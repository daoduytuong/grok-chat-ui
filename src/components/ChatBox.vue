<template>
    <div class="chat-container">
      <div ref="chatHistory" class="chat-history">
        <div
          v-for="(msg, index) in messages"
          :key="index"
          :class="['message', msg.role === 'user' ? 'user-message' : 'bot-message']"
        >
          <strong>{{ msg.role === 'user' ? 'You' : 'Grok' }}:</strong>
          <div v-html="renderContent(msg.content)" class="message-content"></div>
        </div>
      </div>
  
      <form @submit.prevent="sendMessage" class="chat-input">
        <input v-model="newMessage" type="text" placeholder="Type your message..." :disabled="isLoading" />
        <button type="submit" :disabled="isLoading">Send</button>
      </form>
  
      <p v-if="isLoading" class="loading">Sending...</p>
    </div>
  </template>
  
  <script>
  import axios from 'axios';
  import { marked } from 'marked';
  import hljs from 'highlight.js';
  import 'highlight.js/styles/github.css'; // Theme cho code
  
  export default {
    name: 'ChatBox',
    data() {
      return {
        messages: [],
        newMessage: '',
        isLoading: false,
        sessionId: 'user1',
      };
    },
    async created() {
      await this.loadHistory();
    },
    watch: {
      messages() {
        this.$nextTick(() => {
          const chat = this.$refs.chatHistory;
          chat.scrollTop = chat.scrollHeight; // Tự động cuộn xuống
        });
      },
    },
    methods: {
      async loadHistory() {
        try {
          const response = await axios.get(`http://localhost:3000/history/${this.sessionId}`);
          this.messages = response.data.messages;
        } catch (error) {
          console.error('Error loading history:', error);
          this.messages.push({ role: 'assistant', content: 'Error: Could not load history' });
        }
      },
      async sendMessage() {
        if (!this.newMessage.trim()) return;
  
        this.messages.push({ role: 'user', content: this.newMessage });
        const messageToSend = this.newMessage;
        this.newMessage = '';
        this.isLoading = true;
  
        try {
          const response = await axios.post('http://localhost:3000/chat', {
            sessionId: this.sessionId,
            message: messageToSend,
          });
          this.messages.push({ role: 'assistant', content: response.data.response });
        } catch (error) {
          console.error('Error sending message:', error);
          this.messages.push({ role: 'assistant', content: 'Error: Could not reach the server' });
        } finally {
          this.isLoading = false;
        }
      },
      renderContent(content) {
        // Tách nội dung JSON nếu có
        const codeBlockMatch = content.match(/```json\n([\s\S]*?)\n```/);
        if (codeBlockMatch) {
            const jsonStr = codeBlockMatch[1];
            try {
            const data = JSON.parse(jsonStr);
            if (data.type === 'form') {
                // Tạo HTML cho form
                let formHtml = '<form>';
                data.fields.forEach(field => {
                formHtml += `<label>${field.label}: <input type="${field.type}" /></label><br>`;
                });
                formHtml += '</form>';
                return marked(content.replace(codeBlockMatch[0], '')) + formHtml;
            }
            } catch (e) {
            console.error('Error parsing JSON:', e);
            }
        }
        return marked(content);
        },
    },
  };
  </script>
  
  <style scoped>
  .chat-container {
    border: 1px solid #ccc;
    border-radius: 5px;
    padding: 10px;
    height: 500px;
    display: flex;
    flex-direction: column;
  }
  
  .chat-history {
    flex: 1;
    overflow-y: auto;
    padding: 10px;
    background: #f9f9f9;
  }
  
  .message {
    margin: 10px 0;
    padding: 8px;
    border-radius: 5px;
  }
  
  .user-message {
    background: #9bbce0;
    color: white;
    text-align: right;
  }
  
  .bot-message {
    background: #e9ecef;
    color: #333;
    text-align: left;
  }
  
  .message-content {
    margin-top: 5px;
  }
  
  .chat-input {
    display: flex;
    gap: 10px;
    padding: 10px 0;
  }
  
  input {
    flex: 1;
    padding: 5px;
    border: 1px solid #ccc;
    border-radius: 5px;
  }
  
  button {
    padding: 5px 15px;
    background: #007bff;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
  }
  
  button:disabled {
    background: #ccc;
    cursor: not-allowed;
  }
  
  .loading {
    text-align: center;
    color: #666;
  }
  
  /* Style cho code từ Highlight.js */
  pre {
    background: #f4f4f4;
    padding: 10px;
    border-radius: 5px;
    overflow-x: auto;
  }
  
  code {
    font-family: 'Courier New', Courier, monospace;
  }
  </style>