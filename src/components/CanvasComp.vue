<template>
  <div class="CanvasComp">
    <div>
      <button @click="changeColor('turquoise')">eau</button>
      <button @click="changeColor('green')">herbe</button>
      <button @click="changeColor('sienna')">terre</button>
      <button @click="changeColor('grey')">pierre</button>
    </div>
    <canvas id="mapEditor" width="400" height="400"></canvas>
    <button @click="downloadCanvasImage()" class="download">
      Télécharger Map
    </button>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from "vue";

const color = ref("sienna");

const changeColor = (string: string) => {
  color.value = string;
};

let canvas: HTMLCanvasElement;
let ctx: any;
const gridSize = 25;

const mapCanvas = document.createElement('canvas');
mapCanvas.width = 16;
mapCanvas.height = 16;
const mapCtx = mapCanvas.getContext('2d');

const blocks: any[] = [];

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

const drawSquare = (x: number, y: number) => {
  const gridX = Math.floor(x / gridSize) * gridSize;
  const gridY = Math.floor(y / gridSize) * gridSize;

  ctx.fillStyle = color.value;
  ctx.fillRect(gridX, gridY, gridSize, gridSize);

  blocks.push({ x: gridX, y: gridY, color: color.value });
};

const redrawBlocks = () => {
  blocks.forEach((block) => {
    ctx.fillStyle = block.color;
    ctx.fillRect(block.x, block.y, gridSize, gridSize);
  });
};

const downloadCanvasImage = () => {
  redrawBlocks();
  const mapImage = generateMapImage();
  const downloadLink = document.createElement('a');
  downloadLink.href = mapImage;
  downloadLink.download = 'map16x16.png';

  downloadLink.click();

  drawGrid();
};

const generateMapImage = () => {
  mapCtx.clearRect(0, 0, mapCanvas.width, mapCanvas.height);

  for (let y = 0; y < 16; y++) {
    for (let x = 0; x < 16; x++) {
      const imageData = ctx.getImageData(x * 25, y * 25, 25, 25);
      const color = averageColor(imageData);

      if (color.alpha > 0) {
        mapCtx.fillStyle = `rgba(${color.r}, ${color.g}, ${color.b}, ${color.alpha})`;
        mapCtx.fillRect(x, y, 1, 1);
      }
    }
  }

  const mapImage = mapCanvas.toDataURL("image/png");
  return mapImage;
};

const averageColor = (imageData) => {
  const data = imageData.data;
  let r = 0, g = 0, b = 0, a = 0;
  for (let i = 0; i < data.length; i += 4) {
    r += data[i];
    g += data[i + 1];
    b += data[i + 2];
    a += data[i + 3];
  }
  const pixelCount = data.length / 4;
  return {
    r: Math.round(r / pixelCount),
    g: Math.round(g / pixelCount),
    b: Math.round(b / pixelCount),
    alpha: Math.round(a / pixelCount) / 255
  };
};

onMounted(() => {
  canvas = document.querySelector("canvas");

  ctx = canvas.getContext("2d");

  drawGrid();

  canvas.addEventListener("click", (event) => {
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
