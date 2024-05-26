<script>
export default {
  data() {
    return {
      isHighlighted: false,
      files: [],
      result: null,
      message: null,
      showModal: false,
      modalImage: null
    };
  },
  methods: {
    highlight() {
      this.isHighlighted = true;
    },
    unhighlight() {
      this.isHighlighted = false;
    },
    handleDrop(event) {
      this.unhighlight();
      const dt = event.dataTransfer;
      const files = dt.files;
      this.handleFiles(files);
    },
    handleFiles(files) {
      for (let file of files) {
        this.previewFile(file);
      }
    },
    previewFile(file) {
      const reader = new FileReader();
      reader.readAsDataURL(file);
      reader.onloadend = () => {
        this.files.push({
          name: file.name,
          preview: reader.result,
          file: file
        });
      };
    },
    deleteFile(index) {
      this.files.splice(index, 1);
    },
    async compareFiles() {
      if (this.files.length !== 2) {
        alert('Пожалуйста, загрузите два файла для сравнения.');
        return;
      }

      const formData = new FormData();
      formData.append('file1', this.files[0].file);
      formData.append('file2', this.files[1].file);

      try {
        const response = await fetch('http://localhost:5000/upload', {
          method: 'POST',
          body: formData
        });

        if (response.ok) {
          const contentType = response.headers.get("content-type");
          if (contentType && contentType.includes("application/json")) {
            const data = await response.json();
            this.message = data.error || data.message;
            this.result = null;
          } else {
            const blob = await response.blob();
            const resultURL = URL.createObjectURL(blob);
            this.result = resultURL;
            this.message = null;
          }
        } else {
          const errorData = await response.json();
          this.message = errorData.error || 'Произошла ошибка';
        }
      } catch (error) {
        console.error('Ошибка при загрузке файлов:', error);
        this.message = 'Произошла ошибка';
      }
    },
    deleteResult() {
      this.result = null;
    },
    triggerFileInput() {
      document.getElementById('file-input').click();
    },
    handleInputChange(event) {
      const files = event.target.files;
      this.handleFiles(files);
    },
    openModal(image) {
      this.modalImage = image;
      this.showModal = true;
    },
    closeModal() {
      this.showModal = false;
    }
  }
};
</script>

<template>
  <div>
    <div id="drop-area" 
         @dragenter.prevent="highlight" 
         @dragover.prevent="highlight" 
         @dragleave.prevent="unhighlight"
         @drop.prevent="handleDrop" 
         :class="{ highlight: isHighlighted }">
      Перетащите файлы сюда
    </div>
    <input type="file" id="file-input" @change="handleInputChange" multiple style="display: none;">
    <button @click="triggerFileInput">Загрузить файлы</button>
    <div id="file-list">
      <div v-for="(file, index) in files" :key="file.name" class="file-item">
        <img 
          :src="file.preview" 
          :alt="file.name" 
          class="file-image"
          @click="openModal(file.preview)" />
        <button class="delete-button" @click="deleteFile(index)">
          <svg width="24" height="24" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg" class="icon-svg">
            <path d="M6.34315 6.34338L17.6569 17.6571M17.6569 6.34338L6.34315 17.6571" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />
          </svg>
        </button>
      </div>
    </div>
    <button @click="compareFiles" :disabled="files.length !== 2">Сравнить изображения</button>
    <div v-if="result || message" class="result-container" :class="{ 'no-differences': message }">
      <button v-if="result" class="delete-result-button" @click="deleteResult">
        <svg width="24" height="24" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg" class="icon-svg">
          <path d="M6.34315 6.34338L17.6569 17.6571M17.6569 6.34338L6.34315 17.6571" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />
        </svg>
      </button>
      <img v-if="result" :src="result" alt="Результат сравнения" class="result-image" @click="openModal(result)" />
      <p v-if="message" class="no-differences-message">{{ message }}</p>
    </div>
    <div v-if="showModal" class="modal" @click.self="closeModal">
      <img :src="modalImage" alt="Увеличенное изображение" class="modal-image" />
    </div>
  </div>
</template>

<style scoped>
#drop-area {
  border: 2px dashed #ccc;
  border-radius: 20px;
  width: 300px;
  height: 200px;
  padding: 20px;
  text-align: center;
  font-size: 20px;
  color: #aaa;
  margin: 50px auto;
}

#drop-area.highlight {
  border-color: #21bbc0;
}

#file-list {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  margin-top: 20px;
}

.file-item {
  position: relative;
  margin: 10px;
}

.file-image {
  max-width: 300px;
  max-height: 300px;
  object-fit: cover;
  border-radius: 5px;
  cursor: pointer; /* Добавляем курсор-указатель */
}

.delete-button,
.delete-result-button {
  position: absolute;
  margin: 6px;
  padding: 0;
  top: 0;
  right: 0;
  background: none;
  cursor: pointer;
}

.icon-svg {
  background-color: #21bbc0;
  stroke: #fff;
  border-radius: 6px;
}

.result-container {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 40vw;
  aspect-ratio: 4 / 3;
  margin: 20px auto;
  border: 2px solid #21bbc0;
  padding: 10px;
  border-radius: 10px;
  overflow: hidden;
}

.result-container.no-differences {
  border-color: red;
}

.result-image {
  width: 100%;
  height: 100%;
  object-fit: cover; /* Поддерживает соотношение сторон, обрезая по необходимости */
  cursor: pointer; /* Добавляем курсор-указатель */
}

.no-differences-message {
  color: red;
  font-size: 18px;
  text-align: center;
}

button {
  margin: 10px;
  padding: 10px 20px;
  background-color: #21bbc0;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background-color: rgba(0, 0, 0, 0.8);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-image {
  max-width: 90%;
  max-height: 90%;
}
</style>
