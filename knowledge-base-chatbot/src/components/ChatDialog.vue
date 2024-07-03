<template>
  <div :class="['c', { 'c-fullscreen': isFullscreen }]">
    <div class="head">
      <div class="lh">
        <img src="@/assets/conv.png" class="icon" @click="jump_conv" />
        <img src="@/assets/db.png" class="icon" @click="jump_db" />
      </div>
      <div class="rh">
        <img src="@/assets/minus.png" class="icon" @click="mini" />
        <img src="@/assets/expand.png" class="icon" @click="full" />
        <img src="@/assets/cross.png" class="icon" @click="$emit('close')" />
      </div>
    </div>
    
    <div v-if="createdb" class="c-db">
      <div class="k-header1">
        <img src="@/assets/rename.png" class="k-item-sub-icon" />创建知识库
      </div>

      <div>
        <label>名字</label>
        <input v-model="newKnowledgeBase.name" placeholder="添加名字" class="bn">
      </div>
      <div>
        <label>描述</label>
        <input v-model="newKnowledgeBase.description" placeholder="添加描述" class="bn">
      </div>
      <div>
        <label>索引模型</label>
        <input v-model="newKnowledgeBase.indexModel" placeholder="索引模型" class="bn">
      </div>
      
      <div>
        <label>上传数据</label>
        <div class="upload-box" @click="triggerFileInput" @drop.prevent="handleDrop" @dragover.prevent>
          <input type="file" class="file-input" @change="handleFileUpload" ref="fileInput">
          <p>点击上传或拖动文件到此</p>
          <p>支持超多格式！</p>
        </div>
        <div class="upload-buttons">
          <button class="local-file" @click="triggerFileInput">本地文件</button>
          <button class="web-link">网页链接</button>
        </div>
      </div>

      <div>
        <label>采样温度</label>
        <input type="range" v-model="newKnowledgeBase.samplingTemperature" min="0" max="1" step="0.01">
      </div>
      <button @click="createKnowledgeBase" class="fin_bt">完成</button>
    </div>

    <div v-else-if="db" class="know-b">
      <div class="k-header">
        <h2 class="k-h-t">我的知识库</h2>
        <h4 @click="createdb = true" class="k-h-b">新建</h4>
      </div>

      <div class="k-item" v-for="(kb, index) in knowledgeBases" :key="kb.name" >
        <div class="k-item-l1" >
          <img src="@/assets/db-icon.png" class="k-item-icon" @click="ret_jp(kb.name)">
          <h3 class="k-t" @click="ret_jp(kb.name)">{{ kb.name }}</h3>
          <img src="@/assets/select.png" class="k-item-select" @click="toggleDropdown(index)">
        </div>

        <p class="k-item-l2" @click="ret_jp(kb.name)">{{ kb.description }}</p>

        <div class="k-item-l3" @click="ret_jp(kb.name)">
          <span>
            <img src="@/assets/gpt.png" class="k-item-sub-icon">
            <h>GPT3.5</h>
          </span>
          <span>
            <img src="@/assets/web.png" class="k-item-sub-icon">
            <h>Web 站点同步</h>
          </span>
        </div>

        <div v-if="dropdownIndex === index" class="dropdown-menu">
          <div class="dropdown-item" @click="renameItem(index)">
            <img src="@/assets/rename.png" class="dropdown-icon"> 重命名
          </div>
          <div class="dropdown-item" @click="deleteItem(index)">
            <img src="@/assets/delete.png" class="dropdown-icon"> 删除
          </div>
        </div>
      </div>
    </div>

    <div v-else class="add">
      <div class="content" @click="jump">
        <img src="@/assets/down.png" class="down" />
        <span class="ass_name">{{ this.selectedAssistant.name }}</span>
      </div>
    </div>

    <div class="dialog" v-if="!db && !createdb">
      <div class="messages" ref="messagesContainer">
        <div v-for="(message, index) in messages" :key="index" :class="['message-wrapper', message.sender]">
          <img v-if="message.sender === 'bot'" src="@/assets/icon.jpg" alt="bot" class="message-icon" />
          <div :class="['message', message.sender]">
            {{ message.text }}
          </div>
        </div>
      </div>

      <div class="input-area">
        <img src="@/assets/mic.png" class="input-icon" @click="processVoiceMessage" />
        <input v-model="userInput" @keyup.enter="sendMessage" placeholder="Type your message..." />
        <button @click="sendMessage" class="bt">
          <img src="@/assets/send.png" class="send-icon" />
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      userInput: '',
      messages: [],
      dropdownIndex: null,
      db: false,
      isFullscreen: false,
      createdb: false,
      selectedAssistant: { name: 'AI方案助理' },
      knowledgeBases: [
        { name: 'JAVA 核心技术', description: '这个知识库还没有介绍~' },
        { name: 'C 语言程序设计', description: '这个知识库还没有介绍~' }
      ],
      newKnowledgeBase: {
        name: '',
        description: '',
        indexModel: '',
        file: null,
        samplingTemperature: 0.5
      }
    };
  },
  methods: {
    toggleDropdown(index) {
      this.dropdownIndex = this.dropdownIndex === index ? null : index;
    },
    renameItem(index) {
      const newName = prompt("请输入新的名称:", this.knowledgeBases[index].name);
      if (newName !== null) {
        this.knowledgeBases[index].name = newName;
      }
      this.dropdownIndex = null;
    },
    deleteItem(index) {
      this.knowledgeBases.splice(index, 1);
      this.dropdownIndex = null;
    },
    async sendMessage() {
      if (this.userInput.trim() === '') return;

      this.messages.push({ sender: 'user', text: this.userInput });

      try {
        const response = await axios.post('http://localhost:8081/api/messages', {
          text: this.userInput
        });

        this.messages.push({ sender: 'bot', text: response.data.text });
      } catch (error) {
        console.error('Error sending message:', error);
        this.messages.push({ sender: 'bot', text: '发送消息时出错，请稍后再试。' });
      }

      this.userInput = '';
      this.scrollToBottom();
    },
    triggerFileInput() {
      this.$refs.fileInput.click();
    },
    scrollToBottom() {
      this.$nextTick(() => {
        const messagesContainer = this.$refs.messagesContainer;
        if (messagesContainer) {
          messagesContainer.scrollTop = messagesContainer.scrollHeight;
        }
      });
    },
    full() {
      this.isFullscreen = true;
    },
    mini() {
      this.isFullscreen = false;
    },
    selectAssistant(assistant) {
      this.selectedAssistant.name = assistant;
    },
    jump() {
      this.db = true;
    },
    jump_conv() {
      this.db = false;
      this.createdb = false;
    },
    jump_db() {
      this.db = false;
      this.createdb = true;
    },
    handleFileUpload(event) {
      this.newKnowledgeBase.file = event.target.files[0];
    },
    handleDrop(event) {
      const files = event.dataTransfer.files;
      this.newKnowledgeBase.file = files[0];
    },
    createKnowledgeBase() {
      if(this.newKnowledgeBase.name== '' || this.newKnowledgeBase.description==''){
        this.newKnowledgeBase = {
          name: '',
          description: '',
          indexModel: '',
          file: null,
          samplingTemperature: 0.5,
        }
        this.createdb = false;
      }
      else{
        this.knowledgeBases.push({ ...this.newKnowledgeBase });
        this.createdb = false;
        this.newKnowledgeBase = {
          name: '',
          description: '',
          indexModel: '',
          file: null,
          samplingTemperature: 0.5
        };
      }
    },
    ret_jp(name){
      this.selectAssistant(name);
      this.db = false;
    },
  }
};
</script>

