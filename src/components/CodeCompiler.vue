<script setup lang="ts">
import { ref, watch, onMounted, nextTick } from 'vue'

const languages = [
  { name: 'JavaScript', value: 'javascript', starter: 'console.log("Hello from JS!");\nconst sum = 1 + 2;\nconsole.log(`1 + 2 = ${sum}`);' },
  { name: 'HTML/CSS', value: 'html', starter: '<h1 style="color: #27c93f;">Hello World</h1>\n<p>Try changing this text!</p>\n<button onclick="alert(\'Hi!\')">Click Me</button>' },
  { name: 'Vue 3', value: 'vue', starter: `<div id="app">
  <h1 class="text-accent">{{ message }}</h1>
  <button @click="count++" class="btn">Count is: {{ count }}</button>
</div>

<script>
  const { createApp, ref } = Vue
  createApp({
    setup() {
      const message = ref('Hello from Vue 3!')
      const count = ref(0)
      return { message, count }
    }
  }).mount('#app')
<\/script>

<style>
  .text-accent { color: #27c93f; font-family: sans-serif; }
  .btn { background: #333; color: white; border: 1px solid #444; padding: 10px 20px; border-radius: 8px; cursor: pointer; margin-top: 10px; }
  .btn:hover { background: #444; }
</style>` },
  { name: 'React', value: 'react', starter: `function App() {
  const [count, setCount] = React.useState(0);
  return (
    <div style={{ fontFamily: 'sans-serif', color: '#c9d1d9' }}>
      <h1 style={{ color: '#61dafb' }}>Hello from React!</h1>
      <p>Interactive counter in the browser:</p>
      <button 
        onClick={() => setCount(count + 1)}
        style={{ padding: '10px 20px', background: '#61dafb', border: 'none', borderRadius: '5px', cursor: 'pointer' }}
      >
        Count is {count}
      </button>
    </div>
  );
}

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<App />);` },
  { name: 'Python', value: 'python', starter: 'print("Hello from Python WASM!")\nsum_val = 1 + 2\nprint(f"1 + 2 = {sum_val}")\nimport math\nprint(f"Pi is {math.pi}")' },
  { name: 'PHP', value: 'php', starter: '<?php\n\necho "Hello from PHP!\\n";\n$sum = 1 + 2;\necho "1 + 2 = " . $sum . "\\n";\n\n$arr = ["WebAssembly", "PHP", "Vue"];\nforeach($arr as $item) {\n    echo "Supported: " . $item . "\\n";\n}' }
]

const selectedLang = ref(languages[0])
const code = ref(selectedLang.value.starter)
const output = ref('')
const isRunning = ref(false)
const iframeRef = ref<HTMLIFrameElement | null>(null)

watch(selectedLang, async (newVal) => {
  code.value = newVal.starter
  output.value = ''
  if (['html', 'vue', 'react'].includes(newVal.value)) {
    await nextTick()
    renderPreview()
  }
})

const loadScript = (src: string) => {
  return new Promise((resolve, reject) => {
    if (document.querySelector(`script[src="${src}"]`)) return resolve(true)
    const script = document.createElement('script')
    script.src = src
    script.onload = resolve
    script.onerror = reject
    document.head.appendChild(script)
  })
}

