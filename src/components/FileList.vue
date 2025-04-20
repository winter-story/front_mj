<template>
    <div>
      <h1>HWP 파일 업로드 및 조회</h1>
  
      <!-- 파일 업로드fff -->
      <input type="file" ref="fileInput" @change="handleFileChange" accept=".hwp" />
      <button @click="uploadFile">등록</button>
  
      <!-- 🔍 검색창 -->
      <div style="margin-top: 10px;">
        <input type="text" v-model="searchQuery" placeholder="파일 이름 검색" />
        <button @click="searchFiles">검색</button>
      </div>
  
      <!-- 파일 목록 -->
      <ul v-if="files.length > 0">
        <li v-for="(file, index) in files" :key="index">
          <a :href="`http://localhost:3000/download/${encodeURIComponent(file.name)}`" target="_blank">
            {{ file.name }}
          </a>
          - <small>{{ file.modified }}</small>
        </li>
      </ul>
      <p v-else>파일이 없습니다.</p>
  
      <!-- 페이지네이션 -->
      <div v-if="totalPages > 1">
        <button @click="goToPage(currentPage - 1)" :disabled="currentPage === 1">이전</button>
  
        <button
          v-for="page in totalPages"
          :key="page"
          @click="goToPage(page)"
          :style="{ fontWeight: currentPage === page ? 'bold' : 'normal' }"
        >
          {{ page }}
        </button>
  
        <button @click="goToPage(currentPage + 1)" :disabled="currentPage === totalPages">다음</button>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    name: 'FileList',
    data() {
      return {
        files: [],
        selectedFile: null,
        currentPage: 1,
        pageSize: 10,
        totalPages: 1,
        searchQuery: ''
      };
    },
    methods: {
      handleFileChange(event) {
        this.selectedFile = event.target.files[0];
      },
      uploadFile() {
        if (!this.selectedFile) {
          alert('파일을 선택하세요');
          return;
        }
  
        const formData = new FormData();
        formData.append('file', this.selectedFile);
  
        fetch('http://localhost:3000/upload/', {
          method: 'POST',
          body: formData
        })
          .then(res => res.json())
          .then(data => {
            alert(data.message || '업로드 완료');
            this.selectedFile = null;
            this.$refs.fileInput.value = '';
            this.goToPage(1);
          })
          .catch(err => {
            console.error('업로드 실패:', err);
            alert('업로드 중 오류 발생');
          });
      },
      goToPage(page) {
        if (page < 1 || page > this.totalPages) return;
  
        const url = `http://localhost:3000/files/?page=${page}&page_size=${this.pageSize}&search=${encodeURIComponent(this.searchQuery)}`;
        fetch(url)
          .then(res => res.json())
          .then(data => {
            this.files = data.results;
            this.totalPages = data.total_pages;
            this.currentPage = data.current_page;
          })
          .catch(err => {
            console.error('파일 목록 불러오기 오류:', err);
          });
      },
      searchFiles() {
        this.goToPage(1); // 검색 시 항상 1페이지부터 시작
      }
    },
    mounted() {
      this.goToPage(1);
    }
  };
  </script>
  
  <style scoped>
  button {
    margin: 5px;
  }
  </style>
  