<template>
  <main>
    <div>
      <input  type="file" @change="handleFileChange" />
      <button @click="fetchData" class="buttonstyle">Обработать</button>
    </div>

    <div id="main" class="flex-around">
      <div v-if="croppedImages.length > 0">
        <div class="flex-around" v-for="(img, index) in croppedImages" :key="index">
          <div style="display: flex; justify-content: space-between">
            <select name="" id="">
              <template v-for="(item, index) in linesType">
                <option value="item.value">{{ item.name }}</option>
              </template>
              <option value="" selected>Не требуется</option>
            </select>
            <input type="text" :value="lines[index].LineText"/>
          </div>
          <img :src="img" :alt="'Cropped Image ' + index" />
        </div>
      </div>

      <div v-if="imageUrl">
        <img :src="imageUrl" ref="image" style="max-width: 100%" />
      </div>
    </div>
  </main>
</template>
<style scoped>
body {
  font-family:Verdana, Geneva, Tahoma, sans-serif;
}

  main {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 20px;
  }

  .flex-around {
    gap: 100px;
    display: flex;
    flex-wrap: wrap;
    padding-left: 20px;
    padding-top: 70px;
  }

  .image-container {
    width: 90%;
    max-width: 400px;
    border: 1px solid #ddd;
    border-radius: 5px;
    margin-bottom: 20px;
  }

  .image-details {
    display: flex;
    justify-content: space-between;
    padding: 10px;
  }
  select, .text-input {
    flex: 1;
    padding: 5px;
    border-radius: 5px;
  }

  select {
    margin-right: 10px;
  }

  .input-container {
    margin-bottom: 20px;
  }

  .input-container input[type="file"] {
    display: none;
  }

  .input-container button {
    background-color: #4CAF50;
    color: white;
    padding: 10px 20px;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    transition: background-color 0.3s ease;
  }

  .input-container button:hover {
    background-color: #45a049;
  }

  .text-input {
    margin-top: 10px;
    padding: 5px;
    border: 1px solid #ccc;
  }
  .buttonstyle {
    background-color: #4CAF50;
    color: white;
    padding: 10px 20px;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    transition: background-color 0.3s ease;
  }

  .buttonstyle:hover {
    background-color: #45a049;
  }
</style>





<script setup>
import { ref } from 'vue'
import axios from 'axios'

const file = ref(null)
const imageUrl = ref(null)
const croppedImages = ref([])
const lines = ref([])

const handleFileChange = (event) => {
  const selectedFile = event.target.files[0]
  file.value = selectedFile

  const reader = new FileReader()
  reader.onload = (e) => {
    imageUrl.value = e.target.result
  }
  reader.readAsDataURL(selectedFile)
}

const linesType = [
  {
    value: 'CVnumber',
    name: 'Номер трудовой книги'
  },
  {
    value: 'surname',
    name: 'Фамилия'
  },
  {
    value: 'name',
    name: 'Имя'
  },
  {
    value: 'patronymic',
    name: 'Отчество'
  },
  {
    value: 'education',
    name: 'Образование'
  },
  {
    value: 'profesion',
    name: 'Профессия, Специальность'
  },
  {
    value: 'date',
    name: 'Дата заполнения'
  }
]

const fetchData = async () => {
  const formData = new FormData()
  formData.append('file', file.value)
  formData.append('language', 'rus')
  formData.append('isOverlayRequired', 'true')
  formData.append('apikey', 'K83655257488957')
  formData.append('isTable', 'true')

  try {
    const response = await axios.post('https://api.ocr.space/parse/image', formData, {
      headers: {
        'Content-Type': 'multipart/form-data'
      }
    })
    const data = response.data
    lines.value = data.ParsedResults[0].TextOverlay.Lines
    console.log(lines);
    if (data.OCRExitCode === 1 && data.ParsedResults && data.ParsedResults.length > 0) {
      cropMultipleImages(lines)
    } else {
      console.error('No text found or an error occurred.')
    }
  } catch (error) {
    console.error('There was an error!', error)
  }
}

const cropMultipleImages = (lines) => {
  const imageElement = document.createElement('img')
  imageElement.src = imageUrl.value

  imageElement.onload = () => {
    const canvas = document.createElement('canvas')
    const ctx = canvas.getContext('2d')

    croppedImages.value = []
    lines.value.forEach((line) => {
      const left = Math.min(...line.Words.map((word) => word.Left))
      const top = Math.min(...line.Words.map((word) => word.Top))
      const right = Math.max(...line.Words.map((word) => word.Left + word.Width))
      const bottom = Math.max(...line.Words.map((word) => word.Top + word.Height))
      const width = right - left
      const height = bottom - top

      canvas.width = width
      canvas.height = height

      ctx.drawImage(imageElement, left, top, width, height, 0, 0, width, height)

      const croppedImageUrl = canvas.toDataURL()
      croppedImages.value.push(croppedImageUrl)
    })
  }
}
</script>

