<template>
    <div class="chat-icon-container">
      <div 
        class="chat-icon"  @mouseover="hovered = true" @mouseleave="hovered = false" @click="toggleChat"  v-if="!chatVisible">
        
        <transition name="fade">
          <div class="icon" v-if="!hovered">
            <img src="@/assets/icon.jpg">
          </div>
        </transition>

        <div class="bot1" v-if="hovered">
            <img src="@/assets/bot.png" class="bot">
        </div>
        <transition name="expand">
          
          <div class="message" v-if="hovered">
            <img src="@/assets/icon.jpg" class="small-icon">
            Hi, let's have a chat...
          </div>
        </transition>
      </div>

      <ChatDialog v-if="chatVisible" @close="toggleChat" />
      
    </div>
  </template>
  
  <script>
  import ChatDialog from './ChatDialog.vue';
  
  export default {
    data() {
      return {
        hovered: false,
        chatVisible: false
      };
    },
    methods: {
      toggleChat() {
        this.chatVisible = !this.chatVisible;
      }
    },
    components: {
      ChatDialog
    }
  };
  </script>
  
  <style scoped>
  .chat-icon-container {
    position: fixed;
    bottom: 20px;
    right: 20px;
  }
  
  .chat-icon {
    display: flex;
    align-items: center;
  }
  
  .icon img {
    width: 50px;
    height: 50px;
    transition: opacity 0.5s;
  }
  
  .message {
    display: flex;
    align-items: center;
    background-color: white;
    border: 2px solid #4CAF50;
    border-radius: 25px;
    padding: 5px 15px;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
    margin-left: 10px;
  }
  
  .message .small-icon {
    width: 30px;
    height: 30px;
    margin-right: 10px;
  }
  
  .fade-enter-active, .fade-leave-active {
    transition: opacity 0.5s;
  }
  
  .fade-enter, .fade-leave-to {
    opacity: 0;
  }
  
  .expand-enter-active, .expand-leave-active {
    transition: opacity 0.5s;
  }
  
  .expand-enter, .expand-leave-to {
    opacity: 0;
  }

  
  </style>
  