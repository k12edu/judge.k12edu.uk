<template>
    <div class="discussion-container" style="min-height: 100vh;">
      <h1>{{ topicTitle }}</h1>
  
      <!-- 顯示留言 -->
      <div v-if="loading" class="loading">載入中...</div>
      <div v-else>
        <div v-if="comments.length > 0" class="comments-section">
          <h2>留言區</h2>
          <div v-for="comment in comments" :key="comment.id" class="comment-item">
            <p><strong>{{ comment.author }}:</strong> {{ comment.content }}</p>
            <p class="comment-time">{{ formatDate(comment.timestamp) }}</p>
          </div>
        </div>
        <div v-else class="no-comments">目前還沒有留言，快來留下第一則吧！</div>
      </div>
  
      <!-- 留言輸入框 -->
      <div class="comment-input-section">
        <h2>新增留言</h2>
        <textarea
          v-model="newComment"
          placeholder="輸入您的留言..."
          rows="4"
          class="comment-input"
        ></textarea>
        <button @click="postComment" :disabled="posting" class="comment-button">
          {{ posting ? "發送中..." : "發送留言" }}
        </button>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        topicTitle: "討論主題", // 主題標題，可以從父組件或 API 接收
        comments: [], // 用於存放留言的陣列
        newComment: "", // 新留言內容
        loading: false, // 是否正在載入留言
        posting: false, // 是否正在發送留言
      };
    },
    methods: {
      // 獲取留言列表
      async fetchComments() {
        this.loading = true;
        try {
          const response = await fetch("https://your-api.com/discussion/comments"); // 替換為你的 API
          const data = await response.json();
          this.comments = data.comments; // 假設 API 回傳 { comments: [...] }
        } catch (error) {
          console.error("獲取留言失敗：", error);
        } finally {
          this.loading = false;
        }
      },
      // 發送留言
      async postComment() {
        if (!this.newComment.trim()) {
          alert("留言內容不能為空！");
          return;
        }
  
        this.posting = true;
        try {
          const response = await fetch("https://your-api.com/discussion/comments", {
            method: "POST",
            headers: {
              "Content-Type": "application/json",
            },
            body: JSON.stringify({
              content: this.newComment,
            }),
          });
  
          if (response.ok) {
            const newCommentData = await response.json();
            this.comments.push(newCommentData); // 將新留言新增至列表
            this.newComment = ""; // 清空輸入框
          } else {
            console.error("留言發送失敗：", response.statusText);
          }
        } catch (error) {
          console.error("發送留言時發生錯誤：", error);
        } finally {
          this.posting = false;
        }
      },
      // 格式化時間戳
      formatDate(timestamp) {
        const date = new Date(timestamp);
        return date.toLocaleString();
      },
    },
    created() {
      this.fetchComments(); // 組件創建時獲取留言
    },
  };
  </script>
  
  <style scoped>
  .discussion-container {
    padding: 20px;
    max-width: 800px;
    margin: 0 auto;
  }
  
  h1 {
    text-align: center;
    margin-bottom: 20px;
  }
  
  .comments-section {
    margin-top: 20px;
  }
  
  .comment-item {
    border-bottom: 1px solid #ddd;
    padding: 10px 0;
  }
  
  .comment-item:last-child {
    border-bottom: none;
  }
  
  .comment-time {
    font-size: 0.8rem;
    color: gray;
  }
  
  .no-comments {
    text-align: center;
    color: #666;
    margin-top: 20px;
  }
  
  .comment-input-section {
    margin-top: 30px;
  }
  
  .comment-input {
    width: 100%;
    padding: 10px;
    margin-bottom: 10px;
    border: 1px solid #ddd;
    border-radius: 5px;
    font-size: 1rem;
    resize: none;
  }
  
  .comment-button {
    padding: 10px 20px;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 5px;
    font-size: 1rem;
    cursor: pointer;
  }
  
  .comment-button:disabled {
    background-color: #ccc;
    cursor: not-allowed;
  }
  
  .comment-button:hover:not(:disabled) {
    background-color: #0056b3;
  }
  </style>
  