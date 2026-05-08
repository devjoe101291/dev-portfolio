<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const activePads = ref<Record<string, boolean>>({
  kick: false,
  snare: false,
  hihat: false,
  openhat: false,
  tomHigh: false,
  tomLow: false,
  ride: false,
  clap: false
})

const isReady = ref(false)

let audioCtx: AudioContext | null = null
const buffers = ref<Record<string, AudioBuffer>>({})

const sounds = {
  kick: '/sounds/kick.wav',
  snare: '/sounds/snare.wav',
  hihat: '/sounds/hihat.wav',
  openhat: '/sounds/openhat.wav',
  ride: '/sounds/ride.wav',
  tom: '/sounds/tom.wav'
}

const loadSamples = async () => {
  if (!audioCtx) return
  
  for (const [name, url] of Object.entries(sounds)) {
    try {
      const response = await fetch(url)
      const arrayBuffer = await response.arrayBuffer()
      const audioBuffer = await audioCtx.decodeAudioData(arrayBuffer)
      buffers.value[name] = audioBuffer
    } catch (e) {
      console.error(`Failed to load ${name}`, e)
    }
  }
}

const initAudio = async () => {
  if (!audioCtx) {
    audioCtx = new (window.AudioContext || (window as any).webkitAudioContext)()
    await loadSamples()
  }
  if (audioCtx.state === 'suspended') {
    await audioCtx.resume()
  }
  isReady.value = true
}

const playSound = (name: string, playbackRate = 1) => {
  if (!audioCtx || !buffers.value[name]) return
  
  const source = audioCtx.createBufferSource()
  source.buffer = buffers.value[name]
  source.playbackRate.value = playbackRate
  source.connect(audioCtx.destination)
  source.start(0)
  
  // Logic for animation pad triggering
  let padName = name
  if (name === 'tom') {
    padName = playbackRate > 1 ? 'tomHigh' : 'tomLow'
  }
  
  triggerPad(padName)
}

const triggerPad = (pad: string) => {
  activePads.value[pad] = true
  setTimeout(() => {
    activePads.value[pad] = false
  }, 100)
}

const handleKeydown = (e: KeyboardEvent) => {
  if (!isReady.value || e.repeat) return
  
  switch (e.key.toLowerCase()) {
    case 'a': playSound('ride'); break;
    case 's': playSound('openhat'); break;
    case 'd': playSound('hihat'); break;
    case 'f': playSound('tom', 1.5); break; // High Tom
    case 'j': playSound('tom', 0.8); break; // Low Tom
    case 'k': playSound('snare'); break;
    case 'l': playSound('kick'); break;
    case ' ': playSound('kick'); e.preventDefault(); break;
  }
}

onMounted(() => {
  window.addEventListener('keydown', handleKeydown)
})

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeydown)
  if (audioCtx) audioCtx.close()
})
</script>

