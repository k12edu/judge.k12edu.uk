<template>
    <div class="message-board" style="min-height: 100vh;">
      <div v-if="loading" class="loading">
        <p>載入中...</p>
      </div>
  
      <!-- 顯示討論區的資料 -->
      <div v-else>
        <h2>討論區</h2>
        <div class="message-container">
          <div v-for="message in messages" :key="message.id" class="message-item">
            <h3>{{ message.title }}</h3>
            <p>{{ message.content }}</p>
          </div>
        </div>
  
        <!-- 分頁 -->
        <div class="pagination">
          <button :disabled="currentPage === 1" @click="changePage(currentPage - 1)">上一頁</button>
          <span>{{ currentPage }} / {{ totalPages }}</span>
          <button :disabled="currentPage === totalPages" @click="changePage(currentPage + 1)">下一頁</button>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        messages: [], // 存放從 API 獲取的討論區資料
        currentPage: 1, // 當前頁數
        totalPages: 1, // 總頁數
        loading: true, // 是否正在載入
      };
    },
    methods: {
      // 更換頁數
      changePage(page) {
        this.currentPage = page;
        this.fetchMessages();
      },
      // 從 API 請求資料
      async fetchMessages() {
        try {
          const response = await fetch(`https://api.example.com/messages?page=${this.currentPage}`);
          const data = await response.json();
  
          this.messages = data.messages;
          this.totalPages = data.totalPages;
        } catch (error) {
          console.error("Error fetching messages:", error);
        } finally {
          this.loading = false;
        }
      },
    },
    created() {
      this.fetchMessages();
    },
  };
  </script>
  
  <style scoped>
  .message-board {
    padding: 20px;
  }
  
  .message-container {
    margin-bottom: 20px;
  }
  
  .message-item {
    border-bottom: 1px solid #ccc;
    margin-bottom: 10px;
    padding-bottom: 10px;
  }
  
  .pagination {
    text-align: center;
    margin-top: 20px;
  }
  
  .pagination button {
    padding: 5px 10px;
    margin: 0 5px;
    border: 1px solid #ccc;
    background-color: #f4f4f4;
    cursor: pointer;
  }
  
  .pagination button:disabled {
    background-color: #e0e0e0;
    cursor: not-allowed;
  }
  </style>
  