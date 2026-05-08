<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const isPlaying = ref(false)
const activeInstruments = ref<Record<string, boolean>>({
  kick: false,
  snare: false,
  hihat: false,
  bass: false,
  chord: false,
  lead: false
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

// Simple synth functions using Web Audio API
const playKick = () => {
  if (!audioCtx) return
  const osc = audioCtx.createOscillator()
  const gainNode = audioCtx.createGain()
  osc.connect(gainNode)
  gainNode.connect(audioCtx.destination)

  osc.type = 'sine'
  osc.frequency.setValueAtTime(150, audioCtx.currentTime)
  osc.frequency.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.5)
  
  gainNode.gain.setValueAtTime(1, audioCtx.currentTime)
  gainNode.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.5)

  osc.start(audioCtx.currentTime)
  osc.stop(audioCtx.currentTime + 0.5)
  triggerAnimation('kick')
}

const playSnare = () => {
  if (!audioCtx) return
  const bufferSize = audioCtx.sampleRate * 0.2
  const buffer = audioCtx.createBuffer(1, bufferSize, audioCtx.sampleRate)
  const data = buffer.getChannelData(0)
  for (let i = 0; i < bufferSize; i++) {
    data[i] = Math.random() * 2 - 1
  }

  const noiseSource = audioCtx.createBufferSource()
  noiseSource.buffer = buffer
  
  const noiseFilter = audioCtx.createBiquadFilter()
  noiseFilter.type = 'bandpass'
  noiseFilter.frequency.value = 1000

  const noiseGain = audioCtx.createGain()
  noiseGain.gain.setValueAtTime(1, audioCtx.currentTime)
  noiseGain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.2)

  noiseSource.connect(noiseFilter)
  noiseFilter.connect(noiseGain)
  noiseGain.connect(audioCtx.destination)

  // Add a quick sine sweep for the 'snap'
  const osc = audioCtx.createOscillator()
  const oscGain = audioCtx.createGain()
  osc.type = 'triangle'
  osc.connect(oscGain)
  oscGain.connect(audioCtx.destination)
  osc.frequency.setValueAtTime(250, audioCtx.currentTime)
  oscGain.gain.setValueAtTime(0.5, audioCtx.currentTime)
  oscGain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.1)

  osc.start(audioCtx.currentTime)
  osc.stop(audioCtx.currentTime + 0.1)
  noiseSource.start(audioCtx.currentTime)
  triggerAnimation('snare')
}

const playHiHat = () => {
  if (!audioCtx) return
  const bufferSize = audioCtx.sampleRate * 0.1
  const buffer = audioCtx.createBuffer(1, bufferSize, audioCtx.sampleRate)
  const data = buffer.getChannelData(0)
  for (let i = 0; i < bufferSize; i++) {
    data[i] = Math.random() * 2 - 1
  }

  const noiseSource = audioCtx.createBufferSource()
  noiseSource.buffer = buffer
  
  const bandpass = audioCtx.createBiquadFilter()
  bandpass.type = 'bandpass'
  bandpass.frequency.value = 10000

  const gainNode = audioCtx.createGain()
  gainNode.gain.setValueAtTime(0.5, audioCtx.currentTime)
  gainNode.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.1)

  noiseSource.connect(bandpass)
  bandpass.connect(gainNode)
  gainNode.connect(audioCtx.destination)

  noiseSource.start(audioCtx.currentTime)
  triggerAnimation('hihat')
}

const playBass = () => {
  if (!audioCtx) return
  const osc = audioCtx.createOscillator()
  const gainNode = audioCtx.createGain()
  osc.connect(gainNode)
  gainNode.connect(audioCtx.destination)

  osc.type = 'sawtooth'
  // E1 note
  osc.frequency.setValueAtTime(41.20, audioCtx.currentTime)
  
  const filter = audioCtx.createBiquadFilter()
  filter.type = 'lowpass'
  filter.frequency.setValueAtTime(400, audioCtx.currentTime)
  filter.frequency.exponentialRampToValueAtTime(50, audioCtx.currentTime + 0.5)

  osc.connect(filter)
  filter.connect(gainNode)
  gainNode.disconnect() // disconnect from direct to use filter

  gainNode.gain.setValueAtTime(0.8, audioCtx.currentTime)
  gainNode.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.5)

  osc.start(audioCtx.currentTime)
  osc.stop(audioCtx.currentTime + 0.5)
  triggerAnimation('bass')
}

