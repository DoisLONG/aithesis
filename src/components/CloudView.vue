<script setup>
import { ref } from 'vue'

const emit = defineEmits(['copy-to-edge', 'back-edge'])

const messages = ref([
  {
    role: 'assistant',
    content: '我是云端科研助手，请问有什么可以帮您？',
    sources: null
  }
])

const useKb = ref(true)
const useArxiv = ref(false)
const loading = ref(false)
const inputText = ref('')
const uploadedFileName = ref('')
const hasUploaded = ref(false)

const mockCloudRag = (query) => {
  const sources = [
    'arXiv:2310.xxxxx - Analysis of AI in Education',
    'IEEE Xplore - Edge Computing for Privacy Preserving',
    'Internal Knowledge Base - 2024 Report.pdf'
  ]
  return new Promise((resolve) => {
    setTimeout(() => {
      const text = `**基于 Intel OPEA 云端知识库的分析：**

针对您的问题“${query}”，现有研究表明：
1. **隐私挑战**：在教育场景下，数据隐私是首要考量，边缘计算（Edge AI）是解决之道。
2. **算力分配**：云端适合处理大规模知识图谱构建，而端侧适合实时推理。

建议架构采用云边协同模式，利用 Intel Core Ultra 的 NPU 进行本地负载分担。`
      resolve({ responseText: text, sources })
    }, 1500)
  })
}

const onSend = async () => {
  const text = inputText.value.trim()
  if (!text) return
  messages.value.push({ role: 'user', content: text, sources: null })
  inputText.value = ''
  loading.value = true
  const { responseText, sources } = await mockCloudRag(text)
  loading.value = false
  messages.value.push({ role: 'assistant', content: responseText, sources })
}

const onFileChange = (e) => {
  const file = e.target.files[0]
  if (!file) {
    uploadedFileName.value = ''
    hasUploaded.value = false
    return
  }
  uploadedFileName.value = file.name
  hasUploaded.value = true
}

const copyToEdge = (msg) => {
  emit('copy-to-edge', msg.content)
}

const backEdge = () => {
  emit('back-edge')
}
</script>

<template>
  <div class="grid-2">
    <!-- 左：资料接入 -->
    <div class="card">
      <h3>📂 资料接入</h3>
      <label style="font-size:13px;">上传参考论文 (PDF)</label>
      <input
        class="file-input"
        type="file"
        accept="application/pdf"
        @change="onFileChange"
      />
      <div class="caption" v-if="hasUploaded">
        <span class="badge-success">
          ✅ 已解析: {{ uploadedFileName }}
        </span>
        <br />
        <span class="caption">向量化完成 (ChromaDB)</span>
      </div>

      <div class="divider"></div>

      <label class="toggle">
        <input type="checkbox" v-model="useKb" />
        启用通用知识库搜索
      </label>
      <label class="toggle">
        <input type="checkbox" v-model="useArxiv" />
        启用 arXiv 实时检索
      </label>

      <div style="margin-top:16px;">
        <button class="btn secondary" type="button" @click="backEdge">
          ↩️ 返回边端页面
        </button>
      </div>
    </div>

    <!-- 右：云端 Chat -->
    <div class="card">
      <h3>🤖 智能问答与生成</h3>
      <div class="chat-box">
        <div class="chat-messages">
          <div
            v-for="(msg, idx) in messages"
            :key="idx"
            class="chat-message"
            :class="msg.role"
          >
            <div v-html="msg.content.replace(/\n/g, '<br/>')"></div>

            <details v-if="msg.role === 'assistant' && msg.sources && msg.sources.length">
              <summary>📚 引用来源 (智能聚合)</summary>
              <ul style="padding-left:18px; margin:4px 0 0;">
                <li v-for="(s, i) in msg.sources" :key="i">{{ s }}</li>
              </ul>
            </details>

            <button
              v-if="msg.role === 'assistant'"
              type="button"
              class="btn secondary"
              style="margin-top:6px;"
              @click="copyToEdge(msg)"
            >
              📋 复制内容到写作区
            </button>
          </div>
        </div>

        <form class="chat-input-row" @submit.prevent="onSend">
          <input
            type="text"
            v-model="inputText"
            placeholder="输入研究问题，例如：边缘计算在论文写作中的应用"
          />
          <button class="btn" type="submit">发送</button>
        </form>

        <div class="loading" :class="{ active: loading }">
          <div class="loading-dot"></div>
          <span>☁️ 云端正在检索文献并生成回答...</span>
        </div>
      </div>
    </div>
  </div>
</template>
