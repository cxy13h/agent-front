<template>
  <div class="app">
    <div class="main-content">
      <div class="panel">
        <div class="panel-header">AI</div>
        <div class="messages">
          <div v-for="(msg, i) in aiMessages" :key="i" class="message ai-msg" v-html="msg"></div>
        </div>
      </div>

      <div class="panel">
        <div class="panel-header">用户</div>
        <div class="messages">
          <div v-for="(msg, i) in userMessages" :key="i" class="message user-msg">
            {{ msg }}
          </div>
        </div>
      </div>
    </div>

    <div class="input-area">
      <input
          v-model="currentInput"
          @keyup.enter="sendMessage"
          placeholder="请输入..."
          :disabled="isLoading"
      />
      <button @click="sendMessage" :disabled="!currentInput.trim() || isLoading">
        {{ isLoading ? '生成中...' : '发送' }}
      </button>
    </div>
  </div>
</template>

<script setup>
import {ref, nextTick, onMounted} from 'vue'
import { marked } from 'marked'

/* ────── 组件状态 ────── */
const currentInput = ref('')
const userMessages = ref([])
const aiMessages   = ref([])
const isLoading    = ref(false)
const sid          = `s_${Date.now()}_${Math.random().toString(36).slice(2,9)}`
const GlobalPrompts = ref('');
/* ────── 正在累积的块 ────── */
let activeKey   = null          // 'state' 或 'content'
let activeIndex = -1            // aiMessages 中的占位下标
let buffer      = ''            // 已收到的 value_chunk

// onMounted(() => {
//   const eventSource = new EventSource(`http://localhost:5000/prompts?session_id=${sid}`);
//
//   // 监听消息并更新 globalVariable
//   eventSource.onmessage = (event) => {
//     GlobalPrompts.value = event.data;
//     // console.log(event.data);
//   };
//
//   // 错误处理
//   eventSource.onerror = (error) => {
//     console.error("EventSource error: ", error);
//   };
// });



