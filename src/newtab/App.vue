<template>
  <div :class="['min-h-screen flex items-center justify-center', theme === 'dark' ? 'bg-dark text-light' : 'bg-light text-dark']">
    <div :class="['w-full max-w-xl px-6', hasFocus ? 'shift-up' : '']">
      <Clock :is-collapsed="hasFocus" :time-str="timeStr" :gregorian="gregorian" :lunar="lunar" />
      <SearchPanel v-model:query="query" v-model:engine="engine" v-model:has-focus="hasFocus" @search="onSearch" />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, watch, onMounted, onUnmounted } from 'vue'
import Clock from './components/Clock.vue'
import SearchPanel from './components/SearchPanel.vue'

const defaultEngine = 'bing_cn'

const engines = {
  bing_cn: { name: 'Bing （国内版）', url: 'https://cn.bing.com/search?q=%s' },
  google: { name: 'Google', url: 'https://www.google.com/search?q=%s' },
} as const

const query = ref('')
const engine = ref<string>(localStorage.getItem('kise:engine') || defaultEngine)
const theme = ref<string>(localStorage.getItem('kise:theme') || 'light')
const hasFocus = ref(false)

// clock state
const timeStr = ref('')
const gregorian = ref('')
const lunar = ref('')
let _timer: any = null
let lunarLib: any = null

async function loadLunarLib() {
  try {
    lunarLib = await import('solarlunar')
  } catch (e) {
    lunarLib = null
  }
}

function formatGregorian(now: Date) {
  const month = now.getMonth() + 1
  const day = now.getDate()
  const weekdays = ['星期日','星期一','星期二','星期三','星期四','星期五','星期六']
  return `${month} 月 ${day} 日 ${weekdays[now.getDay()]}`
}

function updateTime() {
  const now = new Date()
  const hh = String(now.getHours()).padStart(2, '0')
  const mm = String(now.getMinutes()).padStart(2, '0')
  const ss = String(now.getSeconds()).padStart(2, '0')
  timeStr.value = `${hh}:${mm}:${ss}`
  gregorian.value = formatGregorian(now)

  // compute lunar via lunarLib if available
  if (lunarLib && typeof lunarLib.solar2lunar === 'function') {
    try {
      const nowY = now.getFullYear()
      const nowM = now.getMonth() + 1
      const nowD = now.getDate()
      const r = lunarLib.solar2lunar(nowY, nowM, nowD)
      const gz = r.gzYear || r.cny || r.cnYear || r.lunarYear || ''
      const mCn = r.IMonthCn || r.cnMonth || r.monthCn || r.lunarMonth || ''
      const dCn = r.IDayCn || r.cnDay || r.dayCn || r.lunarDay || ''
      lunar.value = `${gz ? gz + '年 ' : ''}${mCn || ''}${dCn || ''}`.trim()
    } catch (e) {
      lunar.value = '农历信息不可用'
    }
  } else {
    // try Intl fallback
    try {
      const opt: any = { year: 'numeric', month: 'long', day: 'numeric', calendar: 'chinese' }
      lunar.value = new Intl.DateTimeFormat('zh-CN-u-ca-chinese', opt).format(now)
    } catch (e) {
      lunar.value = '农历信息不可用'
    }
  }
}

onMounted(() => {
  loadLunarLib()
  updateTime()
  _timer = setInterval(updateTime, 1000)
})

onUnmounted(() => {
  if (_timer) clearInterval(_timer)
})

watch(engine, (v) => localStorage.setItem('kise:engine', v))
watch(theme, (v) => localStorage.setItem('kise:theme', v))

function isLikelyURL(input: string) {
  if (!input) return false
  const hasProtocol = /^https?:\/\//i.test(input)
  if (hasProtocol) return true
  if (/^www\./i.test(input)) return true
  // simple tld check: contains a dot and a known-ish TLD
  if (/\.[a-z]{2,}(\/|$)/i.test(input)) return true
  return false
}

function normalizeURL(input: string) {
  if (/^https?:\/\//i.test(input)) return input
  return 'https://' + input
}

function onSearch() {
  const q = query.value.trim()
  if (!q) return

  if (isLikelyURL(q)) {
    const target = normalizeURL(q)
    // replace current page
    window.location.href = target
    return
  }

  const encoded = encodeURIComponent(q)
  const url = (engines as any)[engine.value].url.replace('%s', encoded)
  // replace current page instead of opening a new tab
  window.location.href = url
}
</script>

<style scoped>
.bg-light { background-color: #f9f9f9; }
.text-dark { color: #222; }
.bg-dark { background-color: #1e1e1e; }
.text-light { color: #eee; }
</style>
