<template>
  <div class="app">
    <!-- 主体内容区 -->
    <div class="main-content">
      <!-- 左侧：AI回复 -->
      <div class="panel">
        <div class="panel-header">AI回复</div>
        <div class="messages">
          <div v-for="(msg, i) in aiMessages" :key="i" class="message ai-msg">
            {{ msg }}
          </div>
        </div>
      </div>

      <!-- 右侧：用户输入 -->
      <div class="panel">
        <div class="panel-header">用户输入</div>
        <div class="messages">
          <div v-for="(msg, i) in userMessages" :key="i" class="message user-msg">
            {{ msg }}
          </div>
        </div>
      </div>
    </div>

    <!-- 底部输入区 -->
    <div class="input-area">
      <input
          v-model="currentInput"
          @keyup.enter="sendMessage"
          placeholder="请输入..."
          :disabled="isLoading"
      />
      <button @click="sendMessage" :disabled="!currentInput.trim() || isLoading">
        发送
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const currentInput = ref('')
const userMessages = ref([])
const aiMessages = ref([])
const isLoading = ref(false)

const sendMessage = async () => {
  if (!currentInput.value.trim() || isLoading.value) return

  const userInput = currentInput.value
  userMessages.value.push(userInput)
  currentInput.value = ''
  isLoading.value = true

  try {
    const response = await fetch('/api/chat', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ message: userInput })
    })

    if (!response.ok) throw new Error('请求失败')

    const reader = response.body.getReader()
    const decoder = new TextDecoder()
    let aiResponse = ''

    aiMessages.value.push('')
    const currentIndex = aiMessages.value.length - 1

    while (true) {
      const { done, value } = await reader.read()
      if (done) break

      const chunk = decoder.decode(value, { stream: true })
      aiResponse += chunk
      aiMessages.value[currentIndex] = aiResponse
    }
  } catch (error) {
    console.error('发送失败:', error)
    aiMessages.value.push('抱歉，发生了错误，请稍后重试。')
  } finally {
    isLoading.value = false
  }
}
</script>

<style scoped>
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
  width: 50%;
  display: flex;
  flex-direction: column;
  background: white;
  overflow: hidden;
}

.panel:first-child {
  border-right: 1px solid #ddd;
}

.panel-header {
  padding: 16px 20px;
  background: #f8f9fa;
  border-bottom: 1px solid #ddd;
  font-weight: 500;
  color: #333;
}

.messages {
  flex: 1;
  overflow-y: auto;
  padding: 20px;
  padding-bottom: 80px; /* 为固定的输入框留出空间 */
}

.message {
  margin-bottom: 12px;
  padding: 10px 14px;
  border-radius: 6px;
  word-break: break-word;
}

.ai-msg {
  background: #f3e5f5;
  color: #6a1b9a;
}

.user-msg {
  background: #e3f2fd;
  color: #1565c0;
}

.input-area {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  border-top: 1px solid #ddd;
  background: white;
  padding: 16px 20px;
  display: flex;
  gap: 12px;
  box-shadow: 0 -2px 8px rgba(0, 0, 0, 0.1);
  z-index: 100;
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