<template>
  <div class="terminal overflow-hidden relative group">
    <div class="absolute inset-0 pointer-events-none opacity-[0.03] bg-gradient-to-b from-transparent via-accent to-transparent h-[100%] animate-scanline z-20"></div>
    
    <div class="terminal-header">
      <div class="flex gap-2">
        <div class="w-3 h-3 rounded-full bg-[#ff5f56]"></div>
        <div class="w-3 h-3 rounded-full bg-[#ffbd2e]"></div>
        <div class="w-3 h-3 rounded-full bg-[#27c93f]"></div>
      </div>
      <div class="mx-auto font-mono text-[0.8rem] text-gray-500 flex items-center gap-2">
        <svg class="w-4 h-4 text-accent" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19V6l12-3v13M9 19c0 1.105-1.343 2-3 2s-3-.895-3-2 1.343-2 3-2 3 .895 3 2zm12-3c0 1.105-1.343 2-3 2s-3-.895-3-2 1.343-2 3-2 3 .895 3 2zM9 10l12-3"></path></svg>
        acoustic_drum_kit.sh
      </div>
    </div>

    <div class="p-6 md:p-8 bg-obsidian/50 relative z-10 flex flex-col items-center">
      <div class="w-full max-w-[500px]">
        <div class="text-center mb-6">
          <p class="text-gray-400 font-mono text-sm mb-2">
            > loading acoustic samples... [OK]<br>
            > calibrating dynamics... [OK]<br>
            > engine ready.
          </p>
          <p class="text-accent font-mono text-xs hidden md:block">
            Play with your keyboard: A S D F J K L (SPACE for Kick)
          </p>
        </div>

        <!-- Drum Kit UI Layout -->
        <div class="relative w-full aspect-video bg-obsidian/30 rounded-xl border border-gray-800/50 p-4 mb-4 overflow-hidden">
          <!-- Play Overlay -->
          <div v-if="!isReady" class="absolute inset-0 z-30 bg-obsidian/80 backdrop-blur-sm flex flex-col items-center justify-center p-6 text-center">
            <div class="mb-6 p-4 rounded-full bg-accent/10 border border-accent/20 animate-pulse">
              <svg class="w-12 h-12 text-accent" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19V6l12-3v13M9 19c0 1.105-1.343 2-3 2s-3-.895-3-2 1.343-2 3-2 3 .895 3 2zm12-3c0 1.105-1.343 2-3 2s-3-.895-3-2 1.343-2 3-2 3 .895 3 2zM9 10l12-3"></path></svg>
            </div>
            <h4 class="text-xl font-bold mb-2">Virtual Drum Kit</h4>
            <p class="text-gray-400 text-sm mb-6 max-w-[300px]">Unlock the kit to start your session. Keyboard support will be enabled.</p>
            <button 
              @click="initAudio"
              class="px-10 py-3 bg-accent text-obsidian font-bold rounded-full hover:scale-105 transition-transform shadow-lg shadow-accent/20 uppercase tracking-widest text-sm"
            >
              Play
            </button>
          </div>

          <!-- Cymbals Row -->
          <div class="flex justify-between px-8 mb-4">
             <button @mousedown="playSound('ride')" @touchstart.prevent="playSound('ride')"
              class="w-24 h-24 rounded-full border-4 border-yellow-600/30 bg-yellow-500/10 flex items-center justify-center transition-all active:scale-95 shadow-xl"
              :class="activePads.ride ? 'bg-yellow-500/40 border-yellow-400 scale-95 shadow-yellow-500/50' : 'hover:bg-yellow-500/20'">
              <span class="text-yellow-500 font-bold text-xs">RIDE</span>
            </button>

            <div class="flex gap-4">
              <button @mousedown="playSound('openhat')" @touchstart.prevent="playSound('openhat')"
                class="w-16 h-16 rounded-full border-4 border-accent/30 bg-accent/10 flex items-center justify-center transition-all active:scale-95"
                :class="activePads.openhat ? 'bg-accent/40 border-accent scale-95' : 'hover:bg-accent/20'">
                <span class="text-accent font-bold text-[10px]">OPEN</span>
              </button>
              <button @mousedown="playSound('hihat')" @touchstart.prevent="playSound('hihat')"
                class="w-16 h-16 rounded-full border-4 border-accent/30 bg-accent/10 flex items-center justify-center transition-all active:scale-95"
                :class="activePads.hihat ? 'bg-accent/40 border-accent scale-95' : 'hover:bg-accent/20'">
                <span class="text-accent font-bold text-[10px]">HH</span>
              </button>
            </div>
          </div>

          <!-- Toms & Snare Row -->
          <div class="flex justify-center gap-6 mb-4">
            <button @mousedown="playSound('tom', 1.5)" @touchstart.prevent="playSound('tom', 1.5)"
              class="w-20 h-20 rounded-full border-4 border-blue-500/30 bg-blue-500/10 flex items-center justify-center transition-all active:scale-95"
              :class="activePads.tomHigh ? 'bg-blue-500/40 border-blue-400 scale-95' : 'hover:bg-blue-500/20'">
              <span class="text-blue-400 font-bold text-xs">TOM 1</span>
            </button>
            <button @mousedown="playSound('tom', 0.8)" @touchstart.prevent="playSound('tom', 0.8)"
              class="w-24 h-24 rounded-full border-4 border-blue-600/30 bg-blue-600/10 flex items-center justify-center transition-all active:scale-95"
              :class="activePads.tomLow ? 'bg-blue-600/40 border-blue-400 scale-95' : 'hover:bg-blue-600/20'">
              <span class="text-blue-400 font-bold text-xs">TOM 2</span>
            </button>
          </div>

          <!-- Bottom Row: Snare & Kick -->
          <div class="flex flex-col items-center">
            <div class="flex gap-12 items-end">
              <button @mousedown="playSound('snare')" @touchstart.prevent="playSound('snare')"
                class="w-28 h-28 rounded-full border-8 border-red-500/30 bg-red-500/10 flex items-center justify-center transition-all active:scale-95 shadow-lg"
                :class="activePads.snare ? 'bg-red-500/40 border-red-400 scale-95 shadow-red-500/50' : 'hover:bg-red-500/20'">
                <span class="text-red-400 font-bold">SNARE</span>
              </button>
              
              <button @mousedown="playSound('kick')" @touchstart.prevent="playSound('kick')"
                class="w-32 h-32 rounded-full border-8 border-gray-700 bg-gray-800 flex items-center justify-center transition-all active:scale-95 shadow-2xl relative overflow-hidden"
                :class="activePads.kick ? 'scale-105 border-accent shadow-accent/20' : 'hover:bg-gray-700'">
                <div v-if="activePads.kick" class="absolute inset-0 bg-accent/10 animate-pulse"></div>
                <span class="text-gray-400 font-black text-xl">KICK</span>
              </button>
            </div>
          </div>

        </div>

        <div class="text-center">
           <p class="text-gray-500 font-mono text-[0.6rem]">
            Acoustic samples provided by Wes Bos (JS30). High-fidelity audio triggered via Web Audio API.
          </p>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
@keyframes scanline {
  0% { transform: translateY(-100%); }
  100% { transform: translateY(100%); }
}
</style>
