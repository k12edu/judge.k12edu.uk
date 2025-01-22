<template>
  <div class="profile-container" style="min-height: 100vh;">
    <h1>個人檔案</h1>
    
    <div v-if="loading">載入中...</div>
    <div v-else-if="error" class="error-message">{{ error }}</div>
    <div v-else>
      <div class="profile-field">
        <label for="field1">欄位1：</label>
        <span>{{ profileData.field1 }}</span>
      </div>
      <div class="profile-field">
        <label for="field2">欄位2：</label>
        <span>{{ profileData.field2 }}</span>
      </div>
      <div class="profile-field">
        <label for="field3">欄位3：</label>
        <span>{{ profileData.field3 }}</span>
      </div>
      <div class="profile-field">
        <label for="field4">欄位4：</label>
        <span>{{ profileData.field4 }}</span>
      </div>
      <!-- 可以根據需求添加更多欄位 -->
    </div>
  </div>
</template>

<script>
export default {
  name: 'UserProfile',
  data() {
    return {
      profileData: {},   // 儲存從後端獲取的資料
      loading: true,     // 載入狀態
      error: null,       // 錯誤訊息
    };
  },
  methods: {
    async fetchProfileData() {
      const url = 'https://your-backend-url.com/api/user-profile'; // 替換為你的後端 API 路徑
      try {
        const response = await fetch(url);
        if (!response.ok) {
          throw new Error(`HTTP 錯誤：${response.status}`);
        }
        const data = await response.json();
        this.profileData = data; // 將接收到的資料賦值給 profileData
      } catch (err) {
        this.error = '無法取得個人資料，請稍後再試';
        console.error(err);
      } finally {
        this.loading = false;
      }
    },
  },
  mounted() {
    this.fetchProfileData(); // 組件掛載時請求資料
  },
};
</script>

<style scoped>
.profile-container {
  padding: 20px;
  background-color: #f5f5f5;
  border-radius: 8px;
  width: 60%;
  margin: 0 auto;
}

h1 {
  text-align: center;
}

.profile-field {
  margin: 10px 0;
  display: flex;
  justify-content: space-between;
}

label {
  font-weight: bold;
}

.error-message {
  color: red;
  text-align: center;
}
</style>
