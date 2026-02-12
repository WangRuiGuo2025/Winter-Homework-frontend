<script setup>
    import { ref , inject } from 'vue'
    import { ElMessage } from 'element-plus'
    import axios from 'axios'

    const UserName = ref('')
    const Password = ref('')

    //引入全局方法
    const toRegister = inject('toRegister')
    const toHome = inject('toHome')
    const loginsuccess = inject('loginsuccess')
    const updateUserName = inject('updateUserName')

    //登录按钮的小函数
    //TODO：实现后端的数据库调用判断
    const OnLogin=async()=>{
        try{
            const response = await axios.get("https://winter-homework-backend.onrender.com/login",{
                params:{
                    username:UserName.value,
                    password:Password.value
                }
            })
        if(response.data.code===1){
            loginsuccess()

            // 这里必须调用父组件提供的方法来更新 userName（AI帮助）
            updateUserName(UserName.value)
            toHome()
            ElMessage({
                message: '登录成功',
                type: 'success',
            })
        }else{
            ElMessage({
                message: '登录失败',
                type: 'error',
            })
        }
    }catch(e){
        ElMessage({
            message: '登录失败',
            type: 'error',
        })
    }
}
        // if(UserName.value==='1'&&Password.value==='1'){
        //     loginsuccess()
        //     toHome()
        //     ElMessage({
        //         message: '登录成功',
        //         type: 'success',
        //     })
        // }else{
        //     ElMessage({
        //         message: '登录失败',
        //         type: 'error',
        //     })
        // }
</script>

<template>
    <div class="Login_Content">
        <el-text class="Login-text" size="large">登陆页面</el-text>
        <br>
        <div class="Login-UserName">
            <el-text class="UserName-text">用户名：</el-text>
            <el-input class="UserName-input" v-model="UserName" placeholder="请输入账户名"></el-input>
        </div>
        <div class="Login-Password">
            <el-text class="Password-text">密码：</el-text>
            <el-input class="Password-input" v-model="Password" placeholder="请输入密码" type="password" show-password></el-input>
        </div>
        <el-link class="Register" type="primary" @click="toRegister()">注册</el-link>
        <el-button class="Login-Button" type="primary" @click="OnLogin">登录</el-button>
    </div>
</template>

<style scoped>
    .Login_Content{
        background-color: var(--color-border);
        padding: 25px;
        border-radius: 10px;
        display: flex;
        flex-direction: column;
    }

    .Login-text{
        font-size: x-large;
        font-weight: bolder;
        align-items: center;
        justify-items: center;
        color: var(--color-text);
    }

    .Login-UserName{
        display: flex;
        align-items: center;
        margin-bottom: 15px;
    }

    .UserName-text{
        width: 80px;
        text-align: right;
        margin-right: 10px;
        font-weight: bold;
        color: var(--color-text);
    }

    .Login-Password{
        display: flex;
        align-items: center;
        margin-bottom: 15px;
    }
    
    .UserName-input{
        width: 250px;
    }

    .UserName-input :deep(.el-input__wrapper) {
        background-color: var(--color-fff-border-hover);
    }
    
    .UserName-input :deep(.el-input__inner) {
        color: var(--color-text);
    }

    .Password-text{
        width: 80px;
        text-align: right;
        margin-right: 10px;
        font-weight: bold;
        color: var(--color-text);
    }

    .Password-input{
        width: 250px;
    }

    .Password-input :deep(.el-input__wrapper) {
        background-color: var(--color-fff-border-hover);
    }

    .Password-input :deep(.el-input__inner) {
        color: var(--color-text);
    }

    .Register{
        margin-bottom: 15px;
    }

    .Login-Button{
        margin-bottom: 15px;
    }
    
</style>
