<template>
  <div id="app">
    <h1>Grok Chatbot</h1>
    <ChatBox />
  </div>
</template>

<script>
import ChatBox from './components/ChatBox.vue';

export default {
  name: 'App',
  components: {
    ChatBox,
  },
};
</script>

<style>
#app {
  font-family: Arial, sans-serif;
  max-width: 70vw;
  margin: 0 auto;
  padding: 20px;
}
h1 {
  text-align: center;
  color: #333;
}
</style>