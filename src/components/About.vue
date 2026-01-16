<script setup>
import axios from 'axios';
import { onMounted, ref } from 'vue';
import { ElMessage } from 'element-plus';

//我看了我这个应该不是最优解，我看AI是给我cardData的数据定义，不清楚有啥优点
//反正写完：）
const title = ref('')
const body = ref('')
const footer = ref('')

const getAbout = async() => {
  try {
    const Response = await axios.get("http://localhost:8000/about")
    if (Response.data) {
      title.value = Response.data.title;
      body.value = Response.data.body;
      footer.value = Response.data.footer;
    }
  } catch (e) {
    ElMessage({
      message: '获取失败',
      type: 'error',
    })
  }
}

onMounted(() => {
  getAbout()
})
    
</script>

<template>
  <div class="about">
    <el-card style="width: 480px;background-color:var(--color-f-background-soft);">
      <template #header>
        <div class="card-header">
          <span class="card-header">{{title}}</span>
        </div>
      </template>
      <p class="text-item">{{body}}</p>
      <template #footer class="footer">{{ footer }}</template>
    </el-card>
  </div>
</template>

<style scoped>
  .about{
    display:flex ;
    justify-content: center;
    min-height: calc(100vh - 60px);
    flex-direction: column;
    align-items: center;
  }

  .card-header{
    font-size: xx-large;
    font-weight: bolder;
  }
  
  .text-item,.footer{
    font-size: medium;
    font-weight: normal;
  }
  
</style>