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
const result = ref(null)
const resultType = ref('')
const lastSourceText = ref('')
const loading = ref(false)

/** ================= 1. 真实接口配置 ================= **/

// 开发阶段：后端跑在本机 5000 端口
// 后面上专用机器，可以改成 http://<your-host>:5000，或者从环境变量里读
const API_BASE = 'http://localhost:5000'

// 映射任务类型 -> 后端路径
const endpointMap = {
  '语法检查': '/api/grammar-check',
  '规范润色': '/api/polish',
  '智能翻译': '/api/translate'
}

// 调用后端接口
const callEdgeApi = async (taskType, text) => {
  const path = endpointMap[taskType]
  if (!path) {
    throw new Error(`未知任务类型: ${taskType}`)
  }
  const url = `${API_BASE}${path}`

  const formData = new FormData()
  // 和 curl 一致：text_content=xxx
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
    // ✅ 这里根据你现在的返回结构优先用 data.new
    // 如果之后 polish/translate 返回字段名不一样，可以在这里统一兼容
    return (
      data.new ||
      data.result ||
      data.text ||
      data.data ||
      JSON.stringify(data, null, 2)
    )
  } else {
    // 纯文本返回
    const textRes = await resp.text()
    return textRes
  }
}

/** ================= 2. mock 兜底（接口挂了还能 demo） ================= **/

const mockEdgeInference = (text, taskType) => {
  return new Promise((resolve) => {
    setTimeout(() => {
      let out = text
      if (taskType === '语法检查') {
        out = text
          .replace(' is ', ' are (修正) ')
          .replace(/data/g, 'data (建议: datasets)')
        out += '\n\n[检测到 2 处潜在语法问题]'
      } else if (taskType === '规范润色') {
        out =
          '**[润色结果]**\nThis study comprehensively investigates the impact of Artificial Intelligence... (Deepseek R1 14b 优化)'
      } else if (taskType === '智能翻译') {
        out =
          '**[译文]**\n本研究探讨了人工智能在现代教育系统中的影响...'
      }
      resolve(out)
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
  try {
    let res
    try {
      // 🚀 优先调用真实后端接口
      res = await callEdgeApi(taskType, effective)
    } catch (err) {
      console.error('调用后端接口失败，使用 mock 结果作为兜底:', err)
      // ❗ 如果你不想兜底，可以直接 throw 或提示错误
      res = await mockEdgeInference(effective, taskType)
    }
    result.value = res
    resultType.value = label
    lastSourceText.value = effective
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
    // fallback
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
            <span>AI 处理后</span>
            <!-- 复制按钮 -->
            <button
              type="button"
              class="copy-icon-btn"
              @click="copyResult"
              title="复制结果"
            >
              📋
            </button>
          </div>
          <div class="panel success">
            {{ result }}
          </div>
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
</style>
