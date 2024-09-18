<template>
  <div class="login">
    <input type="text" v-model="username" placeholder="Enter username" />
    <button @click="login">Login</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      username: '',
    };
  },
  methods: {
    login() {
      if (this.username) {
        const user = { name: this.username, id: Date.now() };
        this.$emit('loginSuccess', user);
      }
    },
  },
};
</script>
