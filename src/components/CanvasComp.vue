<template>
  <div class="CanvasComp">
    <div>
      <button
        @click="
          () => {
            color = 'turquoise';
          }
        "
      >
        eau
      </button>
      <button
        @click="
          () => {
            color = 'green';
          }
        "
      >
        herbe
      </button>
      <button
        @click="
          () => {
            color = 'sienna';
          }
        "
      >
        terre
      </button>
      <button
        @click="
          () => {
            color = 'grey';
          }
        "
      >
        pierre
      </button>
    </div>
    <canvas id="mapEditor" width="400" height="400"></canvas>
    <button @click="downloadCanvasImage()" class="download">Télécharger Map</button>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from "vue";

const color = ref("blue");

let canvas;
let ctx;
const gridSize = 25;

const blocks = [];

const drawGrid = () => {
  ctx.strokeStyle = "#ddd";
  for (let x = 0; x <= canvas.width; x += gridSize) {
    ctx.beginPath();
    ctx.moveTo(x, 0);
    ctx.lineTo(x, canvas.height);
    ctx.stroke();
  }
  for (let y = 0; y <= canvas.height; y += gridSize) {
    ctx.beginPath();
    ctx.moveTo(0, y);
    ctx.lineTo(canvas.width, y);
    ctx.stroke();
  }
};

const drawSquare = (x, y) => {
  const gridX = Math.floor(x / gridSize) * gridSize;
  const gridY = Math.floor(y / gridSize) * gridSize;

  ctx.fillStyle = color.value;
  ctx.fillRect(gridX, gridY, gridSize, gridSize);

  blocks.push({ x: gridX, y: gridY, color: color.value });
};

const redrawBlocks = () => {
  blocks.forEach(block => {
    ctx.fillStyle = block.color;
    ctx.fillRect(block.x, block.y, gridSize, gridSize);
  });
};

const downloadCanvasImage = () => {
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  redrawBlocks();

  const image = canvas.toDataURL();
  const downloadLink = document.createElement("a");
  downloadLink.href = image;
  downloadLink.download = "map400x400.png";
  downloadLink.click();

  drawGrid();
};

onMounted(() => {
  canvas = document.querySelector("canvas");
  ctx = canvas.getContext("2d");

  drawGrid();

  canvas.addEventListener("click", event => {
    const rect = canvas.getBoundingClientRect();
    const x = event.clientX - rect.left;
    const y = event.clientY - rect.top;
    drawSquare(x, y);
    drawGrid();
  });
});
</script>

<style scoped>
.CanvasComp {
  display: flex;
  flex-direction: column;
}

#mapEditor {
  border: 6px solid #213547;
  margin-bottom: 2em;
}

button {
  margin: 0 1em 2em 0;
}
</style>
