<template>
    <div class="main-content" style="min-height: 100vh;">
        <!-- 公告區域 -->
        <div class="announcement-section">
        <h2>最新公告</h2>
        <div v-if="announcements.length" class="announcement-list">
          <div v-for="announcement in announcements" :key="announcement.id" class="announcement-card">
            <h3>{{ announcement.title }}</h3>
            <p>{{ announcement.announcement_description }}</p>
          </div>
        </div>
        <div v-else class="no-announcements">
          <p>目前沒有公告</p>
        </div>
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
        announcements: [],
      }
    },
    props: {

    },
    inject: ['api_url', 'access_token'],
    methods:{
      async fetchAnnouncements() {
      try {
        const defaultParams = {
          page: 1,
          per_page: 3,
        };

        const queryParams = new URLSearchParams(defaultParams);
        const response = await fetch(`${this.api_url}/onlinejudge/api/announcement/list?${queryParams.toString()}`, {
          method: 'GET',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${this.access_token}`,
          },
        });
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        const data = await response.json();
        this.announcements = data.announcements;
      } catch (error) {
        console.error("取得公告時發生錯誤：", error);
      }
    },
    },

    mounted(){
      this.fetchAnnouncements();
    }
  }
  </script>
  
  <!-- Add "scoped" attribute to limit CSS to this component only -->
   
  <style scoped>
    .announcement-list {
      width: 80%;
      display: flex;
      flex-direction: column;
      gap: 15px;
    }

    .announcement-card {
      background: #f9f9f9;
      padding: 15px;
      border-radius: 8px;
      box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
      transition: transform 0.2s;
      min-height: 200px;
      text-align: left;
      text-overflow: ellipsis;
    }

    .announcement-card:hover {
      transform: translateY(-5px);
    }

    .announcement-card h3 {
      margin: 0 0 10px 0;
      font-size: 1.5rem;
      color: #333;
    }

    .announcement-card p {
      margin: 0;
      color: #555;
    }

    .no-announcements {
      margin-top: 20px;
      font-size: 1.2rem;
      color: #777;
    }
    .main-content {
        display: flex;
        flex-direction: column;
        align-items: center;
        margin: 20px;
        padding: 20px;
    }

    .announcement-section {
      display: flex;
      flex-direction: column;
      align-items: center;
    width: 100%;
    max-width: 800px;
    padding: 20px;
    margin-bottom: 40px;
    border: 1px solid #ddd;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    background-color: rgb(206, 167, 117);
    border-image-source: url('@/assets/tile_0000.png');
    border-width: 10px;
    border-style: solid;
    border-image-slice: 6 fill;
    border-image-repeat: repeat;

    }

    .announcement-section h2 {
    margin-bottom: 10px;
    color: #333;
    }

    .site-intro-section {
    width: 100%;
    max-width: 800px;
    text-align: center;
    background-color: rgb(206, 167, 117);
    padding: 20px;
    border: 1px solid #ddd;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    border-image-source: url('@/assets/tile_0000.png');
    border-width: 10px;
    border-style: solid;
    border-image-slice: 6 fill;
    border-image-repeat: repeat;
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
    border-radius: 8px;
    padding: 15px;
    width: 240px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    border-image-source: url('@/assets/tile_0002.png');
    border-width: 10px;
    border-style: solid;
    border-image-slice: 6 fill;
    border-image-repeat: repeat;
    }

    .feature h4 {
    margin-bottom: 10px;
    color: #444444;
    }

    .feature p {
    color: #444444;
    font-size: 14px;
    line-height: 1.5;
    }
  </style>