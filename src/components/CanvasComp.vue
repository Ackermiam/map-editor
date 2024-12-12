<template>
  <div class="CanvasComp">
    <div>
      <button @click="changeColor('black')">obstacle</button>
      <button @click="changeColor('white')">effacer</button>
    </div>
    <canvas id="mapEditor" width="400" height="400"></canvas>
    <button @click="exportColoredBlocks()" class="download">
      Exporter Map
    </button>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from "vue";

const color = ref("black");

const changeColor = (string: string) => {
  color.value = string;
};

let canvas: HTMLCanvasElement;
let ctx: any;
const gridSize = 25;

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
  const gridX = Math.floor(x / gridSize);
  const gridY = Math.floor(y / gridSize);

  const existingBlockIndex = blocks.findIndex((block) => block.x === gridX && block.z === gridY);

  if (color.value === "white") {
    if (existingBlockIndex !== -1) {
      blocks.splice(existingBlockIndex, 1);
    }
    ctx.clearRect(gridX * gridSize, gridY * gridSize, gridSize, gridSize);
    ctx.strokeStyle = "#ddd";
    ctx.strokeRect(gridX * gridSize, gridY * gridSize, gridSize, gridSize);
  } else {
    ctx.fillStyle = color.value;
    ctx.fillRect(gridX * gridSize, gridY * gridSize, gridSize, gridSize);

    if (existingBlockIndex === -1) {
      blocks.push({ x: gridX, z: gridY, color: color.value });
    } else {
      blocks[existingBlockIndex].color = color.value;
    }
  }
};

const exportColoredBlocks = () => {
  const coloredBlocks = blocks
    .filter((block) => block.color === "black")
    .map(({ x, z }) => ({ x, z }));
  console.log(JSON.stringify(coloredBlocks, null, 2));
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