const playChord = () => {
  if (!audioCtx) return
  // E minor chord
  const freqs = [164.81, 196.00, 246.94] // E3, G3, B3
  
  freqs.forEach(freq => {
    const osc = audioCtx!.createOscillator()
    const gainNode = audioCtx!.createGain()
    osc.connect(gainNode)
    gainNode.connect(audioCtx!.destination)

    osc.type = 'triangle'
    osc.frequency.setValueAtTime(freq, audioCtx!.currentTime)
    
    gainNode.gain.setValueAtTime(0.2, audioCtx!.currentTime)
    gainNode.gain.linearRampToValueAtTime(0.1, audioCtx!.currentTime + 0.1)
    gainNode.gain.exponentialRampToValueAtTime(0.01, audioCtx!.currentTime + 0.8)

    osc.start(audioCtx!.currentTime)
    osc.stop(audioCtx!.currentTime + 0.8)
  })
  triggerAnimation('chord')
}

const playLead = () => {
  if (!audioCtx) return
  const osc = audioCtx.createOscillator()
  const gainNode = audioCtx.createGain()
  
  const filter = audioCtx.createBiquadFilter()
  filter.type = 'lowpass'
  filter.frequency.value = 2000

  osc.connect(filter)
  filter.connect(gainNode)
  gainNode.connect(audioCtx.destination)

  osc.type = 'square'
  // Random pentatonic note
  const notes = [329.63, 392.00, 440.00, 493.88, 587.33] // E4, G4, A4, B4, D5
  const freq = notes[Math.floor(Math.random() * notes.length)]
  osc.frequency.setValueAtTime(freq, audioCtx.currentTime)
  
  gainNode.gain.setValueAtTime(0.15, audioCtx.currentTime)
  gainNode.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.3)

  osc.start(audioCtx.currentTime)
  osc.stop(audioCtx.currentTime + 0.3)
  triggerAnimation('lead')
}

const triggerAnimation = (instrument: string) => {
  activeInstruments.value[instrument] = true
  setTimeout(() => {
    activeInstruments.value[instrument] = false
  }, 100)
}

// Simple sequencer
let step = 0
let sequencerInterval: number | null = null

const toggleSequencer = () => {
  initAudio()
  isPlaying.value = !isPlaying.value
  
  if (isPlaying.value) {
    step = 0
    sequencerInterval = setInterval(() => {
      // Basic drum beat
      if (step % 4 === 0) playKick()
      if (step % 4 === 2) playSnare()
      playHiHat()
      
      // Bass line
      if (step % 8 === 0 || step % 8 === 3) playBass()
      
      // Chords
      if (step % 16 === 0 || step % 16 === 10) playChord()
      
      // Lead riff (randomized slightly)
      if (Math.random() > 0.7 && step % 2 !== 0) playLead()
      
      step = (step + 1) % 16
    }, 250) // 120 BPM 8th notes
  } else {
    if (sequencerInterval) clearInterval(sequencerInterval)
  }
}

