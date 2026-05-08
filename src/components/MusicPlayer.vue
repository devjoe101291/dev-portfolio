<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const isPlaying = ref(false)
const volume = ref(0.5)
const audio = ref<HTMLAudioElement | null>(null)
const currentTrack = ref(0)

const tracks = [
  {
    title: 'Lofi Radio (Live)',
    artist: 'Programmer Vibes',
    url: 'https://stream.zeno.fm/0r0xa792kwzuv'
  },
  {
    title: 'Chill Focus',
    artist: 'Ambient Tech',
    url: 'https://www.soundhelix.com/examples/mp3/SoundHelix-Song-17.mp3'
  },
  {
    title: 'Terminal Flow',
    artist: 'Synthwave',
    url: 'https://www.soundhelix.com/examples/mp3/SoundHelix-Song-8.mp3'
  }
]

const togglePlay = () => {
  if (!audio.value) return
  if (isPlaying.value) {
    audio.value.pause()
  } else {
    audio.value.play().catch(e => console.error("Playback failed:", e))
  }
  isPlaying.value = !isPlaying.value
}

const updateVolume = (e: Event) => {
  const target = e.target as HTMLInputElement
  volume.value = parseFloat(target.value)
  if (audio.value) {
    audio.value.volume = volume.value
  }
}

const nextTrack = () => {
  currentTrack.value = (currentTrack.value + 1) % tracks.length
  if (audio.value) {
    audio.value.src = tracks[currentTrack.value].url
    if (isPlaying.value) audio.value.play()
  }
}

const isMinimized = ref(false)

onMounted(() => {
  audio.value = new Audio(tracks[currentTrack.value].url)
  audio.value.volume = volume.value
  audio.value.loop = true
  
  // Auto-minimize on mobile
  if (window.innerWidth < 768) {
    isMinimized.value = true
  }
})

onUnmounted(() => {
  if (audio.value) {
    audio.value.pause()
    audio.value = null
  }
})
</script>

<template>
  <div class="fixed bottom-6 right-6 z-50 transition-all duration-500" :class="isMinimized ? 'translate-y-2 translate-x-2' : ''">
    <div v-if="!isMinimized" class="terminal-music-player bg-panel border border-gray-800 rounded-lg p-4 shadow-2xl transition-all duration-300 hover:border-accent/50 w-64 backdrop-blur-md relative overflow-hidden">
      <!-- CRT scanline overlay for the player -->
      <div class="absolute inset-0 pointer-events-none opacity-[0.03] bg-gradient-to-b from-transparent via-accent to-transparent h-[100%] animate-scanline"></div>
      
      <div class="flex items-center justify-between mb-3 border-b border-gray-800 pb-2 relative z-10">
        <div class="flex items-center gap-2">
          <div class="w-2 h-2 rounded-full" :class="isPlaying ? 'bg-accent animate-pulse' : 'bg-gray-600'"></div>
          <span class="font-mono text-[0.7rem] text-gray-500 uppercase tracking-widest">audio_output.sys</span>
        </div>
        <div class="flex gap-2">
          <button @click="nextTrack" class="text-gray-500 hover:text-accent transition-colors" title="Next Track">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 5l7 7-7 7M5 5l7 7-7 7"></path>
            </svg>
          </button>
          <button @click="isMinimized = true" class="text-gray-500 hover:text-accent transition-colors" title="Minimize">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"></path>
            </svg>
          </button>
        </div>
      </div>

      <div class="flex items-center gap-4 relative z-10">
        <button 
          @click="togglePlay"
          class="w-12 h-12 rounded-full border border-accent/20 flex items-center justify-center hover:bg-accent/10 transition-all group shrink-0"
        >
          <svg v-if="!isPlaying" class="w-6 h-6 text-accent group-hover:scale-110 transition-transform" fill="currentColor" viewBox="0 0 24 24">
            <path d="M8 5v14l11-7z"></path>
          </svg>
          <svg v-else class="w-6 h-6 text-accent group-hover:scale-110 transition-transform" fill="currentColor" viewBox="0 0 24 24">
            <path d="M6 19h4V5H6v14zm8-14v14h4V5h-4z"></path>
          </svg>
        </button>

        <div class="flex-1 overflow-hidden">
          <div class="font-mono text-xs text-accent truncate">{{ tracks[currentTrack].title }}</div>
          <div class="font-mono text-[0.6rem] text-gray-500 truncate">{{ tracks[currentTrack].artist }}</div>
          
          <div class="mt-2 flex items-center gap-2">
            <svg class="w-3 h-3 text-gray-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15.536 8.464a5 5 0 010 7.072m2.828-9.9a9 9 0 010 12.728M5.586 15H4a1 1 0 01-1-1v-4a1 1 0 011-1h1.586l4.707-4.707C10.923 3.663 12 4.109 12 5v14c0 .891-1.077 1.337-1.707.707L5.586 15z"></path>
            </svg>
            <input 
              type="range" 
              min="0" 
              max="1" 
              step="0.01" 
              :value="volume" 
              @input="updateVolume"
              class="w-full h-1 bg-gray-800 rounded-lg appearance-none cursor-pointer accent-accent opacity-50 hover:opacity-100 transition-opacity"
            >
          </div>
        </div>
      </div>

      <!-- Simple Visualizer -->
      <div v-if="isPlaying" class="mt-3 flex items-end justify-center gap-[2px] h-4 relative z-10">
        <div v-for="i in 16" :key="i" 
             class="w-1 bg-accent/30 rounded-t-sm animate-music-bar"
             :style="{ height: Math.random() * 100 + '%', animationDelay: (i * 0.1) + 's' }">
        </div>
      </div>
    </div>

    <!-- Minimized State -->
    <button 
      v-else 
      @click="isMinimized = false"
      class="bg-panel border border-gray-800 p-3 rounded-full shadow-2xl hover:border-accent/50 transition-all group flex items-center justify-center"
      title="Show Player"
    >
      <div v-if="isPlaying" class="flex gap-[2px] items-end h-4 w-4 mr-1">
        <div class="w-[2px] bg-accent animate-music-bar h-[40%]"></div>
        <div class="w-[2px] bg-accent animate-music-bar h-[100%] [animation-delay:0.2s]"></div>
        <div class="w-[2px] bg-accent animate-music-bar h-[60%] [animation-delay:0.4s]"></div>
      </div>
      <svg v-else class="w-5 h-5 text-gray-500 group-hover:text-accent" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19V6l12-3v13M9 19c0 1.105-1.343 2-3 2s-3-.895-3-2 1.343-2 3-2 3 .895 3 2zm12-3c0 1.105-1.343 2-3 2s-3-.895-3-2 1.343-2 3-2 3 .895 3 2zM9 10l12-3"></path>
      </svg>
    </button>
  </div>
</template>

<style scoped>
.animate-music-bar {
  animation: music-bar 1.2s ease-in-out infinite;
}

@keyframes music-bar {
  0%, 100% { height: 20%; }
  50% { height: 100%; }
}

@keyframes scanline {
  0% { transform: translateY(-100%); }
  100% { transform: translateY(100%); }
}

input[type=range]::-webkit-slider-thumb {
  -webkit-appearance: none;
  height: 8px;
  width: 8px;
  border-radius: 50%;
  background: #10b981; /* Fallback for accent */
  cursor: pointer;
}
</style>
