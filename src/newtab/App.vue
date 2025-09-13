<template>
  <div :class="['min-h-screen flex items-center justify-center', theme === 'dark' ? 'bg-dark text-light' : 'bg-light text-dark']">
    <div class="w-full max-w-xl px-6">
      <div class="text-center mb-12">
        <div class="clock">
          <div class="time" aria-live="polite">{{ timeStr }}</div>
          <div class="dates">
            <span class="gregorian">{{ gregorian }}</span>
            <span class="sep">&nbsp;&nbsp;•&nbsp;&nbsp;</span>
            <span class="lunar">{{ lunar }}</span>
          </div>
        </div>
      </div>

      <form @submit.prevent="onSearch" class="flex items-center gap-3">
        <div class="engine-toggle" role="tablist" aria-label="搜索引擎选择">
          <button type="button" :class="['engine-btn', engine === 'bing_cn' ? 'active' : '']" @click="engine = 'bing_cn'" title="Microsoft Bing">
            <span class="svg-wrap">
              <!-- Bing SVG -->
              <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 640 640" width="20" height="20" aria-hidden="true"><path d="M96 96L310.6 96L310.6 310.6L96 310.6L96 96zM329.4 96L544 96L544 310.6L329.4 310.6L329.4 96zM96 329.4L310.6 329.4L310.6 544L96 544L96 329.4zM329.4 329.4L544 329.4L544 544L329.4 544L329.4 329.4z" fill="currentColor"/></svg>
            </span>
          </button>
          <button type="button" :class="['engine-btn', engine === 'google' ? 'active' : '']" @click="engine = 'google'" title="Google">
            <span class="svg-wrap">
              <!-- Google SVG -->
              <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512" width="20" height="20" aria-hidden="true"><path d="M500 261.8C500 403.3 403.1 504 260 504 122.8 504 12 393.2 12 256S122.8 8 260 8c66.8 0 123 24.5 166.3 64.9l-67.5 64.9c-88.3-85.2-252.5-21.2-252.5 118.2 0 86.5 69.1 156.6 153.7 156.6 98.2 0 135-70.4 140.8-106.9l-140.8 0 0-85.3 236.1 0c2.3 12.7 3.9 24.9 3.9 41.4z" fill="currentColor"/></svg>
            </span>
          </button>
        </div>

        <div :class="['input-wrap', { 'has-query': query, 'has-focus': hasFocus } ]">
          <input v-model="query" class="input-search" type="search" placeholder="搜索或输入网址，按回车搜索"
            @focus="hasFocus = true" @blur="hasFocus = false" />
          <div class="input-icon" aria-hidden="true">
            <transition name="icon-slide" mode="out-in">
              <div :key="isLikelyURL(query) ? 'url' : 'search'" class="icon-inner" v-if="query">
                <template v-if="isLikelyURL(query)">
                  <!-- url icon (表示将作为网址跳转) -->
                  <svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 -960 960 960" width="24px" fill="currentColor"><path d="M440-280H280q-83 0-141.5-58.5T80-480q0-83 58.5-141.5T280-680h160v80H280q-50 0-85 35t-35 85q0 50 35 85t85 35h160v80ZM320-440v-80h320v80H320Zm200 160v-80h160q50 0 85-35t35-85q0-50-35-85t-85-35H520v-80h160q83 0 141.5 58.5T880-480q0 83-58.5 141.5T680-280H520Z"/></svg>
                </template>
                <template v-else>
                  <!-- search icon (表示将作为搜索) -->
                  <svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 -960 960 960" width="24px" fill="currentColor"><path d="M784-120 532-372q-30 24-69 38t-83 14q-109 0-184.5-75.5T120-580q0-109 75.5-184.5T380-840q109 0 184.5 75.5T640-580q0 44-14 83t-38 69l252 252-56 56ZM380-400q75 0 127.5-52.5T560-580q0-75-52.5-127.5T380-760q-75 0-127.5 52.5T200-580q0 75 52.5 127.5T380-400Z"/></svg>
                </template>
              </div>
            </transition>
          </div>
        </div>
      </form>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue'
import { onMounted, onUnmounted } from 'vue'

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

/* basic input appearance */
.input-search {
  background: transparent;
  border: none; /* remove native border; we draw animated border in ::before */
  padding: 0.75rem 1rem;
  border-radius: 10px;
  font-size: 1rem;
  line-height:1;
}

