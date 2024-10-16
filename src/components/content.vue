
<template>
  <div v-if="cityName" class="content">
    <el-card class="general-info">
      <h3 class="weather-title">基本信息</h3>
      <el-radio-group v-model="infoSource" @change="changeInfoSource" class="weather-radio-group">
        <el-radio-button value="维基百科">维基百科</el-radio-button>
        <el-radio-button value="通义千问">通义千问</el-radio-button>
      </el-radio-group>

      <el-card >
        <h1 class="info-card" >{{cityName}}</h1>
        <p style="line-height: 1.5;max-width: 800px">{{ generalInfo }}</p>
      </el-card>
    </el-card>

    <el-card class="general-info">
      <h3 class="weather-title">景点</h3>
      <el-carousel v-if="scenicList" arrow="never" height="250px" :autoplay="false" class="carousel-padding" indicator-position="outside">
        <el-carousel-item v-for="(scenic, index) in scenicList.showapi_res_body.result" :key="index">
          <div class="scenic-item">
            <img :src="scenic.newPicUrl" alt="scenic image" class="scenic-image" />
            <div class="scenic-info">
              <h3>{{ scenic.scenicName }}</h3>
              <p><strong>地址:</strong> {{ scenic.address }}</p>
              <p><strong>开放时间:</strong> {{ scenic.bizTime }}</p>
              <p><strong>票价:</strong> {{ scenic.salePrice }} 元</p>
            </div>
          </div>
        </el-carousel-item>
      </el-carousel>
    </el-card>


    <div class="sections">
      <el-card class="section-card">
        <h3 class="weather-title">天气预报</h3>
        <el-carousel v-if="weatherData" arrow="never" height="200px" :autoplay="false"  indicator-position="outside">
          <el-carousel-item v-for="(day, index) in weatherData.results[0].daily" :key="index"  :style="{ backgroundColor: getBackgroundColor(day.text_day) }">
            <div class="weather-info">
              <div class="weather-location">{{ weatherData.results[0].location.name }}</div>
              <div class="weather-date">{{ day.date }}</div>
              <div class="weather-temperature">{{ day.text_day }}</div>
              <div class="weather-temperature">最高 {{ day.high }}° 最低 {{ day.low }}°</div>
              <div class="weather-details">风向: {{ day.wind_direction }} 风速: {{ day.wind_speed }}km/h</div>
            </div>
          </el-carousel-item>
        </el-carousel>
      </el-card>

      <el-card  class="section-card">
        <div class="advice-setting">
          <h3 class="weather-title">旅游建议</h3>
          <span @click="getAdvice">
        <el-icon size="25" color="blue"><Refresh /></el-icon>
      </span>
        </div>
        <p class="advice-item">{{ advice }}</p>
      </el-card>

      <el-card class="section-card">
        <h3 class="weather-title">美食</h3>
        <el-carousel v-if="foodData" arrow="never" height="200px" :autoplay="false" class="carousel-padding" indicator-position="outside">
          <el-carousel-item v-for="(meal, index) in foodData.showapi_res_body.mealList" :key="index">
            <div class="meal-item">
              <h3>{{ meal.name }}</h3>
              <p class="food-desc">{{ meal.description }}</p>
            </div>
          </el-carousel-item>
        </el-carousel>
      </el-card>

      <el-card class="section-card">
        <h3 class="weather-title">高校</h3>
        <el-carousel arrow="never" height="200px" :autoplay="false" class="carousel-padding" indicator-position="outside">
          <el-carousel-item v-if='schoolData' v-for="(school, index) in schoolData" :key="index">
            <div class="school-info">
              <h3>{{ school.school_name }}</h3>
              <p><strong>城市:</strong> {{ school.school_city }}</p>
              <p><strong>层次:</strong> {{ school.school_level }}</p>
              <p><strong>官网:</strong> <a :href="school.school_website" target="_blank">{{ school.school_website }}</a></p>
            </div>
          </el-carousel-item>
        </el-carousel>
      </el-card>
    </div>

    <el-card class="map-section">
      <h3>在线地图</h3>
      <div class="map-container">
        <iframe :src="mapUrl" width="100%" height="400"></iframe>
      </div>
    </el-card>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';
import { defineProps, watch } from 'vue';
const cityName = ref();
// 使用 defineProps 接收来自父组件的cityName
const props = defineProps({
  cityName: String
});


// 发现cityName改变时调用各api
watch(() => props.cityName, (newValue) => {
  cityName.value = newValue;
  changeInfoSource();
  getWeather();
  getMap();
  getFood();
  getSchool();
  getScreen();
  getAdvice();
});

const infoSource = ref('维基百科');
const generalInfo = ref();
const weatherData = ref();
const mapUrl = ref('');
const foodData =ref()
const schoolData = ref()
const scenicList =ref()
const advice=ref()


