<template>
    <div class="problem-list-container" style="min-height: 100vh;">
      <h1>題目列表</h1>
  
      <!-- 顯示多選標籤 -->
      <div class="tag-filter">
        <label for="tags">選擇標籤:</label>
        <div class="tags">
          <label v-for="tag in tags" :key="tag" class="tag-option">
            <input
              type="checkbox"
              :value="tag"
              v-model="selectedTags"
              class="checkbox-input"
              @change="fetchProblems"
            />
            <span class="tag-text">{{ tag }}</span>
          </label>
        </div>
      </div>
  
      <!-- 顯示題目列表 -->
      <div v-if="loading" class="loading">載入中...</div>
      <div v-else>
        <div v-for="problem in problems" :key="problem.id" class="problem-item">
          <h3>{{ problem.title }}</h3>
          <p>{{ problem.is_solved }}</p>
          <p><strong>標籤:</strong> {{ problem.tag.join(', ') }}</p>
        </div>
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
        problems: [],  // 顯示的題目資料
        tags: [],      // 標籤選項
        selectedTags: [],  // 用來存儲選中的標籤
        loading: false,
        currentPage: 1,
        total_page: 0,
        per_page:10,
      };
    },
    computed: {
      totalPages() {
        return this.total_page;
      },

    },
    inject: ['api_url', 'access_token'],
    methods: {
      changePage(page) {
        this.currentPage = page;
        this.fetchProblems();
      },
      async fetchProblems() {
        this.loading = true;
        try {
          let query_list="";
          if(this.selectedTags.length>0){
            query_list=this.selectedTags[0];
            for(let i=1;i<this.selectedTags.length;i++){
              query_list+=`,${this.selectedTags[i]}`;
            }
          }
          else{
            query_list=null;
          }
          const defaultParams = {
            page: this.currentPage,
            per_page: this.per_page,
            query_tag: query_list,
          };

          const queryParams = new URLSearchParams(defaultParams)
          const response = await fetch(`${this.api_url}/onlinejudge/api/problem/list?${queryParams.toString()}`, {
            method: 'GET',
            headers: {
              'Content-Type': 'application/json',
              'Authorization': `Bearer ${this.access_token}`,
            },
          });
          const data = await response.json();
          if(this.total_page==0) this.total_page = data.total_page;
          this.problems = data.problems;  // 假設回應資料包含一個問題列表
        } catch (error) {
          console.error('獲取題目資料錯誤:', error);
        } finally {
          this.loading = false;
        }
      },
      async fetchTags() {
        this.loading = true;
        try {
          const defaultParams = {};

          const queryParams = new URLSearchParams(defaultParams)
          const response = await fetch(`${this.api_url}/onlinejudge/api/tags/list?${queryParams.toString()}`, {
            method: 'GET',
            headers: {
              'Content-Type': 'application/json',
              'Authorization': `Bearer ${this.access_token}`,
            },
          });
          const data = await response.json();
          this.tags = data.tags;  // 假設回應資料包含一個問題列表
        } catch (error) {
          console.error('獲取tag資料錯誤:', error);
        }
      }
    },
    mounted() {
      if(this.tags.length==0){
        this.fetchTags();
      }
      this.fetchProblems();  // 在組件創建時獲取題目資料
    }
  };
  </script>
  
  <style scoped>
  .problem-list-container {
    padding: 20px;
  }
  
  .tag-filter {
    margin-bottom: 20px;
  }
  
  .tags {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
  }
  
  .tag-option {
    display: flex;
    align-items: center;
    padding: 5px 10px;
    border: 2px solid #ffd06b;
    border-radius: 5px;
    cursor: pointer;
    transition: background-color 0.3s, border-color 0.3s;
  }
  
  .tag-option:hover {
    background-color: #ff9838;
    border-color: #ff9838;
  }
  
  .checkbox-input {
    margin-right: 10px;
  }
  
  .tag-text {
    font-weight: bold;
    color: #333;
  }
  
  .problem-item {
    border: 1px solid #ccc;
    margin: 10px 0;
    padding: 10px;
  }
  
  .loading {
    text-align: center;
    font-size: 18px;
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
  