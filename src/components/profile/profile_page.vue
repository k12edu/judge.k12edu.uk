<template>
  <div class="profile-container">
    <div class="top-bar">
      <p class="back-button" @click="goBack">返回上一頁</p>
      <p v-if="this.$route.query.id==0 && isLogin" class="logout-button" @click="handleLogout">登出</p>
    </div>

    <h1>個人檔案</h1>

    <div v-if="loading">載入中...</div>
    <div v-else-if="error" class="error-message">{{ error }}</div>
    <div v-else>
      <div class="profile-field">
        <label>ID：</label>
        <span>{{ profileData.id }}</span>
      </div>
      <div class="profile-field">
        <label>使用者名稱：</label>
        <span>{{ profileData.user_name }}</span>
      </div>
      <div class="profile-field">
        <label>Email：</label>
        <span>{{ profileData.email }}</span>
      </div>
      <div class="profile-field">
        <label>已解程式題數：</label>
        <span>{{ profileData.num_of_solved_program_problem }}</span>
      </div>
      <div class="profile-field">
        <label>已解自然題數：</label>
        <span>{{ profileData.num_of_solved_natural_problem }}</span>
      </div>
      <div class="profile-field">
        <label>已解數學題數：</label>
        <span>{{ profileData.num_of_solved_math_problem }}</span>
      </div>
      <div class="profile-field">
        <label>註冊時間：</label>
        <span>{{ formatDate(profileData.create_time) }}</span>
      </div>
    </div>
    <div v-if="this.$route.query.id==0" class="top-bar">
      <p v-if="isLogin" class="logout-button" @click="goToHistory">提交紀錄</p>
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
  inject: ['api_url', 'access_token', 'logout', 'isLogin'],
  methods: {
    goBack() {
      this.$router.go(-1);
    },
    goToHistory(){
      this.$router.push({ path: `/submission-history`, });
    },
    async fetchProfileData() {
      try {
        const userId = this.$route.query.id;
        const response = await fetch(`${this.api_url}/onlinejudge/api/user/${userId}`, {
          method: 'GET',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${this.access_token}`,
          },
        });

        if (!response.ok) {
          throw new Error('無法獲取個人資料');
        }

        const data = await response.json();
        this.profileData = data; // 更新 profileData
      } catch (err) {
        this.error = '無法取得個人資料，請稍後再試';
        console.error(err);
      } finally {
        this.loading = false;
      }
    },
    formatDate(timestamp) {
      if (!timestamp) return "無日期資訊";
      const date = new Date(timestamp);
      return date.toLocaleDateString();
    },
    handleLogout() {
      this.logout(); // 調用提供的登出方法
    }
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
  margin: 20px auto;
  min-height: 100vh;
  border-image-source: url('@/assets/tile_0000.png');
  border-width: 10px;
  border-style: solid;
  border-image-slice: 6 fill;
  border-image-repeat: repeat;
}

.top-bar {
  display: flex;
  justify-content: space-between;
  padding: 10px 20px;
}

.back-button, .logout-button {
  padding: 6px 12px;
  background: #f0f0f0;
  cursor: pointer;
  border-radius: 6px;
  transition: background 0.3s;
  border-image-source: url('@/assets/tile_0000.png');
  border-width: 10px;
  border-style: solid;
  border-image-slice: 6 fill;
  border-image-repeat: repeat;
}

.back-button:hover, .logout-button:hover {
  background: #ddd;
}

h1 {
  text-align: center;
}

.profile-field {
  margin: 10px 0;
  display: flex;
  justify-content: space-between;
  border-bottom: 1px solid #ccc;
  padding-bottom: 5px;
}

label {
  font-weight: bold;
}

.error-message {
  color: red;
  text-align: center;
}
</style>
