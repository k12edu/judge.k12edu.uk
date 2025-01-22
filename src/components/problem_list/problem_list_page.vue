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
            />
            <span class="tag-text">{{ tag }}</span>
          </label>
        </div>
      </div>
  
      <!-- 顯示題目列表 -->
      <div v-if="loading" class="loading">載入中...</div>
      <div v-else>
        <div v-for="problem in filteredProblems" :key="problem.id" class="problem-item">
          <h3>{{ problem.title }}</h3>
          <p>{{ problem.description }}</p>
          <p><strong>標籤:</strong> {{ problem.tags.join(', ') }}</p>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        problems: [],  // 顯示的題目資料
        tags: ['圖論','動態規劃'],      // 標籤選項
        selectedTags: [],  // 用來存儲選中的標籤
        loading: false,
      };
    },
    computed: {
      // 根據選中的標籤過濾題目
      filteredProblems() {
        if (this.selectedTags.length === 0) {
          return this.problems;  // 如果沒有選擇標籤，顯示所有題目
        }
        return this.problems.filter(problem =>
          problem.tags.some(tag => this.selectedTags.includes(tag))
        );
      }
    },
    methods: {
      async fetchProblems() {
        this.loading = true;
        try {
          const response = await fetch('https://your-api.com/problems'); // 假設這是你的 API
          const data = await response.json();
          this.problems = data.problems;  // 假設回應資料包含一個問題列表
          this.tags = [...new Set(this.problems.flatMap(problem => problem.tags))];  // 提取所有標籤
        } catch (error) {
          console.error('獲取題目資料錯誤:', error);
        } finally {
          this.loading = false;
        }
      }
    },
    created() {
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
  </style>
  