.engine-toggle{display:inline-flex;align-items:center;background:transparent;border-radius:10px;padding:6px;gap:12px}
.engine-btn{display:inline-flex;align-items:center;justify-content:center;width:40px;height:40px;border-radius:8px;border:1px solid var(--accent-border);background:transparent;color:var(--text-dark);cursor:pointer;transition:box-shadow .22s cubic-bezier(.2,.8,.2,1),transform .2s ease}
.engine-btn svg{display:block}
/* hover: use glow border like input, no vertical movement */
.engine-btn:hover{transform:none;box-shadow:0 10px 30px var(--accent-glow), 0 0 0 4px rgba(139,92,246,0.06)}
.engine-btn.active{background:var(--accent);color:#fff;box-shadow:0 10px 30px rgba(139,92,246,0.18), 0 0 0 4px rgba(139,92,246,0.10);transform:none}

/* input-wrap: we draw an animated border/highlight with ::before so icon is not affected */
.input-wrap{position:relative;flex:1}
.input-wrap::before{
  content: '';
  position: absolute;
  inset: 0;
  border-radius: 12px;
  pointer-events: none;
  /* faint purple outline by default (use variables) */
  box-shadow: 0 0 0 1px rgba(139,92,246,0.06), 0 0 0 1px var(--accent-border);
  transition: box-shadow .28s cubic-bezier(.2,.8,.2,1), opacity .22s cubic-bezier(.2,.8,.2,1), transform .22s cubic-bezier(.2,.8,.2,1);
  opacity: 1;
  transform: scale(0.995);
}
.input-wrap.has-focus::before{
  opacity: 1;
  transform: scale(1);
  /* enhanced highlight on focus using accent variables */
  box-shadow: 0 10px 30px var(--accent-glow), 0 0 0 4px rgba(139,92,246,0.08);
}

/* hover effect: soft shadow increase for the whole search control (no movement) */
.input-wrap:hover::before{
  box-shadow: 0 10px 30px var(--accent-glow), 0 0 0 4px rgba(139,92,246,0.10);
}

.input-wrap:hover .input-icon{box-shadow:0 10px 30px rgba(0,0,0,0.08)}
.input-wrap:hover .input-icon svg{filter:drop-shadow(0 4px 12px rgba(0,0,0,0.06))}

/* Ensure the input and icon stack above the animated border */
.input-wrap .input-search{position:relative;z-index:2;transition:color .22s cubic-bezier(.2,.8,.2,1),transform .22s cubic-bezier(.2,.8,.2,1),letter-spacing .22s cubic-bezier(.2,.8,.2,1);color:var(--text-dark)}
.input-wrap .input-icon{z-index:3}

/* animate input text appearance: color and slight upward nudge when active */
.input-search{color:rgba(34,34,34,0);} 
.input-wrap.has-focus .input-search,
.input-wrap.has-query .input-search{
  color:var(--text-dark);
  transform:translateY(-2px);
  letter-spacing:0.2px;
}
.input-search::placeholder{
  color: rgba(34,34,34,0.45);
  transition: opacity .32s cubic-bezier(.2,.8,.2,1), color .32s cubic-bezier(.2,.8,.2,1), transform .32s cubic-bezier(.2,.8,.2,1);
  opacity: 1;
}
/* hide placeholder smoothly when focusing or when there's text */
.input-wrap.has-focus .input-search::placeholder,
.input-wrap.has-query .input-search::placeholder{
  opacity: 0;
  transform: translateY(-4px);
}

/* icon area and animation (icon unaffected by border) */
.input-icon{position:absolute;right:16px;top:50%;transform:translateY(-50%) translateX(8px);opacity:0;pointer-events:none;transition:transform .32s cubic-bezier(.2,.8,.2,1),opacity .32s cubic-bezier(.2,.8,.2,1);color:var(--text-dark)}
.input-icon svg{display:block}

/* show icon when query exists */
.input-wrap.has-query .input-icon{opacity:1;transform:translateY(-50%) translateX(0)}

/* input width transition for smooth shrink when icon appears */
.input-search{width:100%;box-sizing:border-box;transition:width .22s cubic-bezier(.2,.8,.2,1),padding-right .22s cubic-bezier(.2,.8,.2,1)}
.input-wrap.has-query .input-search{width:calc(100% - 56px)}

/* icon-slide: consistent right->center enter/leave */
.icon-slide-enter-from{opacity:0;transform:translateX(12px) translateY(0)}
.icon-slide-enter-to{opacity:1;transform:translateX(0) translateY(0)}
.icon-slide-leave-from{opacity:1;transform:translateX(0) translateY(0)}
.icon-slide-leave-to{opacity:0;transform:translateX(12px) translateY(0)}
.icon-slide-enter-active,.icon-slide-leave-active{transition:opacity .32s cubic-bezier(.2,.8,.2,1),transform .32s cubic-bezier(.2,.8,.2,1)}

/* vertical nudge during switch */
.icon-inner{display:inline-block}
.icon-slide-enter-active .icon-inner{transform:translateY(-6px)}
.icon-slide-leave-active .icon-inner{transform:translateY(6px)}

/* clock styles */
.clock{display:block;margin-bottom:2.5rem}
.time{font-size:4.5rem;line-height:1;font-weight:700;color:var(--text-dark)}
.dates{margin-top:0.5rem;font-size:1rem;color:rgba(34,34,34,0.7)}
.dates .sep{color:rgba(34,34,34,0.45)}
</style>
