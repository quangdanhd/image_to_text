<template>
  <div class="container mt-2 image-to-text">
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
                ref="fileInput"
                class="form-control d-none"
                type="file"
                accept="image/png, image/gif, image/jpeg"
              />
            </div>
            <div class="col-12">
              <div class="row g-2">
                <div class="col-8">
                  <input
                    v-model="imgLink"
                    type="text"
                    class="form-control"
                    placeholder="Paste image link"
                  />
                </div>
                <div class="col-4">
                  <button
                    @click="submitRecognition($event)"
                    type="submit"
                    class="btn btn-outline-primary w-100"
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
          {{ message }}
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
      imgLink: null,
      imgPreview: null,
      imgLoadError: false,
      loading: false,
      error: false,
      message: null,
    };
  },
  methods: {
    langChange(key) {
      this.activeLang = key;
    },
    imageChanged() {
      if (!this.imgLink) {
        return;
      }
      this.message = null;
      this.loading = true;
      this.imgLoadError = false;
      this.imgPreview = this.imgLink;
    },
    onImgLoad() {
      if (this.imgLoadError) {
        this.loading = false;
        return;
      }
      console.log(1);
      this.imageRecognition();
    },
    replaceByErrorImg(e) {
      this.imgLoadError = true;
      e.target.src = require("./assets/error.png");
    },
    imageRecognition() {
      try {
        Tesseract.recognize(this.imgPreview, this.activeLang, {
          logger: (m) => {
            console.log(m);
          },
          errorHandler: (err) => {
            console.log(err);
            return err;
          },
        }).then(({ data: { text } }) => {
          this.loading = false;
          console.log(text);
        });
      } catch (e) {
        this.error = true;
        this.message = e.message;
      }
    },
    submitRecognition(e) {
      e.preventDefault();
    },
  },
  watch: {
    imgLink() {
      this.imageChanged();
    },
  },
};
</script>
