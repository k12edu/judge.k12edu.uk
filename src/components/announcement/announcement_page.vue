<template>
    <div class="announcement-page" style="min-height: 100vh;">
      <h1>公告</h1>
      <div v-if="announcements.length">
        <ul>
          <li v-for="announcement in paginatedAnnouncements" :key="announcement.id">
            <h3>{{ announcement.title }}</h3>
            <p>{{ announcement.content }}</p>
          </li>
        </ul>
      </div>
      <div v-else>
        <p>目前沒有公告。</p>
      </div>
      <div class="pagination">
        <button @click="changePage(page)" :class="{ active: currentPage === page }" v-for="page in totalPages" :key="page">
          {{ page }}
        </button>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        announcements: [], // 全部公告
        currentPage: 1,    // 當前頁數
        pageSize: 5,       // 每頁顯示幾筆資料
      };
    },
    computed: {
      totalPages() {
        return Math.ceil(this.announcements.length / this.pageSize);
      },
      paginatedAnnouncements() {
        const start = (this.currentPage - 1) * this.pageSize;
        return this.announcements.slice(start, start + this.pageSize);
      },
    },
    methods: {
      async fetchAnnouncements() {
        try {
          const response = await fetch("https://your-api-endpoint.com/announcements");
          if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
          }
          const data = await response.json();
          this.announcements = data; // 假設 API 回傳的資料是一個公告陣列
        } catch (error) {
          console.error("取得公告時發生錯誤：", error);
        }
      },
      changePage(page) {
        this.currentPage = page;
      },
    },
    mounted() {
      this.fetchAnnouncements();
    },
  };
  </script>
  
  <style scoped>
  .announcement-page {
    padding: 20px;
  }
  
  .pagination {
    display: flex;
    gap: 10px;
    margin-top: 20px;
  }
  
  button {
    padding: 10px 20px;
    border: 1px solid #ddd;
    background-color: white;
    cursor: pointer;
  }
  
  button.active {
    background-color: #007bff;
    color: white;
  }
  </style>
  