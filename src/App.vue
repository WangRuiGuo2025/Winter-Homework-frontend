<script setup>
  import Login from './components/Login.vue';
  import Register from './components/Register.vue';
  import Home from './components/Home.vue';
  import About from './components/About.vue';
  import Write from './components/Write.vue';
  import Footer from './components/Footer.vue';

  import { Search } from '@element-plus/icons-vue';
  import { ref, provide, onMounted, h} from 'vue'
  import { ElLoading , ElNotification } from 'element-plus';

  const userName = ref('')
  const PageStatus = ref('Home')
  const activeIndex = ref('Home')
  const Status = ref('None')
  const SearchContent = ref('') 
  const fullscreenLoading = ref(false)
  
  //（AI帮助）
  const triggerSearch = ref(() => {})

  //（AI帮助）
  const handleSearch = () => {
    // 先去掉首尾空格
    const keyword = SearchContent.value.trim()
    if (PageStatus.value === 'Home') {
      // 直接调用搜索函数
      triggerSearch.value(keyword)
    } else {
      toHome()
      setTimeout(() => {
        triggerSearch.value(keyword)
      }, 100)
    }
  }

  let loadingDelay = 0

  // （AI帮助）随机数公式：Math.floor(Math.random() * (最大值 - 最小值)) + 最小值
  const openFullScreen = () => {
    const loading = ElLoading.service({
      lock: true,
      text: 'Loading',
      background: 'rgba(0, 0, 0, 0.7)',
    })
    loadingDelay = Math.floor(Math.random() * 250) + 100
    setTimeout(() => {
      loading.close()
    }, loadingDelay)
  }

  const toLogin = () => { 
    activeIndex.value = 'Login'
    PageStatus.value = 'Login' 
    openFullScreen()
  } 
  const toRegister = () => {
    activeIndex.value = 'Register'
    PageStatus.value = 'Register' 
    openFullScreen()
  } 
  const toHome = () => { 
    activeIndex.value = 'Home'
    PageStatus.value = 'Home' 
    openFullScreen()
  } 
  const toWrite = () => { 
    activeIndex.value = 'Write'
    PageStatus.value = 'Write' 
    openFullScreen()
  } 
  const toAbout = () => { 
    activeIndex.value = 'About'
    PageStatus.value = 'About' 
    openFullScreen()
  } 
  const loginsuccess =()=>{
    Status.value = 'success'
  }

  const updateUserName = (name) => {
    userName.value = name
  }

  provide('updateUserName', updateUserName)
  provide('PageStatus', PageStatus)
  provide('Status', Status)
  provide('loginsuccess', loginsuccess)
  provide('toLogin', toLogin)
  provide('toRegister', toRegister)
  provide('toHome', toHome)
  provide('toWrite', toWrite)
  provide('toAbout', toAbout)
  provide('userName', userName)
  provide('SearchContent', SearchContent)
  
  //（AI帮助）
  provide('registerSearchHandler', (fn) => {
    triggerSearch.value = fn
  })

  const Logout=()=>{
    Status.value = 'None'
    PageStatus.value = 'Home'
    activeIndex.value = 'Home'
    localStorage.removeItem('token')
    localStorage.removeItem('userInfo')
    localStorage.removeItem('loginStatus')
    localStorage.removeItem('userName')
    sessionStorage.clear()
    window.location.reload()
  }

  const Notification = () => {
    ElNotification({
      title: 'ⓘ 每日提醒',
      customClass: 'daily-notification',
      message: h('i', { style: 'color: teal' }, (() => {
          const currentDate = new Date();
          const year = currentDate.getFullYear();
          const month = String(currentDate.getMonth() + 1);
          const day = String(currentDate.getDate());
          return `今天是${year}年${month}月${day}日`;
        })()
      ),
    });
  };

  onMounted(() => {
    // 从localStorage恢复登录状态
    const savedStatus = localStorage.getItem('loginStatus')
    const savedUserName = localStorage.getItem('userName')
    
    if (savedStatus === 'success' && savedUserName) {
      Status.value = 'success'
      userName.value = savedUserName
    }
    openFullScreen()
    setTimeout(() => {
      Notification()
    }, loadingDelay)
  })
