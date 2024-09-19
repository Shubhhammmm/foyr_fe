<template>
  <div class="login-container">
    <div class="login-box">
      <h2>Login</h2>
      <input
        type="text"
        v-model="username"
        class="login-input"
        placeholder="Enter username"
      />
      <input
        type="password"
        v-model="password"
        class="login-input"
        placeholder="Enter password"
      />
      <button class="login-btn" @click="login">Login</button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      username: '',
      password: '',
    };
  },
  methods: {
    login() {
      if (this.username && this.password) {
        const user = { name: this.username, id: Date.now() };
        
        this.$emit('loginSuccess', user);
      } else {
        alert('Please enter both username and password.');
      }
    },
  },
};
</script>

<style scoped>
/* Login Page Styling */
.login-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  /* background-color: #f4f7f6; */
}

.login-box {
  background-color: white;
  padding: 2rem;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  width: 300px;
  text-align: center;
}

h2 {
  font-size: 24px;
  margin-bottom: 1.5rem;
  color: #333;
}

.login-input {
  width: 100%;
  padding: 10px;
  margin-bottom: 1.5rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 16px;
  box-sizing: border-box;
}

.login-input:focus {
  outline: none;
  border-color: #007bff;
}

.login-btn {
  width: 100%;
  padding: 10px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  font-weight: bold;
}

.login-btn:hover {
  background-color: #0056b3;
}

.login-btn:focus {
  outline: none;
}
</style>
