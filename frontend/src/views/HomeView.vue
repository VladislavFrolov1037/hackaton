<template>
  <div>
    <input type="file" @change="handleFileChange">
    <button @click="fetchData">ЛЕСГОУ</button>
    <div v-if="imageUrl">
      <img :src="imageUrl" ref="image" style="max-width: 100%;">
    </div>
    <div v-if="croppedImages.length > 0">
      <h3>Cropped Images:</h3>
      <div v-for="(img, index) in croppedImages" :key="index">
        <img :src="img" :alt="'Cropped Image ' + index">
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';

const file = ref(null);
const imageUrl = ref(null);
const croppedImages = ref([]);

const handleFileChange = (event) => {
  const selectedFile = event.target.files[0];
  file.value = selectedFile;

  const reader = new FileReader();
  reader.onload = (e) => {
    imageUrl.value = e.target.result;
  };
  reader.readAsDataURL(selectedFile);
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
      const words = data.ParsedResults[0].TextOverlay.Lines.flatMap(line => line.Words);
      cropMultipleImages(words);
    } else {
      console.error('No text found or an error occurred.');
    }
  } catch (error) {
    console.error("There was an error!", error);
  }
};

const cropMultipleImages = (words) => {
  const imageElement = document.createElement('img');
  imageElement.src = imageUrl.value;

  imageElement.onload = () => {
    const canvas = document.createElement('canvas');
    const ctx = canvas.getContext('2d');

    croppedImages.value = [];

    words.forEach(word => {
      const { Left: left, Top: top, Width: width, Height: height } = word;

      canvas.width = width;
      canvas.height = height;

      ctx.drawImage(imageElement, left, top, width, height, 0, 0, width, height);

      const croppedImageUrl = canvas.toDataURL();
      croppedImages.value.push(croppedImageUrl);
    });
  };
};
</script>

<style scoped>



</style>
