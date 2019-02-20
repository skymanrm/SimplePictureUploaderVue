<template>
  <div id="app">
    <h2>Test Simple Picture Uploader</h2>

    <simple-picture-uploader
      v-model="photo"
      :width="400"
      :height="400"
    >
      <template v-slot:empty>
        <div class="centered">Tap for select file<br /> or drop file here</div>
      </template>
      <template v-slot:over>
        <div class="centered">Drop file</div>
      </template>
    </simple-picture-uploader>
    <div>
      <p>Put cursor on image and scroll for scaling
        <br />
      or Drag image for framing</p>
    </div>
  </div>
</template>

<script>
import SimplePictureUploader from '../src/SimplePictureUploader';

export default {
  name: 'App',
  components: {
    SimplePictureUploader,
  },
  data: () => ({
    photo: null,
  }),
};
</script>

<style>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }
  .centered {
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
  }
</style>
