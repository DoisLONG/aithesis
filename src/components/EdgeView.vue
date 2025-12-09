<script setup>
import { ref, watch } from 'vue'

const props = defineProps({
  prefillText: {
    type: String,
    default: ''
  }
})

const edgeText = ref(props.prefillText || '')
const sourceText = ref(null)
const fileStatus = ref('')
const result = ref(null)        // 纯文本结果，用来复制
const resultHtml = ref('')      // 带高亮的 HTML
const resultType = ref('')
const lastSourceText = ref('')
const loading = ref(false)
const hasDiff = ref(false)      // 是否存在原文与新文本的差异（仅语法/润色）

/** ================= 工具函数：HTML 转义 & 简单 diff ================= **/

const escapeHtml = (str = '') => {
  return str
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#39;')
}

const toHtml = (str = '') =>
  escapeHtml(str).replace(/\n/g, '<br/>')

/**
 * 简单 diff 高亮：找到首尾相同部分，只对中间「不同段」做高亮
 * original: 后端返回的 raw
 * modified: 后端返回的 new（处理后文本）
 */
const buildDiffHtml = (original = '', modified = '') => {
  if (!original || !modified || original === modified) {
    return toHtml(modified)
  }

  const lenO = original.length
  const lenM = modified.length
  let start = 0
  while (
    start < lenO &&
    start < lenM &&
    original[start] === modified[start]
  ) {
    start++
  }

  let endO = lenO - 1
  let endM = lenM - 1
  while (
    endO >= start &&
    endM >= start &&
    original[endO] === modified[endM]
  ) {
    endO--
    endM--
  }

  const prefix = modified.slice(0, start)
  const changed = modified.slice(start, endM + 1)
  const suffix = modified.slice(endM + 1)

  return (
    toHtml(prefix) +
    '<span class="diff-changed">' +
    escapeHtml(changed) +
    '</span>' +
    toHtml(suffix)
  )
}

/** ================= 1. 真实接口配置 ================= **/

const API_BASE = 'http://localhost:5000'

const endpointMap = {
  '语法检查': '/api/grammar-check',
  '规范润色': '/api/polish',
  '智能翻译': '/api/translate'
}

// 调后端：返回统一结构 { text, raw, newText }
const callEdgeApi = async (taskType, text) => {
  const path = endpointMap[taskType]
  if (!path) throw new Error(`未知任务类型: ${taskType}`)
  const url = `${API_BASE}${path}`

  const formData = new FormData()
  formData.append('text_content', text)

  const resp = await fetch(url, {
    method: 'POST',
    body: formData
  })

  if (!resp.ok) {
    throw new Error(`HTTP ${resp.status}`)
  }

  const contentType = resp.headers.get('content-type') || ''
  if (contentType.includes('application/json')) {
    const data = await resp.json()
    const rawFromApi = data.raw || text
    const newFromApi =
      data.new ||
      data.result ||
      data.text ||
      data.data ||
      null
    const outText = newFromApi || rawFromApi
    return {
      text: outText,
      raw: rawFromApi,
      newText: newFromApi
    }
  } else {
    const textRes = await resp.text()
    return {
      text: textRes,
      raw: text,
      newText: null
    }
  }
}

/** ================= 2. mock 兜底（接口挂了还能 demo） ================= **/

const mockEdgeInference = (text, taskType) => {
  return new Promise((resolve) => {
    setTimeout(() => {
      let raw = text
      let newText = null

      if (taskType === '语法检查') {
        newText = text
          .replace(' is ', ' are (修正) ')
          .replace(/data/g, 'data (建议: datasets)')
        newText += '\n\n[检测到 2 处潜在语法问题]'
      } else if (taskType === '规范润色') {
        newText =
          'This study comprehensively investigates the impact of Artificial Intelligence... (Deepseek R1 14b 优化)'
      } else if (taskType === '智能翻译') {
        newText = '本研究探讨了人工智能在现代教育系统中的影响...'
      }

      const outText = newText || raw
      resolve({
        text: outText,
        raw,
        newText
      })
    }, 800)
  })
}

/** ================= 3. 监听 prefillText 变化 ================= **/

watch(
  () => props.prefillText,
  (val) => {
    if (val) {
      edgeText.value = val
      sourceText.value = null
    }
  }
)

/** ================= 4. 文件上传 ================= **/

const onFileChange = (e) => {
  const file = e.target.files[0]
  if (!file) {
    sourceText.value = null
    fileStatus.value = ''
    return
  }
  if (file.type === 'text/plain') {
    const reader = new FileReader()
    reader.onload = (evt) => {
      sourceText.value = evt.target?.result || ''
      fileStatus.value = `✅ 已加载 TXT: ${file.name}`
      edgeText.value = sourceText.value
    }
    reader.readAsText(file, 'utf-8')
  } else if (file.type === 'application/pdf') {
    fileStatus.value =
      `✅ 已上传 PDF: ${file.name}\n前端 demo 暂未解析 PDF 文本，可在后续对接后端解析。`
  } else {
    fileStatus.value = '不支持的文件类型，请上传 TXT 或 PDF。'
  }
}

/** ================= 5. 点击工具按钮 ================= **/

