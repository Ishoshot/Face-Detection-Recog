<template>
  <div class="container">
    <div v-if="message" class="fade-out">
      <i class="fa fa-spinner fa-spin fa-5x"></i>
      <h1>Please Wait...</h1>
    </div>

    <div :class="{'d-none' : messag}" v-else>
      <div class="row justify-content-center">
        <div class="col-md-10">
          <div class="embed-responsive embed-responsive-4by3">
            <video id="video" class="embed-responsive-item" autoplay muted></video>
          </div>
          <canvas class="canvas" id="overlay"></canvas>
        </div>
      </div>
    </div>

    <div v-if="messag">
      <i class="fa fa-eye fa-5x"></i>
      <h1>Almost...</h1>
    </div>
  </div>
</template>

<script>
import * as faceapi from "face-api.js";

export default {
  name: "First",

  props: {},

  data() {
    return {
      message: true,
      messag: ""
    };
  },

  mounted() {
    faceapi.nets.tinyFaceDetector.loadFromUri("./models");
    faceapi.nets.faceLandmark68Net.loadFromUri("./models");
    faceapi.nets.faceLandmark68TinyNet.loadFromUri("./models");
    faceapi.nets.faceRecognitionNet.loadFromUri("./models");
    faceapi.nets.faceExpressionNet.loadFromUri("./models");
    faceapi.nets.ageGenderNet.loadFromUri("./models");
    console.log(faceapi.nets);
    setTimeout(() => {
      this.message = false;
    }, 100);

    setTimeout(() => {
      this.messag = "load";
    }, 200);

    this.startVideo();
  },

  methods: {
    async startVideo() {
      // const video = document.querySelector("video");
      const stream = await navigator.mediaDevices.getUserMedia({
        video: true
      });
      document.querySelector("video").srcObject = stream;
      this.messag = "";
      this.onPlay();
    },
    async onPlay() {
      let a = document.querySelector("video").play();
      if (a) {
        setInterval(async () => {
          const displaySize = {
            width: document.querySelector("video").clientWidth,
            height: document.querySelector("video").clientHeight
          };
          const canvas = document.getElementById("overlay");
          // document.body.append(canvas);
          faceapi.matchDimensions(canvas, displaySize);
          const detections = await faceapi
            .detectAllFaces(
              document.querySelector("video"),
              new faceapi.TinyFaceDetectorOptions()
            )
            .withFaceLandmarks()
            .withFaceExpressions()
            .withAgeAndGender()
            .withFaceDescriptors();
          const resizedResults = faceapi.resizeResults(detections, displaySize);
          console.log(detections);
          canvas
            .getContext("2d")
            .clearRect(
              0,
              0,
              document.querySelector("video").offsetWidth,
              document.querySelector("video").offsetHeight
            );
          faceapi.draw.drawDetections(canvas, resizedResults);
          faceapi.draw.drawFaceLandmarks(canvas, resizedResults);
          const minProbability = 0.05;
          faceapi.draw.drawFaceExpressions(
            canvas,
            resizedResults,
            minProbability
          );
        }, 1000);
        console.log("Playing");
      } else {
        alert("Not Playing");
      }
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->

<style>
.canvas {
  position: absolute;
  top: 0;
  left: 0;
  z-index: 10;
  /* margin-top: 6%; */
}

#video {
  position: absolute;
  /* top: 0; */
  left: 0;
}
</style>
