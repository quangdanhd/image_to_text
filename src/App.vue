<template>
  <img alt="Vue logo" src="./assets/logo.png" />
  <button
    class="btn btn-primary"
    data-bs-target="#collapseTarget"
    data-bs-toggle="collapse"
  >
    Bootstrap collapse
  </button>
  <div class="collapse py-2" id="collapseTarget">
    This is the toggle-able content!
  </div>
</template>

<script>
// import Tesseract from 'tesseract.js';

export default {
  name: "App",
  components: {},
  created() {
    // Tesseract.recognize(
    //     'https://tesseract.projectnaptha.com/img/eng_bw.png',
    //     'eng',
    //     { logger: m => console.log(m) }
    // ).then(({ data: { text } }) => {
    //   console.log(text);
    // })
  },
};
</script>
