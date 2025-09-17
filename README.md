# KiseTab

KiseTab 是一个基于 Vue 3 + Vite 的轻量 Chrome 新标签页扩展，集成了 CRXJS 插件与 TypeScript 支持。

本仓库实现了一个替代默认新标签页的界面：包含实时时钟（支持公历与农历）、可切换的搜索引擎（Bing / Google）、智能网址识别与流畅的交互动画。该项目设计为易于定制与扩展。

## 主要功能

- 替代默认新标签页
- 中央搜索框：智能识别网址 / 搜索词，按回车直接跳转或搜索
- 搜索引擎切换：Bing（国内）和 Google
- 实时显示 24 小时制时钟、当日公历和农历
- 精致的交互动画
- 使用 Vue 3 `<script setup>` + TypeScript，组件化良好，易于维护

## 项目结构

- `src/newtab/` 新标签页实现（页面、样式、组件）
	- `components/Clock.vue` 时钟与日期展示组件
	- `components/SearchPanel.vue` 搜索框与引擎切换组件
- `src/popup/` 浏览器工具栏弹出页面（项目简介页）
- `manifest.config.ts` CRXJS / manifest 配置入口
- `vite.config.ts` Vite 配置

## 开发与运行

先安装依赖：

```bash
npm install
```

启动开发热更新（Vite dev server）：

```bash
npm run dev
```

构建生产包（会先运行类型检查）：

```bash
npm run build
```

构建后，CRXJS 会在 `dist/` 目录生成可加载的扩展包。要在 Chrome 中加载 unpacked 扩展：

1. 打开 `chrome://extensions/`。
2. 打开右上角的“开发者模式（Developer mode）”。
3. 点击“加载已解压的扩展程序（Load unpacked）”，选择项目的 `dist/` 目录。

## 定制

- 主题色与全局样式：`src/newtab/style.css`（包含 CSS 变量，如 `--accent` / `--accent-glow`），建议在此管理颜色与视觉变量。
- 搜索引擎配置：在 `src/newtab/App.vue`（或对应组件）中维护 `engines` 配置对象，可以添加更多引擎并持久化到 `localStorage`。
- 农历支持：使用 `solarlunar` 动态导入以减小首屏体积；若缺失会回退到 `Intl.DateTimeFormat('zh-CN-u-ca-chinese')`（浏览器支持前提）。

## 许可证

[MIT](./LICENSE)