const runCode = async () => {
  isRunning.value = true
  output.value = ''
  
  try {
    if (selectedLang.value.value === 'javascript') {
      output.value = '> Executing JS locally...\n'
      const logs: string[] = []
      const originalLog = console.log
      console.log = (...args) => logs.push(args.join(' '))
      try {
        new Function(code.value)()
        output.value = logs.join('\n') || '> Execution completed.'
      } catch (e) {
        output.value = `Error: ${e}`
      }
      console.log = originalLog
    } 
    
    else if (['html', 'vue', 'react'].includes(selectedLang.value.value)) {
      renderPreview()
    }

    else if (selectedLang.value.value === 'python') {
      output.value = '> Loading Python runtime (Pyodide WASM)...\n'
      await loadScript('https://cdn.jsdelivr.net/pyodide/v0.25.0/full/pyodide.js')
      const pyodide = await (window as any).loadPyodide()
      output.value = ''
      pyodide.setStdout({
        batched: (text: string) => { output.value += text + '\n' }
      })
      await pyodide.runPythonAsync(code.value)
    }

    else if (selectedLang.value.value === 'php') {
      output.value = '> Executing PHP via Piston API...\n'
      // Switching to a more reliable Piston instance with fallback
      const response = await fetch('https://emkc.org/api/v2/piston/execute', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          language: 'php',
          version: '8.2.3',
          files: [{ name: 'main.php', content: code.value }]
        })
      })
      const data = await response.json()
      if (data.run) {
        output.value = data.run.output || (data.run.stderr ? `Error:\n${data.run.stderr}` : '> PHP Execution completed.')
      } else {
        output.value = `> API Error: ${data.message || 'Unknown error'}`
      }
    }
  } catch (error) {
    output.value = `> System Error: ${error}\n\nPlease try again or switch language.`
  } finally {
    isRunning.value = false
  }
}

const renderPreview = () => {
  if (!iframeRef.value) return
  const doc = iframeRef.value.contentDocument
  if (!doc) return

  let content = ''
  
  if (selectedLang.value.value === 'html') {
    content = code.value
  } else if (selectedLang.value.value === 'vue') {
    content = `
      <script src="https://unpkg.com/vue@3/dist/vue.global.js"><\/script>
      ${code.value}
    `
  } else if (selectedLang.value.value === 'react') {
    content = `
      <div id="root"></div>
      <script src="https://unpkg.com/react@18/umd/react.development.js"><\/script>
      <script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"><\/script>
      <script src="https://unpkg.com/@babel/standalone/babel.min.js"><\/script>
      <script type="text/babel">${code.value}<\/script>
    `
  }

  doc.open()
  doc.write(`
    <!DOCTYPE html>
    <html>
    <head>
      <style>
        body { background: #0d1117; color: #c9d1d9; font-family: sans-serif; padding: 20px; line-height: 1.6; }
      </style>
    </head>
    <body>${content}</body>
    </html>
  `)
  doc.close()
}

watch(code, () => {
  if (['html', 'vue', 'react'].includes(selectedLang.value.value)) renderPreview()
})

onMounted(() => {
  if (['html', 'vue', 'react'].includes(selectedLang.value.value)) setTimeout(renderPreview, 500)
})
</script>

