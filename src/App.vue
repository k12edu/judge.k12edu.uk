<template>
  <div class="v-container">
    <div class="menu-container" style="height: 70px;">
      <router-link style="width: 120px;" to="/" class="menu-item">OnlineJudge</router-link>
      <nav class="menu">
        <router-link to="/announcement" class="menu-item">公告</router-link>
        <router-link to="/problem_list" class="menu-item">題目</router-link>
        <router-link to="/forum" class="menu-item">討論區</router-link>
        <router-link to="/rank" class="menu-item">排行榜</router-link>
      </nav>
      <div style="width: 150px;" v-if="!isLogin" id="google-signin-button" @click="googleLogin"></div>
      <router-link style="width: 150px;" v-if="isLogin" to="/profile" class="menu-item">個人資料</router-link>
    </div>
    <router-view></router-view>
    <footer class="footer">
      <div class="footer-container">
        <div>
          <p>© 2025 Your Company. All Rights Reserved.</p>
        </div>
      </div>
    </footer>
  </div>
</template>
<script>
  export default {
    data() {
      return {
        isLogin: false,
        googleClientId: '63473080805-na5r3r5d4m3ibnk1f7kvjgp7n1grnaoe.apps.googleusercontent.com', // 替換為你的 Google OAuth 客戶端 ID
        access_token: '',
      };
    },
    mounted() {
      const currentDomain = window.location.hostname;
      console.log(currentDomain);
      if(currentDomain=='localhost'){
        this.api_url='http://127.0.0.1:60000';
        this.r_url='http://localhost:8080/';
        console.log('test1');
      }
      else{
        this.api_url='https://api.k12edu.us.kg';
        this.r_url='https://teacher.k12edu.us.kg/';
        console.log('test2');
      }
      this.loadGoogleScript();
    },
    methods: {
      // 動態加載 Google API 客戶端腳本
      loadGoogleScript() {
        const script = document.createElement('script');
        script.src = 'https://accounts.google.com/gsi/client';
        script.async = true;
        script.defer = true;
        script.onload = this.initializeGoogleSignIn; // 當腳本加載完成時初始化
        document.head.appendChild(script);
      },
      // 初始化 Google Sign-In
      initializeGoogleSignIn() {
        // 使用 window.google 確保全域變數
        if (typeof window.google === 'undefined') {
          console.error('Google API 未正確加載');
          return;
        }

        window.google.accounts.id.initialize({
          client_id: this.googleClientId,
          scope: 'openid https://www.googleapis.com/auth/userinfo.email https://www.googleapis.com/auth/userinfo.profile',
          callback: this.handleCredentialResponse, // 當用戶成功登入時的回調函數
        });
        window.google.accounts.id.renderButton(
          document.getElementById('google-signin-button'), // 指定渲染按鈕的位置
          {
            theme: 'outline', // 按鈕樣式，可選 "filled_blue" 或 "outline"
            size: 'large',    // 按鈕大小，可選 "small", "medium", "large"
          }
        );
      },
      // 處理登入回應
      handleCredentialResponse(response) {
        console.log('Google Sign-In Credential Response:', response);
        // 解析 JWT 資料（如需要）
        const token = response.credential;
        this.sendIdTokenToBackend(token);
      },
      sendIdTokenToBackend(idToken) {
          fetch(`${this.api_url}/accounts/api/google-login/`, {
              method: 'POST',
              headers: {
                  'Content-Type': 'application/json',
              },
              body: JSON.stringify({ id_token: idToken }), // 修改成傳 id_token
          })
          .then(response => response.json())
          .then(data => {
              // 處理 Django 回傳的 JWT
              if (data.access) {
                  this.access_token = data.access;
                  this.isLogin = true;
                  localStorage.setItem('jwt', data.access);
                  localStorage.setItem('refresh', data.refresh);
                  console.log('JWT token received and stored:', data);
              } else {
                  console.error('JWT not received:', data);
              }
          })
          .catch(error => console.error('Error sending id_token to backend:', error));
      },
    },
  };



</script>
<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
.v-container{
  display: flex;
  flex-direction: column;
  align-items: center;
}

.h-comtainer{
  display: flex;
  flex-direction: row;
  justify-content: center;
}
</style>

<style scoped>
  .menu-container {
    display: flex;
    justify-content: space-between; /* 左右對齊 */
    align-items: center; /* 垂直居中 */
    background-color: #f4f4f4;
    margin: 0 auto; /* 自動分配外邊距，讓內容居中 */
    padding: 0 20px; /* 左右邊距，讓內容不緊貼邊界 */
    max-width: 100%; /* 最大寬度，限制內容的總寬度 */
    width: 100%; /* 確保在小屏幕時撐滿 */
    box-sizing: border-box; /* 包含內邊距 */
  }


/* 菜單容器 */
  .menu {
    display: flex;
    gap: 20px; /* 每個按鈕之間的間距 */
  }

  /* 菜單項目 (按鈕樣式) */
  .menu-item {
    text-decoration: none; /* 移除超連結預設的下劃線 */
    padding: 10px 20px;
    border: 2px solid #ffd06b;
    border-radius: 5px;
    color: #000000;
    font-weight: bold;
    background-color: white;
    transition: background-color 0.3s, color 0.3s, box-shadow 0.3s;
    text-align: center;
    user-select: none; 
    cursor: pointer;
  }

  .menu-item:hover {
    background-color: #ff9838;
    color: white;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  }

  .menu-item:active {
    transform: scale(0.95); /* 點擊時的縮放效果 */
  }
  .footer {
  background-color: #333;
  color: white;
  padding: 20px 10px;
  position: relative;
  bottom: 0;
  width: 100%;
}

.footer-container {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-wrap: wrap; /* 讓內容在小螢幕上自動換行 */
  margin: 0 auto;
}
</style>
