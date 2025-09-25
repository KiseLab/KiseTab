<script setup lang="ts">
import ApiKeyForm from './components/ApiKeyForm.vue'
import GeolocationSettings from './components/GeolocationSettings.vue'
const repo = "https://github.com/KiseLab/KiseTab";
</script>

<template>
  <div class="popup-root">
    <div class="header">
      <div class="title">KiseTab</div>
      <div class="subtitle">简洁可定制的新标签页替代 Chrome 插件</div>
    </div>

    <ApiKeyForm />
    <GeolocationSettings />

    <div class="footer">
      <span><a :href="repo" target="_blank" class="footer-link">GitHub</a> | © KiseLab</span>
    </div>
  </div>
</template>

<style scoped>
.popup-root {
  width: 320px;
  padding: 18px;
  box-sizing: border-box;
  font-family: Inter, system-ui, Arial, sans-serif;
  background: var(--bg-light);
  color: var(--text-dark);
  border-radius: 12px;
}
.header {
  margin-bottom: 12px;
}
.title {
  font-size: 18px;
  font-weight: 700;
  color: var(--text-dark);
  margin-bottom: 6px;
}
.subtitle {
  font-size: 13px;
  color: rgba(34, 34, 34, 0.7);
}
.links {
  display: flex;
  align-items: center;
  gap: 10px;
  margin: 14px 0;
}
.repo {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  color: var(--text-dark);
  text-decoration: none;
  padding: 8px;
  border-radius: 8px;
}
.repo .gh {
  fill: var(--text-dark);
  transition: fill 0.22s ease, transform 0.22s ease;
}
.repo:hover .gh {
  fill: var(--accent);
  transform: translateY(-2px);
}
.repo-label {
  font-size: 13px;
}
.open-btn {
  margin-left: auto;
  padding: 8px 12px;
  border-radius: 8px;
  background: var(--accent);
  color: #fff;
  border: none;
  cursor: pointer;
}
.footer {
  margin-top: 12px;
  font-size: 12px;
  color: rgba(34, 34, 34, 0.55);
}
</style>
