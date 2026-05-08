<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const activePads = ref<Record<string, boolean>>({
  kick: false,
  snare: false,
  hihatClosed: false,
  hihatOpen: false,
  tomHigh: false,
  tomLow: false,
  crash: false,
  ride: false
})

let audioCtx: AudioContext | null = null

const initAudio = () => {
  if (!audioCtx) {
    audioCtx = new (window.AudioContext || (window as any).webkitAudioContext)()
  }
  if (audioCtx.state === 'suspended') {
    audioCtx.resume()
  }
}

// Highly tuned Web Audio Drum Synthesis
const playKick = () => {
  if (!audioCtx) return
  const osc = audioCtx.createOscillator()
  const gain = audioCtx.createGain()
  osc.connect(gain)
  gain.connect(audioCtx.destination)

  const time = audioCtx.currentTime
  osc.frequency.setValueAtTime(150, time)
  osc.frequency.exponentialRampToValueAtTime(0.001, time + 0.5)
  
  gain.gain.setValueAtTime(1, time)
  gain.gain.exponentialRampToValueAtTime(0.001, time + 0.5)

  osc.start(time)
  osc.stop(time + 0.5)
  triggerPad('kick')
}

const playSnare = () => {
  if (!audioCtx) return
  const time = audioCtx.currentTime

  // Snare Body
  const osc = audioCtx.createOscillator()
  const oscGain = audioCtx.createGain()
  osc.type = 'triangle'
  osc.connect(oscGain)
  oscGain.connect(audioCtx.destination)
  osc.frequency.setValueAtTime(250, time)
  oscGain.gain.setValueAtTime(0.7, time)
  oscGain.gain.exponentialRampToValueAtTime(0.01, time + 0.2)
  osc.start(time)
  osc.stop(time + 0.2)

  // Snare Noise (Wires)
  const bufferSize = audioCtx.sampleRate * 0.2
  const buffer = audioCtx.createBuffer(1, bufferSize, audioCtx.sampleRate)
  const data = buffer.getChannelData(0)
  for (let i = 0; i < bufferSize; i++) {
    data[i] = Math.random() * 2 - 1
  }

  const noise = audioCtx.createBufferSource()
  noise.buffer = buffer
  
  const noiseFilter = audioCtx.createBiquadFilter()
  noiseFilter.type = 'highpass'
  noiseFilter.frequency.value = 1000

  const noiseGain = audioCtx.createGain()
  noiseGain.gain.setValueAtTime(1, time)
  noiseGain.gain.exponentialRampToValueAtTime(0.01, time + 0.2)

  noise.connect(noiseFilter)
  noiseFilter.connect(noiseGain)
  noiseGain.connect(audioCtx.destination)

  noise.start(time)
  triggerPad('snare')
}

const playHiHat = (isOpen = false) => {
  if (!audioCtx) return
  const time = audioCtx.currentTime
  const decay = isOpen ? 0.5 : 0.1

  // Use multiple square waves for metallic sound
  const ratios = [1, 1.3420, 1.2312, 1.6532, 1.9523, 2.1523]
  const baseFreq = 400
  const bandpass = audioCtx.createBiquadFilter()
  bandpass.type = 'bandpass'
  bandpass.frequency.value = 10000

  const highpass = audioCtx.createBiquadFilter()
  highpass.type = 'highpass'
  highpass.frequency.value = 7000

  const gain = audioCtx.createGain()
  gain.gain.setValueAtTime(0.5, time)
  gain.gain.exponentialRampToValueAtTime(0.01, time + decay)

  ratios.forEach(ratio => {
    const osc = audioCtx!.createOscillator()
    osc.type = 'square'
    osc.frequency.value = baseFreq * ratio
    osc.connect(bandpass)
    osc.start(time)
    osc.stop(time + decay)
  })

  bandpass.connect(highpass)
  highpass.connect(gain)
  gain.connect(audioCtx.destination)

  triggerPad(isOpen ? 'hihatOpen' : 'hihatClosed')
}

const playTom = (isHigh = true) => {
  if (!audioCtx) return
  const osc = audioCtx.createOscillator()
  const gain = audioCtx.createGain()
  osc.connect(gain)
  gain.connect(audioCtx.destination)

  const time = audioCtx.currentTime
  const startFreq = isHigh ? 200 : 100
  osc.frequency.setValueAtTime(startFreq, time)
  osc.frequency.exponentialRampToValueAtTime(0.01, time + 0.5)
  
  gain.gain.setValueAtTime(1, time)
  gain.gain.exponentialRampToValueAtTime(0.01, time + 0.5)

  osc.start(time)
  osc.stop(time + 0.5)
  triggerPad(isHigh ? 'tomHigh' : 'tomLow')
}

