<template>
  <div class="chat">
    <div class="messages">
      <div v-for="msg in messages" :key="msg.id">{{ msg.user }}: {{ msg.text }}</div>
    </div>
    <input v-model="message" @keyup.enter="sendMessage" placeholder="Type a message" />
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
    });
  },
  methods: {
    sendMessage() {
      if (this.message) {
        const msg = { user: this.user.name, text: this.message };
        this.socket.emit('sendMessage', msg);
        this.message = '';
      }
    }
  }
};
</script>
