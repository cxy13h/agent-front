<!--<template>-->
<!--  <div class="app">-->
<!--    &lt;!&ndash; 主体内容区 &ndash;&gt;-->
<!--    <div class="main-content">-->
<!--      &lt;!&ndash; 左侧：AI回复 &ndash;&gt;-->
<!--      <div class="panel">-->
<!--        <div class="panel-header">AI回复</div>-->
<!--        <div class="messages">-->
<!--          <div v-for="(msg, i) in aiMessages" :key="i" class="message ai-msg">-->
<!--            {{ msg }}-->
<!--          </div>-->
<!--        </div>-->
<!--      </div>-->

<!--      &lt;!&ndash; 右侧：用户输入 &ndash;&gt;-->
<!--      <div class="panel">-->
<!--        <div class="panel-header">用户输入</div>-->
<!--        <div class="messages">-->
<!--          <div v-for="(msg, i) in userMessages" :key="i" class="message user-msg">-->
<!--            {{ msg }}-->
<!--          </div>-->
<!--        </div>-->
<!--      </div>-->
<!--    </div>-->

<!--    &lt;!&ndash; 底部输入区 &ndash;&gt;-->
<!--    <div class="input-area">-->
<!--      <input-->
<!--          v-model="currentInput"-->
<!--          @keyup.enter="sendMessage"-->
<!--          placeholder="请输入..."-->
<!--          :disabled="isLoading"-->
<!--      />-->
<!--      <button @click="sendMessage" :disabled="!currentInput.trim() || isLoading">-->
<!--        {{ isLoading ? '发送中...' : '发送' }}-->
<!--      </button>-->
<!--    </div>-->
<!--  </div>-->
<!--</template>-->

<!--<script setup>-->
<!--import { ref, onMounted } from 'vue'-->

<!--const currentInput = ref('')-->
<!--const userMessages = ref([])-->
<!--const aiMessages = ref([])-->
<!--const isLoading = ref(false)-->
<!--const sessionId = ref(`session_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`)-->

<!--const sendMessage = async () => {-->
<!--  if (!currentInput.value.trim() || isLoading.value) return-->

<!--  const userInput = currentInput.value-->
<!--  userMessages.value.push(userInput)-->
<!--  currentInput.value = ''-->
<!--  isLoading.value = true-->

<!--  try {-->
<!--    const response = await fetch('http://localhost:5000/api/chat', {-->
<!--      method: 'POST',-->
<!--      headers: { 'Content-Type': 'application/json' },-->
<!--      body: JSON.stringify({-->
<!--        message: userInput,-->
<!--        session_id: sessionId.value-->
<!--      })-->
<!--    })-->

<!--    if (!response.ok) throw new Error('请求失败')-->

<!--    const reader = response.body.getReader()-->
<!--    const decoder = new TextDecoder()-->
<!--    let aiResponse = ''-->

<!--    aiMessages.value.push('')-->
<!--    const currentIndex = aiMessages.value.length - 1-->

<!--    while (true) {-->
<!--      const { done, value } = await reader.read()-->
<!--      if (done) break-->

<!--      const chunk = decoder.decode(value, { stream: true })-->
<!--      aiResponse += chunk-->
<!--      aiMessages.value[currentIndex] = aiResponse-->
<!--    }-->
<!--  } catch (error) {-->
<!--    console.error('发送失败:', error)-->
<!--    aiMessages.value.push('抱歉，发生了错误，请稍后重试。')-->
<!--  } finally {-->
<!--    isLoading.value = false-->
<!--  }-->
<!--}-->

<!--// 初始化时显示会话ID（可选）-->
<!--onMounted(() => {-->
<!--  console.log('会话ID:', sessionId.value)-->
<!--})-->
<!--</script>-->

<!--<style scoped>-->
<!--.app {-->
<!--  position: fixed;-->
<!--  top: 0;-->
<!--  left: 0;-->
<!--  right: 0;-->
<!--  bottom: 0;-->
<!--  display: flex;-->
<!--  flex-direction: column;-->
<!--  margin: 0;-->
<!--  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;-->
<!--  background: #f5f5f5;-->
<!--  overflow: hidden;-->
<!--}-->

<!--.main-content {-->
<!--  flex: 1;-->
<!--  display: flex;-->
<!--  min-height: 0;-->
<!--  overflow: hidden;-->
<!--}-->

<!--.panel {-->
<!--  width: 50%;-->
<!--  display: flex;-->
<!--  flex-direction: column;-->
<!--  background: white;-->
<!--  overflow: hidden;-->
<!--}-->

<!--.panel:first-child {-->
<!--  border-right: 1px solid #ddd;-->
<!--}-->

<!--.panel-header {-->
<!--  padding: 16px 20px;-->
<!--  background: #f8f9fa;-->
<!--  border-bottom: 1px solid #ddd;-->
<!--  font-weight: 500;-->
<!--  color: #333;-->
<!--}-->

<!--.messages {-->
<!--  flex: 1;-->
<!--  overflow-y: auto;-->
<!--  padding: 20px;-->
<!--  padding-bottom: 80px; /* 为固定的输入框留出空间 */-->
<!--}-->

<!--.message {-->
<!--  margin-bottom: 12px;-->
<!--  padding: 10px 14px;-->
<!--  border-radius: 6px;-->
<!--  word-break: break-word;-->
<!--  white-space: pre-wrap; /* 保留换行和空格 */-->
<!--}-->