const playCrash = () => {
  if (!audioCtx) return
  const time = audioCtx.currentTime
  const bufferSize = audioCtx.sampleRate * 2.0
  const buffer = audioCtx.createBuffer(1, bufferSize, audioCtx.sampleRate)
  const data = buffer.getChannelData(0)
  for (let i = 0; i < bufferSize; i++) {
    data[i] = (Math.random() * 2 - 1) * Math.exp(-i / (audioCtx.sampleRate * 0.5)) // pre-apply decay to noise
  }

  const noise = audioCtx.createBufferSource()
  noise.buffer = buffer
  
  const filter = audioCtx.createBiquadFilter()
  filter.type = 'bandpass'
  filter.frequency.value = 8000
  filter.Q.value = 0.5

  const gain = audioCtx.createGain()
  gain.gain.setValueAtTime(1.5, time)
  gain.gain.exponentialRampToValueAtTime(0.01, time + 2.0)

  noise.connect(filter)
  filter.connect(gain)
  gain.connect(audioCtx.destination)

  noise.start(time)
  triggerPad('crash')
}

const triggerPad = (pad: string) => {
  activePads.value[pad] = true
  setTimeout(() => {
    activePads.value[pad] = false
  }, 100)
}

// Keyboard mapping for finger drumming
const handleKeydown = (e: KeyboardEvent) => {
  if (e.repeat) return
  initAudio()
  switch (e.key.toLowerCase()) {
    case 'a': playCrash(); break;
    case 's': playHiHat(true); break; // Open HH
    case 'd': playHiHat(false); break; // Closed HH
    case 'f': playTom(true); break; // High Tom
    case 'j': playTom(false); break; // Low Tom
    case 'k': playSnare(); break;
    case 'l': playKick(); break;
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
        drum_kit_emulator.sh
      </div>
    </div>

    <div class="p-6 md:p-8 bg-obsidian/50 relative z-10 flex flex-col items-center">
      <div class="w-full max-w-[500px]">
        <div class="text-center mb-6">
          <p class="text-gray-400 font-mono text-sm mb-2">
            > initializing drum synthesizer... [OK]<br>
            > loading accurate 808/909 style waveforms... [OK]<br>
            > ready for input.
          </p>
          <p class="text-accent font-mono text-xs hidden md:block">
            Pro Tip: Use your keyboard to play! (A S D F J K L)
          </p>
        </div>

        <div class="grid grid-cols-3 gap-3 md:gap-4 mt-8">
          
          <button @mousedown="() => { initAudio(); playCrash(); }" @touchstart.prevent="() => { initAudio(); playCrash(); }"
            class="h-24 border border-gray-700 rounded flex flex-col items-center justify-center transition-all duration-75 relative overflow-hidden group"
            :class="activePads.crash ? 'bg-[#ffbd2e]/20 border-[#ffbd2e] shadow-[0_0_15px_rgba(255,189,46,0.3)] scale-95' : 'bg-panel hover:bg-gray-800'">
            <div class="text-gray-300 font-bold tracking-wider mb-1">CRASH</div>
            <div class="text-[0.6rem] text-gray-500 font-mono hidden md:block">[A]</div>
            <div v-if="activePads.crash" class="absolute bottom-0 left-0 h-1 bg-[#ffbd2e] w-full animate-ping"></div>
          </button>

          <button @mousedown="() => { initAudio(); playHiHat(true); }" @touchstart.prevent="() => { initAudio(); playHiHat(true); }"
            class="h-24 border border-gray-700 rounded flex flex-col items-center justify-center transition-all duration-75 relative overflow-hidden group"
            :class="activePads.hihatOpen ? 'bg-accent/20 border-accent shadow-[0_0_15px_rgba(16,185,129,0.3)] scale-95' : 'bg-panel hover:bg-gray-800'">
            <div class="text-gray-300 font-bold tracking-wider mb-1">OPEN HH</div>
            <div class="text-[0.6rem] text-gray-500 font-mono hidden md:block">[S]</div>
            <div v-if="activePads.hihatOpen" class="absolute bottom-0 left-0 h-1 bg-accent w-full animate-ping"></div>
          </button>

          <button @mousedown="() => { initAudio(); playHiHat(false); }" @touchstart.prevent="() => { initAudio(); playHiHat(false); }"
            class="h-24 border border-gray-700 rounded flex flex-col items-center justify-center transition-all duration-75 relative overflow-hidden group"
            :class="activePads.hihatClosed ? 'bg-accent/20 border-accent shadow-[0_0_15px_rgba(16,185,129,0.3)] scale-95' : 'bg-panel hover:bg-gray-800'">
            <div class="text-gray-300 font-bold tracking-wider mb-1">HI-HAT</div>
            <div class="text-[0.6rem] text-gray-500 font-mono hidden md:block">[D]</div>
            <div v-if="activePads.hihatClosed" class="absolute bottom-0 left-0 h-1 bg-accent w-full animate-ping"></div>
          </button>

          <button @mousedown="() => { initAudio(); playTom(true); }" @touchstart.prevent="() => { initAudio(); playTom(true); }"
            class="h-24 border border-gray-700 rounded flex flex-col items-center justify-center transition-all duration-75 relative overflow-hidden group"
            :class="activePads.tomHigh ? 'bg-[#27c93f]/20 border-[#27c93f] shadow-[0_0_15px_rgba(39,201,63,0.3)] scale-95' : 'bg-panel hover:bg-gray-800'">
            <div class="text-gray-300 font-bold tracking-wider mb-1">HI TOM</div>
            <div class="text-[0.6rem] text-gray-500 font-mono hidden md:block">[F]</div>
            <div v-if="activePads.tomHigh" class="absolute bottom-0 left-0 h-1 bg-[#27c93f] w-full animate-ping"></div>
          </button>

          <button @mousedown="() => { initAudio(); playTom(false); }" @touchstart.prevent="() => { initAudio(); playTom(false); }"
            class="h-24 border border-gray-700 rounded flex flex-col items-center justify-center transition-all duration-75 relative overflow-hidden group"
            :class="activePads.tomLow ? 'bg-[#27c93f]/20 border-[#27c93f] shadow-[0_0_15px_rgba(39,201,63,0.3)] scale-95' : 'bg-panel hover:bg-gray-800'">
            <div class="text-gray-300 font-bold tracking-wider mb-1">LOW TOM</div>
            <div class="text-[0.6rem] text-gray-500 font-mono hidden md:block">[J]</div>
            <div v-if="activePads.tomLow" class="absolute bottom-0 left-0 h-1 bg-[#27c93f] w-full animate-ping"></div>
          </button>

          <button @mousedown="() => { initAudio(); playSnare(); }" @touchstart.prevent="() => { initAudio(); playSnare(); }"
            class="h-24 border border-gray-700 rounded flex flex-col items-center justify-center transition-all duration-75 relative overflow-hidden group col-span-3 md:col-span-1"
            :class="activePads.snare ? 'bg-[#ff5f56]/20 border-[#ff5f56] shadow-[0_0_15px_rgba(255,95,86,0.3)] scale-95' : 'bg-panel hover:bg-gray-800'">
            <div class="text-gray-300 font-bold tracking-wider mb-1">SNARE</div>
            <div class="text-[0.6rem] text-gray-500 font-mono hidden md:block">[K]</div>
            <div v-if="activePads.snare" class="absolute bottom-0 left-0 h-1 bg-[#ff5f56] w-full animate-ping"></div>
          </button>

          <button @mousedown="() => { initAudio(); playKick(); }" @touchstart.prevent="() => { initAudio(); playKick(); }"
            class="h-24 border border-gray-700 rounded flex flex-col items-center justify-center transition-all duration-75 relative overflow-hidden group col-span-3 mt-2"
            :class="activePads.kick ? 'bg-[#ff5f56]/20 border-[#ff5f56] shadow-[0_0_15px_rgba(255,95,86,0.3)] scale-95' : 'bg-panel hover:bg-gray-800'">
            <div class="text-gray-300 font-bold tracking-wider mb-1">KICK DRUM</div>
            <div class="text-[0.6rem] text-gray-500 font-mono hidden md:block">[L]</div>
            <div v-if="activePads.kick" class="absolute bottom-0 left-0 h-1 bg-[#ff5f56] w-full animate-ping"></div>
          </button>

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
