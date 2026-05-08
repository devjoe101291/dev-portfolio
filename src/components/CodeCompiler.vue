<script setup lang="ts">
import { ref, watch, onMounted, nextTick } from 'vue'

const languages = [
  { name: 'JavaScript', value: 'javascript', starter: 'console.log("Hello from JS!");\nconst sum = 1 + 2;\nconsole.log(`1 + 2 = ${sum}`);' },
  { name: 'HTML/CSS', value: 'html', starter: '<h1 style="color: #27c93f;">Hello World</h1>\n<p>Try changing this text!</p>\n<button onclick="alert(\'Hi!\')">Click Me</button>' },
  { name: 'Vue 3', value: 'vue', starter: `<div id="app">\n  <h1 class="text-accent">{{ message }}</h1>\n  <button @click="count++" class="btn">Count is: {{ count }}</button>\n</div>\n\n<script>\n  const { createApp, ref } = Vue\n  createApp({\n    setup() {\n      const message = ref('Hello from Vue 3!')\n      const count = ref(0)\n      return { message, count }\n    }\n  }).mount('#app')\n<\/script>\n\n<style>\n  .text-accent { color: #27c93f; font-family: sans-serif; }\n  .btn { background: #333; color: white; border: 1px solid #444; padding: 10px 20px; border-radius: 8px; cursor: pointer; margin-top: 10px; }\n  .btn:hover { background: #444; }\n</style>` },
  { name: 'React', value: 'react', starter: `function App() {\n  const [count, setCount] = React.useState(0);\n  return (\n    <div style={{ fontFamily: 'sans-serif', color: '#c9d1d9' }}>\n      <h1 style={{ color: '#61dafb' }}>Hello from React!</h1>\n      <button \n        onClick={() => setCount(count + 1)}\n        style={{ padding: '10px 20px', background: '#61dafb', border: 'none', borderRadius: '5px', cursor: 'pointer' }}\n      >\n        Count is {count}\n      </button>\n    </div>\n  );\n}\n\nconst root = ReactDOM.createRoot(document.getElementById('root'));\nroot.render(<App />);` },
  { name: 'React Native', value: 'reactnative', starter: `// Mocking React Native Components\nconst View = ({ children, style }) => <div style={{ ...style, display: 'flex', flexDirection: 'column' }}>{children}</div>;\nconst Text = ({ children, style }) => <span style={style}>{children}</span>;\nconst StyleSheet = { create: (obj) => obj };\n\nfunction MobileApp() {\n  return (\n    <View style={styles.container}>\n      <Text style={styles.title}>Hello from React Native!</Text>\n      <Text style={styles.subtitle}>Cross-platform mobile development.</Text>\n    </View>\n  );\n}\n\nconst styles = StyleSheet.create({\n  container: { padding: 20, alignItems: 'center' },\n  title: { fontSize: 24, color: '#61dafb', fontWeight: 'bold' },\n  subtitle: { color: '#888', marginTop: 10 }\n});\n\nconst root = ReactDOM.createRoot(document.getElementById('root'));\nroot.render(<MobileApp />);` },
  { name: 'Laravel', value: 'laravel', starter: `<?php\n\n// Mocking Laravel Route Engine\nclass Route {\n    public static function get($uri, $callback) {\n        echo "Laravel [v10.x] Route: GET $uri\\n";\n        if (is_callable($callback)) return $callback();\n    }\n}\n\nRoute::get('/api/profile', function() {\n    return print_r(['user' => 'Joey Ventulan', 'status' => 'Active'], true);\n});` },
  { name: 'CodeIgniter', value: 'codeigniter', starter: `<?php\n\nclass Home extends BaseController {\n    public function index() {\n        echo "CodeIgniter 4 Controller executing...\\n";\n        echo "Action: Displaying Dashboard";\n    }\n}\n\n$controller = new Home();\n$controller->index();` },
  { name: 'WordPress', value: 'wordpress', starter: `<?php\n\n// Mocking WP Action Hooks\nfunction add_action($hook, $callback) {\n    echo "WordPress Hook Registered: $hook\\n";\n    $callback();\n}\n\nadd_action('init', function() {\n    echo "WP_Plugin_Initialized: Custom Portfolio Extension active.";\n});` },
  { name: 'Python', value: 'python', starter: 'print("Hello from Python WASM!")\nsum_val = 1 + 2\nprint(f"1 + 2 = {sum_val}")' },
  { name: 'PHP', value: 'php', starter: '<?php\necho "Hello from PHP!\\n";' }
]

