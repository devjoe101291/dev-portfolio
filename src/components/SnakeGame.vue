<script setup lang="ts">
import { ref, onMounted, onUnmounted, watch } from 'vue'

const GRID_SIZE = 20
const INITIAL_SPEED = 150

const snake = ref([{ x: 10, y: 10 }])
const food = ref({ x: 15, y: 15 })
const direction = ref({ x: 1, y: 0 })
const nextDirection = ref({ x: 1, y: 0 })
const isPlaying = ref(false)
const isGameOver = ref(false)
const score = ref(0)
const highScore = ref(0)
const speed = ref(INITIAL_SPEED)
let gameInterval: ReturnType<typeof setInterval> | null = null

const isSnake = (x: number, y: number) => {
  return snake.value.some(segment => segment.x === x && segment.y === y)
}

const isHead = (x: number, y: number) => {
  return snake.value[0].x === x && snake.value[0].y === y
}

const isFood = (x: number, y: number) => {
  return food.value.x === x && food.value.y === y
}

const spawnFood = () => {
  let newFood
  while (true) {
    newFood = {
      x: Math.floor(Math.random() * GRID_SIZE),
      y: Math.floor(Math.random() * GRID_SIZE)
    }
    if (!isSnake(newFood.x, newFood.y)) break
  }
  food.value = newFood
}

const startGame = () => {
  snake.value = [{ x: 10, y: 10 }]
  direction.value = { x: 1, y: 0 }
  nextDirection.value = { x: 1, y: 0 }
  score.value = 0
  speed.value = INITIAL_SPEED
  isGameOver.value = false
  isPlaying.value = true
  spawnFood()
  
  if (gameInterval) clearInterval(gameInterval)
  gameInterval = setInterval(gameLoop, speed.value)
}

const stopGame = () => {
  isPlaying.value = false
  if (gameInterval) clearInterval(gameInterval)
}

const gameOver = () => {
  isGameOver.value = true
  isPlaying.value = false
  if (score.value > highScore.value) {
    highScore.value = score.value
  }
  if (gameInterval) clearInterval(gameInterval)
}

const gameLoop = () => {
  direction.value = nextDirection.value
  
  const newHead = {
    x: snake.value[0].x + direction.value.x,
    y: snake.value[0].y + direction.value.y
  }

  // Check wall collision
  if (
    newHead.x < 0 || 
    newHead.x >= GRID_SIZE || 
    newHead.y < 0 || 
    newHead.y >= GRID_SIZE
  ) {
    gameOver()
    return
  }

  // Check self collision
  if (isSnake(newHead.x, newHead.y)) {
    gameOver()
    return
  }

  snake.value.unshift(newHead)

  // Check food collision
  if (newHead.x === food.value.x && newHead.y === food.value.y) {
    score.value += 10
    speed.value = Math.max(50, INITIAL_SPEED - Math.floor(score.value / 30) * 10)
    spawnFood()
    
    // Update interval with new speed
    if (gameInterval) clearInterval(gameInterval)
    gameInterval = setInterval(gameLoop, speed.value)
  } else {
    snake.value.pop()
  }
}

const handleKeydown = (e: KeyboardEvent) => {
  // Prevent default scrolling for arrow keys
  if (['ArrowUp', 'ArrowDown', 'ArrowLeft', 'ArrowRight', ' '].includes(e.key)) {
    e.preventDefault()
  }

  if (e.key === ' ' && !isPlaying.value) {
    startGame()
    return
  }

  if (!isPlaying.value) return

  switch (e.key) {
    case 'ArrowUp':
    case 'w':
    case 'W':
      if (direction.value.y !== 1) nextDirection.value = { x: 0, y: -1 }
      break
    case 'ArrowDown':
    case 's':
    case 'S':
      if (direction.value.y !== -1) nextDirection.value = { x: 0, y: 1 }
      break
    case 'ArrowLeft':
    case 'a':
    case 'A':
      if (direction.value.x !== 1) nextDirection.value = { x: -1, y: 0 }
      break
    case 'ArrowRight':
    case 'd':
    case 'D':
      if (direction.value.x !== -1) nextDirection.value = { x: 1, y: 0 }
      break
  }
}

// Controls for mobile
const handleControlClick = (dx: number, dy: number) => {
  if (!isPlaying.value) return
  if (dx !== 0 && direction.value.x !== -dx) nextDirection.value = { x: dx, y: 0 }
  if (dy !== 0 && direction.value.y !== -dy) nextDirection.value = { x: 0, y: dy }
}

onMounted(() => {
  window.addEventListener('keydown', handleKeydown)
})

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeydown)
  if (gameInterval) clearInterval(gameInterval)
})
</script>

