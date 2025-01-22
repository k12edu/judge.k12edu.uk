<template>
    <div class="rank-board-page" style="min-height: 100vh;">
      <h1>排行榜</h1>
      <div v-if="rankings.length">
        <ul>
          <li v-for="(ranking, index) in paginatedRankings" :key="ranking.userId">
            <p>{{ index + 1 }}. {{ ranking.username }} - {{ ranking.score }} 分</p>
          </li>
        </ul>
      </div>
      <div v-else>
        <p>目前沒有排名資料。</p>
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
        rankings: [],   // 排行榜資料
        currentPage: 1, // 當前頁數
        pageSize: 5,    // 每頁顯示的排名數量
      };
    },
    computed: {
      totalPages() {
        return Math.ceil(this.rankings.length / this.pageSize);
      },
      paginatedRankings() {
        const start = (this.currentPage - 1) * this.pageSize;
        return this.rankings.slice(start, start + this.pageSize);
      },
    },
    methods: {
      async fetchRankings() {
        try {
          const response = await fetch("https://your-api-endpoint.com/rankings");
          if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
          }
          const data = await response.json();
          this.rankings = data; // 假設 API 回傳的資料是包含排行榜用戶和分數的陣列
        } catch (error) {
          console.error("取得排行榜時發生錯誤：", error);
        }
      },
      changePage(page) {
        this.currentPage = page;
      },
    },
    mounted() {
      this.fetchRankings();
    },
  };
  </script>
  
  <style scoped>
  .rank-board-page {
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
  