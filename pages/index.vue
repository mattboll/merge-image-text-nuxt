<template>
  <div class="image-merger">
    <form @submit.prevent="mergeImageAndText">
      <div>
        <label for="imageInput">Choisir une image:</label>
        <input type="file" id="imageInput" @change="handleImageUpload" accept="image/*" class="text-sm text-stone-500
   file:mr-5 file:py-1 file:px-3 file:border-[1px]
   file:text-xs file:font-medium
   file:bg-stone-50 file:text-stone-700
   hover:file:cursor-pointer hover:file:bg-blue-50
   hover:file:text-blue-700" />
      </div>
      <div>
        <label for="textInput">Texte:</label>
        <input type="text" id="textInput" v-model="text" class="bg-gray-200 appearance-none border-2 border-gray-200 rounded w-full py-2 px-4 text-gray-700 leading-tight focus:outline-none focus:bg-white focus:border-purple-500" />
      </div>
      <div>
        <label for="textInput">Couleur du texte:</label>
        <input type="color" id="colorPicker" v-model="textColor" />
      </div>
      <button class="flex-shrink-0 bg-teal-500 hover:bg-teal-700 border-teal-500 hover:border-teal-700 text-sm border-4 text-white py-1 px-2 rounded" type="submit">Générer l'image</button>
    </form>
    <div v-if="canvasVisible">
      <canvas ref="canvas" width="360" height="540"></canvas>
      <button class="flex-shrink-0 bg-teal-500 hover:bg-teal-700 border-teal-500 hover:border-teal-700 text-sm border-4 text-white py-1 px-2 rounded" @click="downloadImage">Télécharger l'image</button>
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
      textColor: '#FF4A52',
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
            ctx.fillStyle = this.textColor;

            const textWidth = ctx.measureText(this.text ).width;

            ctx.fillText(this.text, (canvas.width / 2) - (textWidth / 2), 50);
            const soutient = "soutient";
            const soutientWidth = ctx.measureText(soutient ).width;
            ctx.fillText(soutient, (canvas.width / 2) - (soutientWidth / 2), 100);
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
        saveAs(blob, 'soutient_front_populaire.png');
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
