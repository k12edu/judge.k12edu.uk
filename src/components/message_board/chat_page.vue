<template>
  <div class="problem-list-container">
    <div class="back-div" >
      <p class="back-button" @click="goBack">返回上一頁</p>
      <p class="back-button" @click="goToProblem">前往題目</p>
    </div>
    <h1 v-if="article" class="title">{{ article.title }}</h1>
    <div class="items-per-page" style="margin: 20px 0px;">
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
      <div class="problem-list">
        <div v-if="article" class="problem-item">
          <div class="problem-number">作者: {{ article.author }}</div>
          <div class="problem-title">題目: {{ article.problem_title }}</div>
          <div class="problem-title">標題: {{ article.title }}</div>
          <div class="problem-content">{{ article.content }}</div>
        </div>
        <div v-for="comment in comments" :key="comment.id" class="problem-item">
          <div class="problem-number">作者: {{ comment.author }}</div>
          <div class="problem-content">{{ comment.content }}</div>
        </div>
      </div>
    </div>

    <div class="pagination">
      <button @click="changePage(page)" :class="{ active: currentPage === page }" v-for="page in displayedPages" :key="page">
        {{ page }}
      </button>
    </div>

    <div class="new-comment">
      <h2>新增留言</h2>
      <input v-model="title" type="text" placeholder="輸入標題" class="input" />
      <textarea v-model="content" placeholder="輸入內容" class="textarea"></textarea>
      <button @click="submitComment" class="submit-btn">發布</button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      comments: [],
      article: {},
      loading: false,
      currentPage: 1,
      total_page: 0,
      per_page: 10,
      title: '',
      content: ''
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
    goToProblem() {
        this.$router.push({ path: `/problem`, query: { problemId: this.article.problem_id } });
      },
    goBack() {
      this.$router.go(-1);
    },
    async submitComment() {
      if (!this.title || !this.content) {
        alert('標題與內容不能為空');
        return;
      }
      try {
        let data = {
          'article_id': this.$route.query.articleId,
          'content': this.content,
        };
        const response = await fetch(`${this.api_url}/onlinejudge/comment/create/`, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${this.access_token}`,
          },
          body: JSON.stringify(data),  
        });
        const res = await response.json();
        console.log(res);
        alert('留言發佈成功！');
        this.fetchArticles();
        this.title = '';
        this.content = '';
      } catch (error) {
        console.error('發佈失敗', error);
        alert('發佈失敗，請稍後再試');
      }
    },
    changeItemPerPage() {
      this.total_page = 0;
      this.changePage(1);
    },
    changePage(page) {
      this.currentPage = page;
      this.fetchArticles();
    },
    async fetchArticles() {
      this.loading = true;
      try {
        const queryParams = new URLSearchParams({
          page: this.currentPage,
          per_page: this.per_page,
        });
        const response = await fetch(`${this.api_url}/onlinejudge/article/${this.$route.query.articleId}/comments/?${queryParams.toString()}`, {
          method: 'GET',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${this.access_token}`,
          },
        });
        const data = await response.json();
        this.total_page = data.total_page;
        this.comments = data.comments;
        this.article = data.article;
      } catch (error) {
        console.error('獲取資料錯誤:', error);
      } finally {
        this.loading = false;
      }
    },
  },
  mounted() {
    this.fetchArticles();
  }
};
</script>

<style scoped>
.back-div{
  display: flex;
  width: 100%;
  justify-content: space-between;
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
.new-comment {
  max-width: 600px;
  margin: 20px auto;
  padding: 20px 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
  background: #f9f9f9;
}
.input, .textarea {
  width: 80%;
  padding: 10px;
  margin: 20px 0;
  border: 1px solid #ccc;
  border-radius: 5px;
}
.textarea {
  margin: 20px 0;
  height: 150px;
  resize: none;
}
.submit-btn {
  width: 100%;
  padding: 10px;
  background: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}
.submit-btn:hover {
  background: #0056b3;
}

.problem-list-container {
  padding: 20px;
  width: 80%;
  margin: 0 auto;
}

.problem-list {
  display: flex;
  flex-direction: column;
  gap: 20px;
  width: 100%;
}

.problem-item {
  border: 1px solid #ccc;
  padding: 15px;
  border-radius: 5px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  text-align: left;
}

.problem-content {
  background: #f1f1f1;
  padding: 10px;
  border-radius: 5px;
}
</style>
