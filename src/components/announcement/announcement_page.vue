<template>
  <div class="announcement-page">
    <div class="back-div" >
      <p class="back-button" @click="goBack">返回上一頁</p>
    </div>
    <h1>公告</h1>
    <div class="items-per-page">
      <label for="perPage">每頁顯示數量:</label>
      <input
        type="number"
        id="perPage"
        v-model="pageSize"
        min="1"
        @input="changeItemPerPage"
        placeholder="輸入數字"
      />
    </div>
    <div v-if="announcements.length" class="announcement-list">
      <div v-for="announcement in announcements" :key="announcement.id" class="announcement-card">
        <h3>{{ announcement.title }}</h3>
        <p>{{ announcement.announcement_description }}</p>
      </div>
    </div>
    <div v-else class="no-announcements">
      <p>目前沒有公告</p>
    </div>
    <div class="pagination">
      <button @click="changePage(page)" class="btn" :class="{ active: currentPage === page }" v-for="page in displayedPages" :key="page">
        {{ page }}
      </button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      announcements: [], // 全部公告
      currentPage: 1,    // 當前頁數
      pageSize: 5,       // 每頁顯示幾筆資料
      total_page: 0,
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
    changeItemPerPage(){
        this.total_page=0;
        this.changePage(1);
      },
    async fetchAnnouncements() {
      try {
        const defaultParams = {
          page: this.currentPage,
          per_page: this.pageSize,
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
        if(this.total_page!=data.total_page) this.total_page = data.total_page;
        this.announcements = data.announcements;
      } catch (error) {
        console.error("取得公告時發生錯誤：", error);
      }
    },
    changePage(page) {
      this.currentPage = page;
      this.fetchAnnouncements();
    },
  },
  mounted() {
    this.total_page=0;
    this.fetchAnnouncements();
  },
};
</script>

<style scoped>
.back-div{
  display: flex;
  width: 100%;
  justify-content: left;
  padding: 10px 20px;
}
.btn{
  border-image-source: url('@/assets/tile_0002.png');
  border-width: 10px;
  border-style: solid;
  border-image-slice: 6 fill;
  border-image-repeat: repeat;
  color: white;
}
.back-button {
  width: 100px;
  background: #f0f0f0;
  padding: 6px 6px;
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
.announcement-page {
  padding: 20px;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 70%;
}

.announcement-list {
  width: 80%;
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.announcement-card {
  /* background: #f9f9f9; */
  padding: 15px;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s;
  min-height: 200px;
  text-align: left;
  border-image-source: url('@/assets/tile_0001.png');
  border-width: 10px;
  border-style: solid;
  border-image-slice: 6 fill;
  border-image-repeat: repeat;
}

.announcement-card:hover {
  transform: translateY(-5px);
}

.announcement-card h3 {
  margin: 0 0 10px 0;
  font-size: 1.5rem;
  color: #dfdfdf;
}

.announcement-card p {
  margin: 0;
  color: #dfdfdf;
}

.no-announcements {
  margin-top: 20px;
  font-size: 1.2rem;
  color: #777;
}

.pagination {
  display: flex;
  gap: 10px;
  margin-top: 20px;
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
