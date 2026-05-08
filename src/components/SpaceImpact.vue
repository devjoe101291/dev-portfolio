<script setup lang="ts">
import { ref, onMounted, onUnmounted, watch } from 'vue'

const isPlaying = ref(false)
const isGameOver = ref(false)
const score = ref(0)
const highScore = ref(0)
const canvasRef = ref<HTMLCanvasElement | null>(null)

// Game constants
const CANVAS_WIDTH = 400
const CANVAS_HEIGHT = 200
const PLAYER_SPEED = 3
const BULLET_SPEED = 7
const ENEMY_SPEED_BASE = 2

// Game State
let player = { x: 20, y: CANVAS_HEIGHT / 2, width: 20, height: 10, dy: 0, dx: 0 }
let bullets: Array<{ x: number, y: number, width: number, height: number }> = []
let enemies: Array<{ x: number, y: number, width: number, height: number, speed: number }> = []
let particles: Array<{ x: number, y: number, vx: number, vy: number, life: number }> = []
let animationFrameId: number | null = null
let lastEnemySpawn = 0

// Key state
const keys = {
  ArrowUp: false,
  ArrowDown: false,
  ArrowLeft: false,
  ArrowRight: false,
  w: false,
  s: false,
  a: false,
  d: false,
  ' ': false
}

const startGame = () => {
  player = { x: 20, y: CANVAS_HEIGHT / 2, width: 20, height: 10, dy: 0, dx: 0 }
  bullets = []
  enemies = []
  particles = []
  score.value = 0
  isGameOver.value = false
  isPlaying.value = true
  lastEnemySpawn = Date.now()
  
  if (animationFrameId) cancelAnimationFrame(animationFrameId)
  gameLoop()
}

const spawnEnemy = () => {
  const size = Math.random() > 0.8 ? 20 : 15 // Sometimes spawn bigger enemies
  enemies.push({
    x: CANVAS_WIDTH,
    y: Math.random() * (CANVAS_HEIGHT - size),
    width: size,
    height: size,
    speed: ENEMY_SPEED_BASE + Math.random() * 2 + (score.value / 100)
  })
}

const createExplosion = (x: number, y: number) => {
  for (let i = 0; i < 10; i++) {
    particles.push({
      x,
      y,
      vx: (Math.random() - 0.5) * 5,
      vy: (Math.random() - 0.5) * 5,
      life: 1.0
    })
  }
}

const gameLoop = () => {
  if (!isPlaying.value) return
  
  const ctx = canvasRef.value?.getContext('2d')
  if (!ctx) return

  // Clear canvas
  ctx.clearRect(0, 0, CANVAS_WIDTH, CANVAS_HEIGHT)

  // Player movement
  player.dy = 0
  player.dx = 0
  if (keys.ArrowUp || keys.w) player.dy = -PLAYER_SPEED
  if (keys.ArrowDown || keys.s) player.dy = PLAYER_SPEED
  if (keys.ArrowLeft || keys.a) player.dx = -PLAYER_SPEED
  if (keys.ArrowRight || keys.d) player.dx = PLAYER_SPEED

  player.y += player.dy
  player.x += player.dx

  // Boundaries
  player.y = Math.max(0, Math.min(CANVAS_HEIGHT - player.height, player.y))
  player.x = Math.max(0, Math.min(CANVAS_WIDTH - player.width, player.x))

  // Draw Player (Ship)
  ctx.fillStyle = '#ffffff'
  ctx.shadowBlur = 10
  ctx.shadowColor = '#ffffff'
  ctx.beginPath()
  ctx.moveTo(player.x, player.y)
  ctx.lineTo(player.x + player.width, player.y + player.height / 2)
  ctx.lineTo(player.x, player.y + player.height)
  ctx.closePath()
  ctx.fill()

  // Engine flame
  ctx.fillStyle = '#10b981'
  ctx.shadowColor = '#10b981'
  if (Math.random() > 0.3) {
    ctx.fillRect(player.x - 5, player.y + 2, 5, player.height - 4)
  }

  // Update and draw bullets
  ctx.fillStyle = '#10b981'
  for (let i = bullets.length - 1; i >= 0; i--) {
    let b = bullets[i]
    b.x += BULLET_SPEED
    ctx.fillRect(b.x, b.y, b.width, b.height)
    
    // Remove off-screen bullets
    if (b.x > CANVAS_WIDTH) bullets.splice(i, 1)
  }

  // Spawn enemies
  const now = Date.now()
  if (now - lastEnemySpawn > Math.max(500, 1500 - score.value * 5)) {
    spawnEnemy()
    lastEnemySpawn = now
  }

  // Update and draw enemies
  ctx.fillStyle = '#ff5f56'
  ctx.shadowColor = '#ff5f56'
  for (let i = enemies.length - 1; i >= 0; i--) {
    let e = enemies[i]
    e.x -= e.speed
    ctx.fillRect(e.x, e.y, e.width, e.height)

    // Player collision
    if (
      player.x < e.x + e.width &&
      player.x + player.width > e.x &&
      player.y < e.y + e.height &&
      player.y + player.height > e.y
    ) {
      createExplosion(player.x, player.y)
      gameOver()
      return
    }

    // Bullet collision
    for (let j = bullets.length - 1; j >= 0; j--) {
      let b = bullets[j]
      if (
        b.x < e.x + e.width &&
        b.x + b.width > e.x &&
        b.y < e.y + e.height &&
        b.y + b.height > e.y
      ) {
        // Destroy enemy and bullet
        createExplosion(e.x + e.width/2, e.y + e.height/2)
        enemies.splice(i, 1)
        bullets.splice(j, 1)
        score.value += 10
        break
      }
    }

    // Remove off-screen enemies
    if (e.x + e.width < 0) {
      enemies.splice(i, 1)
    }
  }

  // Update and draw particles
  ctx.shadowBlur = 0
  for (let i = particles.length - 1; i >= 0; i--) {
    let p = particles[i]
    p.x += p.vx
    p.y += p.vy
    p.life -= 0.05
    if (p.life <= 0) {
      particles.splice(i, 1)
    } else {
      ctx.fillStyle = `rgba(16, 185, 129, ${p.life})`
      ctx.fillRect(p.x, p.y, 2, 2)
    }
  }

  animationFrameId = requestAnimationFrame(gameLoop)
}

