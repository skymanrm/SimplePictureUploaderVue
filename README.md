# Simple Picture Uploader for Vue

![Example](https://github.com/skymanrm/SimplePictureUploaderVue/blob/master/static/example.gif?raw=true)

Features

* Drag drop support
* Preview
* Scaling
* Framing
* Working with v-model 

## Installation via npm

``` bash
npm install simplepictureuploader --save
```

## Build Setup

``` bash
# install dependencies
npm install

# serve examples
npm run dev
```

Example usage
```vue
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
```

SimplePictureUploader will put result encoded in base64 to specified field (v-model). For prevent lags, field updates only once in a second.
