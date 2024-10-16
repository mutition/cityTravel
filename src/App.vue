<template>
  <body>

  <div  class="city-info-container">
    <he @update-city="handleCityUpdate"></he>
    <content :city-name="cityName"></content>
  </div>
  </body>
</template>

<script setup>
import he from './components/header.vue';
import content from './components/content.vue';
import { ref } from 'vue';
import axios from 'axios';


const cityName = ref('');

// 定义处理子组件事件的方法
const handleCityUpdate = (newCity) => {
  cityName.value = newCity;
};


</script>;

<style scoped>

.city-info-container {
  width: 80%;
  margin: 0 auto;
}

body {
  min-height: 1000px;
  background-image: url('@/assets/images.jpg');
  background-size: cover;
  background-repeat: no-repeat;
  background-attachment: fixed;
  background-position: center;
}
</style>