<style scoped>
.upload-box {
  border: 1px dashed #ccc;
  border-radius: 10px;
  padding: 10px;
  text-align: center;
  color: #999;
  cursor: pointer;
  margin-bottom: 10px;
}

.file-input {
  display: none;
}

.upload-buttons {
  display: flex;
  justify-content: space-between;
}

.upload-buttons button.local-file {
  background-color: white;
  color: black;
  border: 1px solid #ccc;
  border-radius: 20px;
  font-size: 12px;
  width: 14vw;
  height: 5vh;
  margin-right: 20px;
}

.upload-buttons button.web-link {
  background-color: white;
  color: black;
  border: 1px solid #ccc;
  border-radius: 20px;
  font-size: 12px;
  width: 14vw;
  height: 5vh;
}

.upload-buttons button:hover {
  background-color: #f5f5f5;
}

.c {
  position: fixed;
  bottom: 20px;
  right: 20px;
  width: 400px;
  height: 650px;
  background-color: white;
  border: 2px solid #4CAF50;
  border-radius: 15px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
  display: flex;
  flex-direction: column;
  overflow: hidden;
  transition: all 0.3s ease;
}

.c-fullscreen {
  width: 100%;
  height: 100%;
  bottom: 0;
  right: 0;
  border-radius: 0;
}

.head {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  border-bottom: 1px solid #ccc;
  background-color: #f5f5f5;
}

.hl {
  display: flex;
  align-items: center;
}

.lh .icon {
  width: 25px;
  height: 25px;
  margin-left: 10px;
  padding: 3px;
}

.rh {
  display: flex;
  align-items: center;
}

.rh .icon {
  cursor: pointer;
  width: 24px;
  height: 24px;
  margin-right: 10px;
  padding: 3px;
}

.add {
  display: flex;
  flex-direction: column;
  border-bottom: 1px solid #ccc;
}

