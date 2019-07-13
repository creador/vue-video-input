vue-video-input
=============

Video file input Vue.js component with video canvas preview, drag and drop, information detection, gif preview creation and more.


## Installation

### npm

``` sh
npm install --save vue-video-input
```
### yarn

``` sh
yarn add vue-video-input
```

## Usage

```HTML
<template>
  <div class="hello">    
    <video-input 
      ancho="600" 
      alto="600" 
      radio="0"
      mensaje="DRAG VIDEO" 
      @info="onInfo">
    </video-input>
  </div>
</template>
```

```javascript
<script>
import VideoInput from 'vue-video-input'

export default {
  name: 'app',
  data () {
    return {
    }
  },
  components: {
    VideoInput
  },
  methods: {
    onInfo (video) {
      console.log('New video selected!')
      if (video.base64) {
        console.log('Base64 of video loaded.', video.base64)
      } else {
        console.log('FileReader API not supported: use the <form>, Luke!')
      }
    }
  }
}
</script>
```

You can also use it directly in the browser through [unpkg's CDN](https://unpkg.com/vue-video-input) (or [jsDelivr](https://cdn.jsdelivr.net/npm/vue-video-input)):

```html
<!DOCTYPE html>
<html>
<head>
  <script src="https://unpkg.com/vue"></script>
  <script src="https://unpkg.com/vue-video-input"></script>
  <title>In the browser!</title>
</head>
<body>
  <div id="app">
    <p>{{ message }}</p>
    <video-input></video-input>
  </div>
  <script>
    var app = new Vue({
      el: '#app',
      data: {
        message: 'Hello Vue!'
      },
      components: {
        'video-input': VideoInput
      }
    })
  </script>
</body>
</html>
```


## Props

- **ancho, alto**: (pixels, optional) the maximum width and height of the preview container. The video will be resized and centered to cover this area. Either ancho or alto is required.
- **margin**: (pixels, optional) the margin around the preview container. Default value: 0.
- **radio**: (percentage, optional) The border-radius value for the container. Set *radius="50"* to get a circular container. Default value: 0.
- **mensaje**: (string, optional) Message to display inside container when no video has being selected.
- **tipo**: (media type, optional) the accepted video type(s), e.g. video/mp4, video/avi, etc. Default value: 'video/*'. 
- **maxmb**: (MB, optional) the maximum accepted file size in megabytes.
- **duracion**: (int, optional) the amount of seconds the video must have. Default -1=disabled.
- **ratio**: (string, optional) the required aspect ratio the video must have.
- **preview_ancho**: (pixels, optional) the width for the gif preview file to be created.
- **preview_alto**: (pixels, optional) the height for the gif preview file to be created.
   
## Events

- **info**: emitted on (successful) video change. Passes an object with the fields: mbytes (amount of megabytes that the video has), duracion (amount of seconds the video has), ratio (aspect ratio of the video), ancho (original width of video), height (original height of video), base64 (Base64 Data URI string with video data).
- **change**: emitted on (successful) video change. Passes a field called 'archivo' with the input object file.
- **preview**: emitted while the animated gif datafile is being and/or has finished created, along with an object with *creando* (true is gif preview is still being created), *preview* (Base64 Data URI string with GIF data).
- **error**: emitted on error, along with an object with *tipo* (maxmb, ratio, formato), *mensaje*.
  
## TODOs

- Translate properties/events to English
- Add tests

## Contributions

All contributions are welcome, as long as they are within the scope of the project. Please open a new issue before submitting a pull request.

You should follow the Javascript Standard Style guidelines:
https://github.com/feross/standard/blob/master/RULES.md#javascript-standard-style