const askTongyi = async (content) => {
  const apiEndpoint = 'https://dashscope.aliyuncs.com/compatible-mode/v1/chat/completions';
  const apiKey = 'sk-1fc2f2739d444a1690d390e9cfdd8b0c';

  const requestBody = {
    model: 'qwen-coder-turbo',
    messages: [
      {
        role: 'system',
        content: 'You are a helpful assistant.'
      },
      {
        role: 'user',
        content: content
      }
    ]
  };

  try {
    const response = await axios.post(apiEndpoint, requestBody, {
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${apiKey}`
      }
    });
    console.log(response.data.choices[0].message.content);
    if (response.data && response.data.choices && response.data.choices.length > 0) {
      return response.data.choices[0].message.content;
    } else {
      return '无法获取回答';
    }
  } catch (error) {
    console.error('请求失败:', error);
    return '请求失败，请检查网络或 API 配置';
  }
};

const changeInfoSource = async () => {
  console.log(`切换信息来源到 ${infoSource.value}`);
  generalInfo.value=(`切换信息来源到 ${infoSource.value}`);
  if (!cityName.value.trim()) {
    return;
  }
  try {
    let response;
    if (infoSource.value === '维基百科') {
      // 调用维基百科 API
      response = await axios.get(`https://zh.wikipedia.org/api/rest_v1/page/summary/${encodeURIComponent(cityName.value)}`);
      generalInfo.value = response.data.extract;// 将简介内容赋值给 generalInfo
    } else if (infoSource.value === '通义千问') {
      // 调用通义千问 API
      response=await askTongyi(`给我${cityName.value}的简介，字数在两百字左右，给我一段话不要分段`)
      generalInfo.value = response; // 将简介内容赋值给 generalInfo
    }
  } catch (error) {
    generalInfo.value = '无法获取信息';
    console.error(error);
  }
};

const getScreen =async () => {
   const response = await axios.get(`https://route.showapi.com/1681-1?appKey=D20Bcb27196B4E18927C4FB3C46C7c72&key=${encodeURIComponent(cityName.value)}&pageSize=5&page=1`);
   scenicList.value = response.data;
}

const getWeather =async () => {
  try {
    const response = await axios.get(`https://api.seniverse.com/v3/weather/daily.json?key=Sm7cDteCVc_DP9wC5&location=${encodeURIComponent(cityName.value)}&language=zh-Hans&unit=c&start=0&days=6`);
    weatherData.value=response.data;
    return true;
  } catch (error) {
    console.error('请求失败:', error);
    return false;
  }

}

const getFood = async () => {
  const response=await axios.get(`https://route.showapi.com/3215-1?appKey=D20Bcb27196B4E18927C4FB3C46C7c72&area=${encodeURIComponent(cityName.value)}`);
  foodData.value=response.data
}

const getSchool = async () => {
  const response=await axios.get(`http://hn.api.yesapi.net/?s=App.Common_University.Search&return_data=5&school_name=大学&school_city=${encodeURIComponent(cityName.value)}&top_num=5&app_key=0DFA0B6E0D51F7CDE4614078056DC309&sign=347D7082C559F467724F52D941FDA5DE`);
  schoolData.value=response.data.data.schools
}

const getMap = async() => {
  mapUrl.value = `https://www.google.com/maps?q=${cityName.value}&output=embed`;
}

const getAdvice = async () => {
  advice.value="正在生成旅游建议..."
  advice.value=await askTongyi(`给我${cityName.value}的旅游建议，包括几月份合适旅游，住在哪里等，字数在200字左右，给我一段话不要分段`)
}

//按照天气信息选择卡片背景颜色
const getBackgroundColor = (weather) => {
  switch (weather) {
    case '晴':
      return 'lightyellow';
    case '多云':
      return 'lightgray';
    default:
      return 'lightblue';
  }
};
</script>

<style scoped>
.content {
  display: flex;
  flex-direction: column;
}
.general-info {
  margin-bottom: 10px;
}

.info-card{
  margin-top: 0;
}
.sections {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
  margin-bottom: 20px;
  justify-content: space-between;
}


.section-card {
  width: 48%;
}

.scenic-item {
  display: flex;
  align-items: center;
  padding-left: 275px;
  padding-top: 20px;
}

.scenic-image {
  width: 40%;
  max-width: 200px;
  height: auto;
  margin-right: 40px;
  border-radius: 8px;
}

.scenic-info {
  flex: 1;
}

.scenic-info h3 {
  font-size: 24px;
  margin-bottom: 7px;
}

.scenic-info p {
  font-size: 16px;
  margin: 5px 0;
  max-width: 250px;
}
.weather-radio-group{
  margin-bottom: 10px;
}
.weather-title {
  text-align: left;
  margin-top: 0;
  font-weight: bold;
  margin-bottom: 10px;
  font-size: 24px;
}

.weather-card {
  margin-bottom: 20px;
  padding: 0 30px 0 0;
  text-align: center;
  color: #000;
}

.weather-info {
  padding-left: 50px;
  padding-top: 10px;
  font-size: 16px;
}

.weather-location {
  font-size: 18px;
  font-weight: bold;
}

.weather-date {
  font-size: 14px;
  margin-bottom: 5px;
}

.weather-temperature {
  font-size: 24px;
  font-weight: bold;
  margin: 10px 0;
}

.weather-details {
  font-size: 14px;
  margin-bottom: 10px;
}

.weather-extra {
  display: flex;
  justify-content: center;
  gap: 10px;
}

.food-desc {
  line-height: 1.5;
  max-width: 350px;
}
.carousel-padding {
  padding-top: 0;
  padding-left: 50px;
}

.carousel-item-padding{
  padding-top: 20px;
  margin-left: 50px;
}

.advice-setting{
  display: flex;
  align-items: center;
  gap: 10px;
}

.advice-item{
  padding: 0 20px 0 20px;
  line-height: 1.5;
}
.map-section {
  margin-bottom: 20px;
}

.map-card {
  height: 300px;
}

#map-container {
  height: 100%;
  background-color: #f0f0f0;
  display: flex;
  align-items: center;
  justify-content: center;
}

</style>