const selectedLang = ref(languages[0])
const code = ref(selectedLang.value.starter)
const output = ref('')
const isRunning = ref(false)
const isFullScreen = ref(false)
const iframeRef = ref<HTMLIFrameElement | null>(null)

watch(selectedLang, async (newVal) => {
  code.value = newVal.starter
  output.value = ''
  if (['html', 'vue', 'react', 'reactnative'].includes(newVal.value)) {
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

const runPHP = async (source: string) => {
  try {
    const response = await fetch('https://ce.judge0.com/submissions?wait=true', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ source_code: source, language_id: 68, stdin: "" })
    })
    if (response.ok) {
      const data = await response.json()
      return data.stdout || data.stderr || data.compile_output || '> Execution completed.'
    }
  } catch (e) { console.warn('Judge0 failed, trying Piston mirror...') }

  try {
    const response = await fetch('https://piston.deno.dev/api/v2/execute', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ language: 'php', version: '8.2.3', files: [{ name: 'main.php', content: source }] })
    })
    const data = await response.json()
    if (data.run) return data.run.output || data.run.stderr || '> Execution completed.'
  } catch (e) { console.warn('Piston Mirror failed.') }

  return '> System Error: Engines busy.'
}

const runCode = async () => {
  isRunning.value = true
  output.value = '> Initializing environment...\n'

  try {
    if (selectedLang.value.value === 'javascript') {
      const logs: string[] = []
      const originalLog = console.log
      console.log = (...args) => logs.push(args.join(' '))
      try {
        new Function(code.value)()
        output.value = logs.join('\n') || '> Execution completed.'
      } catch (e) { output.value = `Error: ${e}` }
      console.log = originalLog
    }
    else if (['html', 'vue', 'react', 'reactnative'].includes(selectedLang.value.value)) {
      renderPreview()
    }
    else if (selectedLang.value.value === 'python') {
      output.value = '> Loading Python runtime (Pyodide WASM)...\n'
      await loadScript('https://cdn.jsdelivr.net/pyodide/v0.25.0/full/pyodide.js')
      const pyodide = await (window as any).loadPyodide()
      output.value = ''
      pyodide.setStdout({ batched: (text: string) => { output.value += text + '\n' } })
      await pyodide.runPythonAsync(code.value)
    }
    else if (['php', 'laravel', 'codeigniter', 'wordpress'].includes(selectedLang.value.value)) {
      output.value = `> Booting ${selectedLang.value.name} Environment...\n`
      output.value = await runPHP(code.value)
    }
  } catch (error) {
    output.value = `> Execution Error: ${error}`
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
    content = `<script src="https://unpkg.com/vue@3/dist/vue.global.js"><\/script>${code.value}`
  } else if (['react', 'reactnative'].includes(selectedLang.value.value)) {
    content = `<div id="root"></div><script src="https://unpkg.com/react@18/umd/react.development.js"><\/script><script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"><\/script><script src="https://unpkg.com/@babel/standalone/babel.min.js"><\/script><script type="text/babel">${code.value}<\/script>`
  }

  doc.open()
  doc.write(`<!DOCTYPE html><html><head><style>body { background: #0d1117; color: #c9d1d9; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Helvetica, Arial, sans-serif; padding: 20px; line-height: 1.6; }</style></head><body>${content}</body></html>`)
  doc.close()
}

watch(code, () => {
  if (['html', 'vue', 'react', 'reactnative'].includes(selectedLang.value.value)) renderPreview()
})

onMounted(() => {
  if (['html', 'vue', 'react', 'reactnative'].includes(selectedLang.value.value)) setTimeout(renderPreview, 500)
})

const toggleFullScreen = () => {
  isFullScreen.value = !isFullScreen.value
  document.body.style.overflow = isFullScreen.value ? 'hidden' : ''
}
</script>

