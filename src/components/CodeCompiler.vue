<script setup lang="ts">
import { ref, watch } from 'vue'

const languages = [
  { name: 'PHP', value: 'php', version: '8.2.3', starter: '<?php\n\necho "Hello from PHP!\\n";\n$sum = 1 + 2;\necho "1 + 2 = " . $sum . "\\n";' },
  { name: 'JavaScript', value: 'javascript', version: '18.15.0', starter: 'console.log("Hello from JS!");\nconst sum = 1 + 2;\nconsole.log(`1 + 2 = ${sum}`);' },
  { name: 'Python', value: 'python', version: '3.10.0', starter: 'print("Hello from Python!")\nsum_val = 1 + 2\nprint(f"1 + 2 = {sum_val}")' },
  { name: 'C++', value: 'c++', version: '10.2.0', starter: '#include <iostream>\n\nint main() {\n    std::cout << "Hello from C++!" << std::endl;\n    return 0;\n}' }
]

const selectedLang = ref(languages[0])
const code = ref(selectedLang.value.starter)
const output = ref('')
const isRunning = ref(false)

watch(selectedLang, (newVal) => {
  code.value = newVal.starter
  output.value = ''
})

const runCode = async () => {
  isRunning.value = true
  output.value = '> Initializing environment...\n'
  
  try {
    const response = await fetch('https://emkc.org/api/v2/piston/execute', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        language: selectedLang.value.value,
        version: selectedLang.value.version,
        files: [{ 
          name: 'main',
          content: code.value 
        }]
      })
    })

    const data = await response.json()
    
    if (data.run) {
      output.value = data.run.output || (data.run.stderr ? `Error:\n${data.run.stderr}` : '> Execution completed with no output.')
    } else {
      output.value = `> Error: ${data.message || 'Unable to reach the execution engine.'}`
    }
  } catch (error) {
    output.value = `> Network Error: ${error}`
  } finally {
    isRunning.value = false
  }
}
</script>

<template>
  <div class="terminal overflow-hidden relative group h-[500px] flex flex-col">
    <!-- CRT overlay -->
    <div class="absolute inset-0 pointer-events-none opacity-[0.03] bg-gradient-to-b from-transparent via-accent to-transparent h-[100%] animate-scanline z-20"></div>
    
    <div class="terminal-header flex justify-between items-center px-4">
      <div class="flex gap-2">
        <div class="w-3 h-3 rounded-full bg-[#ff5f56]"></div>
        <div class="w-3 h-3 rounded-full bg-[#ffbd2e]"></div>
        <div class="w-3 h-3 rounded-full bg-[#27c93f]"></div>
      </div>
      
      <div class="flex items-center gap-4">
        <select 
          v-model="selectedLang" 
          class="bg-obsidian border border-gray-700 text-gray-400 font-mono text-[0.7rem] px-2 py-1 rounded focus:outline-none focus:border-accent"
        >
          <option v-for="lang in languages" :key="lang.value" :value="lang">
            {{ lang.name }}
          </option>
        </select>
        
        <button 
          @click="runCode" 
          :disabled="isRunning"
          class="bg-accent/20 hover:bg-accent/30 text-accent font-mono text-[0.7rem] px-3 py-1 rounded transition-all flex items-center gap-2 disabled:opacity-50"
        >
          <svg v-if="!isRunning" class="w-3 h-3" fill="currentColor" viewBox="0 0 20 20"><path d="M4.5 3L15.5 10L4.5 17V3Z"/></svg>
          <svg v-else class="w-3 h-3 animate-spin" fill="none" viewBox="0 0 24 24"><circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle><path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path></svg>
          {{ isRunning ? 'RUNNING...' : 'RUN' }}
        </button>
      </div>
    </div>

    <div class="flex-1 flex flex-col md:flex-row border-t border-gray-800">
      <!-- Editor Area -->
      <div class="flex-1 relative border-b md:border-b-0 md:border-r border-gray-800">
        <div class="absolute top-2 right-2 text-[0.6rem] text-gray-600 font-mono z-10">EDITOR</div>
        <textarea
          v-model="code"
          spellcheck="false"
          class="w-full h-full bg-transparent text-gray-300 font-mono text-sm p-4 resize-none focus:outline-none selection:bg-accent/30 custom-scrollbar"
        ></textarea>
      </div>

      <!-- Output Area -->
      <div class="flex-1 bg-black/20 relative flex flex-col">
        <div class="absolute top-2 right-2 text-[0.6rem] text-gray-600 font-mono z-10">CONSOLE</div>
        <div class="flex-1 p-4 font-mono text-sm overflow-y-auto custom-scrollbar">
          <pre class="text-gray-400 whitespace-pre-wrap">{{ output || '> Output will appear here...' }}</pre>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.custom-scrollbar::-webkit-scrollbar {
  width: 6px;
}
.custom-scrollbar::-webkit-scrollbar-track {
  background: transparent;
}
.custom-scrollbar::-webkit-scrollbar-thumb {
  background: #1f2937;
  border-radius: 3px;
}
.custom-scrollbar::-webkit-scrollbar-thumb:hover {
  background: #374151;
}
@keyframes scanline {
  0% { transform: translateY(-100%); }
  100% { transform: translateY(100%); }
}
</style>
