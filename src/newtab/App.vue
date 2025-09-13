<template>
  <div :class="['min-h-screen flex items-center justify-center', theme === 'dark' ? 'bg-dark text-light' : 'bg-light text-dark']">
    <div class="w-full max-w-xl px-6">
      <div class="text-center mb-8">
        <h1 class="text-5xl font-extrabold">KiseTab</h1>
      </div>

      <form @submit.prevent="onSearch" class="flex items-center gap-3">
        <div class="engine-toggle" role="tablist" aria-label="搜索引擎选择">
          <button type="button" :class="['engine-btn', engine === 'bing_cn' ? 'active' : '']" @click="engine = 'bing_cn'" title="Microsoft Bing">
            <!-- Bing SVG -->
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 640 640" width="20" height="20" aria-hidden="true"><path d="M96 96L310.6 96L310.6 310.6L96 310.6L96 96zM329.4 96L544 96L544 310.6L329.4 310.6L329.4 96zM96 329.4L310.6 329.4L310.6 544L96 544L96 329.4zM329.4 329.4L544 329.4L544 544L329.4 544L329.4 329.4z" fill="currentColor"/></svg>
          </button>
          <button type="button" :class="['engine-btn', engine === 'google' ? 'active' : '']" @click="engine = 'google'" title="Google">
            <!-- Google SVG (FontAwesome provided) -->
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512" width="20" height="20" aria-hidden="true"><path d="M500 261.8C500 403.3 403.1 504 260 504 122.8 504 12 393.2 12 256S122.8 8 260 8c66.8 0 123 24.5 166.3 64.9l-67.5 64.9c-88.3-85.2-252.5-21.2-252.5 118.2 0 86.5 69.1 156.6 153.7 156.6 98.2 0 135-70.4 140.8-106.9l-140.8 0 0-85.3 236.1 0c2.3 12.7 3.9 24.9 3.9 41.4z" fill="currentColor"/></svg>
          </button>
        </div>

        <input v-model="query" class="flex-1 input-search" type="search" placeholder="搜索或输入网址，按回车搜索" />
      </form>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue'

const defaultEngine = 'bing_cn'

const engines = {
  bing_cn: { name: 'Bing （国内版）', url: 'https://cn.bing.com/search?q=%s' },
  google: { name: 'Google', url: 'https://www.google.com/search?q=%s' },
  duck: { name: 'DuckDuckGo', url: 'https://duckduckgo.com/?q=%s' },
  baidu: { name: 'Baidu', url: 'https://www.baidu.com/s?wd=%s' },
} as const

const query = ref('')
const engine = ref<string>(localStorage.getItem('kise:engine') || defaultEngine)
const theme = ref<string>(localStorage.getItem('kise:theme') || 'light')

watch(engine, (v) => localStorage.setItem('kise:engine', v))
watch(theme, (v) => localStorage.setItem('kise:theme', v))

function onSearch() {
  const q = query.value.trim()
  if (!q) return
  const encoded = encodeURIComponent(q)
  const url = (engines as any)[engine.value].url.replace('%s', encoded)
  window.open(url, '_blank')
}
</script>

<style scoped>
.bg-light { background-color: #f9f9f9; }
.text-dark { color: #222; }
.bg-dark { background-color: #1e1e1e; }
.text-light { color: #eee; }

.input-search {
  background: transparent;
  border: 1px solid rgba(0,0,0,0.08);
  padding: 0.75rem 1rem;
  border-radius: 8px;
  font-size: 1rem;
}

.engine-toggle{display:inline-flex;align-items:center;background:transparent;border-radius:10px;padding:4px;gap:6px}
.engine-btn{display:inline-flex;align-items:center;justify-content:center;width:40px;height:40px;border-radius:8px;border:1px solid rgba(0,0,0,0.06);background:transparent;color:var(--text-dark);cursor:pointer;transition:all .18s ease}
.engine-btn svg{display:block}
.engine-btn:hover{transform:translateY(-3px);box-shadow:0 6px 18px rgba(0,0,0,0.06)}
.engine-btn.active{background:var(--accent);color:#fff;box-shadow:0 8px 24px rgba(139,92,246,0.18);transform:translateY(-2px)}

.input-search{background:transparent;border:1px solid rgba(0,0,0,0.06);padding:0.75rem 1rem;border-radius:10px;font-size:1rem;transition:box-shadow .14s ease,border-color .14s ease}
.input-search:focus{outline:none;box-shadow:0 8px 20px rgba(0,0,0,0.06);border-color:var(--accent)}
</style>