<template>
  <div class="terminal overflow-hidden relative group h-[550px] flex flex-col border border-gray-800 rounded-lg shadow-2xl">
    <div class="absolute inset-0 pointer-events-none opacity-[0.03] bg-gradient-to-b from-transparent via-accent to-transparent h-[100%] animate-scanline z-20"></div>
    
    <div class="terminal-header flex justify-between items-center px-4 py-2 bg-obsidian border-b border-gray-800">
      <div class="flex gap-2">
        <div class="w-3 h-3 rounded-full bg-[#ff5f56]"></div>
        <div class="w-3 h-3 rounded-full bg-[#ffbd2e]"></div>
        <div class="w-3 h-3 rounded-full bg-[#27c93f]"></div>
      </div>
      
      <div class="flex items-center gap-3">
        <div class="flex items-center bg-black/40 rounded px-2 py-1 border border-gray-700">
          <span class="text-[0.65rem] text-gray-500 font-mono mr-2">LANG:</span>
          <select v-model="selectedLang" class="bg-transparent text-accent font-mono text-[0.7rem] focus:outline-none cursor-pointer">
            <option v-for="lang in languages" :key="lang.value" :value="lang">{{ lang.name.toUpperCase() }}</option>
          </select>
        </div>
        
        <button 
          v-if="!['html', 'vue', 'react'].includes(selectedLang.value)"
          @click="runCode" :disabled="isRunning"
          class="bg-accent text-obsidian font-mono font-bold text-[0.7rem] px-4 py-1 rounded transition-all hover:brightness-110 active:scale-95 disabled:opacity-50 flex items-center gap-2 shadow-lg"
        >
          <svg v-if="!isRunning" class="w-3 h-3" fill="currentColor" viewBox="0 0 20 20"><path d="M4.5 3L15.5 10L4.5 17V3Z"/></svg>
          <svg v-else class="w-3 h-3 animate-spin" fill="none" viewBox="0 0 24 24"><circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle><path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path></svg>
          {{ isRunning ? 'EXECUTING...' : 'RUN' }}
        </button>
        <div v-else class="text-[0.65rem] text-accent/60 font-mono italic animate-pulse">Live Preview Active</div>
      </div>
    </div>

    <div class="flex-1 flex flex-col md:flex-row">
      <div class="flex-1 relative border-b md:border-b-0 md:border-r border-gray-800 bg-[#1e1e1e]/50">
        <div class="absolute top-2 left-4 text-[0.6rem] text-gray-600 font-mono z-10 pointer-events-none uppercase">Source_Code</div>
        <textarea v-model="code" spellcheck="false" class="w-full h-full bg-transparent text-gray-300 font-mono text-sm p-8 pt-10 resize-none focus:outline-none selection:bg-accent/30 custom-scrollbar leading-relaxed"></textarea>
      </div>

      <div class="flex-1 bg-black/10 relative flex flex-col">
        <div class="absolute top-2 left-4 text-[0.6rem] text-gray-600 font-mono z-10 pointer-events-none uppercase">
          {{ ['html', 'vue', 'react'].includes(selectedLang.value) ? 'Live_Render' : 'System_Output' }}
        </div>
        <div v-show="['html', 'vue', 'react'].includes(selectedLang.value)" class="flex-1 pt-8"><iframe ref="iframeRef" class="w-full h-full border-none"></iframe></div>
        <div v-show="!['html', 'vue', 'react'].includes(selectedLang.value)" class="flex-1 p-8 pt-10 font-mono text-sm overflow-y-auto custom-scrollbar">
          <pre class="text-gray-400 whitespace-pre-wrap">{{ output || '> Ready for execution...' }}</pre>
        </div>
      </div>
    </div>
    
    <div class="px-4 py-2 bg-obsidian border-t border-gray-800 flex justify-between items-center">
       <div class="flex items-center gap-4">
         <span class="text-[0.6rem] text-gray-500 font-mono flex items-center gap-1">
           <span class="w-1.5 h-1.5 rounded-full" :class="isRunning ? 'bg-accent animate-pulse' : 'bg-gray-700'"></span>
           STATUS: {{ isRunning ? 'BUSY' : 'IDLE' }}
         </span>
         <span class="text-[0.6rem] text-gray-500 font-mono flex items-center gap-1">
           <span class="w-1.5 h-1.5 rounded-full bg-blue-500"></span>
           RUNTIME: {{ ['javascript', 'python'].includes(selectedLang.value) ? 'BROWSER_WASM' : (['vue', 'react', 'html'].includes(selectedLang.value) ? 'LIVE_FRAME' : 'EXTERNAL_API') }}
         </span>
       </div>
       <div class="text-[0.6rem] text-gray-600 font-mono uppercase tracking-widest">Joey_Ventulan // Dev_Lab v3.0</div>
    </div>
  </div>
</template>

<style scoped>
.custom-scrollbar::-webkit-scrollbar { width: 6px; }
.custom-scrollbar::-webkit-scrollbar-track { background: transparent; }
.custom-scrollbar::-webkit-scrollbar-thumb { background: #1f2937; border-radius: 3px; }
.custom-scrollbar::-webkit-scrollbar-thumb:hover { background: #374151; }
@keyframes scanline { 0% { transform: translateY(-100%); } 100% { transform: translateY(100%); } }
</style>
