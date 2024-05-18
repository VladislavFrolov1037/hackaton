<template>
  <div>
    <input type="file" @change="handleFileChange">
    <button @click="fetchData">ЛЕСГОУ</button>
    <div v-if="ocrResult">
      <h3>OCR Result:</h3>
      <pre>{{ ocrResult }}</pre>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';

const file = ref(null);
const ocrResult = ref(null);

const handleFileChange = (event) => {
  const selectedFile = event.target.files[0];
  file.value = selectedFile;
};

const fetchData = async () => {
  const formData = new FormData();
  formData.append('file', file.value);
  formData.append('language', 'rus');
  formData.append('isOverlayRequired', 'true');
  formData.append('apikey', 'K83655257488957');
  formData.append('isTable', 'true');

  try {
    const response = await axios.post("https://api.ocr.space/parse/image", formData, {
      headers: {
        'Content-Type': 'multipart/form-data'
      }
    });

    const data = response.data;
    if (data.OCRExitCode === 1 && data.ParsedResults && data.ParsedResults.length > 0) {
      ocrResult.value = data.ParsedResults[0];
      console.log(ocrResult.value)
    } else {
      ocrResult.value = 'No text found or an error occurred.';
    }
  } catch (error) {
    console.error("There was an error!", error);
    ocrResult.value = 'An error occurred while processing the image.';
  }
}
</script>

<style scoped>

</style>
