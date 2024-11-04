<template>
  <div>
    <div>
      <button
        @click="
          () => {
            color = 'blue';
          }
        "
      >
        blue
      </button>
      <button
        @click="
          () => {
            color = 'green';
          }
        "
      >
        green
      </button>
      <button
        @click="
          () => {
            color = 'brown';
          }
        "
      >
        brown
      </button>
      <button @click="downloadMapImage()">Télécharger Map</button>
    </div>
    <canvas id="mapEditor" width="400" height="400"></canvas>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from "vue";

const color = ref("blue");

const mapCanvas = document.createElement("canvas");
mapCanvas.width = 16;
mapCanvas.height = 16;
const mapCtx = mapCanvas.getContext("2d");

let originalCanvas;
let ctx;
const gridSize = 25;

const drawGrid = () => {
  for (let x = 0; x <= originalCanvas.width; x += gridSize) {
    ctx.beginPath();
    ctx.moveTo(x, 0);
    ctx.lineTo(x, originalCanvas.height);
    ctx.stroke();
  }
  for (let y = 0; y <= originalCanvas.height; y += gridSize) {
    ctx.beginPath();
    ctx.moveTo(0, y);
    ctx.lineTo(originalCanvas.width, y);
    ctx.stroke();
  }
};

const drawSquare = (x, y) => {
  const gridX = Math.floor(x / gridSize) * gridSize;
  const gridY = Math.floor(y / gridSize) * gridSize;
  ctx.fillStyle = color.value;
  ctx.fillRect(gridX, gridY, gridSize, gridSize);
};

const generateMapImage = () => {
  for (let y = 0; y < 16; y++) {
    for (let x = 0; x < 16; x++) {
      const imageData = ctx.getImageData(x * 25, y * 25, 25, 25);
      const color = averageColor(imageData);

      if (!(color.r === 0 && color.g === 0 && color.b === 0)) {
        mapCtx.fillStyle = `rgb(${color.r}, ${color.g}, ${color.b})`;
        mapCtx.fillRect(x, y, 1, 1);
      }
    }
  }

  const mapImage = mapCanvas.toDataURL();
  console.log(mapImage);
  return mapImage;
};

const averageColor = (imageData) => {
  const data = imageData.data;
  let r = 0,
    g = 0,
    b = 0;
  for (let i = 0; i < data.length; i += 4) {
    r += data[i];
    g += data[i + 1];
    b += data[i + 2];
  }
  const pixelCount = data.length / 4;
  return {
    r: Math.round(r / pixelCount),
    g: Math.round(g / pixelCount),
    b: Math.round(b / pixelCount),
  };
};

const downloadMapImage = () => {
  const mapImage = generateMapImage();

  const downloadLink = document.createElement("a");
  downloadLink.href = mapImage;
  downloadLink.download = "map16x16.png";

  downloadLink.click();
};
onMounted(() => {
  originalCanvas = document.querySelector("canvas");
  ctx = originalCanvas.getContext("2d");
  drawGrid();

  originalCanvas.addEventListener("click", (event) => {
    const rect = originalCanvas.getBoundingClientRect();
    const x = event.clientX - rect.left;
    const y = event.clientY - rect.top;
    drawSquare(x, y);
  });
});
</script>

<style scoped>
#mapEditor {
  border: 6px solid #213547;
}
</style>