const gameOver = () => {
  isGameOver.value = true
  isPlaying.value = false
  if (score.value > highScore.value) {
    highScore.value = score.value
  }
}

const fireBullet = () => {
  if (!isPlaying.value) return
  bullets.push({
    x: player.x + player.width,
    y: player.y + player.height / 2 - 1,
    width: 6,
    height: 2
  })
}

let fireInterval: ReturnType<typeof setInterval> | null = null

const handleKeydown = (e: KeyboardEvent) => {
  if (keys.hasOwnProperty(e.key)) {
    keys[e.key as keyof typeof keys] = true
    if (['ArrowUp', 'ArrowDown', 'ArrowLeft', 'ArrowRight', ' '].includes(e.key)) {
      e.preventDefault()
    }
  }

  if (e.key === ' ') {
    if (!isPlaying.value) {
      startGame()
    } else if (!fireInterval) {
      fireBullet()
      fireInterval = setInterval(fireBullet, 200) // Auto-fire while held
    }
  }
}

const handleKeyup = (e: KeyboardEvent) => {
  if (keys.hasOwnProperty(e.key)) {
    keys[e.key as keyof typeof keys] = false
  }
  if (e.key === ' ') {
    if (fireInterval) {
      clearInterval(fireInterval)
      fireInterval = null
    }
  }
}

// Mobile controls
const handleMobileFireDown = () => {
  if (!isPlaying.value) return
  fireBullet()
  if (!fireInterval) fireInterval = setInterval(fireBullet, 200)
}

const handleMobileFireUp = () => {
  if (fireInterval) {
    clearInterval(fireInterval)
    fireInterval = null
  }
}

