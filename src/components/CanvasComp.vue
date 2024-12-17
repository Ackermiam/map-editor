<template>
  <div class="CanvasComp">
    <div>
      <button @click="changeColor('black')">obstacle</button>
      <button @click="changeColor('purple')">personnage</button>
      <button @click="changeColor('white')">effacer</button>
      <button @click="exportColoredBlocks()" class="download">
        Télécharger Map
      </button>
    </div>
    <canvas id="mapEditor" width="600" height="600"></canvas>
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
    // Effacer une case
    if (existingBlockIndex !== -1) {
      blocks.splice(existingBlockIndex, 1);
    }
    ctx.clearRect(gridX * gridSize, gridY * gridSize, gridSize, gridSize);
    ctx.strokeStyle = "#ddd";
    ctx.strokeRect(gridX * gridSize, gridY * gridSize, gridSize, gridSize);
  } else if (color.value === "purple") {
    // Effacer l'ancienne case violette
    const previousPurpleIndex = blocks.findIndex((block) => block.color === "purple");
    if (previousPurpleIndex !== -1) {
      const previousBlock = blocks[previousPurpleIndex];
      blocks.splice(previousPurpleIndex, 1);
      ctx.clearRect(previousBlock.x * gridSize, previousBlock.z * gridSize, gridSize, gridSize);
      ctx.strokeStyle = "#ddd";
      ctx.strokeRect(previousBlock.x * gridSize, previousBlock.z * gridSize, gridSize, gridSize);
    }
    // Ajouter la nouvelle case violette
    ctx.fillStyle = color.value;
    ctx.fillRect(gridX * gridSize, gridY * gridSize, gridSize, gridSize);
    blocks.push({ x: gridX, z: gridY, color: color.value });
  } else {
    // Ajouter d'autres cases (ex: black)
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
  const characterPlacement = blocks.find((block) => block.color === "purple");

  const level = blocks
    .filter((block) => block.color === "black")
    .map(({ x, z }) => ({ x, z }));

  const result = {
    characterPlacement: characterPlacement
      ? { x: characterPlacement.x, z: characterPlacement.z }
      : null,
    level,
  };

  // Affichage dans la console
  console.log(JSON.stringify(result, null, 2));

  // Création du Blob avec le contenu JSON
  const blob = new Blob([JSON.stringify(result, null, 2)], { type: "application/json" });

  // Création d'un lien pour le téléchargement
  const link = document.createElement("a");
  link.href = URL.createObjectURL(blob);
  link.download = "mapData.json";  // Nom du fichier téléchargé

  // Simulation d'un clic pour télécharger le fichier
  link.click();
};

const fillBorders = () => {
  const rows = canvas.height / gridSize;
  const cols = canvas.width / gridSize;

  for (let x = 0; x < cols; x++) {
    drawSquare(x * gridSize, 0); // Ligne du haut
    drawSquare(x * gridSize, (rows - 1) * gridSize); // Ligne du bas
  }

  for (let y = 0; y < rows; y++) {
    drawSquare(0, y * gridSize); // Colonne de gauche
    drawSquare((cols - 1) * gridSize, y * gridSize); // Colonne de droite
  }
};

onMounted(() => {
  canvas = document.querySelector("canvas");

  ctx = canvas.getContext("2d");

  fillBorders();
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
}

button {
  margin: 0 1em 2em 0;
}

.download {
  background: rgb(42, 136, 42);
  color: white;
}
</style>
