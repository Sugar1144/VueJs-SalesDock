<template>
  <!-- <img alt="Vue logo" src="./assets/logo.png"> -->
  <DiscountMain />
</template>

<script>
import DiscountMain from './components/DiscountMain.vue'

export default {
  name: 'App',
  components: {
    DiscountMain
  }
}
</script>

<style lang="scss">
#app {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #333333;
}

@import url('https://fonts.googleapis.com/css2?family=Lato:wght@300;400;700;900&display=swap');
@import url('https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.6.0/css/all.min.css');

body,
html {
  background-color: #F3F3F5 !important;
  font-size: 14px;
  font-family: 'Lato', sans-serif !important;
}

.modal-content {
  border-radius: 0 !important;

  .modal-title {
    font-size: 20px;
    line-height: 28px;
    font-weight: 700;
  }

  .modal-footer {
    display: none;
  }

  .btn-close {
    width: 0.5em;
    height: 0.5em;
    box-sizing: border-box;
    margin-right: 5px !important;
  }
}
</style>