.content {
  display: flex;
  padding: 10px;
  cursor: pointer;
}

.down {
  margin-left: 10px;
  height: 20px;
  width: 20px;
}

.dialog {
  flex: 1;
  display: flex;
  flex-direction: column;
  padding: 10px;
  overflow: hidden;
}

.messages {
  flex: 1;
  overflow-y: auto;
  margin-bottom: 8px;
}

.message-wrapper {
  display: flex;
  align-items: flex-end;
  margin-bottom: 10px;
}

.message-icon {
  width: 30px;
  height: 30px;
  margin-right: 10px;
}

.message {
  padding: 10px;
  border-radius: 10px;
  max-width: 70%;
  word-wrap: break-word;
}

.message.user {
  background-color: #f5f5f5;
  align-self: flex-end;
  margin-left: auto;
  border-top-right-radius: 10px;
  border-bottom-right-radius: 0;
  border-bottom-left-radius: 10px;
}

.message.bot {
  background-color: #f5f5f5;
  align-self: flex-start;
  margin-right: auto;
  border-top-left-radius: 10px;
  border-bottom-left-radius: 0;
  border-bottom-right-radius: 10px;
}

.input-area {
  display: flex;
  align-items: center;
  border-top: 1px solid #ccc;
  padding-top: 8px;
}

.input-icon {
  width: 30px;
  height: 30px;
  margin-right: 10px;
}

input {
  flex: 1;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 20px;
  margin-right: 10px;
}

button {
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 50%;
  cursor: pointer;
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.send-icon {
  width: 20px;
  height: 20px;
}

.sd {
  background-color: white;
}

.bt {
  background-color: white;
  color: black;
}

.ass_name {
  text-align: left;
  margin-left: 10px;
}

.know-b {
  flex: 1;
  padding: 0 10px 10px 10px;
  overflow-y: auto;
}

.k-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 0;
  border-bottom: 1px solid #ccc;
}

.k-item {
  border: 1px solid #ccc;
  border-radius: 10px;
  padding: 10px;
  margin: 10px 0;
  position: relative; 
}

.k-header h2 {
  margin: 0;
  font-size: 18px;
  color: #1d541e;
}

.k-header1 {
  display: flex;
  margin-top: 2px;
  margin-left: 5px;
  border-bottom: 1px solid #ccc;
  font-size: 17px;
  color: #1d541e;
}

.k-h-b {
  background-color: #aad5b9;
  color: #1d541e;
  border: none;
  border-radius: 10px;
  cursor: pointer;
  width: 70px;
  height: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0;
  margin-right: 5px;
  padding: 0;
  font-size: 16px;
}

.k-item-l1 {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 10px;
}

.k-item-icon {
  width: 30px;
  height: 30px;
  margin-right: 10px;
}

.k-t {
  font-size: 18px;
  font-weight: normal;
  margin: 0;
  flex-grow: 1;
}

.k-item-select {
  width: 20px;
  height: 20px;
  cursor: pointer;
}

.k-item-l2 {
  color: #999;
  margin: 10px 0;
}

.k-item-l3 {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.k-item-l3 span {
  display: flex;
  align-items: center;
  margin-top: 15px;
}

.k-item-sub-icon {
  width: 20px;
  height: 20px;
  margin-right: 5px;
}

.dropdown-menu {
  position: absolute;
  top: 40px;
  right: 10px;
  border: 1px solid #e0e0e0;
  background-color: #fff;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  border-radius: 5px;
  z-index: 1000;
  display: flex;
  flex-direction: column;
  padding: 5px 0;
}

.dropdown-item {
  padding: 4px 6px;
  background: none;
  border: none;
  text-align: left;
  cursor: pointer;
  display: flex;
  align-items: center;
  width: 100%;
  font-size: 14px;
  box-sizing: border-box;
}

.dropdown-item:hover {
  background-color: #f0f0f0;
  padding: 4px 6px;
}

.dropdown-icon {
  width: 16px;
  height: 16px;
  margin-right: 10px;
}

.c-db {
  padding: 10px;
  background-color: #fff;
  margin: 0;
}

.c-db h2 {
  font-size: 20px;
  margin-bottom: 10px;
}

.c-db label {
  display: block;
  margin-top: 5px;
  margin-left: 2px;
  margin-right: 2px;
  margin-bottom: 5px;
}

.bn {
  width: 95%;
  border-radius: 10px;
  height: 20px;
}

.c-db input[type="range"] {
  width: 94%;
  padding: 8px;
  margin-bottom: 10px;
  background-color: green;
}

.c-db button {
  display: block;
  width: 100%;
  padding: 10px;
  background-color: #b4dab5;
  color: rgb(24, 22, 22);
  border: none;
  border-radius: 15px;
  cursor: pointer;
  margin-bottom: 10px;
}
</style>