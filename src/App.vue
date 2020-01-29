<template>
  <div>
    <v-app>
      <v-container>
        <h2>Image Text OCR</h2>
        <v-col cols="12" style="height: 100%; height: 60px;">
          <v-file-input
            v-model="imageFile"
            accept="image/*"
            label="Select Image File"
            prepend-icon="mdi-camera"
            @change="onImagePicked()">
          </v-file-input>
        </v-col>
        <v-col v-show="imageFile != null">
          <v-row>
            <v-col cols="9" md="9">
              <div id="crop-image-block" style="height:450px;"></div>
            </v-col>
            <v-col cols="3" md="3" align="right">
              <v-select
                v-model="recognizeLanguage"
                :items="languageItems"
                menu-props="auto"
                label="Language"
              ></v-select>
              <p>
                <v-btn small dark color="warning" v-on:click="recognize()">Recognize</v-btn>
              </p>
              <v-textarea
                outlined
                label="Recognize Output"
                v-model="recognizeOutput"
                rows="10"
              ></v-textarea>
            </v-col>
          </v-row>
        </v-col>
      </v-container>
      <v-overlay opacity="0.85" :value="overlay">
        <v-progress-circular rotate="-90" size="100" width="15" :value="recognizeProgress" color="amber">
          {{ recognizeProgress }} %
        </v-progress-circular>
      </v-overlay>
    </v-app>
  </div>
</template>

<script>
import Cropper from 'cropperjs';
import 'cropperjs/dist/cropper.css';
import Tesseract from 'tesseract.js';
import TesseractConfig from './configs/tesseract-config.js';

export default {
  name: 'app',
  data: function () {
    return {
      recognizeLanguage: 'eng',
      imageFile: null,
      cropper: null,
      recognizeOutput: '',
      languageItems: TesseractConfig.languages,
      overlay: false,
      recognizeProgress: 0,
    };
  },
  mounted: function () {
  },
  methods: {
    initCanvas: function() {
      let canvas = document.createElement('canvas');
      let cropperBlock = document.querySelector('#crop-image-block');
      while (cropperBlock.firstChild) {
        cropperBlock.removeChild(cropperBlock.firstChild);
      }
      cropperBlock.appendChild(canvas);
      this.canvas = canvas;
    },
    onImagePicked: function() {
      this.initCanvas();
      let selectedFile = this.imageFile;
      let reader = new FileReader();
      reader.onload = () => {
        let image = new Image();
        image.src = reader.result;
        image.onload = () => {
          this.canvas.width = image.width;
          this.canvas.height = image.height;
          this.canvas.getContext('2d').drawImage(image, 0, 0, this.canvas.width, this.canvas.height);
          this.cropper = new Cropper(this.canvas, {});
        };
      }
      reader.readAsDataURL(selectedFile);
    },
    recognize: function() {
      let dataUrl = this.cropper.getCroppedCanvas().toDataURL('image/jpeg');
      this.overlay = true;
      this.recognizeProgress = 0;
      Tesseract.recognize(dataUrl, this.recognizeLanguage, {
        logger: (m) => {
          if (m.status == "recognizing text") {
            this.recognizeProgress = Math.round(m.progress * 100);
          }
        }
      }).then(({ data: { text } }) => {
        this.overlay = false;
        this.recognizeOutput = text;
      });
    }
  },
  components: {
  }
}
</script>

<style>
</style>
