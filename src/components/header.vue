<template>
  <div class="header">
    <h1 class="title">中国城市旅游指南</h1>
    <el-input
        v-model="cityName"
        placeholder="请输入城市名称"
        size="large"
        style="width: 500px;height: 50px"
        class="input-border-style"
    >
      <template #append>
        <el-button @click="searchCity" size="large" type="success" plain>搜索</el-button>
      </template>
    </el-input>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import { defineEmits } from 'vue';
import { ElMessage } from 'element-plus'
import axios from "axios";

const isCity = ref(false);
const cityName = ref('');
const weatherData=ref()
const emit = defineEmits(['update-city']);
const sendCityUpdate = () => {
  emit('update-city', cityName.value);
};
const getWeather =async () => {
  try {
    const response = await axios.get(`https://api.seniverse.com/v3/weather/daily.json?key=Sm7cDteCVc_DP9wC5&location=${encodeURIComponent(cityName.value)}&language=zh-Hans&unit=c&start=0&days=6`);
    weatherData.value=response.data;
    return true;
  } catch (error) {
    console.error('请求失败:', error); // 打印错误信息
    return false; // 返回 false 表示请求失败
  }

}
const searchCity = async () => {
  if (cityName.value.trim() !== '') {
    // 打印城市名称或执行相应的搜索逻辑
    console.log('搜索的城市:', cityName.value);
    // 这里可以调用 API 或其他逻辑来搜索城市信息
    if (!await getWeather())  {
      ElMessage({
        message: '输入的城市不存在，请重新输入',
        type: 'error',
      });
      isCity.value=false
      return
    }
  } else {
    console.log('请输入有效的城市名称');
    return
  }
  isCity.value=true
  sendCityUpdate()
};
</script>

<style scoped>
>>>.input-border-style .el-input__wrapper {
  background-color: lightgray;
}
.title {
  color: orange;
}
.header {
  margin-bottom: 20px;
  padding-top: 20px;
  text-align: center;
}
</style>



