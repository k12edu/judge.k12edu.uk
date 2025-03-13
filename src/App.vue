<template>
  <div class="v-container" style="max-width: 100vw;">
    <div class="menu-container" style="height: 70px;">
      <router-link style="width: 120px;" to="/" class="menu-item">OnlineJudge</router-link>
      <nav class="menu">
        <router-link to="/announcement" class="menu-item">公告</router-link>
        <router-link to="/problem_list" class="menu-item">題目</router-link>
        <router-link to="/forum" class="menu-item">討論區</router-link>
        <router-link to="/rank" class="menu-item">排行榜</router-link>
      </nav>
      <div style="width: 150px;" v-if="!isLogin" id="google-signin-button" @click="googleLogin"></div>
      <router-link style="width: 150px;" v-if="isLogin" :to="{ path: '/profile', query: { id: 0 } }" class="menu-item">個人資料</router-link>
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
  import { toRef } from 'vue';
  export default {
    data() {
      return {
        isLogin: false,
        googleClientId: '917774825923-ki9dogspvhie7m0pfblhsec1mfa5guvi.apps.googleusercontent.com', // 替換為你的 Google OAuth 客戶端 ID
        // googleClientId: '63473080805-na5r3r5d4m3ibnk1f7kvjgp7n1grnaoe.apps.googleusercontent.com', // 替換為你的 Google OAuth 客戶端 ID
        // 917774825923-ki9dogspvhie7m0pfblhsec1mfa5guvi.apps.googleusercontent.com
        access_token: '',
        api_url:"",
        r_url:""
      };
    },
    provide(){
      return {
        isLogin: toRef(this, 'isLogin'),
        access_token: toRef(this, 'access_token'),
        api_url: toRef(this, 'api_url'),
        logout: this.logout
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
        this.api_url='https://api.k12edu.uk';
        this.r_url='https://teacher.k12edu.uk/';
        console.log('test2');
      }
      this.loadGoogleScript();
      if(localStorage.getItem("jwt")===null) this.getTokenFromCookie();
      this.access_token = localStorage.getItem('jwt');
      
      if(this.checkAndRefreshToken()==false){
        this.isLogin=false;
        
      }
      else{
        this.isLogin=true;
      }
      
    },
    methods: {
      async getTokenFromCookie() {
      
      try {
        const response = await fetch(
          `${this.api_url}/accounts/api/get_token_from_cookie/`,
          {
            method: "GET",
            credentials: "include", // 確保請求攜帶 Cookie
          }
        );

        const data = await response.json();

        if (response.ok) {
          console.log("已登入:", data.token);
          this.isLogin = true;
          this.access_token = data.token;
        } else {
          console.warn("未登入:", data.message);
        }
      } catch (error) {
        console.error("請求失敗:", error);
      }
    },

    logout() {
      document.cookie =
        "token=; path=/; SameSite=Lax; domain=.k12edu.uk; expires=Thu, 01 Jan 1970 00:00:00 GMT";
      this.access_token = "";
      this.isLogin = false;
      this.$router.push({ name: "home" });
    },

    loadGoogleScript() {
      const script = document.createElement("script");
      script.src = "https://accounts.google.com/gsi/client";
      script.async = true;
      script.defer = true;
      script.onload = this.initializeGoogleSignIn;
      document.head.appendChild(script);
    },

    initializeGoogleSignIn() {
      if (typeof window.google === "undefined") {
        console.error("Google API 未正確加載");
        return;
      }

      window.google.accounts.id.initialize({
        client_id: this.googleClientId,
        scope:
          "openid https://www.googleapis.com/auth/userinfo.email https://www.googleapis.com/auth/userinfo.profile",
        callback: this.handleCredentialResponse,
      });
      window.google.accounts.id.renderButton(
        document.getElementById("google-signin-button"),
        {
          theme: "outline",
          size: "large",
        }
      );
    },

    handleCredentialResponse(response) {
      console.log("Google Sign-In Credential Response:", response);
      const token = response.credential;
      this.sendIdTokenToBackend(token);
    },

    sendIdTokenToBackend(idToken) {
      fetch(`${this.api_url}/accounts/api/google-login2/`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ id_token: idToken }),
        credentials: "include", // 確保回傳的 JWT 存入 Cookie
      })
        .then((response) => response.json())
        .then((data) => {
          if (data.access) {
            console.log("JWT token received and stored in cookie:", data);
            this.isLogin = true;
            this.getTokenFromCookie();
          } else {
            console.error("JWT not received:", data);
          }
        })
        .catch((error) =>
          console.error("Error sending id_token to backend:", error)
        );
    },
  },
};
</script>


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
    flex-wrap: nowrap; /* 確保內容不換行 */
    overflow: hidden; /* 避免溢出 */
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
    border: 2px solid lightblue;
    border-radius: 5px;
    color: #000000;
    font-weight: bold;
    background-color: white;
    transition: background-color 0.3s, color 0.3s, box-shadow 0.3s;
    text-align: center;
    user-select: none; 
    cursor: pointer;
    white-space: nowrap; /* 避免文字換行 */
  }

  .menu-item:hover {
    background-color: lightblue;
    color: white;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  }

  .menu-item:active {
    transform: scale(0.95); /* 點擊時的縮放效果 */
  }
  .footer {
    background-color: #333;
    color: white;
    padding: 20px 0px;
    bottom: 0;
    width: 100%;
  }

.footer-container {
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 0 auto;
}
</style>
