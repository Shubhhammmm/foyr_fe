<template>
  <div class="chat-container">
    <div class="messages">
      <div v-for="msg in messages" :key="msg.id" class="message">
        <strong>{{ msg.user }}:</strong> {{ msg.text }}
      </div>
    </div>
    <div class="input-container">
      <input 
        v-model="message" 
        @keyup.enter="sendMessage" 
        placeholder="Type a message" 
        class="message-input"
      />
      <button @click="sendMessage" class="send-button">Send</button>
    </div>
  </div>
</template>

<script>
import io from 'socket.io-client';

export default {
  props: ['user'],
  data() {
    return {
      socket: null,
      message: '',
      messages: []
    };
  },
  mounted() {
    this.socket = io('https://foyr-be.onrender.com');
    
    this.socket.on('newMessage', (msg) => {
      this.messages.push(msg);
      this.scrollToBottom();  // Scroll to the latest message
    });
  },
  methods: {
    sendMessage() {
      if (this.message) {
        const msg = { user: this.user.name, text: this.message };
        this.socket.emit('sendMessage', msg);
        this.message = '';  // Clear input after sending
      }
    },
    scrollToBottom() {
      const messagesContainer = this.$el.querySelector('.messages');
      messagesContainer.scrollTop = messagesContainer.scrollHeight;
    }
  }
};
</script>


<style scoped>
.chat-container {
  display: flex;
  flex-direction: column;
  width: 100%;
  max-width: 600px;
  margin: 0 auto;
  border: 1px solid #ddd;
  border-radius: 8px;
  overflow: hidden;
  background-color: #fff;
}

.messages {
  flex-grow: 1;
  padding: 10px;
  height: 300px;
  overflow-y: auto;
  background-color: #f9f9f9;
  border-bottom: 1px solid #ddd;
}

.message {
  margin-bottom: 10px;
  padding: 8px 10px;
  border-radius: 5px;
  background-color: #e6f7ff;
  font-size: 14px;
}

.message strong {
  color: #007bff;
}

.input-container {
  display: flex;
  padding: 10px;
  background-color: #fff;
  border-top: 1px solid #ddd;
}

.message-input {
  flex-grow: 1;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
}

.send-button {
  margin-left: 10px;
  padding: 10px 15px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

.send-button:hover {
  background-color: #0056b3;
}
</style>

