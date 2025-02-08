<template>
  <div class="v-container" style="min-height: 100vh; width: 60%; gap: 20px;">
    <div v-if="problem" class="h-container">
      <h1>{{ problem.title }}</h1>
    </div>
    <div class="problem-description">
      <h2>題目敘述</h2>
      <div v-if="loading">載入中...</div>
      <div v-else-if="error" class="error-message">{{ error }}</div>
      <div v-else class="description-content" v-html="description"></div>
    </div>
    <!-- 編輯器容器 -->
    <button @click="submitCode" class="btn">提交</button>
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
import { EditorView, basicSetup } from 'codemirror'
import { EditorState } from '@codemirror/state'
import { java } from '@codemirror/lang-java'
import { python } from '@codemirror/lang-python'
import { cpp } from '@codemirror/lang-cpp'

export default {
  name: 'CodeEditor',
  data() {
    return {
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
    }
  },
  inject: ['api_url', 'access_token'],
  methods: {
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
        console.error(err);
      } finally {
        this.loading = false;
      }
    },

    // 初始化編輯器
    initializeEditor() {
      const myTheme = EditorView.theme({
        "&": {
          textAlign: "left", // 確保文字靠左對齊
        },
      });
      const borderedTheme = EditorView.theme({
        "&": {
          border: "1px solid #ccc", // 邊框
          borderRadius: "4px", // 圓角邊框
          padding: "4px", // 內部間距
        },
      });
      const noOutlineTheme = EditorView.theme({
        "&.cm-editor.cm-focused": {
          outline: "none", // 移除整個編輯器的外框
        },
        ".cm-selectionMatch": {
          outline: "none", // 移除選取匹配的虛線框
        },
      });

      const cmOptions = {
        doc: this.codeContent, // 初始內容
        extensions: [basicSetup, python(), myTheme, borderedTheme, noOutlineTheme],
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
        // 清除編輯器的父容器內容
        this.editor.destroy();
        this.editor = null;
        
        const myTheme = EditorView.theme({
        "&": {
          textAlign: "left", // 確保文字靠左對齊
        },
      });
      const borderedTheme = EditorView.theme({
        "&": {
          border: "1px solid #ccc", // 邊框
          borderRadius: "4px", // 圓角邊框
          padding: "4px", // 內部間距
        },
      });
      const noOutlineTheme = EditorView.theme({
        "&.cm-editor.cm-focused": {
          outline: "none", // 移除整個編輯器的外框
        },
        ".cm-selectionMatch": {
          outline: "none", // 移除選取匹配的虛線框
        },
      });
      const ext = [basicSetup, python(), myTheme, borderedTheme, noOutlineTheme]
      if (this.language === 'java') {
          ext.push(java());
        } else if (this.language === 'python') {
          ext.push(python());
        } else if (this.language === 'c' || this.language === 'cpp') {
          ext.push(cpp());
        }
      const cmOptions = {
        doc: this.codeContent, // 初始內容
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
      console.log(codeContent);
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
  },
};
</script>

<style scoped>
.btn {
    text-decoration: none; /* 移除超連結預設的下劃線 */
    padding: 10px 20px;
    border: 2px solid #22a507;
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
    background-color: #13a30e;
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
}
.error-message {
  color: red;
}
</style>
