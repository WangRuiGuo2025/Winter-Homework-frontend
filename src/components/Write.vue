<script setup>
import axios from 'axios'
import { ref, inject } from 'vue'
import { ElMessage } from 'element-plus'

const title = ref('')
const content = ref('')

const toHome = inject('toHome')
const userName = inject('userName')

const submit = async () => {
  try {
    const response = await axios.post("https://winter-homework-backend.onrender.com/articles/submit", {
        title: title.value,
        content: content.value,
        publisher: userName.value
    })

    if (response.data.code === 1) {
      toHome()
      ElMessage({
          message: '发布成功',
          type: 'success',
      })
    } else {
      ElMessage({
          message: '发布失败',
          type: 'error',
      })
    }
  } catch (e) {
    ElMessage({
        message: '发布失败',
        type: 'error',
    })
  }
}
</script>

<template>
  <div class="artical">
    <el-text class="artical-text">发文章</el-text>
    <div class="title">
      <el-text class="title-text">标题：</el-text>
      <el-input class="title-input" v-model="title" style="width: 75%;margin-bottom: 10px;" placeholder="请输入标题" />
    </div>
    <div class="content">
      <el-text class="content-text">内容：</el-text> 
      <el-input
        v-model="content"
        class="content-input"
        style="width: 75%"
        :rows="10"
        type="textarea"
        placeholder="请输入文章内容"
      />
    </div>
    <div class="submit">
      <el-button type="primary" @click="submit" style="margin-top: 20px;">提交</el-button>
    </div>
  </div>
</template>

<style scoped>
  .artical{
    display:flex ;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    min-height: calc(100vh - 60px);
    background-color: var(--color-border);
  }

  .title,.content{
    display: flex;
    width: 100%;
    align-items: center;
    justify-content: center;
  }
  
  .artical-text{
    font-size: xx-large;
    font-weight: bolder;
    color: var(--color-text);
  }
  
  .title-text,.content-text{
    font-size: medium;
    font-weight: normal;
    color: var(--color-text);
  }

  .title-input :deep(.el-input__wrapper),
  .content-input :deep(.el-input__wrapper) {
    background-color: var(--color-fff-border-hover) !important;
  }
  .title-input :deep(.el-input__inner){
    color: var(--color-text);
  }

  .content-input :deep(.el-textarea__inner) {
    background-color: var(--color-fff-border-hover) !important;
    color: var(--color-text)!important;
  }

</style>