<template>
  <div
    class="terminal overflow-hidden relative group flex flex-col border border-gray-800 rounded-lg shadow-2xl transition-all duration-300"
    :class="isFullScreen ? 'fixed inset-4 z-[9999] bg-obsidian/95 backdrop-blur-md' : 'h-[550px]'">
    <div
      class="absolute inset-0 pointer-events-none opacity-[0.03] bg-gradient-to-b from-transparent via-accent to-transparent h-[100%] animate-scanline z-20">
    </div>

    <div class="terminal-header flex justify-between items-center px-4 py-2 bg-obsidian border-b border-gray-800">
      <div class="flex gap-2">
        <button @click="toggleFullScreen" class="w-3 h-3 rounded-full bg-[#ff5f56] hover:brightness-110 shadow-sm"
          title="Toggle Fullscreen"></button>
        <div class="w-3 h-3 rounded-full bg-[#ffbd2e] shadow-sm"></div>
        <div class="w-3 h-3 rounded-full bg-[#27c93f] shadow-sm"></div>
      </div>

      <div class="flex items-center gap-3">
        <div class="flex items-center bg-black/40 rounded px-2 py-1 border border-gray-700">
          <span class="text-[0.65rem] text-gray-500 font-mono mr-2">ENV:</span>
          <select v-model="selectedLang"
            class="bg-transparent text-accent font-mono text-[0.7rem] focus:outline-none cursor-pointer">
            <option v-for="lang in languages" :key="lang.value" :value="lang">{{ lang.name.toUpperCase() }}</option>
          </select>
        </div>

        <button v-if="!['html', 'vue', 'react', 'reactnative'].includes(selectedLang.value)" @click="runCode"
          :disabled="isRunning"
          class="bg-accent text-obsidian font-mono font-bold text-[0.7rem] px-4 py-1 rounded transition-all hover:brightness-110 active:scale-95 disabled:opacity-50 flex items-center gap-2 shadow-lg">
          {{ isRunning ? 'BOOTING...' : 'RUN' }}
        </button>
        <div v-else class="text-[0.65rem] text-accent/60 font-mono italic animate-pulse">Live Preview Active</div>
      </div>
    </div>

    <div class="flex-1 flex flex-col md:flex-row min-h-0">
      <div class="flex-1 relative border-b md:border-b-0 md:border-r border-gray-800 bg-[#1e1e1e]/50">
        <div class="absolute top-2 left-4 text-[0.6rem] text-gray-600 font-mono z-10 pointer-events-none uppercase">
          SOURCE_{{ selectedLang.value }}</div>
        <textarea v-model="code" spellcheck="false"
          class="w-full h-full bg-transparent text-gray-300 font-mono text-sm p-8 pt-10 resize-none focus:outline-none selection:bg-accent/30 custom-scrollbar leading-relaxed"></textarea>
      </div>

      <div class="flex-1 bg-black/10 relative flex flex-col">
        <div class="absolute top-2 left-4 text-[0.6rem] text-gray-600 font-mono z-10 pointer-events-none uppercase">
          {{ ['html', 'vue', 'react', 'reactnative'].includes(selectedLang.value) ? 'Live_Render' : 'System_Output' }}
        </div>
        <div v-show="['html', 'vue', 'react', 'reactnative'].includes(selectedLang.value)" class="flex-1 pt-8"><iframe
            ref="iframeRef" class="w-full h-full border-none"></iframe></div>
        <div v-show="!['html', 'vue', 'react', 'reactnative'].includes(selectedLang.value)"
          class="flex-1 p-8 pt-10 font-mono text-sm overflow-y-auto custom-scrollbar">
          <pre class="text-gray-400 whitespace-pre-wrap">{{ output || '> System ready...' }}</pre>
        </div>
      </div>
    </div>

    <div class="px-4 py-2 bg-obsidian border-t border-gray-800 flex justify-between items-center">
      <div class="flex items-center gap-4">
        <span class="text-[0.6rem] text-gray-500 font-mono flex items-center gap-1">
          <span class="w-1.5 h-1.5 rounded-full" :class="isRunning ? 'bg-accent animate-pulse' : 'bg-gray-700'"></span>
          STATUS: {{ isRunning ? 'BUSY' : 'IDLE' }}
        </span>
      </div>
      <div class="text-[0.6rem] text-gray-600 font-mono uppercase tracking-widest hidden md:block">
        {{ isFullScreen ? 'Maximize Mode' : 'Standard View' }} // Joey_Ventulan // Dev_Lab v4.5
      </div>
      <button @click="toggleFullScreen" class="text-[0.6rem] text-accent font-mono hover:underline uppercase">
        {{ isFullScreen ? '[ Exit Full View ]' : '[ Full View ]' }}
      </button>
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
  0% {
    transform: translateY(-100%);
  }

  100% {
    transform: translateY(100%);
  }
}
</style>
