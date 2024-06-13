<template>
  <div class="image-merger">
    <form @submit.prevent="mergeImageAndText">
      <div>
        <label for="imageInput">Choisir une image:</label>
        <input type="file" id="imageInput" @change="handleImageUpload" accept="image/*" class="block w-full text-sm text-gray-900 border border-gray-300 rounded-lg cursor-pointer bg-gray-50 dark:text-gray-400 focus:outline-none dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400" />
      </div>
      <div>
        <label for="textInput">Texte:</label>
        <input type="text" id="textInput" v-model="text" class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500" />
      </div>
      <button class="focus:outline-none text-white bg-green-700 hover:bg-green-800 focus:ring-4 focus:ring-green-300 font-medium rounded-lg text-sm px-5 py-2.5 me-2 mb-2 dark:bg-green-600 dark:hover:bg-green-700 dark:focus:ring-green-800" type="submit">Générer l'image</button>
    </form>
    <div v-if="canvasVisible">
      <canvas ref="canvas" width="300" height="600"></canvas>
      <button class="focus:outline-none text-white bg-red-700 hover:bg-red-800 focus:ring-4 focus:ring-red-300 font-medium rounded-lg text-sm px-5 py-2.5 me-2 mb-2 dark:bg-red-600 dark:hover:bg-red-700 dark:focus:ring-red-900" @click="downloadImage">Télécharger l'image</button>
    </div>
  </div>
</template>

<script>
import { saveAs } from 'file-saver';

export default {
  data() {
    return {
      image: null,
      text: '',
      canvasVisible: false,
    };
  },
  methods: {
    handleImageUpload(event) {
      const file = event.target.files[0];
      const reader = new FileReader();

      reader.onload = (e) => {
        this.image = new Image();
        this.image.src = e.target.result;
      };

      if (file) {
        reader.readAsDataURL(file);
      }
    },
    mergeImageAndText() {
      if (this.image && this.text) {
        this.canvasVisible = true; // Ensure canvas is visible before accessing it
        this.$nextTick(() => {
          const canvas = this.$refs.canvas;
          const ctx = canvas.getContext('2d');

          this.image.onload = () => {
            // Clear the canvas
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Calculate the cropping and drawing coordinates
            const aspectRatio = this.image.width / this.image.height;
            const canvasAspectRatio = canvas.width / canvas.height;
            let sourceX, sourceY, sourceWidth, sourceHeight;

            if (aspectRatio > canvasAspectRatio) {
              // Image is wider relative to the canvas
              sourceHeight = this.image.height;
              sourceWidth = this.image.height * canvasAspectRatio;
              sourceX = (this.image.width - sourceWidth) / 2;
              sourceY = 0;
            } else {
              // Image is taller relative to the canvas
              sourceWidth = this.image.width;
              sourceHeight = this.image.width / canvasAspectRatio;
              sourceX = 0;
              sourceY = (this.image.height - sourceHeight) / 2;
            }

            // Draw the image on the canvas
            ctx.drawImage(
              this.image,
              sourceX,
              sourceY,
              sourceWidth,
              sourceHeight,
              0,
              0,
              canvas.width,
              canvas.height
            );

            // Draw the text on the canvas
            ctx.font = '30px Arial';
            ctx.fillStyle = 'black';
            ctx.fillText(this.text, 50, 50);
          };

          if (this.image.complete) {
            // If the image is already loaded, manually trigger the onload handler
            this.image.onload();
          }
        });
      }
    },
    downloadImage() {
      const canvas = this.$refs.canvas;
      canvas.toBlob((blob) => {
        saveAs(blob, 'image_with_text.png');
      });
    },
  },
};
</script>

<style>
.image-merger {
  display: flex;
  flex-direction: column;
  align-items: center;
}
</style>