</script>

<template>
  <div class="Menu">
    <el-menu
      :default-active="activeIndex"
      class="el-menu-demo"
      mode="horizontal"
      :ellipsis="false"
    >
      <el-menu-item index="0">
        <img
          style="width: 50px"
          src="./assets/logo.svg"
          alt="Element logo"
        />
      </el-menu-item>
        <el-input 
          class="search-input" 
          placeholder="请输入内容以查询文章" 
          style="width: 300px" 
          v-model="SearchContent"
          @keyup.enter="handleSearch"
        >
        <template #append>
          <el-button type="primary" :icon="Search" @click.stop="handleSearch"></el-button>
        </template>
      </el-input>
      <template v-if="Status === 'success'">
        <el-menu-item index="Home" @click="toHome()">主页</el-menu-item>
        <el-menu-item index="Write" @click="toWrite()">写文章</el-menu-item>
        <el-menu-item index="About" @click="toAbout()">关于</el-menu-item>
        <el-sub-menu index="User" popper-class="user-dropdown">
          <template #title>{{ userName ? `用户（${userName}）` : '用户' }}</template>
          <el-menu-item index="Logout" @click="Logout()">退出登录</el-menu-item>
        </el-sub-menu>
      </template>
      <template v-else>
        <el-menu-item index="Home" @click="toHome()">主页</el-menu-item>
        <el-menu-item index="Login" @click="toLogin()">登录</el-menu-item>
        <el-menu-item index="Register" @click="toRegister()">注册</el-menu-item>
        <el-menu-item index="About" @click="toAbout()">关于</el-menu-item>
      </template>
    </el-menu>
  </div>

  <div class="Content">
    <div class="Login" v-if="PageStatus==='Login'">
      <Login/>
    </div>
    <div class="Register" v-else-if="PageStatus==='Register'">
      <Register/>
    </div>
    <div class="About" v-else-if="PageStatus==='About'">
      <About/>
    </div>
    <div class="Write" v-else-if="PageStatus==='Write' && Status === 'success'">
      <Write/>
    </div>
    <div class="Home" v-else>
      <Home/>
    </div>
  </div>
  <Footer/>
</template>

<style>
  .user-dropdown {
    --el-menu-bg-color: var(--color-background-soft); 
    --el-menu-text-color: var(--color-text);
  }

  .daily-notification {
    background-color: var(--color-background-mute) !important;
    color: var(--color-text) !important;
    border: 1px solid var(--color-border) !important;
  }

  /* 修复内容区域透明度(AI帮助) */
  .daily-notification .el-notification__content {
    background-color: transparent !important;
    color: var(--color-text) !important;
  }
  
  .daily-notification .el-notification__title {
    color: var(--color-heading) !important;
  }

</style>

<style scoped>
  .el-menu-demo {
    background-color: var(--color-f-border-hover);
    border-bottom: 1px solid #dcdfe6;
    --el-menu-text-color: var(--color-text); 
  }

  .el-menu--horizontal > .el-menu-item:nth-child(1) {
    margin-right: auto;
  }

  .search-input :deep(.el-input__wrapper) {
    background-color: var(--color-f-border-hover);
  }

  .search-input :deep(.el-input-group__append) {
    background-color: var(--color-ff-border-hover);
  }

  .search-input :deep(.el-input__inner) {
    color: var(--color-text);
  }

  .Content{
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: calc(100vh - 180px);
    padding: 20px;
  }

  .Login{ 
    width: 400px; 
  }

  .Register{ 
    width: 400px; 
  }

  .Write{ 
    width: 75%; 
  }

  .Home { 
    width: 90%; 
    padding: 25px; 
  }

  .Error_Content{
    background-color: var(--color-f-background-soft);
    padding: 25px;
    border-radius: 10px;
    display: flex;
    flex-direction: column;
  }

  .Error{
    width: 400px;
    margin: 20px;
    font-weight: bolder;
  }
</style>