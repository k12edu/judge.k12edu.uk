<template>
  <div class="v-container" style="min-height: 100vh; width: 60%; gap: 20px;">
    <div class="back-div" >
      <p class="back-button" @click="goBack">返回上一頁</p>
    </div>
    <div v-if="problem" class="h-container">
      <h1>{{ problem.title }}</h1>
    </div>
    <div class="problem-description">
      <h2>題目敘述</h2>
      <div v-if="loading">載入中...</div>
      <div v-else-if="error" class="error-message">{{ error }}</div>
      <div v-else class="description-content">{{ description }}</div>
    </div>
    <div v-if="problem" class="v-container">
      <p>標籤: {{ getTag}}</p>
    </div>
    <!-- 編輯器容器 -->
    <div class="h-container">
      <button style="margin: 0px 10px;" @click="submitCode" class="btn">提交</button>
      <button style="margin: 0px 10px;" @click="gotoArticleList" class="btn">討論區</button>
    </div>
    <select v-model="language" @change="changeLanguage">
      <option value="java">Java</option>
      <option value="python">Python</option>
      <option value="c">C</option>
      <option value="cpp">C++</option>
    </select>
    
    <div ref="editorContainer" class="codemirror-container"></div>
    
    <!-- 語言選擇器 -->
  </div>
</template>

<script>
import { EditorView, basicSetup } from "codemirror";
import { EditorState } from "@codemirror/state";
import { keymap } from "@codemirror/view";
import { indentWithTab } from "@codemirror/commands";
import { java } from "@codemirror/lang-java";
import { python } from "@codemirror/lang-python";
import { cpp } from "@codemirror/lang-cpp";

