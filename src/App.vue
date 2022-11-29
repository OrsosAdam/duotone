<template>
  <div id="main">
    <p>
      This is the canvas generated with the Duotone function. Use the "Convert
      to Image" button to convert the canvas into a png image below, which can
      be saved on desktop and mobile.
    </p>
    <div>
      <input type="color" id="primary" name="primary" v-model="primary" @input="runDuotone" />
      <label for="primary">primary</label>
    </div>

    <div>
      <input type="color" id="secondary" name="secondary" v-model="secondary" @input="runDuotone" />
      <label for="secondary">secondary</label>
    </div>
    <div>
      <input ref="fileInput" type="file" @change="pickFile" name="fileInput" />
      <label for="fileInput">File:</label>
    </div>
    <canvas id="duotone" width="600" height="600"></canvas>
    <button @click="runDuoTone()">Change Colours</button>
  </div>
</template>

<script setup>
import { onMounted, ref } from "vue";
let primary = ref("#f65e35");
let secondary = ref("#1e3265");
let image = ref();

function pickFile(e) {
  var files = e.target.files || e.dataTransfer.files;
  if (!files.length) return;

  image.value = URL.createObjectURL(files[0]);
  runDuoTone();
}

function duotone(id, src, primaryClr, secondaryClr, actions = () => null) {
  let canvas = document.getElementById(id);
  let ctx = canvas.getContext("2d");

  let downloadedImg = new Image();
  downloadedImg.crossOrigin = ""; // to allow us to manipulate the image without tainting canvas
  downloadedImg.onload = function () {
    ctx.drawImage(downloadedImg, 0, 0, canvas.width, canvas.height); // draws image to canvas on load
    // Converts to grayscale by averaging the values of each pixel
    let imageData = ctx.getImageData(0, 0, 800, 800);
    const pixels = imageData.data;
    for (let i = 0; i < pixels.length; i += 4) {
      const red = pixels[i];
      const green = pixels[i + 1];
      const blue = pixels[i + 2];
      // Using relative luminance to convert to grayscale
      const avg = Math.round((0.299 * red + 0.587 * green + 0.114 * blue) * 1);
      pixels[i] = avg;
      pixels[i + 1] = avg;
      pixels[i + 2] = avg;
    }
    // Puts the grayscaled image data back into the canvas
    ctx.putImageData(imageData, 0, 0);
    // puts the duotone image into canvas with multiply and lighten
    ctx.globalCompositeOperation = "multiply";
    ctx.fillStyle = primaryClr; // colour for highlights
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    // lighten
    ctx.globalCompositeOperation = "lighten";
    ctx.fillStyle = secondaryClr; // colour for shadows
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    // calls any other draws that you want through the function parameter passed in
    actions(ctx);
  };
  downloadedImg.src = src; // source for the image
}
function runDuoTone() {
  console.log("Duotone is running");
  duotone("duotone", image.value, primary.value, secondary.value);
}
onMounted(() => {
  runDuoTone();
});
</script>

<style scoped>
body {
  margin: 0;
  font-size: 16px;
  font-family: "Helvetica Neue", Helvetica;
}

#main {
  width: 600px;
  margin: 50px auto;
  text-align: center;
}

p {
  text-align: left;
  margin: 40px;
}

button {
  font-weight: 600;
  margin: 20px 10px;
  padding: 15px 30px;
  border: 2px solid black;
  background-color: white;
  border-radius: 10px;
  color: black;
  transition: all 0.3s ease;
  text-transform: uppercase;
}

button:hover {
  cursor: pointer;
  background-color: black;
  color: white;
}
</style>
