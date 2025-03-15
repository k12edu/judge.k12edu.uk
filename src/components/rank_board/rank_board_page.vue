<template>
  <div class="rank-board-page" style="min-height: 100vh;">
    <div class="back-div" >
      <p class="back-button" @click="goBack">返回上一頁</p>
    </div>
    <h1>排行榜</h1>
    <div class="items-per-page">
      <label for="perPage">每頁顯示數量:</label>
      <input
        type="number"
        id="perPage"
        v-model="pageSize"
        min="1"
        @input="changeItemPerPage"
        placeholder="輸入每頁顯示的資料筆數"
      />
    </div>
    <div v-if="rankings.length">
      <div class="rank-grid">
        <div class="rank-grid-header">
          <p>排名</p>
          <p>用戶名稱</p>
          <p>解題數量</p>
        </div>
        <div class="rank-grid-item" v-for="(ranking) in rankings" :key="ranking.id">
          <p class="rank_data">{{ ranking.rank }}</p>
          <p class="rank_data link" @click="goToProfile(ranking.id)">{{ ranking.user_name }}</p>
          <p class="rank_data">{{ ranking.num_of_solved_program_problem }}</p>
        </div>
      </div>
    </div>
    <div v-else>
      <p>目前沒有排名資料。</p>
    </div>
    <div class="pagination">
      <button @click="changePage(page)" class="btn" :class="{ active: currentPage === page }" v-for="page in displayedPages" :key="page">
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
    displayedPages() {
      const pagesToShow = 10; // 最多顯示 10 頁
      let startPage = Math.max(1, this.currentPage - Math.floor(pagesToShow / 2));
      let endPage = Math.min(this.totalPages, startPage + pagesToShow - 1);

      // 確保頁數範圍合法，並且不會超過總頁數
      if (endPage - startPage + 1 < pagesToShow) {
        startPage = Math.max(1, endPage - pagesToShow + 1);
      }

      // 生成顯示的頁數範圍
      let pages = [];
      for (let i = startPage; i <= endPage; i++) {
        pages.push(i);
      }
      return pages;
    },
    totalPages() {
      return this.total_pages;
    },
  },
  inject: ['api_url', 'access_token'],
  methods: {
    goBack() {
      this.$router.go(-1);
    },
    goToProfile(userId) {
        this.$router.push({ path: `/profile`, query: { id: userId } });
      },
    changeItemPerPage(){
        this.total_pages=0;
        this.changePage(1);
      },
    async fetchRankings() {
      try {
        const defaultParams = {
          suject: 'program',
          page: this.currentPage,
          num: this.pageSize,
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
        if(this.total_pages!=response_data.total_pages) this.total_pages = response_data.total_pages;
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
.back-div{
  display: flex;
  width: 100%;
  justify-content: left;
  padding: 10px 20px;
}
.back-button {
  width: 100px;
  background: #f0f0f0;
  padding: 6px 6px;
  cursor: pointer;
  border: 0px solid #ccc;
  border-radius: 6px;
  border-image-source: url('@/assets/tile_0000.png');
  border-width: 10px;
  border-style: solid;
  border-image-slice: 6 fill;
  border-image-repeat: repeat;
}
.back-button:hover {
  background: #ddd;
}
button.active {
  font-weight: 700;
  border-image-source: url('@/assets/tile_0003.png');
}
p{
  margin: 0px 0px;
}
.link{
  padding: 10px 5px;
  border-radius: 10px;
  cursor: pointer;
}
.link:hover{
  background-color: lightgray;
}
.rank-board-page {
  padding: 20px;
}

.rank-grid {
  display: grid;
  grid-template-columns: 1fr 2fr 1fr; /* 三個欄位 */
  gap: 10px;
  margin: 20px 0;
  border-image-source: url('@/assets/tile_0000.png');
  border-width: 10px;
  border-style: solid;
  border-image-slice: 6 fill;
  border-image-repeat: repeat;
}

.rank-grid-header {
  font-weight: bold;
  display: contents;
}

.rank-grid-item {
  display: contents;
}

.rank_data {
  justify-content: center;
  align-content: center;
}

.pagination {
  display: flex;
  justify-content: center;
  gap: 5px;
  margin-top: 0px;
}

.btn{
  border-image-source: url('@/assets/tile_0002.png');
  border-width: 10px;
  border-style: solid;
  border-image-slice: 6 fill;
  border-image-repeat: repeat;
  color: white;
}
button {
  padding: 10px 20px;
  border: none;
  background-color: #eee;
  cursor: pointer;
  border-radius: 5px;
  transition: background-color 0.3s;
}

button:hover {
  background-color: #ddd;
}

.items-per-page {
  margin-top: 20px;
  margin-bottom: 20px;
  border-image-source: url('@/assets/tile_0000.png');
  border-width: 10px;
  border-style: solid;
  border-image-slice: 6 fill;
  border-image-repeat: repeat;
}

.items-per-page input {
  padding: 5px 10px;
  font-size: 16px;
  width: 100px;
  margin-left: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
}
</style>
