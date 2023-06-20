<template>
  <v-container class="center" v-if="!isUploading && !imgUploaded">
    <v-card elevation="24" rounded="lg">
      <v-card-title class="text-center text-h6">Upload your image</v-card-title>
      <v-card-subtitle class="text-center text-caption"
        >File should be .jpeg, .png, .jpg</v-card-subtitle
      >
      <v-card-text>
        <div
          class="drop-area bg-grey-lighten-2"
          @dragover="handleDragOver"
          @drop="handleDrop"
        >
          <img src="./assets/image.svg" alt="SVG Image" class="svg-image" />
          Drag & Drop your image here
        </div>

        <v-row justify="center" class="mt-6 mb-4">
        <v-btn @click="choseAFile()" color="indigo" rounded="lg">Choose a file</v-btn>
        </v-row>
      </v-card-text>
    </v-card>
  </v-container>
  <v-container class="center" v-if="isUploading">
    <v-card elevation="12" min-width="300">
      <v-card-title>Uploading...</v-card-title>
      <v-card-text>
        <v-progress-linear
          color="indigo"
          indeterminate
          rounded
          height="8"
        ></v-progress-linear>
      </v-card-text>
    </v-card>
  </v-container>
  <v-container class="center" v-if="imgUploaded">
    <v-card>
    <v-row justify="center" class="mt-4 mb-4">
    <v-icon  size="x-large" color="green">mdi-check-circle</v-icon>
    </v-row>
      <v-card-title class="text-center text-h5">Uploaded Successfully!</v-card-title>

      <v-card-text>
        <v-row justify="center" class="mt-2 mb-2">
          <img :src="imgLink"  height="450" width="600" />
        </v-row>
        <v-row justify="space-evenly">
          <v-text-field v-model="imgLink" variant="plain" class="text-overline"></v-text-field>
          <v-btn class="mt-4 ml-2 mr-2" size="small" rounded="lg" color="indigo" @click="copyToClipBoard()">Copy link</v-btn>
        </v-row>
      </v-card-text>
      <v-row justify="center" class="mt-2 mb-6">
      <v-btn color="red" size="small" rounded="lg" @click="reset()">Upload another image</v-btn>
      </v-row>
    </v-card>
  </v-container>
  <v-snackbar
    v-model="valueSnack" timeout="3000" color="green" location="center"
  >
    Link copied to clipboard<v-icon  @click.native="valueSnack = false">mdi-close</v-icon>
  </v-snackbar>
</template>

<script setup>
import { ref } from "vue"
import axios from "axios"
//change this URL if you want local
const BASE_URL = "https://desarrollogaren.000webhostapp.com/imgUploader/public/"
const isUploading = ref(false)
const imgUploaded = ref(false)
const imgLink = ref("")
const valueSnack = ref(false)
const handleDragOver = (event) => {
  event.preventDefault()
}

const handleDrop = (event) => {
  isUploading.value = true
  event.preventDefault()
  const file = event.dataTransfer.files[0]
  if (file.type == "image/jpeg" || file.type == "image/png" || file.type == "image/jpg") {
    uploadImage(file)
  } else {
    isUploading.value = false
    return
  }
}

const uploadImage = async (image) => {
  const PATH = `${BASE_URL}api/uploadIMG`
  const formData = new FormData()

  // The third parameter is required for server
  formData.append("image", image)

  // Send the compressed image file to server with XMLHttpRequest.
  const response = await axios.post(PATH, formData)
  const data = response.data
  if (data.status) {
    imgLink.value = data.imageUrl
    setTimeout(() => {
      isUploading.value = false
      imgUploaded.value = true
    }, 4000)
  } else {
    isUploading.value = false
  }
}

const copyToClipBoard = () => {
  navigator.clipboard.writeText(imgLink.value)
    .then(() => {
      // Copiado exitoso
      console.log('Texto copiado al portapapeles');
      valueSnack.value = true
    })
    .catch((error) => {
      // Error al copiar
      console.error('Error al copiar el texto al portapapeles', error);
    });
};

const choseAFile = () => {
  const input = document.createElement('input');
  input.type = 'file';
  input.accept = 'image/jpeg, image/png, image/jpg';
  input.onchange = handleFileSelection;
  input.click();
}
const handleFileSelection = (event) => {
  isUploading.value = true

  const file = event.target.files[0];

  if (file) {
    // Procesar el archivo seleccionado aquí
    uploadImage(file);
  }
};
const reset = () => {
  isUploading.value = false
  imgUploaded.value = false
  imgLink.value = ""
}
</script>

<style scoped>
.drop-area {
  width: 300px;
  height: 200px;
  border: 2px dashed gray;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  justify-content: center;
  gap: 8px;
  text-align: center;
}
.svg-image {
  width: 100px;
  height: 100px;
}
.center {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
</style>
