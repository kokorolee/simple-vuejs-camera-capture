
<template>
    <div id="app">
        <div><video ref="video" id="video" width="640" height="480" autoplay playsinline></video></div>
        <div><button id="snap" v-on:click="capture()">Snap Photo</button></div>
        <canvas ref="canvas" id="canvas" width="640" height="480"></canvas>
        <ul>
            <li v-for="c in captures">
                <img v-bind:src="c" height="50" />
            </li>
        </ul>
    </div>
</template>

<script>
import axios from 'axios';
const DOMAIN = 'https://6f23374c.ngrok.io'
export default {
  name: 'app',
  data() {
    return {
      video: {},
      canvas: {},
      captures: [],
      files: [],
      isPlaying: false,
    }
  },
  mounted() {
    let constraints = {
      video: true
    };
    navigator.mediaDevices.getUserMedia(constraints).
    then(this.handleSuccess).catch(this.handleError);
  },
  methods: {
    handleSuccess(stream) {
      this.video = this.$refs.video
      window.stream = stream; // only to make stream available to console
      this.video.srcObject = stream;
    },
    handleError(error) {
      console.log('getUserMedia error: ', error);
    },
    hasGetUserMedia() {
      return !!(navigator.mediaDevices &&
        navigator.mediaDevices.getUserMedia);
    },
    capture() {
        console.log("hehe");
        this.canvas = this.$refs.canvas;
        let context = this.canvas.getContext("2d").drawImage(this.video, 0, 0, 640, 480);
        let base64URL = this.canvas.toDataURL("image/png")
        this.captures.push(base64URL);
        let file = this.toFile(this.dataURLtoBlob(base64URL))
        console.log(file);
        let bodyFormData = new FormData();
        let name = Math.random().toString(36).substring(7);
        bodyFormData.append("product[image]", file);
        bodyFormData.append("product[name]", name);
        // bodyFormData.append("product[image_text]", base64URL);

        axios({
          method: 'post',
          url: `${DOMAIN}/products`,
          data: bodyFormData,
          config: { headers: {
            'Content-Type': 'multipart/form-data',
            'Access-Control-Allow-Origin': '*'
           }}
          })
          .then(function (response) {
              //handle success
              console.log(response);
              alert("uploaded")
          })
          .catch(function (response) {
            console.log(response);
              //handle error
              alert("upload failed")
          });

    },
    dataURLtoBlob(dataURL) {
      let array, binary, i, results;
      binary = atob(dataURL.split(',')[1])
      array = []
      i = 0
      results = []
      while (i < binary.length) {
        array.push(binary.charCodeAt(i))
        results.push(i++)
      }
      return results
    },
    toFile(array){
      return new Blob([new Uint8Array(array)], {
        type: 'image/png'
      })
    },

}
}
</script>

<style>
body: {
  background-color: #F0F0F0;
}

#app {
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

#video {
  background-color: #000000;
}

#canvas {
  display: none;
}

li {
  display: inline;
  padding: 5px;
}
</style>
