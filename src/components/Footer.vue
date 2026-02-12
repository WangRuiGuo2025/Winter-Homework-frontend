<script setup>
import { ref, onMounted } from 'vue'

const years = ref("")
const year = 2025
onMounted(() => {
  const currentYear = new Date().getFullYear()
  if(currentYear === year){
    years.value = year
  }else{
    years.value = `${year} - ${currentYear}`
  }
})
</script>

<template>
  <el-footer class="custom-footer">
    <el-row class="footer-row">
      <el-col :span="24">
        <div class="footer-text">
          <p class="footer-year">© {{ years }} 程序部知识库. 保留所有权利.</p>
          <p class="footer-text">分享技术见解，记录开发历程</p>
        </div>
      </el-col>
    </el-row>
  </el-footer>
</template>

<style scoped>
.custom-footer {
  height: 120px;
  background-color: var(--color-background-soft);
  color: var(--color-f-background-soft);
  padding: 20px 0;
}

.footer-row {
  justify-content: center;
  height: 100%;
}

.footer-text {
  text-align: center;
  font-size: small;
  opacity: 0.8;
}

.footer-year {
  font-size: medium;
  margin-bottom: 10px;
}

.footer-text {
  font-size: small;
}
</style>