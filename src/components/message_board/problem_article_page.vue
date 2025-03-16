<template>
<div class="problem-list-container" style="min-height: 100vh;">
	<div class="back-div" >
      <p class="back-button" @click="goBack">返回上一頁</p>
    </div>
	<h1>討論區</h1>
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

<!-- 顯示題目列表 -->
	<div v-if="loading" class="loading">載入中...</div>
	<div v-else>
		<div class="problem-grid">
			<div v-for="article in articles" :key="article.id" class="problem-item" @click="goToArticle(article.id)">
				<div class="problem-title">主題: {{ article.title }}</div>
				<div class="problem-difficulty">留言數: {{ article.comment_count }}</div>
			</div>
		</div>
	</div>

<!-- 分頁 -->
	<div class="pagination">
		<button @click="changePage(page)" class="btn" :class="{ active: currentPage === page }" v-for="page in displayedPages" :key="page">
		{{ page }}
		</button>
	</div>
	<div class="new-article">
      <h2>新增文章</h2>
      <input v-model="title" type="text" placeholder="輸入標題" class="input" />
      <textarea v-model="content" placeholder="輸入內容" class="textarea"></textarea>
      <button @click="submitArticle" class="submit-btn">發布</button>
    </div>
  </div>
</template>

<script>
export default {
data() {
	return {
		articles: [],  // 顯示的題目資料
		loading: false,
		currentPage: 1,
		total_page: 0,
		per_page:10,
		title: '',
		content: ''
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
	async submitArticle() {
      if (!this.title || !this.content) {
        alert('標題與內容不能為空');
        return;
      }
      try {
        let data={
          'problem_id': this.$route.query.problemId,
          'title': this.title,
          'content': this.content,
        };
        const response = await fetch(`${this.api_url}/onlinejudge/article/create/`, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${this.access_token}`,
          },
          body: JSON.stringify(data),  
        });
        const res = response.json()
        console.log(res);
        alert('文章發佈成功！');
        this.fetchArticles();
        this.title = '';
        this.content = '';
      } catch (error) {
        console.error('發佈失敗', error);
        alert('發佈失敗，請稍後再試');
      }
    },
	changeItemPerPage(){
		this.total_page=0;
		this.changePage(1);
	},
	goToArticle(articleId) {
		this.$router.push({ path: `/forum/disscussion`, query: { articleId: articleId } });
	},
	changePage(page) {
		this.currentPage = page;
		this.fetchArticles();
	},
	async fetchArticles() {
		this.loading = true;
		try {
			const defaultParams = {
				page: this.currentPage,
				per_page: this.per_page,
			};
			const queryParams = new URLSearchParams(defaultParams)
			const response = await fetch(`${this.api_url}/onlinejudge/problem/${this.$route.query.problemId}/articles/?${queryParams.toString()}`, {
				method: 'GET',
				headers: {
					'Content-Type': 'application/json',
					'Authorization': `Bearer ${this.access_token}`,
				},
				});
			const data = await response.json();
			if(this.total_page!=data.total_page) this.total_page = data.total_page;
			this.articles = data.articles;  // 假設回應資料包含一個問題列表
		} catch (error) {
			console.error('獲取題目資料錯誤:', error);
		} finally {
			this.loading = false;	
		}
	},
},
mounted() {
this.fetchArticles();  // 在組件創建時獲取題目資料
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
  border-image-source: url('@/assets/tile_0000.png');
  border-width: 10px;
  border-style: solid;
  border-image-slice: 6 fill;
  border-image-repeat: repeat;
}
.back-button:hover {
  background: #ddd;
}
.new-article {
  max-width: 600px;
  margin: 20px auto;
  padding: 20px 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
  background: #f9f9f9;
  border-image-source: url('@/assets/tile_0000.png');
  border-width: 10px;
  border-style: solid;
  border-image-slice: 6 fill;
  border-image-repeat: repeat;
}
.input, .textarea {
  width: 80%;
  padding: 10px;
  margin: 20px 0;
  border: 1px solid #ccc;
  border-radius: 5px;
  border-image-source: url('@/assets/tile_0000.png');
  border-width: 10px;
  border-style: solid;
  border-image-slice: 6 fill;
  border-image-repeat: repeat;
}
.textarea {
  margin: 20px 0;
  height: 150px;
  resize: none;
  border-image-source: url('@/assets/tile_0000.png');
  border-width: 10px;
  border-style: solid;
  border-image-slice: 6 fill;
  border-image-repeat: repeat;
}
.submit-btn {
  width: 100%;
  padding: 10px;
  background: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  border-image-source: url('@/assets/tile_0003.png');
  border-width: 10px;
  border-style: solid;
  border-image-slice: 6 fill;
  border-image-repeat: repeat;
}

.problem-list-container {
	padding: 20px;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 70%;
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

.problem-grid {
display: grid;
grid-template-columns: repeat(auto-fill, minmax(500px, 1fr));
gap: 20px;
margin-top: 20px;
width: 100%;
}

.problem-item {
border: 1px solid #ccc;
padding: 15px;
border-radius: 5px;
box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
cursor: pointer;
transition: transform 0.3s, box-shadow 0.3s;
border-image-source: url('@/assets/tile_0000.png');
  border-width: 10px;
  border-style: solid;
  border-image-slice: 6 fill;
  border-image-repeat: repeat;
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
button.active {
  font-weight: 700;
  border-image-source: url('@/assets/tile_0003.png');
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