onMounted(() => {
  window.addEventListener('keydown', handleKeydown)
  window.addEventListener('keyup', handleKeyup)
})

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeydown)
  window.removeEventListener('keyup', handleKeyup)
  if (animationFrameId) cancelAnimationFrame(animationFrameId)
  if (fireInterval) clearInterval(fireInterval)
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
        <svg class="w-4 h-4 text-accent" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19.428 15.428a2 2 0 00-1.022-.547l-2.387-.477a6 6 0 00-3.86.517l-.318.158a6 6 0 01-3.86.517L6.05 15.21a2 2 0 00-1.806.547M8 4h8l-1 1v5.172a2 2 0 00.586 1.414l5 5c1.26 1.26.367 3.414-1.415 3.414H4.828c-1.782 0-2.674-2.154-1.414-3.414l5-5A2 2 0 009 10.172V5L8 4z"></path></svg>
        space_impact.exe
      </div>
    </div>

    <div class="p-6 md:p-8 flex flex-col items-center bg-obsidian/50 relative z-10">
      
      <!-- Score Board -->
      <div class="w-full max-w-[400px] flex justify-between mb-4 font-mono text-sm">
        <div class="text-gray-400">SCORE: <span class="text-accent text-lg">{{ score }}</span></div>
        <div class="text-gray-500">HI: <span class="text-white">{{ highScore }}</span></div>
      </div>

      <!-- Game Board -->
      <div class="relative bg-obsidian border-2 border-gray-800 rounded overflow-hidden w-full max-w-[400px]" style="aspect-ratio: 2/1;">
        
        <!-- Starfield Background -->
        <div class="absolute inset-0 opacity-[0.2] bg-[url('data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIyMDAiIGhlaWdodD0iMjAwIj48cGF0aCBkPSJNMTAgMTBoMnYySDEwem01MCA1MGgydjJoLTJ6bTMwLTQwaDJ2MmgtMnptLTUwIDgwaDJ2MmgtMnptLTItNTBoMnYyaC0yem05MCA5MGgydjJoLTJ6bS00MC0zMGgydjJoLTJ6IiBmaWxsPSIjZmZmIi8+PC9zdmc+')] animate-starfield"></div>

        <canvas 
          ref="canvasRef"
          width="400" 
          height="200" 
          class="w-full h-full relative z-10 block"
        ></canvas>

        <!-- Overlays -->
        <div v-if="!isPlaying && !isGameOver" class="absolute inset-0 bg-obsidian/80 backdrop-blur-sm z-30 flex flex-col items-center justify-center">
          <div class="text-accent font-mono text-xl mb-4 font-bold tracking-widest animate-pulse">MISSION_STANDBY</div>
          <button @click="startGame" class="px-6 py-2 border border-accent/50 text-accent font-mono text-sm hover:bg-accent/10 transition-colors rounded">
            LAUNCH [SPACE]
          </button>
        </div>

        <div v-if="isGameOver" class="absolute inset-0 bg-red-900/20 backdrop-blur-sm z-30 flex flex-col items-center justify-center">
          <div class="text-red-500 font-mono text-2xl mb-2 font-bold tracking-widest">SHIP_DESTROYED</div>
          <div class="text-gray-300 font-mono text-sm mb-6">FINAL SCORE: {{ score }}</div>
          <button @click="startGame" class="px-6 py-2 border border-accent/50 text-accent font-mono text-sm hover:bg-accent/10 transition-colors rounded shadow-[0_0_15px_rgba(16,185,129,0.2)]">
            RETRY MISSION
          </button>
        </div>
      </div>

      <!-- Mobile Controls -->
      <div class="mt-6 flex gap-4 w-full max-w-[400px] md:hidden justify-between">
        <div class="grid grid-cols-3 gap-2">
          <div></div>
          <button @mousedown="keys.ArrowUp=true" @mouseup="keys.ArrowUp=false" @touchstart.prevent="keys.ArrowUp=true" @touchend.prevent="keys.ArrowUp=false" class="bg-gray-800/50 hover:bg-gray-700 active:bg-accent/50 p-3 rounded-lg flex justify-center items-center transition-colors">
            <svg class="w-5 h-5 text-gray-300" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 15l7-7 7 7"></path></svg>
          </button>
          <div></div>
          <button @mousedown="keys.ArrowLeft=true" @mouseup="keys.ArrowLeft=false" @touchstart.prevent="keys.ArrowLeft=true" @touchend.prevent="keys.ArrowLeft=false" class="bg-gray-800/50 hover:bg-gray-700 active:bg-accent/50 p-3 rounded-lg flex justify-center items-center transition-colors">
            <svg class="w-5 h-5 text-gray-300" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7"></path></svg>
          </button>
          <button @mousedown="keys.ArrowDown=true" @mouseup="keys.ArrowDown=false" @touchstart.prevent="keys.ArrowDown=true" @touchend.prevent="keys.ArrowDown=false" class="bg-gray-800/50 hover:bg-gray-700 active:bg-accent/50 p-3 rounded-lg flex justify-center items-center transition-colors">
            <svg class="w-5 h-5 text-gray-300" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7" transform="rotate(180 12 12)"></path></svg>
          </button>
          <button @mousedown="keys.ArrowRight=true" @mouseup="keys.ArrowRight=false" @touchstart.prevent="keys.ArrowRight=true" @touchend.prevent="keys.ArrowRight=false" class="bg-gray-800/50 hover:bg-gray-700 active:bg-accent/50 p-3 rounded-lg flex justify-center items-center transition-colors">
            <svg class="w-5 h-5 text-gray-300" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7"></path></svg>
          </button>
        </div>
        <div class="flex items-center">
           <button @mousedown="handleMobileFireDown" @mouseup="handleMobileFireUp" @mouseleave="handleMobileFireUp" @touchstart.prevent="handleMobileFireDown" @touchend.prevent="handleMobileFireUp" class="bg-accent/20 hover:bg-accent/30 active:bg-accent text-accent active:text-obsidian border border-accent/50 p-6 rounded-full flex justify-center items-center transition-colors shadow-[0_0_15px_rgba(16,185,129,0.2)]">
            <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 10V3L4 14h7v7l9-11h-7z"></path></svg>
          </button>
        </div>
      </div>

      <div class="mt-6 text-gray-500 font-mono text-[0.65rem] text-center max-w-[400px]">
        <span class="hidden md:inline">Use arrow keys or WASD to move. Hold SPACE to fire.</span>
        <span class="md:hidden">Use D-pad to move, tap action button to fire.</span>
      </div>
    </div>
  </div>
</template>

<style scoped>
@keyframes scanline {
  0% { transform: translateY(-100%); }
  100% { transform: translateY(100%); }
}

@keyframes starfield-move {
  from { background-position: 0 0; }
  to { background-position: -200px 0; }
}

.animate-starfield {
  animation: starfield-move 10s linear infinite;
}
</style>