const runTool = async (taskType, label) => {
  const effective = sourceText.value || edgeText.value
  if (!effective || !effective.trim()) {
    alert('请先在编辑器中输入内容或上传文件。')
    return
  }
  loading.value = true
  hasDiff.value = false
  try {
    let res
    try {
      res = await callEdgeApi(taskType, effective)
    } catch (err) {
      console.error('调用后端接口失败，使用 mock 结果作为兜底:', err)
      res = await mockEdgeInference(effective, taskType)
    }

    result.value = res.text
    resultType.value = label
    // 原文显示用 raw（如果后端有做规范化），否则用当前输入
    lastSourceText.value = res.raw || effective

    // 构建带高亮的 HTML
    if (
      taskType !== '智能翻译' &&
      res.raw &&
      res.newText &&
      res.raw !== res.newText
    ) {
      // 语法/润色 & 有差异 → 用 diff 高亮
      resultHtml.value = buildDiffHtml(res.raw, res.newText)
      hasDiff.value = true
    } else {
      // 翻译 or 无差异 → 正常展示
      resultHtml.value = toHtml(res.text)
      hasDiff.value = false
    }
  } finally {
    loading.value = false
  }
}

/** ================= 6. 复制结果 ================= **/

const copyResult = async () => {
  if (!result.value) return
  try {
    await navigator.clipboard.writeText(result.value)
    alert('已复制到剪贴板')
  } catch (err) {
    console.error(err)
    const textarea = document.createElement('textarea')
    textarea.value = result.value
    textarea.style.position = 'fixed'
    textarea.style.opacity = '0'
    document.body.appendChild(textarea)
    textarea.focus()
    textarea.select()
    try {
      document.execCommand('copy')
      alert('已复制到剪贴板')
    } catch (e) {
      alert('复制失败，请手动选择文本复制。')
    }
    document.body.removeChild(textarea)
  }
}
</script>

<template>
  <div class="card">
    <h3>🔒 本地隐私写作空间</h3>
    <p class="caption">
      ⚠️ 此处数据仅在本地 Intel Core Ultra NPU 上处理，断网可用。 [cite: 30]
    </p>

    <div class="grid-2" style="margin-top:10px;">
      <!-- 左：编辑器 + 本地文件 -->
      <div>
        <label style="font-size:13px;">上传本地文件 (TXT/PDF)</label>
        <input
          class="file-input"
          type="file"
          accept=".txt,application/pdf"
          @change="onFileChange"
        />
        <div class="caption" style="white-space: pre-wrap;" v-if="fileStatus">
          {{ fileStatus }}
        </div>

        <textarea
          v-model="edgeText"
          class="textarea"
          placeholder="请将云端生成的内容粘贴至此，或直接开始写作..."
        ></textarea>
      </div>

      <!-- 右：智能工具箱 -->
      <div class="toolbox">
        <div>
          <div style="font-weight:600; margin-bottom:4px;">🛠️ 智能工具箱</div>
          <small>Powered by OpenVINO</small>
        </div>

        <button
          type="button"
          class="btn tool-button"
          @click="runTool('语法检查', '语法检查')"
        >
          <span>🔍</span> 语法检查
        </button>
        <button
          type="button"
          class="btn tool-button"
          @click="runTool('规范润色', '润色建议')"
        >
          <span>✨</span> 规范润色
        </button>
        <button
          type="button"
          class="btn tool-button"
          @click="runTool('智能翻译', '翻译结果')"
        >
          <span>🌐</span> 智能翻译
        </button>

        <div class="loading" :class="{ active: loading }">
          <div class="loading-dot"></div>
          <span>本地模型正在处理...</span>
        </div>
      </div>
    </div>

    <!-- 处理结果区域 -->
    <div v-if="result" class="result-section">
      <div class="result-header">
        <span>🎯 处理结果：</span>
        <span class="pill">{{ resultType }}</span>
      </div>
      <div class="two-cols">
        <div>
          <div style="font-size:13px;font-weight:600;margin-bottom:4px;">
            原文
          </div>
          <div class="panel info">
            {{ lastSourceText }}
          </div>
        </div>
        <div>
          <div
            style="font-size:13px;font-weight:600;margin-bottom:4px; display:flex; align-items:center; justify-content:space-between;"
          >
            <div>
              <span>AI 处理后</span>
              <span v-if="hasDiff" class="diff-hint">
                · 已高亮与原文不一致部分
              </span>
            </div>
            <button
              type="button"
              class="copy-icon-btn"
              @click="copyResult"
              title="复制结果"
            >
              📋
            </button>
          </div>
          <div class="panel success" v-html="resultHtml"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.copy-icon-btn {
  border: none;
  background: transparent;
  cursor: pointer;
  font-size: 16px;
  line-height: 1;
  padding: 2px 4px;
  border-radius: 6px;
  transition: background 0.15s ease;
}
.copy-icon-btn:hover {
  background: rgba(15, 23, 42, 0.06);
}

:deep(.diff-changed) {
  font-weight: 600;     /* 字体加粗 */
  font-size: 1.05em;    /* 比周围文字大一点点 */
}
.diff-hint {
  font-size: 11px;
  color: #9ca3af;
  margin-left: 4px;
}
</style>
