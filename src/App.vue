<script setup lang="ts">
import { ref, onMounted } from 'vue'

const typingCommand = ref('cat welcome.txt')
const currentCommand = ref('')
const showWelcome = ref(false)
const logoText = ref('./joey_ventulan')
const isMobileMenuOpen = ref(false)

onMounted(() => {
  let i = 0
  const interval = setInterval(() => {
    if (i < typingCommand.value.length) {
      currentCommand.value += typingCommand.value.charAt(i)
      i++
    } else {
      clearInterval(interval)
      setTimeout(() => {
        showWelcome.value = true
      }, 500)
    }
  }, 100)
})

const handleLogoHover = (isHover: boolean) => {
  logoText.value = isHover ? './joey_ventulan --debug' : './joey_ventulan'
}
</script>

<template>
  <div class="bg-grid"></div>

  <nav class="fixed top-0 w-full py-4 md:py-6 z-50 bg-obsidian/80 backdrop-blur-md border-b border-gray-800">
    <div class="max-w-[1100px] mx-auto px-4 md:px-8 flex justify-between items-center relative">
      <a href="#" 
         class="font-mono font-bold text-accent no-underline text-sm md:text-base"
         @mouseover="handleLogoHover(true)"
         @mouseleave="handleLogoHover(false)">
        {{ logoText }}
      </a>

      <!-- Mobile Menu Toggle -->
      <button 
        class="md:hidden text-gray-400 hover:text-accent focus:outline-none"
        @click="isMobileMenuOpen = !isMobileMenuOpen"
        aria-label="Toggle mobile menu"
      >
        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
          <path v-if="!isMobileMenuOpen" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16"></path>
          <path v-else stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path>
        </svg>
      </button>

      <!-- Desktop Menu -->
      <div class="hidden md:flex gap-8">
        <a href="#services" class="no-underline text-gray-400 font-mono text-sm hover:text-accent transition-colors">services/</a>
        <a href="#projects" class="no-underline text-gray-400 font-mono text-sm hover:text-accent transition-colors">projects/</a>
        <a href="#skills" class="no-underline text-gray-400 font-mono text-sm hover:text-accent transition-colors">skills/</a>
        <a href="#contact" class="no-underline text-gray-400 font-mono text-sm hover:text-accent transition-colors">contact/</a>
      </div>
    </div>

    <!-- Mobile Menu Dropdown -->
    <div v-show="isMobileMenuOpen" class="md:hidden absolute top-full left-0 w-full bg-obsidian/95 backdrop-blur-md border-b border-gray-800 py-4 px-4 flex flex-col gap-4 shadow-xl">
      <a href="#services" @click="isMobileMenuOpen = false" class="no-underline text-gray-400 font-mono text-sm hover:text-accent transition-colors">services/</a>
      <a href="#projects" @click="isMobileMenuOpen = false" class="no-underline text-gray-400 font-mono text-sm hover:text-accent transition-colors">projects/</a>
      <a href="#skills" @click="isMobileMenuOpen = false" class="no-underline text-gray-400 font-mono text-sm hover:text-accent transition-colors">skills/</a>
      <a href="#contact" @click="isMobileMenuOpen = false" class="no-underline text-gray-400 font-mono text-sm hover:text-accent transition-colors">contact/</a>
    </div>
  </nav>

  <main class="max-w-[1100px] mx-auto px-4 md:px-8 overflow-x-hidden">
    <!-- Hero Section -->
    <section class="min-h-screen flex flex-col justify-center pt-16">
      <div class="terminal mb-8">
        <div class="terminal-header">
          <div class="w-3 h-3 rounded-full bg-[#ff5f56]"></div>
          <div class="w-3 h-3 rounded-full bg-[#ffbd2e]"></div>
          <div class="w-3 h-3 rounded-full bg-[#27c93f]"></div>
          <div class="mx-auto font-mono text-[0.8rem] text-gray-500">zsh — 80×24</div>
        </div>
        <div class="terminal-body overflow-x-auto text-xs md:text-[0.95rem]">
          <div class="whitespace-nowrap"><span class="text-accent mr-2">guest@portfolio:~$</span><span class="text-gray-200">whoami</span></div>
          <div class="text-gray-400 mt-2 mb-6">
            Joey Ventulan<br>
            > Full-stack Developer & PHP Specialist<br>
            > Laravel & CodeIgniter Expert<br>
            > Mobile & WordPress Architect
          </div>
          <div class="whitespace-nowrap mt-4"><span class="text-accent mr-2">guest@portfolio:~$</span><span class="text-gray-200">ls -la stack/</span></div>
          <div class="text-gray-400 mt-2 mb-6">
            total 64<br>
            -rw-r--r--  1 joey  staff  2048 May  7 09:30 <span class="text-accent">backend.php</span><br>
            -rw-r--r--  1 joey  staff  1024 May  7 09:30 <span class="text-accent">frontend.js</span><br>
            -rw-r--r--  1 joey  staff  1024 May  7 09:30 <span class="text-accent">mobile.tsx</span><br>
            -rw-r--r--  1 joey  staff   512 May  7 09:30 <span class="text-accent">wordpress.php</span>
          </div>
          <div class="whitespace-nowrap mt-4">
            <span class="text-accent mr-2">guest@portfolio:~$</span>
            <span class="text-gray-200">{{ currentCommand }}</span>
            <span class="inline-block w-2 h-[15px] md:h-[18px] bg-accent ml-1 align-middle animate-blink"></span>
          </div>
          <div v-if="showWelcome" class="text-gray-400 mt-2">
            Crafting seamless digital experiences from server-side logic in Laravel to responsive mobile apps in React Native. 
            I bridge the gap between complex backends and intuitive frontends.
          </div>
        </div>
      </div>
      
      <h1 class="text-4xl md:text-[3.5rem] font-bold mb-4 leading-tight">
        Building Scalable <br>
        <span class="bg-gradient-to-r from-white to-accent bg-clip-text text-transparent">Full-stack</span> Solutions.
      </h1>
      <p class="text-lg md:text-xl text-gray-400 max-w-[600px] mb-8">
        Expert in Laravel, PHP, and Vue.js, delivering high-performance web and mobile applications with focus on API integrity.
      </p>
    </section>

    <!-- Services -->
    <section id="services" class="mt-12 md:mt-16 pt-16 md:pt-20">
      <h2 class="font-mono text-2xl mb-8"><span class="text-gray-600">#</span> expertise_matrix</h2>
      <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
        <div class="card relative">
          <div class="flex justify-between items-center mb-4">
            <div class="text-2xl text-accent">🐘</div>
            <span class="tag">BACKEND</span>
          </div>
          <h3 class="font-mono text-lg font-bold">PHP Ecosystem</h3>
          <p class="text-gray-400 text-sm mt-2">Specializing in Laravel and CodeIgniter to build secure, scalable server-side architectures and RESTful APIs.</p>
        </div>
        <div class="card relative">
          <div class="flex justify-between items-center mb-4">
            <div class="text-2xl text-accent">🌐</div>
            <span class="tag">FRONTEND</span>
          </div>
          <h3 class="font-mono text-lg font-bold">Modern Web UI</h3>
          <p class="text-gray-400 text-sm mt-2">Crafting responsive interfaces using Vue, Tailwind CSS, jQuery, and standard HTML/CSS best practices.</p>
        </div>
        <div class="card relative">
          <div class="flex justify-between items-center mb-4">
            <div class="text-2xl text-accent">📱</div>
            <span class="tag">MOBILE</span>
          </div>
          <h3 class="font-mono text-lg font-bold">Cross-Platform</h3>
          <p class="text-gray-400 text-sm mt-2">Developing high-quality mobile applications with React Native for both iOS and Android platforms.</p>
        </div>
      </div>
    </section>

    <!-- Projects -->
    <section id="projects" class="mt-20 md:mt-32 pt-16 md:pt-20">
      <h2 class="font-mono text-2xl mb-8"><span class="text-gray-600">#</span> selected_works</h2>
      <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
        <div class="card relative">
          <div class="tag absolute top-4 right-4">LARAVEL</div>
          <h3 class="font-mono text-lg font-bold mb-4">ERP Core System</h3>
          <p class="text-gray-400 text-sm">A robust enterprise resource planning system with complex API integrations and automated workflows.</p>
          <div class="mt-6 flex gap-2">
            <span class="tag">PHP</span>
            <span class="tag">MySQL</span>
          </div>
        </div>
        <div class="card relative">
          <div class="tag absolute top-4 right-4">REACT NATIVE</div>
          <h3 class="font-mono text-lg font-bold mb-4">FitTrack Pro</h3>
          <p class="text-gray-400 text-sm">A mobile fitness application with real-time tracking, social features, and offline synchronization.</p>
          <div class="mt-6 flex gap-2">
            <span class="tag">iOS</span>
            <span class="tag">Android</span>
          </div>
        </div>
        <div class="card relative">
          <div class="tag absolute top-4 right-4">WORDPRESS</div>
          <h3 class="font-mono text-lg font-bold mb-4">Portal Engine</h3>
          <p class="text-gray-400 text-sm">Custom WordPress themes and plugins built for high-traffic news portals and e-commerce stores.</p>
          <div class="mt-6 flex gap-2">
            <span class="tag">jQuery</span>
            <span class="tag">PHP</span>
          </div>
        </div>
      </div>
    </section>

    <!-- Skills -->
    <section id="skills" class="mt-20 md:mt-32 pt-16 md:pt-20">
      <div class="terminal">
        <div class="terminal-header">
          <div class="mx-auto font-mono text-[0.8rem] text-gray-500">./fullstack_dev.json</div>
        </div>
        <div class="terminal-body overflow-x-auto text-xs md:text-[0.95rem]">
