<script setup>
import { ref } from 'vue'
import Sidebar from './components/Sidebar.vue'
import CloudView from './components/CloudView.vue'
import EdgeView from './components/EdgeView.vue'

// 当前视图：'edge' | 'cloud'
const view = ref('edge')

// 边端预填内容（从云端复制过来）
const edgePrefillText = ref(
  'The study investigates the impact of artificial intelligence on modern education systems. We conducted a comprehensive analysis of 500 schools. The results shows that AI-powered tools significantly improve student engagement.'
)

// 顶部按钮：进入云端
const goCloud = () => {
  view.value = 'cloud'
}

// 从云端复制到边端写作区
const handleCopyToEdge = (text) => {
  edgePrefillText.value = text
  view.value = 'edge'
}
</script>

<template>
  <div class="app">
    <!-- 左侧侧边栏 -->
    <Sidebar />

    <!-- 右侧主区域 -->
    <main class="main">
      <header class="main-header">
        <div class="main-title">
          <h2><span>🎓</span>科研论文智能体</h2>
          <!-- <small>Based on Intel OPEA Architecture</small> -->
        </div>
        <button class="btn" type="button" @click="goCloud">
          ☁️ 访问云端RAG系统
        </button>
      </header>

      <section class="views">
        <!-- Cloud 视图 -->
        <section v-show="view === 'cloud'" class="view active">
          <CloudView @copy-to-edge="handleCopyToEdge" @back-edge="view = 'edge'" />
        </section>

        <!-- Edge 视图 -->
        <section v-show="view === 'edge'" class="view active">
          <EdgeView :prefill-text="edgePrefillText" />
        </section>
      </section>
    </main>
  </div>
</template>

<style>
:root {
  --bg: #f3f4f6;
  --sidebar-bg: #111827;
  --card-bg: #ffffff;
  --primary: #2563eb;
  --primary-soft: #dbeafe;
  --border: #e5e7eb;
  --text-main: #111827;
  --text-sub: #6b7280;
  --success: #16a34a;
  --info: #0ea5e9;
}

* {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI",
    sans-serif;
  background-color: var(--bg);
  color: var(--text-main);
}

.app {
  display: flex;
  min-height: 100vh;
}

/* ---- Sidebar ---- */
.sidebar {
  width: 270px;
  background: var(--sidebar-bg);
  color: #f9fafb;
  padding: 20px 18px;
  display: flex;
  flex-direction: column;
  gap: 18px;
}

.sidebar h1 {
  font-size: 20px;
  margin: 0 0 8px;
  display: flex;
  align-items: center;
  gap: 6px;
}

.sidebar h1 span {
  font-size: 22px;
}

.sidebar small {
  color: #9ca3af;
}

.sidebar-section {
  border-radius: 14px;
  padding: 12px 12px 10px;
  background: rgba(15, 23, 42, 0.9);
  border: 1px solid rgba(75, 85, 99, 0.7);
}

.sidebar-section h2 {
  margin: 0 0 4px;
  font-size: 14px;
  font-weight: 600;
}

.sb-badge {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  font-size: 13px;
  padding: 4px 8px;
  border-radius: 999px;
  background: rgba(34, 197, 94, 0.1);
  color: #bbf7d0;
  margin-top: 8px;
}

.sb-badge.info {
  background: rgba(56, 189, 248, 0.12);
  color: #e0f2fe;
}

