<!-- <template>-->
<!--  <div class="app">-->
<!--    <div class="main-content">-->
<!--      <div class="panel">-->
<!--        <div class="panel-header">AI 回复 (流式思想过程)</div>-->
<!--        <div class="messages">-->
<!--          <div v-for="(msg, i) in aiMessages" :key="i" class="message ai-msg" v-html="msg"></div>-->
<!--        </div>-->
<!--      </div>-->

<!--      <div class="panel">-->
<!--        <div class="panel-header">用户输入</div>-->
<!--        <div class="messages">-->
<!--          <div v-for="(msg, i) in userMessages" :key="i" class="message user-msg">-->
<!--            {{ msg }}-->
<!--          </div>-->
<!--        </div>-->
<!--      </div>-->
<!--    </div>-->

<!--    <div class="input-area">-->
<!--      <input-->
<!--          v-model="currentInput"-->
<!--          @keyup.enter="sendMessage"-->
<!--          placeholder="请输入..."-->
<!--          :disabled="isLoading"-->
<!--      />-->
<!--      <button @click="sendMessage" :disabled="!currentInput.trim() || isLoading">-->
<!--        {{ isLoading ? '生成中...' : '发送' }}-->
<!--      </button>-->
<!--    </div>-->
<!--  </div>-->
<!--</template>-->

<!--<script setup>-->
<!--import { ref, onMounted } from 'vue'-->
<!--import { marked } from 'marked'; // Import marked for Markdown rendering-->

<!--const currentInput = ref('')-->
<!--const userMessages = ref([])-->
<!--const aiMessages = ref([])-->
<!--const isLoading = ref(false)-->
<!--const sessionId = ref(`session_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`)-->

<!--const sendMessage = async () => {-->
<!--  if (!currentInput.value.trim() || isLoading.value) return;-->

<!--  const userInput = currentInput.value;-->
<!--  userMessages.value.push(userInput);-->
<!--  currentInput.value = '';-->
<!--  isLoading.value = true;-->

<!--  try {-->
<!--    const response = await fetch('http://101.126.145.194:5000/api/chat', {-->
<!--      method: 'POST',-->
<!--      headers: { 'Content-Type': 'application/json' },-->
<!--      body: JSON.stringify({ message: userInput, session_id: sessionId.value }),-->
<!--    });-->

<!--    if (!response.ok) throw new Error(`Request failed with status ${response.status}`);-->

<!--    const reader = response.body.getReader();-->
<!--    const decoder = new TextDecoder();-->
<!--    let buffer = '';-->
<!--    while (true) {-->
<!--      const { value, done: readerDone } = await reader.read();-->
<!--      if (readerDone) {-->
<!--        // 处理可能遗留在缓冲区的最后一部分数据-->
<!--        if (buffer) tryParse(buffer);-->
<!--        break;-->
<!--      }-->

<!--      const chunk = decoder.decode(value, { stream: true });-->
<!--      buffer += chunk;-->

<!--      const parts = buffer.split('\n');-->
<!--      buffer = parts.pop() || ''; // 将不完整的部分放回缓冲区-->

<!--      for (const part of parts) {-->
<!--        tryParse(part);-->
<!--      }-->
<!--    }-->

<!--  } catch (error) {-->
<!--    console.error('Chat failed:', error);-->
<!--    aiMessages.value[currentIndex] = '抱歉，发生了错误，请稍后重试。';-->
<!--  } finally {-->
<!--    isLoading.value = false;-->
<!--  }-->
<!--};-->
<!--// 解析并更新数据的辅助函数-->
<!--function tryParse(jsonString) {-->
<!--  if (!jsonString) return;-->
<!--  try {-->
<!--    let parsed = JSON.parse(jsonString);-->
<!--  } catch (e) {-->
<!--    console.error('JSON 解析失败:', jsonString, e);-->
<!--  }-->
<!--};-->
<!--onMounted(() => {-->
<!--  console.log('Session ID:', sessionId.value);-->
<!--});-->
<!--</script>-->

<!--<style scoped>-->
<!--/* Reset some default styles */-->
<!--:global(body, html) {-->
<!--  margin: 0;-->
<!--  padding: 0;-->
<!--  height: 100%;-->
<!--  overflow: hidden;-->
<!--}-->

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
<!--  font-weight: 600;-->
<!--  color: #333;-->
<!--}-->

<!--.messages {-->
<!--  flex: 1;-->
<!--  overflow-y: auto;-->
<!--  padding: 20px;-->
<!--}-->