export default {
  name: 'CodeEditor',
  data() {
    return {
      request_time:0,
      problem:null,
      description: '', // 題目敘述內容
      loading: true,   // 載入狀態
      error: null,     // 錯誤訊息
      editor: null,    // CodeMirror 編輯器實例
      language: 'python', // 當前語言
      codeContent: '\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n', // 編輯器內容
    };
  },
  props: {
  },
  computed:{
    problemId() {
      return this.$route.query.problemId || ""; // 避免 undefined
    },
    getTag(){
      if(this.problem.tag.length==0) return "";
      let res=this.problem.tag[0];
      for(let i=1;i<this.problem.tag.length;i++){
        res+=","+this.problem.tag[i];
      }
      return res;
    }
  },
  inject: ['api_url', 'access_token','getTokenFromCookie'],
  methods: {
    goBack() {
      this.$router.go(-1);
    },
    gotoArticleList(){
      this.$router.push({ path: `/problem_article_list`, query: { problemId: this.$route.query.problemId } });
    },
    // 獲取題目敘述
    async fetchDescription() {
      try {
          const defaultParams = {
            id: this.problemId,
          };

        const queryParams = new URLSearchParams(defaultParams)
        const response = await fetch(`${this.api_url}/onlinejudge/api/problem/get?${queryParams.toString()}`, {
          method: 'GET',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${this.access_token}`,
          },
        });
        if (!response.ok) {
          throw new Error(`HTTP 錯誤：${response.status}`);
        }
        const data = await response.json();
        this.problem=data.problem;
        this.description = data.problem.problem_description || '無法取得題目內容';
      } catch (err) {
        this.error = '無法取得題目敘述，請稍後再試';
        if(this.request_time<=2){
          this.request_time+=1;
          this.retryFetchDescription();
        }
        console.error(err);
      } finally {
        this.loading = false;
      }
    },
    async retryFetchDescription() {
    // 等待 1 秒
    await new Promise(resolve => setTimeout(resolve, 1000));
    this.getTokenFromCookie();
    // 重新呼叫 fetchDescription 嘗試請求
    alert('bbbw');
    alert(this.access_token);
    this.fetchDescription();
  },
    

    initializeEditor() {
      const myTheme = EditorView.theme({
        "&": { textAlign: "left" },
      });

      const borderedTheme = EditorView.theme({
        "&": { border: "1px solid #ccc", borderRadius: "4px", padding: "4px" },
      });

      const noOutlineTheme = EditorView.theme({
        "&.cm-editor.cm-focused": { outline: "none" },
        ".cm-selectionMatch": { outline: "none" },
      });

      const cmOptions = {
        doc: this.codeContent,
        extensions: [
          basicSetup,
          python(),
          myTheme,
          borderedTheme,
          noOutlineTheme,
          keymap.of([indentWithTab]), // 讓 Tab 插入縮排，而不是切換焦點
        ],
      };

      const state = EditorState.create({
        doc: cmOptions.doc,
        extensions: cmOptions.extensions,
      });

      this.editor = new EditorView({
        state,
        parent: this.$refs.editorContainer,
      });
    },

    // 切換語言
    changeLanguage() {
      if (this.editor) {
        this.editor.destroy();
        this.editor = null;

        const myTheme = EditorView.theme({
          "&": { textAlign: "left" },
        });

        const borderedTheme = EditorView.theme({
          "&": { border: "1px solid #ccc", borderRadius: "4px", padding: "4px" },
        });

        const noOutlineTheme = EditorView.theme({
          "&.cm-editor.cm-focused": { outline: "none" },
          ".cm-selectionMatch": { outline: "none" },
        });

        const ext = [basicSetup, myTheme, borderedTheme, noOutlineTheme, keymap.of([indentWithTab])];

        if (this.language === 'java') {
          ext.push(java());
        } else if (this.language === 'python') {
          ext.push(python());
        } else if (this.language === 'c' || this.language === 'cpp') {
          ext.push(cpp());
        }

        const cmOptions = {
          doc: this.codeContent,
          extensions: ext,
        };

        const state = EditorState.create({
          doc: cmOptions.doc,
          extensions: cmOptions.extensions,
        });

        this.editor = new EditorView({
          state,
          parent: this.$refs.editorContainer,
        });
      }
    },

    // 提交代碼到後端
    async submitCode() {
      const codeContent = this.editor.state.doc.toString();  // 從編輯器取得代碼
      // console.log(codeContent);
      let l=this.language;
      if(l=='python') l='py';
      const url = `${this.api_url}/judge/submit/`; // 替換為你的後端 API 路徑

      try {
        let data2={
            'language': l,
            'code': codeContent,
            'problemId': this.$route.query.problemId,
          };
        console.log(data2);
        const response = await fetch(url, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${this.access_token}`,
          },
          body: JSON.stringify(data2),
        });

        if (!response.ok) {
          throw new Error(`HTTP 錯誤：${response.status}`);
        }

        const data = await response.json();
        console.log('提交成功', data);
        alert('提交成功');
        this.$router.push({ path: `/submission-history`, });
        // 你可以在這裡處理後端回應
      } catch (err) {
        console.error('提交代碼時發生錯誤:', err);
      }
    },
  },

  mounted() {
    console.log(this.$route.query.problemId);
    this.fetchDescription(); // 組件掛載時請求資料
    this.initializeEditor(); // 初始化編輯器
    this.getTokenFromCookie();
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
.btn {
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
  }

  .btn:hover {
    background-color: lightblue;
    color: white;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  }

  .btn:active {
    transform: scale(0.95); /* 點擊時的縮放效果 */
  }
/* 確保編輯器有高度 */
.codemirror-container {
  height: 500px; /* 調整為你需要的高度 */
  width: 100%;  /* 設為 100% 寬度 */
  overflow: auto;
  margin-bottom: 100px;
}
.problem-description {
  border: 1px solid #ccc;
  padding: 16px;
  border-radius: 8px;
  background-color: #f9f9f9;
  width: 100%;
}
.description-content {
  white-space: pre-wrap; /* 保留換行與空白 */
  font-family: Arial, sans-serif;
  line-height: 1.6;
  max-height: none;  /* 確保沒有高度限制 */
  overflow: visible; /* 避免內容被截斷 */
}
.error-message {
  color: red;
}
</style>
