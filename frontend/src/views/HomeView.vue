<template>
  <main>
    <div>
      <input type="file" @change="handleFileChange" />
      <button @click="fetchData">Обработать</button>
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
            <input type="text" />
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

<script setup>
import { ref } from 'vue'
import axios from 'axios'

const file = ref(null)
const imageUrl = ref(null)
const croppedImages = ref([])

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
    const lines = data.ParsedResults[0].TextOverlay.Lines
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
    lines.forEach((line) => {
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
      console.log(croppedImages)
    })
  }
}
</script>

<style scoped>
.flex-around {
  display: flex;
  justify-content: space-around;
}
</style>