.metric {
  margin-top: 10px;
  padding: 6px 8px;
  border-radius: 10px;
  background: rgba(15, 23, 42, 0.8);
  border: 1px solid rgba(55, 65, 81, 0.8);
  font-size: 13px;
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.metric-label {
  color: #9ca3af;
  font-size: 11px;
  text-transform: uppercase;
  letter-spacing: 0.04em;
}

.metric-value {
  font-size: 14px;
  font-weight: 600;
}

.sb-progress-wrapper {
  margin-top: 8px;
  font-size: 12px;
}

.sb-progress-label {
  display: flex;
  justify-content: space-between;
  color: #9ca3af;
  margin-bottom: 4px;
}

.sb-progress-bar {
  width: 100%;
  height: 8px;
  border-radius: 999px;
  background: rgba(31, 41, 55, 0.9);
  overflow: hidden;
}

.sb-progress-bar-inner {
  height: 100%;
  width: 40%;
  background: linear-gradient(90deg, #38bdf8, #22c55e);
}

/* ---- Main ---- */
.main {
  flex: 1;
  padding: 20px 24px;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.main-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 12px;
}

.main-title {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.main-title h2 {
  margin: 0;
  font-size: 22px;
  display: flex;
  align-items: center;
  gap: 8px;
}

.main-title h2 span {
  font-size: 24px;
}

.main-title small {
  color: var(--text-sub);
  font-size: 13px;
}

.btn {
  border: none;
  border-radius: 999px;
  padding: 8px 14px;
  background: var(--primary);
  color: #ffffff;
  font-size: 13px;
  cursor: pointer;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  box-shadow: 0 1px 3px rgba(15, 23, 42, 0.18);
  transition: background 0.15s ease, transform 0.06s ease;
  white-space: nowrap;
}

.btn.secondary {
  background: #e5e7eb;
  color: #111827;
  box-shadow: none;
}

.btn:hover {
  background: #1d4ed8;
  transform: translateY(-0.5px);
}

.btn.secondary:hover {
  background: #d1d5db;
}

.views {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.view {
  display: block;
}

.grid-2 {
  display: grid;
  grid-template-columns: 1.1fr 2fr;
  gap: 16px;
  align-items: flex-start;
}

.card {
  background: var(--card-bg);
  border-radius: 16px;
  padding: 14px 16px 16px;
  border: 1px solid var(--border);
  box-shadow: 0 1px 2px rgba(15, 23, 42, 0.03);
}

.card h3 {
  margin: 0 0 8px;
  font-size: 16px;
  display: flex;
  align-items: center;
  gap: 6px;
}

.card p,
.card small {
  margin: 4px 0;
  font-size: 13px;
  color: var(--text-sub);
}

.divider {
  border-top: 1px dashed var(--border);
  margin: 12px 0;
}

.toggle {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 13px;
  margin-bottom: 6px;
  color: var(--text-main);
}

.toggle input {
  accent-color: var(--primary);
}

.chat-box {
  display: flex;
  flex-direction: column;
  height: 520px;
}

.chat-messages {
  flex: 1;
  border-radius: 12px;
  padding: 10px 10px 12px;
  background: #f9fafb;
  border: 1px solid #e5e7eb;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.chat-message {
  max-width: 80%;
  padding: 8px 10px;
  border-radius: 12px;
  font-size: 13px;
  line-height: 1.5;
  white-space: pre-wrap;
}

.chat-message.user {
  align-self: flex-end;
  background: var(--primary);
  color: #ffffff;
  border-bottom-right-radius: 4px;
}

.chat-message.assistant {
  align-self: flex-start;
  background: #e5e7eb;
  border-bottom-left-radius: 4px;
}

.chat-message details {
  margin-top: 4px;
  font-size: 12px;
}

.chat-message details summary {
  cursor: pointer;
  list-style: none;
}

.chat-message details summary::-webkit-details-marker {
  display: none;
}

.chat-message details summary::before {
  content: "› ";
  display: inline-block;
  transform-origin: center;
  transition: transform 0.12s ease;
}

.chat-message details[open] summary::before {
  transform: rotate(90deg);
}

.chat-input-row {
  margin-top: 10px;
  display: flex;
  gap: 8px;
}

.chat-input-row input[type="text"] {
  flex: 1;
  border-radius: 999px;
  border: 1px solid #d1d5db;
  padding: 8px 12px;
  font-size: 13px;
  outline: none;
}

.chat-input-row input[type="text"]:focus {
  border-color: var(--primary);
  box-shadow: 0 0 0 1px rgba(37, 99, 235, 0.2);
}

.loading {
  margin-top: 8px;
  font-size: 12px;
  color: var(--text-sub);
  display: none;
}

.loading.active {
  display: flex;
  align-items: center;
  gap: 6px;
}

.loading-dot {
  width: 6px;
  height: 6px;
  border-radius: 999px;
  background: var(--primary);
  animation: pulse 1s infinite ease-in-out;
}

@keyframes pulse {
  0% {
    transform: scale(1);
    opacity: 0.6;
  }
  50% {
    transform: scale(1.6);
    opacity: 1;
  }
  100% {
    transform: scale(1);
    opacity: 0.6;
  }
}

.file-input {
  width: 100%;
  font-size: 13px;
  margin: 6px 0 4px;
}

.badge-success {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  font-size: 12px;
  padding: 4px 8px;
  border-radius: 999px;
  background: #ecfdf3;
  color: #15803d;
  margin-top: 6px;
}

.caption {
  font-size: 12px;
  color: var(--text-sub);
}

.textarea {
  width: 100%;
  min-height: 380px;
  resize: vertical;
  padding: 10px 12px;
  border-radius: 12px;
  border: 1px solid #d1d5db;
  font-size: 13px;
  line-height: 1.5;
  font-family: inherit;
  outline: none;
}

.textarea:focus {
  border-color: var(--primary);
  box-shadow: 0 0 0 1px rgba(37, 99, 235, 0.18);
}

.toolbox {
  display: flex;
  flex-direction: column;
  gap: 8px;
  font-size: 14px;
}

.toolbox small {
  color: var(--text-sub);
  font-size: 12px;
  margin-bottom: 6px;
}

.tool-button {
  width: 100%;
  justify-content: flex-start;
}

.tool-button span {
  font-size: 16px;
}

.result-section {
  margin-top: 14px;
  padding-top: 12px;
  border-top: 1px dashed var(--border);
}

.result-header {
  font-size: 15px;
  font-weight: 600;
  margin-bottom: 10px;
  display: flex;
  align-items: center;
  gap: 6px;
}

.two-cols {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
}

.pill {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  font-size: 12px;
  padding: 2px 8px;
  border-radius: 999px;
  background: var(--primary-soft);
  color: #1d4ed8;
}

.panel {
  border-radius: 12px;
  border: 1px solid #e5e7eb;
  padding: 8px 10px;
  background: #f9fafb;
  font-size: 13px;
  white-space: pre-wrap;
}

.panel.info {
  background: #eff6ff;
}

.panel.success {
  background: #ecfdf3;
}

@media (max-width: 960px) {
  .sidebar {
    display: none;
  }
  .main {
    padding: 16px;
  }
  .grid-2,
  .two-cols {
    grid-template-columns: 1fr;
  }
}
</style>
