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


const isCity = ref(false);
const infoSource = ref('维基百科');
const generalInfo = ref();
const weatherData = ref();
const mapUrl = ref('');
const foodData =ref()
const schoolData = ref()
const scenicList =ref()
const advice=ref()


//询问通义简介
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
    // 使用 axios 发送 POST 请求
    const response = await axios.post(apiEndpoint, requestBody, {
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${apiKey}`
      }
    });
    console.log(response.data.choices[0].message.content);
    // 假设 API 返回的数据结构包含回答在 choices 数组中
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

// 调用函数时传递内容

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
      // 调用通义千问 API (需配置代理或使用实际接口)
      response=await askTongyi(`给我${cityName.value}的简介，字数在三百字左右，给我一段话不要分段`)
      generalInfo.value = response; // 将简介内容赋值给 generalInfo
    }
  } catch (error) {
    generalInfo.value = '无法获取信息';
    console.error(error);
  }
};

 const getScreen =async () => {
//   const response = await axios.get(`https://route.showapi.com/1681-1?appKey=D20Bcb27196B4E18927C4FB3C46C7c72&key=${encodeURIComponent(cityName.value)}&pageSize=5&page=1`);
//   scenicList.value = response.data;
}

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
  advice.value=await askTongyi(`给我${cityName.value}的旅游建议，包括几月份合适旅游，住在哪里等，字数在200字左右，给我一段话不要分段`)
}

</script>;

<style scoped>

/* 自定义 el-input 的背景颜色为灰色 */

.city-info-container {
  width: 80%;
  margin: 0 auto;
}

body {
  min-height: 1000px;
  background-image: url('@/assets/images.jpg'); /* 使用本地图片路径 */
  background-size: cover;       /* 背景图适应整个窗口 */
  background-repeat: no-repeat; /* 防止背景图片重复 */
  background-attachment: fixed; /* 背景图固定，不随页面滚动 */
  background-position: center;  /* 背景图片居中对齐 */
}
</style>