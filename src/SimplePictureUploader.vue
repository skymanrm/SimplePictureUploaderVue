<template>
    <div class="pup">
        <div
            class="pup_container"
            @dragenter.stop.prevent="onDragEnter"
            @dragleave.stop.prevent="onDragLeave"
            @click="onClick"
            @drop.stop.prevent="onDrop"
            @dragover.stop.prevent="onDragOver"
            :style="{width: `${width}px`, height: `${height}px`}"
        >
            <div class="filepreview" :style="{display: loadedImage ? 'block' : 'none'}">
                <canvas
                    :style="{width: `${width}px`, height: `${height}px`}"
                    :width="scaledWidth"
                    :height="scaledHeight"
                    ref="canvas"
                    @mousedown.stop.prevent="startMoving($event)"
                    @mouseup.stop.prevent="mouseUp($event)"
                    @click.stop.prevent
                ></canvas>
            </div>
            <slot name="empty" v-if="!somethingOver && !filePreview" class="nomouseevents">
                No file selected, drop file here or click
            </slot>
            <slot name="over" v-if="somethingOver && !filePreview" class="nomouseevents">
                Drop it!
            </slot>
        </div>
        <input type="file" accept="image/*" ref="input" @change="onFileSelected">
    </div>
</template>

<script>
import throttle from 'lodash.throttle';

const MOVING_SPEED = 0.8;
const SCALING_SPEED = 0.0005;
const SCALE_FACTOR = 2;
const CLICK_DISTANCE = 10;

export default {
  name: 'PictureUploadPowered',
  model: {
    prop: 'value',
    event: 'change',
  },
  props: {
    value: String,
    width: {
      type: Number,
      required: true,
    },
    height: {
      type: Number,
      required: true,
    },
  },
  data: () => ({
    somethingOver: false,
    movingStartPoint: null,
    imageParams: { x: 0, y: 0, scale: 1 },
    startImageParams: { x: 0, y: 0, scale: 1 },
    loadedImage: null,
  }),
  computed: {
    filePreview() {
      return this.loadedImage || this.value;
    },
    scaledWidth() {
      return this.width * SCALE_FACTOR;
    },
    scaledHeight() {
      return this.height * SCALE_FACTOR;
    },
  },
  methods: {
    onDragOver(evt) {
      evt.preventDefault();
    },
    onDragEnter() {
      this.somethingOver = true;
    },
    onDragLeave() {
      this.somethingOver = false;
    },
    onDrop(evt) {
      this.somethingOver = false;
      const { files } = evt.dataTransfer;
      this.handleFiles(files);
    },
    onFileSelected(evt) {
      this.somethingOver = false;
      this.handleFiles(evt.target.files);
    },
    onClick() {
      this.$refs.input.click();
    },
    handleFiles(files) {
      const file = files.length > 0 ? files[0] : null;
      if (!file) {
        return;
      }
      if (!file.type.includes('image')) {
        alert('Пожалуйста используйте следующие типы файлов для загрузки: JPG, PNG, BMP');
        return;
      }
      this.readFile(URL.createObjectURL(file));
    },
    readFile(imageSrc) {
      const image = new Image();
      image.onload = () => {
        this.loadedImage = image;
        this.setNormalizeImageParams(0, 0, 1);
      };
      image.src = imageSrc;
    },
    startMoving(evt) {
      this.movingStartPoint = { x: evt.x, y: evt.y };
      this.startImageParams = { ...this.imageParams };
    },
    stopMoving() {
      if (!this.movingStartPoint) {
        return;
      }
      this.movingStartPoint = null;
    },
    mouseUp(evt) {
      const { x, y } = this.movingStartPoint;
      const dist = Math.sqrt(((evt.x - x) ** 2) + ((evt.y - y) ** 2));
      console.log(dist, CLICK_DISTANCE);
      this.stopMoving();
      if (dist < CLICK_DISTANCE) {
        this.onClick();
      }
    },
    moving(evt) {
      if (!this.movingStartPoint) {
        return;
      }
      const dX = MOVING_SPEED * (this.movingStartPoint.x - evt.x);
      const dY = MOVING_SPEED * (this.movingStartPoint.y - evt.y);
      const x = Math.min(0, this.startImageParams.x - dX);
      const y = Math.min(0, this.startImageParams.y - dY);
      this.setNormalizeImageParams(x, y, this.imageParams.scale);
    },
    setNormalizeImageParams(x, y, scale) {
      const { canvas } = this.$refs;
      const minScaleX = canvas.width / this.loadedImage.width;
      const minScaleY = canvas.height / this.loadedImage.height;
      const minScale = Math.max(minScaleX, minScaleY);
      const newScale = Math.min(2, Math.max(minScale, scale || this.imageParams.scale));
      const maxX = -1 * ((this.loadedImage.width * newScale) - canvas.width);
      const maxY = -1 * ((this.loadedImage.height * newScale) - canvas.height);
      this.imageParams = {
        scale: newScale,
        x: Math.max(x, maxX),
        y: Math.max(y, maxY),
      };
    },
    scaling(evt) {
      if (evt.target !== this.$refs.canvas) {
        return;
      }
      evt.preventDefault();
      evt.stopPropagation();
      const scale = Math.max(0, this.imageParams.scale - (SCALING_SPEED * evt.wheelDelta));
      this.setNormalizeImageParams(this.imageParams.x, this.imageParams.y, scale);
    },
    updateCanvas() {
      const { canvas } = this.$refs;
      const ctx = canvas.getContext('2d');
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.drawImage(
        this.loadedImage,
        this.imageParams.x,
        this.imageParams.y,
        this.loadedImage.width * this.imageParams.scale,
        this.loadedImage.height * this.imageParams.scale,
      );
      this.updateModel();
    },
    updateModel: throttle(function () {
      this.$emit('change', this.$refs.canvas.toDataURL());
    }, 1000),
  },
  watch: {
    loadedImage() {
      this.updateCanvas();
    },
    imageParams() {
      this.updateCanvas();
    },
    value(newValue, oldValue) {
      if (!oldValue && newValue && !this.loadedImage) {
        this.readFile(newValue);
      }
    },
  },
  mounted() {
    window.document.addEventListener('mousemove', this.moving);
    window.document.addEventListener('mouseup', this.stopMoving);
    window.document.addEventListener('mousewheel', this.scaling);
  },
  destroyed() {
    window.document.removeEventListener('mousemove', this.moving);
    window.document.removeEventListener('mouseup', this.stopMoving);
    window.document.removeEventListener('mousewheel', this.scaling);
  },
};
</script>

<style scoped>
    .pup {
        display: inline-block;
        padding: 20px;
        background-color: #f7f9fa;
    }

    .pup_container {
        position: relative;
        border: 1px dashed #c1c2c3;
    }

    .nomouseevents {
        pointer-events: none;
    }

    .filepreview canvas {
        cursor: move;
    }

    input[type="file"] {
        display: none;
    }
</style>
