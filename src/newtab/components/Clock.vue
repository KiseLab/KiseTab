<template>
  <div class="text-center mb-12">
    <div :class="['clock', isCollapsed ? 'clock-collapsed' : '']">
      <div class="time" aria-live="polite">{{ timeStr }}</div>
      <div class="dates">
        <span class="gregorian">{{ gregorian }}</span>
        <span class="sep">&nbsp;&nbsp;•&nbsp;&nbsp;</span>
        <span class="lunar">{{ lunar }}</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
defineProps<{
  isCollapsed: boolean,
  timeStr: string,
  gregorian: string,
  lunar: string,
}>()
</script>

<style scoped>
/* clock styles */
.clock{display:block;margin-bottom:2.5rem}
.time{font-size:4.5rem;line-height:1;font-weight:700;color:var(--text-dark)}
.dates{margin-top:0.5rem;font-size:1rem;color:rgba(34,34,34,0.7)}
.dates .sep{color:rgba(34,34,34,0.45)}

/* collapse clock smoothly when input is focused */
.clock{
  /* expanded state (explicit) */
  opacity: 1;
  transform: translateY(0);
  margin-bottom: 2.5rem;
  max-height: 240px; /* ensure room for expanded content */
  overflow: hidden;
  /* same duration and small delay for both appear/disappear to keep symmetry */
  transition: opacity .42s cubic-bezier(.2,.8,.2,1) .08s, transform .42s cubic-bezier(.2,.8,.2,1) .08s, max-height .42s cubic-bezier(.2,.8,.2,1) .08s, margin-bottom .42s cubic-bezier(.2,.8,.2,1) .08s;
}
.clock-collapsed{
  /* collapsed state (same properties inverted) */
  opacity: 0;
  transform: translateY(-18px);
  max-height: 0;
  margin-bottom: 0;
  padding-bottom: 0;
  pointer-events: none;
}
</style>