<pre class="text-gray-400">
{
  <span class="text-blue-400">"backend"</span>: {
    <span class="text-pink-400">"languages"</span>: ["PHP", "JavaScript"],
    <span class="text-pink-400">"frameworks"</span>: ["Laravel", "CodeIgniter"],
    <span class="text-pink-400">"cms"</span>: "WordPress"
  },
  <span class="text-blue-400">"frontend"</span>: {
    <span class="text-pink-400">"frameworks"</span>: ["Vue", "React Native"],
    <span class="text-pink-400">"styling"</span>: ["Tailwind CSS", "Vanilla CSS"],
    <span class="text-pink-400">"libraries"</span>: ["jQuery", "Axios"]
  },
  <span class="text-blue-400">"integration"</span>: {
    <span class="text-pink-400">"apis"</span>: ["RESTful", "JSON-RPC", "GraphQL"],
    <span class="text-pink-400">"tools"</span>: ["Git", "Composer", "NPM"]
  }
}
</pre>
        </div>
      </div>
    </section>

    <!-- Contact -->
    <footer id="contact" class="mt-20 md:mt-32 mb-16 text-center pt-16 md:pt-20">
      <p class="font-mono text-gray-600 mb-4">// establish session?</p>
      <a href="mailto:hello@joey.dev" class="terminal inline-block px-12 py-4 no-underline text-accent font-mono hover:bg-accent/5 transition-colors">
        ssh connect@joey.dev
      </a>
      <div class="mt-16 text-gray-600 text-[0.8rem] font-mono">
        &copy; 2026 joey_ventulan. PHP, Laravel, CI, WordPress, Vue, React Native, Tailwind.
      </div>
    </footer>
  </main>
</template>
