# pictureuploader

> Simple Picture Uploader with scaling, framing

Working with v-model. Support drag-drop. Allow user to scale / position selected image before uploading to server. 

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
