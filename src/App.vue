<template>
  <div class="p-3 pt-2 image-to-text">
    <div class="mb-2 fs-4 fw-bold">Image to Text</div>
    <div class="row g-3">
      <div class="col-12 col-md-5 left">
        <!--Type-->
        <div class="d-flex">
          <div
            v-for="(item, k) in lang"
            @click="langChange(item.key)"
            :key="k"
            class="me-1 rounded p-2 tool-option"
            role="button"
            :class="{
              active: activeLang === item.key,
              'ms-0': k === 0,
              'ms-1': k !== 0,
            }"
          >
            <span class="me-1">{{ item.name }}</span>
          </div>
        </div>
        <!--Input-->
        <div class="mt-3">
          <form class="row g-2">
            <div class="col-12">
              <span
                @click="$refs.fileInput.click()"
                class="link-primary text-decoration-underline"
                role="button"
                >upload a file</span
              ><span class="ms-2">OR</span>
              <input
                @change="changeFile"
                ref="fileInput"
                class="form-control d-none"
                type="file"
                accept="image/png, image/gif, image/jpeg"
                :disabled="loading"
              />
            </div>
            <div class="col-12">
              <div class="row g-2">
                <div class="col-8">
                  <input
                    @paste="imagePaste($event)"
                    v-model="imgLink"
                    type="text"
                    class="form-control"
                    placeholder="Paste image or paste image link"
                    :disabled="loading"
                    autofocus
                  />
                </div>
                <div class="col-4">
                  <button
                    @click="submitRecognition($event)"
                    type="submit"
                    class="btn btn-outline-primary w-100"
                    :disabled="loading"
                  >
                    Text Recognition
                  </button>
                </div>
              </div>
            </div>
          </form>
        </div>
        <!--Image Preview-->
        <div class="mt-5 text-center">
          <img
            v-if="imgPreview"
            class="rounded img-preview"
            :src="imgPreview"
            alt=""
            @load="onImgLoad"
            @error="replaceByErrorImg"
          />
        </div>
      </div>
      <div class="col-12 col-md-7 position-relative right">
        <div
          v-if="message"
          class="text-center mb-3 small"
          :class="{ 'text-danger': error }"
        >
          <span v-html="message"></span>
        </div>
        <div class="mt-3 text-center" style="white-space: break-spaces">
          {{ textResult }}
        </div>
        <div v-if="textResult" class="position-absolute top-0 end-0 me-2">
          <i
            v-if="!copied"
            @click="copyText"
            class="bi bi-clipboard"
            role="button"
          ></i>
          <i v-else class="bi bi-check2-circle"></i>
        </div>
        <HexagonLoader v-if="loading" />
      </div>
    </div>
  </div>
</template>

<script>
import HexagonLoader from "@/components/HexagonLoader.vue";
import Tesseract from "tesseract.js";

export default {
  name: "App",
  components: {
    HexagonLoader,
  },
  data() {
    return {
      lang: [
        {
          key: "eng",
          name: "English",
        },
        {
          key: "vie",
          name: "Vietnamese",
        },
      ],
      activeLang: "eng",
      // img
      imgWatch: false,
      imgLink: null,
      imgPreview: null,
      imgLoadError: false,
      // loading & message
      loading: false,
      error: false,
      message: null,
      // text result
      textResult: null,
      copied: false,
    };
  },
  methods: {
    langChange(key) {
      this.activeLang = key;
    },
    changeFile(e) {
      const file = e.target.files[0];
      this.clean();
      this.imgPreview = URL.createObjectURL(file);
      this.imageRecognition(file);
    },
    clean() {
      this.message = null;
      this.loading = true;
      this.imgLoadError = false;
      this.textResult = null;
    },
    imagePaste(e) {
      const dT = e.clipboardData || window.clipboardData;
      if (typeof dT.files[0] == "undefined") {
        this.imgWatch = !this.imgWatch;
      } else {
        const file = dT.files[0];
        const fileType = file["type"];
        const validImageTypes = ["image/gif", "image/jpeg", "image/png"];
        if (!validImageTypes.includes(fileType)) {
          this.error = true;
          this.message =
            "The file format is not an image file. Please check again.";
          return;
        }
        this.clean();
        this.imgPreview = URL.createObjectURL(file);
        this.imageRecognition(file);
      }
    },
    imageChanged() {
      if (!this.imgLink) {
        return;
      }
      this.message = null;
      this.loading = true;
      this.imgLoadError = false;
      this.textResult = null;
      this.imgPreview = this.imgLink;
    },
    onImgLoad() {
      if (this.imgLoadError) {
        this.loading = false;
        return;
      }
      const vm = this;
      const xhr = new XMLHttpRequest();
      xhr.onreadystatechange = function () {
        if (xhr.readyState === XMLHttpRequest.DONE) {
          if (xhr.status !== 200) {
            vm.error = true;
            vm.message =
              "Unable to recognize text via link.<br>Please copy and paste into the input area,<br>or download the image to your device then use the photo upload feature.";
            vm.loading = false;
            return;
          }
          vm.imageRecognition();
        }
      };
      xhr.open("GET", this.imgPreview, true);
      xhr.send(null);
    },
    replaceByErrorImg(e) {
      this.imgLoadError = true;
      e.target.src = require("./assets/error.png");
    },
    imageRecognition(file = null) {
      let img = this.imgPreview;
      if (file) {
        img = file;
      }
      Tesseract.recognize(img, this.activeLang, {
        logger: (m) => {
          console.log(m);
        },
      }).then(({ data: { text } }) => {
        this.loading = false;
        this.textResult = text;
      });
    },
    submitRecognition(e) {
      e.preventDefault();
      this.imageChanged();
    },
    copyText() {
      this.copied = true;
      setTimeout(() => {
        this.copied = false;
      }, 2000);
      // Copy
      const dummy = document.createElement("textarea");
      document.body.appendChild(dummy);
      dummy.value = this.textResult;
      dummy.select();
      document.execCommand("copy");
      document.body.removeChild(dummy);
    },
  },
  watch: {
    imgWatch() {
      setTimeout(() => {
        this.imageChanged();
      }, 100);
    },
  },
};
</script>
