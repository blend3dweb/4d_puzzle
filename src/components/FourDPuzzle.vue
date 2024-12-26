<script setup>
import { ref, onMounted, computed } from 'vue'

const currentLevel = ref(0)
const playerPosition = ref({ x: 0, y: 0, z: 0, w: 0 })
const gameGrid = ref([])

const levels = [
  {
    id: 0,
    name: "Tutorial",
    grid: generateLevel(4, 4, 4, 2), // x, y, z, w dimensions
    startPosition: { x: 0, y: 0, z: 0, w: 0 },
    endPosition: { x: 3, y: 3, z: 3, w: 1 }
  }
  // Add more levels here
]

function generateLevel(x, y, z, w) {
  // Create 4D array with walkable/blocked spaces
  const grid = Array(x).fill().map(() => 
    Array(y).fill().map(() => 
      Array(z).fill().map(() => 
        Array(w).fill(1)))) // 1 = walkable, 0 = blocked
  return grid
}

function movePlayer(direction) {
  // Handle movement in 4 dimensions
  const newPosition = {...playerPosition.value}
  switch(direction) {
    case 'right': newPosition.x++; break
    case 'left': newPosition.x--; break
    case 'up': newPosition.y++; break
    case 'down': newPosition.y--; break
    case 'forward': newPosition.z++; break
    case 'backward': newPosition.z--; break
    case 'ana': newPosition.w++; break
    case 'kata': newPosition.w--; break
  }
  
  if (isValidMove(newPosition)) {
    playerPosition.value = newPosition
  }
}

function isValidMove(position) {
  const level = levels[currentLevel.value]
  return position.x >= 0 && position.x < level.grid.length &&
         position.y >= 0 && position.y < level.grid[0].length &&
         position.z >= 0 && position.z < level.grid[0][0].length &&
         position.w >= 0 && position.w < level.grid[0][0][0].length &&
         level.grid[position.x][position.y][position.z][position.w] === 1
}

// Computed property to get current w-slice
const currentSlice = computed(() => {
  const level = levels[currentLevel.value]
  return level.grid.map(x => x.map(y => y.map(z => z[playerPosition.value.w])))
})

// Add visual indicators for player and goal
function getCellClass(x, y, z) {
  const pos = playerPosition.value
  const end = levels[currentLevel.value].endPosition
  
  if (x === pos.x && y === pos.y && z === pos.z && pos.w === playerPosition.value.w) {
    return 'player'
  }
  if (x === end.x && y === end.y && z === end.z && end.w === playerPosition.value.w) {
    return 'goal'
  }
  return 'cell'
}
</script>

<template>
  <div class="game-container">
    <div class="level-info">
      <h2>Level {{ currentLevel + 1 }}: {{ levels[currentLevel].name }}</h2>
    </div>
    
    <div class="game-view">
      <div class="grid-3d">
        <div v-for="(xLayer, x) in currentSlice" :key="x" class="x-layer">
          <div v-for="(yLayer, y) in xLayer" :key="y" class="y-layer">
            <div v-for="(cell, z) in yLayer" :key="z" 
                 :class="['cell', getCellClass(x, y, z)]"
                 :style="{
                   transform: `translate3d(${x * 40}px, ${y * 40}px, ${z * 40}px)`
                 }">
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="controls">
      <button @click="movePlayer('left')">Left</button>
      <button @click="movePlayer('right')">Right</button>
      <button @click="movePlayer('up')">Up</button>
      <button @click="movePlayer('down')">Down</button>
      <button @click="movePlayer('forward')">Forward</button>
      <button @click="movePlayer('backward')">Back</button>
      <button @click="movePlayer('ana')">Ana (W+)</button>
      <button @click="movePlayer('kata')">Kata (W-)</button>
    </div>
  </div>
</template>

<style scoped>
.game-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2rem;
}

.controls {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 0.5rem;
}

.grid-3d {
  width: 500px;
  height: 500px;
  border: 2px solid #333;
  perspective: 1000px;
  transform-style: preserve-3d;
  position: relative;
}

.x-layer, .y-layer {
  position: relative;
  transform-style: preserve-3d;
}

.cell {
  position: absolute;
  width: 35px;
  height: 35px;
  background: rgba(200, 200, 200, 0.8);
  border: 1px solid #666;
  transform-style: preserve-3d;
}

.player {
  background: #4CAF50;
  box-shadow: 0 0 10px rgba(76, 175, 80, 0.8);
}

.goal {
  background: #F44336;
  box-shadow: 0 0 10px rgba(244, 67, 54, 0.8);
}

.game-view {
  transform: rotateX(45deg) rotateZ(45deg);
  margin: 100px 0;
}
</style>
