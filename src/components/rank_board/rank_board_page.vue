<template>
  <div class="rank-board-page" style="min-height: 100vh;">
    <h1>排行榜</h1>
    <div v-if="rankings.length">
      <div class="rank-grid">
        <div class="rank-grid-header">
          <p>排名</p>
          <p>用戶名稱</p>
          <p>解題數量</p>
        </div>
        <div class="rank-grid-item" v-for="(ranking) in rankings" :key="ranking.id">
          <p class="rank_data">{{ ranking.rank }}</p>
          <p class="rank_data">{{ ranking.user_name }}</p>
          <p class="rank_data">{{ ranking.num_of_solved_program_problem }}</p>
        </div>
      </div>
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
      total_pages: 0,
      self_data: {},
    };
  },
  computed: {
    totalPages() {
      return this.total_pages;
    },
  },
  inject: ['api_url', 'access_token'],
  methods: {
    async fetchRankings() {
      try {
        const defaultParams = {
          suject: 'program',
          page: this.currentPage,
          num: 10,
        };

        const queryParams = new URLSearchParams(defaultParams)
        const response = await fetch(`${this.api_url}/onlinejudge/api/rank?${queryParams.toString()}`, {
          method: 'GET',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${this.access_token}`,
          },
        });
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        const response_data = await response.json();
        if(this.total_pages==0) this.total_pages = response_data.total_pages;
        this.self_data = response_data.self_data;
        this.rankings = response_data.data; // 假設 API 回傳的資料是包含排行榜用戶和分數的陣列
      } catch (error) {
        console.error("取得排行榜時發生錯誤：", error);
      }
    },
    changePage(page) {
      this.currentPage = page;
      this.fetchRankings();
    },
  },
  mounted() {
    this.total_pages=0;
    this.fetchRankings();
    
  },
};
</script>

<style scoped>
.rank-board-page {
  padding: 20px;
}

.rank-grid {
  display: grid;
  grid-template-columns: 1fr 2fr 1fr; /* 三個欄位 */
  gap: 10px;
  margin-top: 20px;
}

.rank-grid-header {
  font-weight: bold;
  display: contents;
}

.rank-grid-item {
  display: contents;
}

.rank_data {
  justify-content: left;
}

.pagination {
  display: flex;
  justify-content: center;
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
