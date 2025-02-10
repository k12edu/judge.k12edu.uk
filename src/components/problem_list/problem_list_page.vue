<template>
  <div class="problem-list-container" style="min-height: 100vh;">
    <div class="back-div" >
      <p class="back-button" @click="goBack">返回上一頁</p>
    </div>
    <h1>題目列表</h1>
    <div class="items-per-page">
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
    <!-- 顯示多選標籤 -->
    <div class="tag-filter">
      <label for="tags">選擇標籤:</label style="margin-botton: 20px;">
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
      <div class="problem-grid">
        <div v-for="problem in problems" :key="problem.problem_id" class="problem-item" @click="goToProblem(problem.problem_id)">
          <div class="problem-number">#{{ problem.problem_id }}</div>
          <div class="problem-title">{{ problem.title }}</div>
          <div class="problem-difficulty">{{ problem.difficulty }}</div>
          <div class="problem-status">
            <span :class="{'completed': problem.is_solved, 'not-completed': !problem.is_solved}">
              {{ problem.is_solved ? '已完成' : '未完成' }}
            </span>
          </div>
        </div>
      </div>
    </div>

    <!-- 分頁 -->
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
        return this.total_page;
      },

    },
    inject: ['api_url', 'access_token'],
    methods: {
      goBack() {
        this.$router.go(-1);
      },
      changeItemPerPage(){
        this.total_page=0;
        this.changePage(1);
      },
      goToProblem(problemId) {
        this.$router.push({ path: `/problem`, query: { problemId: problemId } });
      },
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
          if(this.total_page!=data.total_page) this.total_page = data.total_page;
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
  .back-div{
  display: flex;
  width: 100%;
  justify-content: left;
  padding: 10px 20px;
}
.back-button {
  width: 100px;
  padding: 6px 6px;
  background: #f0f0f0;
  cursor: pointer;
  border: 0px solid #ccc;
  border-radius: 6px;
}
.back-button:hover {
  background: #ddd;
}
.problem-list-container {
  padding: 20px;
  width: 80%;
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
  border: 2px solid lightblue;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s, border-color 0.3s;
}

.tag-option:hover {
  background-color: lightblue;
  border-color: lightblue;
}

.checkbox-input {
  margin-right: 10px;
}

.tag-text {
  font-weight: bold;
  color: #333;
}

.problem-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 20px;
  margin-top: 20px;
}

.problem-item {
  border: 1px solid #ccc;
  padding: 15px;
  border-radius: 5px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  cursor: pointer;
  transition: transform 0.3s, box-shadow 0.3s;
}

.problem-item:hover {
  transform: translateY(-5px);
  box-shadow: 0 5px 10px rgba(0, 0, 0, 0.2);
}

.problem-number {
  font-size: 18px;
  font-weight: bold;
}

.problem-title {
  font-size: 16px;
  font-weight: 600;
  margin-top: 5px;
}

.problem-difficulty {
  font-size: 14px;
  color: #888;
  margin-top: 5px;
}

.problem-status {
  margin-top: 10px;
}

.problem-status .completed {
  color: green;
  font-weight: bold;
}

.problem-status .not-completed {
  color: red;
  font-weight: bold;
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
.items-per-page {
  margin-top: 20px;
  margin-bottom: 20px;
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