<!--.ai-msg {-->
<!--  background: #f3e5f5;-->
<!--  color: #6a1b9a;-->
<!--}-->

<!--.user-msg {-->
<!--  background: #e3f2fd;-->
<!--  color: #1565c0;-->
<!--}-->

<!--.input-area {-->
<!--  position: fixed;-->
<!--  bottom: 0;-->
<!--  left: 0;-->
<!--  right: 0;-->
<!--  border-top: 1px solid #ddd;-->
<!--  background: white;-->
<!--  padding: 16px 20px;-->
<!--  display: flex;-->
<!--  gap: 12px;-->
<!--  box-shadow: 0 -2px 8px rgba(0, 0, 0, 0.1);-->
<!--  z-index: 100;-->
<!--}-->

<!--.input-area input {-->
<!--  flex: 1;-->
<!--  padding: 10px 14px;-->
<!--  border: 1px solid #ddd;-->
<!--  border-radius: 6px;-->
<!--  font-size: 15px;-->
<!--  outline: none;-->
<!--}-->

<!--.input-area input:focus {-->
<!--  border-color: #1565c0;-->
<!--}-->

<!--.input-area button {-->
<!--  padding: 10px 24px;-->
<!--  background: #1565c0;-->
<!--  color: white;-->
<!--  border: none;-->
<!--  border-radius: 6px;-->
<!--  font-size: 15px;-->
<!--  cursor: pointer;-->
<!--  white-space: nowrap;-->
<!--}-->

<!--.input-area button:hover:not(:disabled) {-->
<!--  background: #0d47a1;-->
<!--}-->

<!--.input-area button:disabled {-->
<!--  background: #ccc;-->
<!--  cursor: not-allowed;-->
<!--}-->

<!--/* 优化滚动条 */-->
<!--.messages::-webkit-scrollbar {-->
<!--  width: 6px;-->
<!--}-->

<!--.messages::-webkit-scrollbar-track {-->
<!--  background: #f1f1f1;-->
<!--}-->

<!--.messages::-webkit-scrollbar-thumb {-->
<!--  background: #bbb;-->
<!--  border-radius: 3px;-->
<!--}-->
<!--</style>-->

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
        {{ isLoading ? '发送中...' : '发送' }}
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const currentInput = ref('')
const userMessages = ref([])
const aiMessages = ref([])
const isLoading = ref(false)
const sessionId = ref(`session_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`)

const sendMessage = async () => {
  if (!currentInput.value.trim() || isLoading.value) return

  const userInput = currentInput.value
  userMessages.value.push(userInput)
  currentInput.value = ''
  isLoading.value = true

  // 添加空的AI消息
  aiMessages.value.push('')
  const currentIndex = aiMessages.value.length - 1
  let currentContent = ''

  try {
    const response = await fetch('http://localhost:5000/api/chat', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        message: userInput,
        session_id: sessionId.value
      })
    })

    if (!response.ok) throw new Error('请求失败')

    const reader = response.body.getReader()
    const decoder = new TextDecoder()
    let buffer = ''

    while (true) {
      const { done, value } = await reader.read()
      if (done) break

      buffer += decoder.decode(value, { stream: true })

      // 按行分割处理事件
      const lines = buffer.split('\n')
      buffer = lines.pop() // 保留最后一个可能不完整的行

      for (const line of lines) {
        if (!line.trim()) continue

        try {
          const event = JSON.parse(line)

          // 处理不同类型的事件
          if (event.type === 'value_chunk' && event.key === 'content') {
            // 累积内容块
            currentContent += event.content

            // 处理内容中的JSON，提取实际文本
            if (currentContent.startsWith('{') || currentContent.startsWith('"')) {
              try {
                const parsed = JSON.parse(currentContent)
                if (typeof parsed === 'string') {
                  aiMessages.value[currentIndex] = parsed
                } else if (parsed.ThinkTool) {
                  aiMessages.value[currentIndex] = `[思考工具: ${parsed.ThinkTool}] ${parsed.ThinkInput}`
                }
              } catch {
                // 如果解析失败，直接显示原始内容
                aiMessages.value[currentIndex] = currentContent
              }
            } else {
              aiMessages.value[currentIndex] = currentContent
            }
          } else if (event.type === 'value_complete' && event.key === 'content') {
            // 一个完整的值结束，重置当前内容
            currentContent = ''
          } else if (event.type === 'tool_executed') {
            // 工具执行完成
            aiMessages.value[currentIndex] += `\n[工具 ${event.tool_name} 执行结果: ${event.observation}]\n`
          } else if (event.type === 'waiting_for_user') {
            // 等待用户输入
            aiMessages.value[currentIndex] += '\n[等待用户回复...]'
          }
        } catch (e) {
          console.error('解析事件失败:', e, line)
        }
      }
    }
  } catch (error) {
    console.error('发送失败:', error)
    aiMessages.value[currentIndex] = '抱歉，发生了错误，请稍后重试。'
  } finally {
    isLoading.value = false
  }
}

// 初始化时显示会话ID（可选）
onMounted(() => {
  console.log('会话ID:', sessionId.value)
})
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
  white-space: pre-wrap; /* 保留换行和空格 */
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