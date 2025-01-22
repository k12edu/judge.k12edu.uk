<template>
    <div class="main-content" style="min-height: 100vh;">
        <!-- 公告區域 -->
        <div class="announcement-section">
        <h2>最新公告</h2>
        <router-view></router-view>
        </div>

        <!-- 網站介紹區域 -->
        <div class="site-intro-section">
        <h3>關於網站</h3>
        <p>
            歡迎來到Online Judge! 此網站提供使用者進行程式題目練習，立即開始您的題目練習並累積經驗!
        </p>
        <div class="features">
            <div class="feature">
            <h4>程式語言</h4>
            <p>支援Python、Java、C、C++四種語言。</p>
            </div>
            <div class="feature">
            <h4>學習範圍</h4>
            <p>題目範圍包括基礎程式功能、資料結構與演算法。</p>
            </div>
            <div class="feature">
            <h4>"遊戲名稱"</h4>
            <p>結合遊戲提高學習動力。</p>
            </div>
        </div>
        </div>
    </div>
  </template>
  
  <script>
  export default {
    name: 'MainBody',
    components:{
      
    },
    data(){
      return {
        bodyTitle: 'Home Page',
      }
    },
    props: {

    }
  }
  </script>
  
  <!-- Add "scoped" attribute to limit CSS to this component only -->
   
  <style scoped>
    .main-content {
        display: flex;
        flex-direction: column;
        align-items: center;
        margin: 20px;
        padding: 20px;
    }

    .announcement-section {
    width: 100%;
    max-width: 800px;
    background-color: #f9f9f9;
    padding: 20px;
    margin-bottom: 40px;
    border: 1px solid #ddd;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    }

    .announcement-section h2 {
    margin-bottom: 10px;
    color: #333;
    }

    .site-intro-section {
    width: 100%;
    max-width: 800px;
    text-align: center;
    background-color: #f0f8ff;
    padding: 20px;
    border: 1px solid #ddd;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    }

    .site-intro-section h3 {
    margin-bottom: 15px;
    color: #2c3e50;
    }

    .site-intro-section p {
    margin-bottom: 20px;
    line-height: 1.6;
    color: #555;
    }

    .features {
    display: flex;
    flex-wrap: wrap;
    gap: 20px;
    justify-content: center;
    }

    .feature {
    background-color: #fff;
    border: 1px solid #ddd;
    border-radius: 8px;
    padding: 15px;
    width: 240px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    }

    .feature h4 {
    margin-bottom: 10px;
    color: #1e90ff;
    }

    .feature p {
    color: #666;
    font-size: 14px;
    line-height: 1.5;
    }
  </style>