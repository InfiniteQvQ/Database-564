<template>
  <div id="app">
    <chatIcon />
  </div>
</template>

<script>
import chatIcon from './components/chatIcon.vue';

export default {
  name: 'App',
  components: {
    chatIcon
  }
}
</script>

<style>
#app {
  position: relative;
  width: 100%;
  height: 100vh;
  background-color: #f5f5f5;
}
</style>