/* 转义 \\n → 换行，并尝试剥掉外围引号 */
const decode = (str) => {
  let t = str.replace(/\\n/g, '\n')     // \n → 换行
      .replace(/\\"/g, '"');
  if ((t.startsWith('"') && t.endsWith('"')) ||
      (t.startsWith("'") && t.endsWith("'"))) {
    try {
      t = t.slice(1, -1)
    } catch {}
  }
  return t
}

/* 收到新 key 时，把上一段定稿 */
function finalizePrev () {
  if (activeIndex !== -1) {
    aiMessages.value[activeIndex] = marked.parse(decode(buffer))
    activeIndex = -1
    buffer      = ''
  }
}

/* 处理一条事件对象 */
function handleEvt (evt) {
  const { type } = evt

  /* tool_executed ─ 直接插入一条 */
  if (type === 'tool_executed') {
    const html = marked.parse(`**${evt.tool_name}**： \n ${decode(evt.observation)}`)
    aiMessages.value.push(html)
    return
  }

  /* key_complete ─ 结束旧块，开启新块 */
  if (type === 'key_complete' && (evt.key === 'state' || evt.key === 'content')) {
    finalizePrev()
    activeKey   = evt.key
    activeIndex = aiMessages.value.push('') - 1  // 新占位
    return
  }

  /* value_chunk ─ 追加到当前块并实时刷新 */
  if (type === 'value_chunk' && evt.key === activeKey) {
    buffer += evt.value
    aiMessages.value[activeIndex] = marked.parse(decode(buffer))
  }
}

/* ────── 发送请求并流式读取 ────── */
async function sendMessage () {
  if (!currentInput.value.trim() || isLoading.value) return
  isLoading.value = true

  userMessages.value.push(currentInput.value)
  const prompt = currentInput.value
  currentInput.value = ''

  try {
    const resp = await fetch('http://localhost:5000/api/chat', {
      method : 'POST',
      headers: { 'Content-Type': 'application/json' },
      body   : JSON.stringify({ message: prompt, session_id: sid }),
    })

    const reader  = resp.body.getReader()
    const decoder = new TextDecoder('utf-8')
    let   buf     = ''

    while (true) {
      const { value, done } = await reader.read()
      if (done) break
      buf += decoder.decode(value, { stream: true })

      let p
      while ((p = buf.indexOf('\n\n')) !== -1) {        // 每段以空行分隔
        let seg = buf.slice(0, p).trim()
        buf     = buf.slice(p + 2)
        seg     = seg.replace(/^data:\s*/, '')          // 若后端无 data: 可删
        if (seg) handleEvt(JSON.parse(seg))
      }
      await nextTick()                                  // 让 DOM 立即刷新
    }
  } catch (e) {
    aiMessages.value.push('❌ 网络或解析错误：' + e.message)
  } finally {
    finalizePrev()                                      // 把最后残片输出
    isLoading.value = false
  }
}
</script>




<style scoped>
/* Reset some default styles */
:global(body, html) {
  margin: 0;
  padding: 0;
  height: 100%;
  overflow: hidden;
}

.app {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  flex-direction: column;
  margin: 0;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  background: #f5f5f5;
  overflow: hidden;
}

.main-content {
  flex: 1;
  display: flex;
  min-height: 0;
  overflow: hidden;
}

.panel {
  display: flex;
  flex-direction: column;
  background: white;
  overflow: hidden;
}

.panel:first-child {
  width: 70%; /* AI回复区域占70% */
  border-right: 1px solid #ddd;
}

.panel:last-child {
  width: 30%; /* 用户输入区域占30% */
}

.panel-header {
  padding: 16px 20px;
  background: #f8f9fa;
  border-bottom: 1px solid #ddd;
  font-weight: 600;
  color: #333;
}

.messages {
  flex: 1;
  overflow-y: auto;
  padding: 20px;
}

.message {
  margin-bottom: 12px;
  padding: 10px 14px;
  border-radius: 8px;
  word-break: break-word;
  white-space: pre-wrap;
  line-height: 1.6;
}

.ai-msg {
  background: #f7f7f7;
  border: 1px solid #e5e5e5;
  color: #333;
}

/* Styles for rendered markdown content inside ai-msg */
.ai-msg :deep(p) {
  margin: 0 0 8px 0;
}
.ai-msg :deep(p:last-child) {
  margin-bottom: 0;
}
.ai-msg :deep(strong) {
  color: #6a1b9a;
}
.ai-msg :deep(pre) {
  background-color: #eef1f3;
  padding: 12px;
  border-radius: 4px;
  white-space: pre-wrap;
  word-break: break-all;
}
.ai-msg :deep(code) {
  font-family: 'Courier New', Courier, monospace;
  background-color: #eef1f3;
  padding: 2px 4px;
  border-radius: 3px;
  font-size: 0.9em;
}
.ai-msg :deep(pre code) {
  padding: 0;
  background-color: transparent;
}

.user-msg {
  background: #e3f2fd;
  color: #1565c0;
}

.input-area {
  border-top: 1px solid #ddd;
  background: white;
  padding: 16px 20px;
  display: flex;
  gap: 12px;
  box-shadow: 0 -2px 8px rgba(0, 0, 0, 0.07);
}

.input-area input {
  flex: 1;
  padding: 10px 14px;
  border: 1px solid #ddd;
  border-radius: 6px;
  font-size: 15px;
  outline: none;
}

.input-area input:focus {
  border-color: #1565c0;
  box-shadow: 0 0 0 2px rgba(21, 101, 192, 0.2);
}

.input-area button {
  padding: 10px 24px;
  background: #1565c0;
  color: white;
  border: none;
  border-radius: 6px;
  font-size: 15px;
  cursor: pointer;
  white-space: nowrap;
  transition: background-color 0.2s;
}

.input-area button:hover:not(:disabled) {
  background: #0d47a1;
}

.input-area button:disabled {
  background: #ccc;
  cursor: not-allowed;
}

/* 优化滚动条 */
.messages::-webkit-scrollbar {
  width: 6px;
}

.messages::-webkit-scrollbar-track {
  background: #f1f1f1;
}

.messages::-webkit-scrollbar-thumb {
  background: #bbb;
  border-radius: 3px;
}
</style>