<template>
  <div class="terminal overflow-hidden relative group">
    <!-- CRT overlay -->
    <div class="absolute inset-0 pointer-events-none opacity-[0.03] bg-gradient-to-b from-transparent via-accent to-transparent h-[100%] animate-scanline z-20"></div>
    
    <div class="terminal-header">
      <div class="flex gap-2">
        <div class="w-3 h-3 rounded-full bg-[#ff5f56]"></div>
        <div class="w-3 h-3 rounded-full bg-[#ffbd2e]"></div>
        <div class="w-3 h-3 rounded-full bg-[#27c93f]"></div>
      </div>
      <div class="mx-auto font-mono text-[0.8rem] text-gray-500 flex items-center gap-2">
        <svg class="w-4 h-4 text-accent" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M14.752 11.168l-3.197-2.132A1 1 0 0010 9.87v4.263a1 1 0 001.555.832l3.197-2.132a1 1 0 000-1.664z"></path><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 12a9 9 0 11-18 0 9 9 0 0118 0z"></path></svg>
        snake_game.exe
      </div>
    </div>

    <div class="p-6 md:p-8 flex flex-col items-center bg-obsidian/50 relative z-10">
      
      <!-- Score Board -->
      <div class="w-full max-w-[400px] flex justify-between mb-4 font-mono text-sm">
        <div class="text-gray-400">SCORE: <span class="text-accent text-lg">{{ score }}</span></div>
        <div class="text-gray-500">HI: <span class="text-white">{{ highScore }}</span></div>
      </div>

      <!-- Game Board -->
      <div 
        class="game-grid bg-obsidian border-2 border-gray-800 rounded relative overflow-hidden"
        :style="{ 
          display: 'grid', 
          gridTemplateColumns: `repeat(${GRID_SIZE}, 1fr)`,
          gridTemplateRows: `repeat(${GRID_SIZE}, 1fr)`,
          width: '100%',
          maxWidth: '400px',
          aspectRatio: '1/1'
        }"
      >
        <!-- Grid Background Pattern -->
        <div class="absolute inset-0 opacity-[0.05] bg-[linear-gradient(#fff_1px,transparent_1px),linear-gradient(90deg,#fff_1px,transparent_1px)] bg-[length:20px_20px]"></div>

        <template v-for="y in GRID_SIZE" :key="'row-'+y">
          <template v-for="x in GRID_SIZE" :key="'cell-'+x+'-'+y">
            <div 
              class="w-full h-full relative z-10 transition-all duration-75"
              :class="{
                'bg-accent shadow-[0_0_10px_rgba(16,185,129,0.8)] rounded-sm scale-90': isSnake(x-1, y-1) && !isHead(x-1, y-1),
                'bg-white shadow-[0_0_15px_rgba(255,255,255,0.9)] rounded-md scale-95 z-20': isHead(x-1, y-1),
                'bg-pink-500 shadow-[0_0_15px_rgba(236,72,153,0.8)] rounded-full scale-75 animate-pulse': isFood(x-1, y-1)
              }"
            ></div>
          </template>
        </template>

        <!-- Overlays -->
        <div v-if="!isPlaying && !isGameOver" class="absolute inset-0 bg-obsidian/80 backdrop-blur-sm z-30 flex flex-col items-center justify-center">
          <div class="text-accent font-mono text-xl mb-4 font-bold tracking-widest animate-pulse">SYSTEM_IDLE</div>
          <button @click="startGame" class="px-6 py-2 border border-accent/50 text-accent font-mono text-sm hover:bg-accent/10 transition-colors rounded">
            INITIALIZE [SPACE]
          </button>
        </div>

        <div v-if="isGameOver" class="absolute inset-0 bg-red-900/20 backdrop-blur-sm z-30 flex flex-col items-center justify-center">
          <div class="text-red-500 font-mono text-2xl mb-2 font-bold tracking-widest">CRITICAL_FAILURE</div>
          <div class="text-gray-300 font-mono text-sm mb-6">SCORE: {{ score }}</div>
          <button @click="startGame" class="px-6 py-2 border border-accent/50 text-accent font-mono text-sm hover:bg-accent/10 transition-colors rounded shadow-[0_0_15px_rgba(16,185,129,0.2)]">
            REBOOT SYSTEM
          </button>
        </div>
      </div>

      <!-- Mobile Controls -->
      <div class="mt-6 grid grid-cols-3 gap-2 w-full max-w-[200px] md:hidden">
        <div></div>
        <button @click="handleControlClick(0, -1)" class="bg-gray-800/50 hover:bg-gray-700 active:bg-accent/50 p-4 rounded-lg flex justify-center items-center transition-colors">
          <svg class="w-6 h-6 text-gray-300" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 15l7-7 7 7"></path></svg>
        </button>
        <div></div>
        <button @click="handleControlClick(-1, 0)" class="bg-gray-800/50 hover:bg-gray-700 active:bg-accent/50 p-4 rounded-lg flex justify-center items-center transition-colors">
          <svg class="w-6 h-6 text-gray-300" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7"></path></svg>
        </button>
        <button @click="handleControlClick(0, 1)" class="bg-gray-800/50 hover:bg-gray-700 active:bg-accent/50 p-4 rounded-lg flex justify-center items-center transition-colors">
          <svg class="w-6 h-6 text-gray-300" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7" transform="rotate(180 12 12)"></path></svg>
        </button>
        <button @click="handleControlClick(1, 0)" class="bg-gray-800/50 hover:bg-gray-700 active:bg-accent/50 p-4 rounded-lg flex justify-center items-center transition-colors">
          <svg class="w-6 h-6 text-gray-300" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7"></path></svg>
        </button>
      </div>

      <div class="mt-6 text-gray-500 font-mono text-[0.65rem] text-center max-w-[400px]">
        <span class="hidden md:inline">Use arrow keys or WASD to control. Press SPACE to start/restart.</span>
        <span class="md:hidden">Use on-screen controls to play.</span>
      </div>
    </div>
  </div>
</template>

<style scoped>
.game-grid {
  box-shadow: inset 0 0 30px rgba(0,0,0,0.8);
}
@keyframes scanline {
  0% { transform: translateY(-100%); }
  100% { transform: translateY(100%); }
}
</style>
