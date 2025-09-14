<template>
  <form @submit.prevent="onSearch" class="flex items-center gap-3 search-panel">
    <div class="engine-toggle" role="tablist" aria-label="搜索引擎选择">
      <button type="button" :class="['engine-btn', localEngine === 'bing_cn' ? 'active' : '']" @click="updateEngine('bing_cn')" title="Microsoft Bing">
        <span class="svg-wrap">
          <!-- Bing SVG -->
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 640 640" width="20" height="20" aria-hidden="true"><path d="M96 96L310.6 96L310.6 310.6L96 310.6L96 96zM329.4 96L544 96L544 310.6L329.4 310.6L329.4 96zM96 329.4L310.6 329.4L310.6 544L96 544L96 329.4zM329.4 329.4L544 329.4L544 544L329.4 544L329.4 329.4z" fill="currentColor"/></svg>
        </span>
      </button>
      <button type="button" :class="['engine-btn', localEngine === 'google' ? 'active' : '']" @click="updateEngine('google')" title="Google">
        <span class="svg-wrap">
          <!-- Google SVG -->
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512" width="20" height="20" aria-hidden="true"><path d="M500 261.8C500 403.3 403.1 504 260 504 122.8 504 12 393.2 12 256S122.8 8 260 8c66.8 0 123 24.5 166.3 64.9l-67.5 64.9c-88.3-85.2-252.5-21.2-252.5 118.2 0 86.5 69.1 156.6 153.7 156.6 98.2 0 135-70.4 140.8-106.9l-140.8 0 0-85.3 236.1 0c2.3 12.7 3.9 24.9 3.9 41.4z" fill="currentColor"/></svg>
        </span>
      </button>
    </div>

    <div :class="['input-wrap', { 'has-query': localQuery, 'has-focus': hasFocus } ]">
      <input :value="localQuery" @input="updateQuery" class="input-search" type="search" placeholder="搜索或输入网址，按回车搜索"
        @focus="$emit('update:hasFocus', true)" @blur="$emit('update:hasFocus', false)" />
      <div class="input-icon" aria-hidden="true">
        <transition name="icon-slide" mode="out-in">
          <div :key="isLikelyURL(localQuery) ? 'url' : 'search'" class="icon-inner" v-if="localQuery">
            <template v-if="isLikelyURL(localQuery)">
              <!-- url icon -->
              <svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 -960 960 960" width="24px" fill="currentColor"><path d="M440-280H280q-83 0-141.5-58.5T80-480q0-83 58.5-141.5T280-680h160v80H280q-50 0-85 35t-35 85q0 50 35 85t85 35h160v80ZM320-440v-80h320v80H320Zm200 160v-80h160q50 0 85-35t35-85q0-50-35-85t-85-35H520v-80h160q83 0 141.5 58.5T880-480q0 83-58.5 141.5T680-280H520Z"/></svg>
            </template>
            <template v-else>
              <!-- search icon -->
              <svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 -960 960 960" width="24px" fill="currentColor"><path d="M784-120 532-372q-30 24-69 38t-83 14q-109 0-184.5-75.5T120-580q0-109 75.5-184.5T380-840q109 0 184.5 75.5T640-580q0 44-14 83t-38 69l252 252-56 56ZM380-400q75 0 127.5-52.5T560-580q0-75-52.5-127.5T380-760q-75 0-127.5 52.5T200-580q0 75 52.5 127.5T380-400Z"/></svg>
            </template>
          </div>
        </transition>
      </div>
    </div>
  </form>
</template>

<script setup lang="ts">
import { toRefs } from 'vue'

const props = defineProps<{
  query: string,
  engine: string,
  hasFocus: boolean,
}>()

const emit = defineEmits(['update:query', 'update:engine', 'update:hasFocus', 'search'])

const { query: localQuery, engine: localEngine } = toRefs(props)

const updateQuery = (event: Event) => {
  emit('update:query', (event.target as HTMLInputElement).value)
}

const updateEngine = (newEngine: string) => {
  emit('update:engine', newEngine)
}

const onSearch = () => {
  emit('search')
}

function isLikelyURL(input: string) {
  if (!input) return false
  const hasProtocol = /^https?:\/\//i.test(input)
  if (hasProtocol) return true
  if (/^www\./i.test(input)) return true
  if (/\.[a-z]{2,}(\/|$)/i.test(input)) return true
  return false
}
</script>

