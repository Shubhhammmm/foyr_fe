<template>
  <div id="app" class="app-container">
    <LoginComponent v-if="!loggedIn" @loginSuccess="onLoginSuccess" />
    <div v-if="loggedIn" class="app-content">
      <WhiteboardComponent :user="user" class="whiteboard-section" />
      <ChatComponent :user="user" class="chat-section" />
    </div>
  </div>
</template>

<script>
import LoginComponent from './components/LoginComponent.vue';
import WhiteboardComponent from './components/WhiteboardComponent.vue';
import ChatComponent from './components/ChatComponent.vue';

export default {
  components: {
    LoginComponent,
    WhiteboardComponent,
    ChatComponent,
  },
  data() {
    return {
      loggedIn: false,
      user: null,
    };
  },
  methods: {
    onLoginSuccess(user) {
      this.user = user;
      this.loggedIn = true;
    },
  },
};
</script>

<style scoped>
/* Main App Container */
.app-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100vh;
  width: 100vw;
  background-color: #f0f2f5;
  padding: 20px;
  margin: 0;
}

/* Content layout when logged in */
.app-content {
  display: flex;
  width: 100%;
  max-width: 1200px;
  height: 80vh;
  background-color: #fff;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  border-radius: 10px;
  overflow: hidden;
}

/* Whiteboard Section */
.whiteboard-section {
  flex: 2;
  background-color: #fafafa;
  border-right: 2px solid #ddd;
  padding: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
}

/* Chat Section */
.chat-section {
  flex: 1;
  background-color: #f9f9f9;
  padding: 20px;
  display: flex;
  flex-direction: column;
}

/* Make the chat window scrollable */
.chat-section {
  overflow-y: auto;
}

</style>
