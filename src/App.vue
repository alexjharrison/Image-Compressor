<template>
  <div id="app">
    <h2 class="text-center mt-5 mb-4">STI Super Awesome Image Compressor</h2>
    <div class="text-center my-4 mx-auto contain d-flex align-items-center justify-content-center">
      <b-form-file
        @change="compressImage"
        browse-text="Upload Full Size Images"
        placeholder="Choose files or drop them here..."
        drop-placeholder="Drop file here..."
        multiple
        accept="image/*"
      />
    </div>
    <b-input-group
      class="contain mb-2 mx-auto d-flex align-items-center"
      v-for="(imgSetting,i) in imgSettings"
      :key="i"
    >
      <label class="mx-2">max height(px)</label>
      <b-input type="number" v-model="imgSetting.maxHeight" />
      <label class="mx-2">max width(px)</label>
      <b-input type="number" v-model="imgSetting.maxWidth" />
      <label class="mx-2">suffix</label>
      <b-input type="text" v-model="imgSetting.suffix" />
      <b-button class="mx-2" size="sm" variant="danger" @click="imgSettings.splice(i,1)">Remove</b-button>
    </b-input-group>
    <div class="text-center mt-3 d-flex justify-content-center">
      <!-- <b-form-checkbox class="mx-5" id="check" v-model="convertToJPEG">Convert To JPEG</b-form-checkbox> -->
      <b-button @click="addSize" variant="primary" size="sm" class="text-center">Add output size +</b-button>
    </div>
    <div class="images d-flex align-items-center justify-content-center m-5">
      <b-img :class="`${twoSecondToggle}`" fluid src="@/assets/bim-with-text.png" />
      <b-img :class="`${!twoSecondToggle}`" fluid src="@/assets/EZ-Path-w-plans.png" />
    </div>
  </div>
</template>

<script>
const JSZip = require("jszip");
import Compressor from "compressorjs";
import { saveAs } from "file-saver";
export default {
  name: "app",
  data() {
    return {
      convertToJPEG: false,
      twoSecondToggle: false,
      zip: new JSZip(),
      imgSettings: [
        {
          maxHeight: 1000,
          maxWidth: 1000,
          suffix: ""
        },
        {
          maxHeight: 2000,
          maxWidth: 2000,
          suffix: "@2x"
        },
        {
          maxHeight: 3000,
          maxWidth: 3000,
          suffix: "@3x"
        },
        {
          maxHeight: 100,
          maxWidth: 100,
          suffix: "@0x"
        }
      ]
    };
  },
  methods: {
    compressImage({ target }) {
      const oldFiles = target.files;
      let count = 0;
      let numImages = this.imgSettings.length * oldFiles.length;
      console.log(oldFiles);
      for (let i = 0; i < oldFiles.length; i++) {
        this.imgSettings.map(imgSetting => {
          new Compressor(oldFiles[i], {
            maxWidth: imgSetting.maxWidth,
            maxHeight: imgSetting.maxHeight,
            // convertSize: this.convertToJPEG ? 5000 : Infinity,
            convertSize: Infinity,

            success: result => {
              const [filename, extension] = oldFiles[i].name.split(".");
              const newName = `${filename}${imgSetting.suffix}.${extension}`;

              this.zip.file(newName, result);
              console.log({ newName, result });
              count++;
              if (count === numImages) {
                this.exportZip();
              }
            }
          });
        });
      }
    },

    addSize() {
      this.imgSettings.push({
        maxHeight: null,
        maxWidth: null,
        suffix: null
      });
    },
    exportZip() {
      console.log(this.zip);
      this.zip.generateAsync({ type: "blob" }).then(function(content) {
        saveAs(content, "images.zip");
      });
    }
  },
  mounted() {
    setTimeout(() => {
      this.twoSecondToggle = !this.twoSecondToggle;
    }, 0);
    setInterval(() => {
      this.twoSecondToggle = !this.twoSecondToggle;
    }, 2000);
  }
};
</script>

<style>
body {
  background: linear-gradient(
    to right bottom,
    #5eff00,
    #2989d8,
    #ff8800,
    #7db9e8
  );
  height: 100vh;
  overflow: hidden;
}
img {
  max-height: 200px;
  transition: transform 2s ease-in-out;
  margin: 0 50px;
}

.contain {
  max-width: 50%;
}

.true {
  transform: rotate(15deg);
}
.false {
  transform: rotate(-15deg);
}

@media only screen and (max-width: 1200px) {
  .contain {
    max-width: 80%;
  }
}
@media only screen and (max-width: 800px) {
  .contain {
    max-width: 100%;
  }
}
</style>