<style scoped>
/* basic input appearance */
.input-search {
  background: transparent;
  border: none;
  padding: 0.75rem 1rem;
  border-radius: 10px;
  font-size: 1rem;
  line-height:1;
}

.engine-toggle{display:inline-flex;align-items:center;background:transparent;border-radius:10px;padding:6px;gap:12px}
.engine-btn{display:inline-flex;align-items:center;justify-content:center;width:40px;height:40px;border-radius:8px;border:1px solid var(--accent-border);background:transparent;color:var(--text-dark);cursor:pointer;transition:box-shadow .22s cubic-bezier(.2,.8,.2,1),transform .2s ease}
.engine-btn svg{display:block}
.engine-btn:hover{transform:none;box-shadow:0 10px 30px var(--accent-glow), 0 0 0 4px rgba(139,92,246,0.06)}
.engine-btn.active{background:var(--accent);color:#fff;box-shadow:0 10px 30px rgba(139,92,246,0.18), 0 0 0 4px rgba(139,92,246,0.10);transform:none}

.input-wrap{position:relative;flex:1}
.input-wrap::before{
  content: '';
  position: absolute;
  inset: 0;
  border-radius: 12px;
  pointer-events: none;
  box-shadow: 0 0 0 1px rgba(139,92,246,0.06), 0 0 0 1px var(--accent-border);
  transition: box-shadow .28s cubic-bezier(.2,.8,.2,1), opacity .22s cubic-bezier(.2,.8,.2,1), transform .22s cubic-bezier(.2,.8,.2,1);
  opacity: 1;
  transform: scale(0.995);
}
.input-wrap.has-focus::before{
  opacity: 1;
  transform: scale(1);
  box-shadow: 0 10px 30px var(--accent-glow), 0 0 0 4px rgba(139,92,246,0.08);
}

.input-wrap:hover::before{
  box-shadow: 0 10px 30px var(--accent-glow), 0 0 0 4px rgba(139,92,246,0.10);
}

.input-wrap:hover .input-icon{box-shadow:0 10px 30px rgba(0,0,0,0.08)}
.input-wrap:hover .input-icon svg{filter:drop-shadow(0 4px 12px rgba(0,0,0,0.06))}

.input-wrap .input-search{position:relative;z-index:2;transition:color .22s cubic-bezier(.2,.8,.2,1),transform .22s cubic-bezier(.2,.8,.2,1),letter-spacing .22s cubic-bezier(.2,.8,.2,1);color:var(--text-dark)}
.input-wrap .input-icon{z-index:3}

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
.input-wrap.has-focus .input-search::placeholder,
.input-wrap.has-query .input-search::placeholder{
  opacity: 0;
  transform: translateY(-4px);
}

.input-icon{position:absolute;right:16px;top:50%;transform:translateY(-50%) translateX(8px);opacity:0;pointer-events:none;transition:transform .32s cubic-bezier(.2,.8,.2,1),opacity .32s cubic-bezier(.2,.8,.2,1);color:var(--text-dark)}
.input-icon svg{display:block}

.input-wrap.has-query .input-icon{opacity:1;transform:translateY(-50%) translateX(0)}

.input-search{width:100%;box-sizing:border-box;transition:width .22s cubic-bezier(.2,.8,.2,1),padding-right .22s cubic-bezier(.2,.8,.2,1)}
.input-wrap.has-query .input-search{width:calc(100% - 56px)}

.icon-slide-enter-from{opacity:0;transform:translateX(12px) translateY(0)}
.icon-slide-enter-to{opacity:1;transform:translateX(0) translateY(0)}
.icon-slide-leave-from{opacity:1;transform:translateX(0) translateY(0)}
.icon-slide-leave-to{opacity:0;transform:translateX(12px) translateY(0)}
.icon-slide-enter-active,.icon-slide-leave-active{transition:opacity .32s cubic-bezier(.2,.8,.2,1),transform .32s cubic-bezier(.2,.8,.2,1)}

.icon-inner{display:inline-block}
.icon-slide-enter-active .icon-inner{transform:translateY(-6px)}
.icon-slide-leave-active .icon-inner{transform:translateY(6px)}

.search-panel{transition:transform .42s cubic-bezier(.2,.8,.2,1) .02s, margin-top .42s cubic-bezier(.2,.8,.2,1) .02s}
.shift-up .search-panel{transform:translateY(-28px)}
</style>