onUnmounted(() => {
  if (sequencerInterval) clearInterval(sequencerInterval)
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
        virtual_band_demo.sh
      </div>
    </div>

    <div class="p-6 md:p-8 bg-obsidian/50 relative z-10 flex flex-col items-center">
      <div class="w-full max-w-[500px]">
        <div class="text-center mb-6">
          <p class="text-gray-400 font-mono text-sm mb-4">
            > initializing audio context... [OK]<br>
            > loading instruments... [OK]<br>
            > click pads to play manually, or start the sequencer.
          </p>
          <button 
            @click="toggleSequencer" 
            class="px-8 py-3 border font-mono text-sm tracking-widest uppercase transition-all duration-300 rounded shadow-[0_0_15px_rgba(16,185,129,0.1)]"
            :class="isPlaying ? 'bg-accent text-obsidian border-accent' : 'border-accent text-accent hover:bg-accent/10'"
          >
            {{ isPlaying ? 'STOP SEQUENCER' : 'AUTO-PLAY DEMO' }}
          </button>
        </div>

        <div class="grid grid-cols-3 gap-3 md:gap-4 mt-8">
          <!-- Drums -->
          <button @mousedown="() => { initAudio(); playKick(); }" @touchstart.prevent="() => { initAudio(); playKick(); }"
            class="h-20 border border-gray-700 rounded flex flex-col items-center justify-center transition-all duration-75 relative overflow-hidden group"
            :class="activeInstruments.kick ? 'bg-accent/20 border-accent shadow-[0_0_15px_rgba(16,185,129,0.3)] scale-95' : 'bg-panel hover:bg-gray-800'">
            <div class="text-gray-300 font-bold tracking-wider mb-1">KICK</div>
            <div class="text-xs text-gray-600 font-mono">DRUMS</div>
            <div v-if="activeInstruments.kick" class="absolute bottom-0 left-0 h-1 bg-accent w-full animate-ping"></div>
          </button>

          <button @mousedown="() => { initAudio(); playSnare(); }" @touchstart.prevent="() => { initAudio(); playSnare(); }"
            class="h-20 border border-gray-700 rounded flex flex-col items-center justify-center transition-all duration-75 relative overflow-hidden group"
            :class="activeInstruments.snare ? 'bg-accent/20 border-accent shadow-[0_0_15px_rgba(16,185,129,0.3)] scale-95' : 'bg-panel hover:bg-gray-800'">
            <div class="text-gray-300 font-bold tracking-wider mb-1">SNARE</div>
            <div class="text-xs text-gray-600 font-mono">DRUMS</div>
            <div v-if="activeInstruments.snare" class="absolute bottom-0 left-0 h-1 bg-accent w-full animate-ping"></div>
          </button>

          <button @mousedown="() => { initAudio(); playHiHat(); }" @touchstart.prevent="() => { initAudio(); playHiHat(); }"
            class="h-20 border border-gray-700 rounded flex flex-col items-center justify-center transition-all duration-75 relative overflow-hidden group"
            :class="activeInstruments.hihat ? 'bg-accent/20 border-accent shadow-[0_0_15px_rgba(16,185,129,0.3)] scale-95' : 'bg-panel hover:bg-gray-800'">
            <div class="text-gray-300 font-bold tracking-wider mb-1">HI-HAT</div>
            <div class="text-xs text-gray-600 font-mono">CYMBAL</div>
            <div v-if="activeInstruments.hihat" class="absolute bottom-0 left-0 h-1 bg-accent w-full animate-ping"></div>
          </button>

          <!-- Instruments -->
          <button @mousedown="() => { initAudio(); playBass(); }" @touchstart.prevent="() => { initAudio(); playBass(); }"
            class="h-20 border border-gray-700 rounded flex flex-col items-center justify-center transition-all duration-75 relative overflow-hidden group"
            :class="activeInstruments.bass ? 'bg-[#ff5f56]/20 border-[#ff5f56] shadow-[0_0_15px_rgba(255,95,86,0.3)] scale-95' : 'bg-panel hover:bg-gray-800'">
            <div class="text-gray-300 font-bold tracking-wider mb-1">BASS</div>
            <div class="text-xs text-gray-600 font-mono">4-STRING</div>
            <div v-if="activeInstruments.bass" class="absolute bottom-0 left-0 h-1 bg-[#ff5f56] w-full animate-ping"></div>
          </button>

          <button @mousedown="() => { initAudio(); playChord(); }" @touchstart.prevent="() => { initAudio(); playChord(); }"
            class="h-20 border border-gray-700 rounded flex flex-col items-center justify-center transition-all duration-75 relative overflow-hidden group"
            :class="activeInstruments.chord ? 'bg-[#ffbd2e]/20 border-[#ffbd2e] shadow-[0_0_15px_rgba(255,189,46,0.3)] scale-95' : 'bg-panel hover:bg-gray-800'">
            <div class="text-gray-300 font-bold tracking-wider mb-1">RHYTHM</div>
            <div class="text-xs text-gray-600 font-mono">GUITAR</div>
            <div v-if="activeInstruments.chord" class="absolute bottom-0 left-0 h-1 bg-[#ffbd2e] w-full animate-ping"></div>
          </button>

          <button @mousedown="() => { initAudio(); playLead(); }" @touchstart.prevent="() => { initAudio(); playLead(); }"
            class="h-20 border border-gray-700 rounded flex flex-col items-center justify-center transition-all duration-75 relative overflow-hidden group"
            :class="activeInstruments.lead ? 'bg-[#27c93f]/20 border-[#27c93f] shadow-[0_0_15px_rgba(39,201,63,0.3)] scale-95' : 'bg-panel hover:bg-gray-800'">
            <div class="text-gray-300 font-bold tracking-wider mb-1">LEAD</div>
            <div class="text-xs text-gray-600 font-mono">SYNTH</div>
            <div v-if="activeInstruments.lead" class="absolute bottom-0 left-0 h-1 bg-[#27c93f] w-full animate-ping"></div>
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