<!--.message {-->
<!--  margin-bottom: 12px;-->
<!--  padding: 10px 14px;-->
<!--  border-radius: 8px;-->
<!--  word-break: break-word;-->
<!--  white-space: pre-wrap;-->
<!--  line-height: 1.6;-->
<!--}-->

<!--.ai-msg {-->
<!--  background: #f7f7f7;-->
<!--  border: 1px solid #e5e5e5;-->
<!--  color: #333;-->
<!--}-->

<!--/* Styles for rendered markdown content inside ai-msg */-->
<!--.ai-msg :deep(p) {-->
<!--  margin: 0 0 8px 0;-->
<!--}-->
<!--.ai-msg :deep(p:last-child) {-->
<!--  margin-bottom: 0;-->
<!--}-->
<!--.ai-msg :deep(strong) {-->
<!--    color: #6a1b9a;-->
<!--}-->
<!--.ai-msg :deep(pre) {-->
<!--    background-color: #eef1f3;-->
<!--    padding: 12px;-->
<!--    border-radius: 4px;-->
<!--    white-space: pre-wrap;-->
<!--    word-break: break-all;-->
<!--}-->
<!--.ai-msg :deep(code) {-->
<!--    font-family: 'Courier New', Courier, monospace;-->
<!--    background-color: #eef1f3;-->
<!--    padding: 2px 4px;-->
<!--    border-radius: 3px;-->
<!--    font-size: 0.9em;-->
<!--}-->
<!--.ai-msg :deep(pre code) {-->
<!--    padding: 0;-->
<!--    background-color: transparent;-->
<!--}-->


<!--.user-msg {-->
<!--  background: #e3f2fd;-->
<!--  color: #1565c0;-->
<!--}-->

