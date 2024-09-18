<template>
  <div id="app">
    <LoginComponent v-if="!loggedIn" @loginSuccess="onLoginSuccess" />
    <WhiteboardComponent v-if="loggedIn" :user="user" />
    <ChatComponent v-if="loggedIn" :user="user" />
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
