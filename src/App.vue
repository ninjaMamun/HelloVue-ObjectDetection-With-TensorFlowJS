<template>
  <div class="d-flex justify-content-center text-center m-5">
    <div>
      <h1 class="text-primary">মেশিনে দেখে &#129315;</h1>
      <h6 class="text-info">Simple Object Detection With TensorFlow</h6>

      <div>
        <div class="m-1">
          <img
            ref="imgRef"
            src="https://i.ibb.co/Xkzfc5Z/Brain-with-digital-circuit-and-programmer-with-laptop-Machine-learning-artificial-intelligence-digit.jpg"
            width="320"
            border="0"
            crossorigin="anonymous"
          />
        </div>
        <video ref="videoRef" autoPlay="true" width="120"></video>
      </div>

      <div v-if="result.length > 0">
        <h3 class="badge badge-primary text-wrap text-capitalize mt-2" style="width: auto; font-size: 30px;">{{ result[0].class }}</h3>
      </div>

      <div v-if="!isstreaming">
        <button class="btn btn-outline-success" @click="openCamera">
          Open Camera
        </button>
      </div>
      <div v-else class="m-2">
        <button @click="stopStreaming" class="btn btn-outline-danger mr-3">Close Camera</button>
        <button @click="snapShot" class="btn btn-outline-success">snapShot</button>
      </div>

      <button class="btn btn-outline-primary mt-2" @click="detect">
        <span v-if="isLoading">Loading</span>
        <span  v-else>Detect</span>
      </button>
      
    </div>
  </div>
</template>

<script>
import { ref } from "vue";
require("@tensorflow/tfjs-backend-cpu");
require("@tensorflow/tfjs-backend-webgl");
const cocoSsd = require("@tensorflow-models/coco-ssd");

export default {
  setup() {
    const imgRef = ref("");
    const result = ref([]);
    const isLoading = ref(false);
    const videoRef = ref("");
    const isstreaming = ref(false);

    async function detect() {
      const img = imgRef.value;

      isLoading.value = true;

      const model = await cocoSsd.load();

      // Classify the image.
      const predictions = await model.detect(img);

      console.log("Predictions: ");
      console.log(predictions);

      result.value = predictions;
      isLoading.value = false;
    }

    async function openCamera() {
      if (navigator.mediaDevices.getUserMedia) {
        const devices = await navigator.mediaDevices.enumerateDevices();
        const cameras = devices.filter(
          (device) => device.kind === "videoinput"
        );
        const camId = cameras[0].deviceId;
        navigator.mediaDevices
          .getUserMedia({ video: { deviceId: { exact: camId } } })
          .then((stream) => {
            isstreaming.value = true;
            videoRef.value.srcObject = stream;
          });
      }
    }

    function stopStreaming() {
      const stream = videoRef.value.srcObject;
      const tracks = stream.getTracks();
      tracks.map((track) => track.stop());
      isstreaming.value = false;
    }

    function snapShot() {
      const canvas = document.createElement("canvas");
      const ctx = canvas.getContext("2d");
      ctx.drawImage(videoRef.value, 0, 0, 200, 200);
      const data = canvas.toDataURL("img/png");
      imgRef.value.setAttribute("src", data);
    }

    return {
      stopStreaming,
      imgRef,
      result,
      detect,
      isLoading,
      openCamera,
      videoRef,
      isstreaming,
      snapShot,
    };
  },
};
</script>

<style>
</style>