<!--.input-area {-->
<!--  border-top: 1px solid #ddd;-->
<!--  background: white;-->
<!--  padding: 16px 20px;-->
<!--  display: flex;-->
<!--  gap: 12px;-->
<!--  box-shadow: 0 -2px 8px rgba(0, 0, 0, 0.07);-->
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
<!--  box-shadow: 0 0 0 2px rgba(21, 101, 192, 0.2);-->
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
<!--  transition: background-color 0.2s;-->
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
    <div class="main-content">
      <div class="panel">
        <div class="panel-header">AI 回复 (流式思想过程)</div>
        <div class="messages">
          <div v-for="(msg, i) in aiMessages" :key="i" class="message ai-msg" v-html="msg"></div>
        </div>
      </div>

      <div class="panel">
        <div class="panel-header">用户输入</div>
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
import { ref, onMounted } from 'vue'
import { marked } from 'marked'; // 导入 marked 用于 Markdown 渲染

const currentInput = ref('') // 用户当前输入
const userMessages = ref([]) // 存储用户消息的数组
const aiMessages = ref([]) // 存储 AI 消息的数组
const isLoading = ref(false) // 控制加载状态
const sessionId = ref(`session_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`) // 会话 ID

// 用于追踪当前正在接收内容的 AI 消息索引
let currentAiMessageIndex = -1;
// 用于追踪当前正在接收的 key 类型 (e.g., 'state', 'content')
let currentKeyType = '';

// 发送消息函数
const sendMessage = async () => {
  if (!currentInput.value.trim() || isLoading.value) return;

  const userInput = currentInput.value;
  userMessages.value.push(userInput); // 将用户输入添加到用户消息列表
  currentInput.value = ''; // 清空输入框
  isLoading.value = true; // 设置加载状态为 true

  // 每次发送新消息时，为 AI 消息创建一个新的占位符，并更新索引
  aiMessages.value.push('');
  currentAiMessageIndex = aiMessages.value.length - 1;
  currentKeyType = ''; // 重置当前 key 类型

  try {
    const response = await fetch('http://101.126.145.194:5000/api/chat', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ message: userInput, session_id: sessionId.value }),
    });

    if (!response.ok) throw new Error(`Request failed with status ${response.status}`);

    const reader = response.body.getReader();
    const decoder = new TextDecoder();
    let buffer = ''; // 缓冲区用于存储不完整的 JSON 字符串

    while (true) {
      const { value, done: readerDone } = await reader.read();
      if (readerDone) {
        // 处理可能遗留在缓冲区的最后一部分数据
        if (buffer) tryParse(buffer);
        break;
      }

      const chunk = decoder.decode(value, { stream: true });
      buffer += chunk;

      // 按换行符分割，处理完整的 JSON 字符串
      const parts = buffer.split('\n');
      buffer = parts.pop() || ''; // 将不完整的部分放回缓冲区

      for (const part of parts) {
        tryParse(part); // 解析每个完整的 JSON 字符串
      }
    }

  } catch (error) {
    console.error('Chat failed:', error);
    // 如果发生错误，更新当前 AI 消息为错误提示
    aiMessages.value[currentAiMessageIndex] = '抱歉，发生了错误，请稍后重试。';
  } finally {
    isLoading.value = false; // 无论成功或失败，都设置加载状态为 false
  }
};
// 解析并更新数据的辅助函数
function tryParse(jsonString) {
  if (!jsonString) return;
  try {
    let parsed = JSON.parse(jsonString);
    console.log(parsed);
    if (parsed.type === 'key_complete') {
      // 如果是 key_complete 类型，设置当前 key 类型，并准备接收后续的 value_chunk
      currentKeyType = parsed.key;
      // 在新的 key_complete 开始时，添加一个新行，并用粗体显示 key
      if (aiMessages.value[currentAiMessageIndex]) {
        aiMessages.value[currentAiMessageIndex] += '\n\n';
      }
      aiMessages.value[currentAiMessageIndex] += `**${parsed.key}:** `;
    } else if (parsed.type === 'value_chunk') {
      // 如果是 value_chunk 类型，追加内容到当前 AI 消息
      // 确保有当前 AI 消息索引并且有 key 类型被设置
      if (currentAiMessageIndex !== -1 && currentKeyType) {
        // 移除 content 值中的引号
        let contentWithoutQuotes = parsed.content.replace(/^"|"$/g, '');
        // *** 关键修改：将 JSON 编码的 \\n 替换为实际的 \n ***
        contentWithoutQuotes = contentWithoutQuotes.replace(/\\n/g, '\n'); //
        aiMessages.value[currentAiMessageIndex] += contentWithoutQuotes;
      }
    } else if (parsed.type === 'tool_executed') {
      // 如果是 tool_executed 类型，直接显示 observation 的值
      if (currentAiMessageIndex !== -1) {
        if (aiMessages.value[currentAiMessageIndex]) {
          aiMessages.value[currentAiMessageIndex] += '\n\n';
        }
        aiMessages.value[currentAiMessageIndex] += `**Tool Executed Observation:** \n\n ${parsed.observation}`;
      }
    }
    // 使用 marked.js 渲染 Markdown
    aiMessages.value[currentAiMessageIndex] = marked.parse(aiMessages.value[currentAiMessageIndex]);

  } catch (e) {
    console.error('JSON 解析失败:', jsonString, e);
  }
}
// // 解析并更新数据的辅助函数
// function tryParse(jsonString) {
//   if (!jsonString) return;
//   try {
//     let parsed = JSON.parse(jsonString);
//     console.log(parsed)
//     if (parsed.type === 'key_complete') {
//       // 如果是 key_complete 类型，设置当前 key 类型，并准备接收后续的 value_chunk
//       currentKeyType = parsed.key;
//       // 在新的 key_complete 开始时，添加一个新行，并用粗体显示 key
//       if (aiMessages.value[currentAiMessageIndex]) {
//         aiMessages.value[currentAiMessageIndex] += '\n\n';
//       }
//       aiMessages.value[currentAiMessageIndex] += `**${parsed.key}:** `;
//     } else if (parsed.type === 'value_chunk') {
//       // 如果是 value_chunk 类型，追加内容到当前 AI 消息
//       // 确保有当前 AI 消息索引并且有 key 类型被设置
//       if (currentAiMessageIndex !== -1 && currentKeyType) {
//         // 移除 content 值中的引号
//         const contentWithoutQuotes = parsed.content.replace(/^"|"$/g, '');
//         aiMessages.value[currentAiMessageIndex] += contentWithoutQuotes;
//       }
//     } else if (parsed.type === 'tool_executed') {
//       // 如果是 tool_executed 类型，直接显示 observation 的值
//       if (currentAiMessageIndex !== -1) {
//         if (aiMessages.value[currentAiMessageIndex]) {
//           aiMessages.value[currentAiMessageIndex] += '\n\n';
//         }
//         aiMessages.value[currentAiMessageIndex] += `**Tool Executed Observation:** \n\n ${parsed.observation}`;
//       }
//     }
//     // 使用 marked.js 渲染 Markdown
//     aiMessages.value[currentAiMessageIndex] = marked.parse(aiMessages.value[currentAiMessageIndex]);
//
//   } catch (e) {
//     console.error('JSON 解析失败:', jsonString, e);
//   }
// };

onMounted(() => {
  console.log('Session ID:', sessionId.value);
});
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