<script setup>
    import { ref , inject } from 'vue'
    import { ElMessage } from 'element-plus'
    import axios from 'axios'

    const UserName = ref('')
    const Password = ref('')

    //引入全局方法
    const toLogin = inject('toLogin')
    const toHome = inject('toHome')

    //注册按钮的小函数
    //TODO：实现后端的数据库调用判断
    const OnRegister=async()=>{
        try{
            const response = await axios.get("http://localhost:8000/register",{
                params:{
                    username:UserName.value,
                    password:Password.value
                }
            })
        if(response.data.code===1){
            toLogin()
            ElMessage({
                message: '注册成功',
                type: 'success',
            })
        }else{
            ElMessage({
                message: '注册失败',
                type: 'error',
            })
        }
    }catch(e){
        ElMessage({
            message: '注册失败',
            type: 'error',
        })
    }
}
        // if(UserName.value==='1'&&Password.value==='1'){
        //     toLogin()
        //     ElMessage({
        //         message: '注册成功',
        //         type: 'success',
        //     })
        // }else{
        //     ElMessage({
        //         message: '注册失败',
        //         type: 'error',
        //     })
        // }

</script>

<template>
    <div class="Register_Content">
        <el-text class="Register-text" size="large">注册页面</el-text>
        <br>
        <div class="Register-UserName">
            <el-text class="UserName-text">用户名：</el-text>
            <el-input class="UserName-input" v-model="UserName" placeholder="请输入账户名"></el-input>
        </div>
        <div class="Register-Password">
            <el-text class="Password-text">密码：</el-text>
            <el-input class="Password-input" v-model="Password" placeholder="请输入密码" type="password" show-password></el-input>
        </div>
        <el-link class="Login" type="primary" @click="toLogin()">登录</el-link>
        <el-button class="Register-Button" type="primary" @click="OnRegister">注册</el-button>
    </div>
</template>

<style scoped>
    .Register_Content{
        background-color: var(--color-f-background-soft);
        padding: 25px;
        border-radius: 10px;
        display: flex;
        flex-direction: column;
    }

    .Register-text{
        font-size: x-large;
        font-weight: bolder;
        align-items: center;
        justify-items: center;
    }

    .Register-UserName{
        display: flex;
        align-items: center;
        margin-bottom: 15px;
    }

    .UserName-text{
        width: 80px;
        text-align: right;
        margin-right: 10px;
        font-weight: bold;
    }

    .Register-Password{
        display: flex;
        align-items: center;
        margin-bottom: 15px;
    }
    
    .UserName-input{
        width: 250px;
    }

    .Password-text{
        width: 80px;
        text-align: right;
        margin-right: 10px;
        font-weight: bold;
    }

    .Password-input{
        width: 250px;
    }

    .Login{
        margin-bottom: 15px;
    }

    .Register-Button{
        margin-bottom: 15px;
    }
    
</style>
