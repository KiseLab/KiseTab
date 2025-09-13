<template>
  <div :class="['min-h-screen flex items-center justify-center', theme === 'dark' ? 'bg-dark text-light' : 'bg-light text-dark']">
    <div class="w-full max-w-xl px-6">
      <div class="text-center mb-8">
        <h1 class="text-5xl font-extrabold">KiseTab</h1>
        <p class="mt-2 text-gray-600 dark:text-gray-300">一个简洁、现代化的新标签页。</p>
      </div>

      <form @submit.prevent="onSearch" class="flex items-center gap-3">
        <select v-model="engine" class="select-engine">
          <option v-for="(e, key) in engines" :key="key" :value="key">{{ e.name }}</option>
        </select>
        <input v-model="query" class="flex-1 input-search" type="search" placeholder="搜索或输入网址" />
        <button class="btn" type="submit">搜索</button>
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

.select-engine {
  border-radius: 8px;
  padding: 0.6rem 0.8rem;
  border: 1px solid rgba(0,0,0,0.08);
  background: white;
}

.btn {
  background: #8b5cf6;
  color: white;
  padding: 0.6rem 1rem;
  border-radius: 8px;
  font-weight: 600;
}
</style>
