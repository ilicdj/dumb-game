<script>
import { ref, onMounted } from 'vue'
import gsap from 'gsap'

export default {
  name: 'Preloader',
  setup() {
    const preloader = ref(null) // Vue ref to bind to template

    onMounted(() => {
      setTimeout(() => {
        gsap.to(preloader.value, {
        opacity: 0,
        display: 'none',
        duration: 1,
        ease: 'power4.inOut',
      })
      }, 4000);
    })

    return { preloader }
  },
}
</script>

<template>
  <div id="preloader" ref="preloader">
    <img id="spinner" src="../../src/assets/logotip.svg" alt="" />
  </div>
</template>

<style scoped lang="scss">
@import '../scss/variables.scss';

#preloader {
  display: flex;
  justify-content: center;
  align-items: center;
  position: absolute;
  top: 0;
  left: 0;
  z-index: 888;
  height: 100vh;
  width: 100%;
  background-color: $black;

  #spinner {
    width: 75px;
    animation: spin 3.5s linear infinite;
  }
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
</style>
