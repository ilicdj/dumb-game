<script setup>
import Scene from './components/Scene.vue'
import Game from './components/Game.vue'
import { ref } from 'vue'
import Preloader from './components/Preloader.vue';

const currentClickCount = ref(0)

const handleClickCount = (count) => {
  currentClickCount.value = count
}
</script>

<template>
  <Preloader />
  <Scene :clickCount="currentClickCount" />
  <Game @update-click-count="handleClickCount" />
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
</style>
