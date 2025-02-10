<template>
    <div class="problem-list-container">
      <div class="back-div">
        <p class="back-button" @click="goBack">返回上一頁</p>
      </div>
      <h1>提交紀錄</h1>
      
      <div class="items-per-page" style="margin-bottom: 20px;">
        <label for="perPage">每頁顯示數量:</label>
        <input
          type="number"
          id="perPage"
          v-model="per_page"
          min="1"
          @input="changeItemPerPage"
          placeholder="輸入每頁顯示的資料筆數"
        />
      </div>
  
      <div v-if="loading" class="loading">載入中...</div>
      <div v-else>
        <div class="submission-grid">
          <div class="grid-header">題號</div>
          <div class="grid-header">題目</div>
          <div class="grid-header">上傳時間</div>
          <div class="grid-header">上傳結果</div>
          
          <template v-for="history in histories" :key="history.submission_id">
            <div>{{ history.problem_id }}</div>
            <div @click="goToProblem(history.problem_id)" class="clickable">{{ history.problem_title }}</div>
            <div>{{ history.submit_time }}</div>
            <div :class="{'success': history.submit_result === 'Accepted', 'failed': history.submit_result !== 'Accepted'}">
              {{ history.submit_result }}
            </div>
          </template>
        </div>
      </div>
  
      <div class="pagination">
        <button @click="changePage(page)" :class="{ active: currentPage === page }" v-for="page in displayedPages" :key="page">
          {{ page }}
        </button>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        histories: [],
        loading: false,
        currentPage: 1,
        total_page: 0,
        per_page: 10
      };
    },
    computed: {
      displayedPages() {
        const pagesToShow = 10;
        let startPage = Math.max(1, this.currentPage - Math.floor(pagesToShow / 2));
        let endPage = Math.min(this.totalPages, startPage + pagesToShow - 1);
        if (endPage - startPage + 1 < pagesToShow) {
          startPage = Math.max(1, endPage - pagesToShow + 1);
        }
        return Array.from({ length: endPage - startPage + 1 }, (_, i) => startPage + i);
      },
      totalPages() {
        return this.total_page;
      }
    },
    inject: ['api_url', 'access_token'],
    methods: {
      goToProblem(problemId) {
        this.$router.push({ path: `/problem`, query: { problemId: problemId } });
      },
      goBack() {
        this.$router.go(-1);
      },
      changeItemPerPage() {
        this.total_page = 0;
        this.changePage(1);
      },
      changePage(page) {
        this.currentPage = page;
        this.fetchHistories();
      },
      async fetchHistories() {
        this.loading = true;
        try {
          const queryParams = new URLSearchParams({
            page: this.currentPage,
            per_page: this.per_page,
          });
          const response = await fetch(`${this.api_url}/onlinejudge/submissions/?${queryParams.toString()}`, {
            method: 'GET',
            headers: {
              'Content-Type': 'application/json',
              'Authorization': `Bearer ${this.access_token}`,
            },
          });
          const data = await response.json();
          this.total_page = data.total_page;
          this.histories = data.data;
        } catch (error) {
          console.error('獲取資料錯誤:', error);
        } finally {
          this.loading = false;
        }
      },
    },
    mounted() {
      this.fetchHistories();
    }
  };
  </script>
  
  <style scoped>
  .problem-list-container {
    padding: 20px;
    width: 80%;
    margin: 0 auto;
  }
  
  .submission-grid {
    display: grid;
    grid-template-columns: 1fr 2fr 2fr 1fr;
    gap: 10px;
    width: 100%;
    text-align: center;
    align-items: center;
  }
  
  .grid-header {
    font-weight: bold;
    background: #f1f1f1;
    padding: 10px;
    border: 1px solid #ccc;
  }
  
  .clickable {
    color: blue;
    text-decoration: underline;
    cursor: pointer;
  }
  
  .success {
    color: green;
  }
  
  .failed {
    color: red;
  }
  
  .back-div {
    display: flex;
    width: 100%;
    justify-content: space-between;
    padding: 10px 20px;
  }
  
  .back-button {
    width: 100px;
    padding: 6px;
    background: #f0f0f0;
    cursor: pointer;
    border-radius: 6px;
  }
  
  .back-button:hover {
    background: #ddd;
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
  