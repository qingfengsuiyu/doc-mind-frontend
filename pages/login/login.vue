<template>
    <view class="container">
        <view class="title">DocMind</view>
        <view class="subtitle">AI 知识库问答系统</view>
        
        <view class="form">
            <input class="input" v-model="username" placeholder="用户名" />
            <input class="input" v-model="password" placeholder="密码" type="password" />
            <button class="btn" @click="handleLogin" :disabled="isLoading"> {{ isLoading ? '登录中...' : '登录' }}</button>
            <button class="btn btn-register" @click="handleRegister">没有账号？去注册</button>
        </view>
    </view>
</template>

<script setup>
import { ref } from 'vue'

const isDev = process.env.NODE_ENV === 'development'
const BASE_URL = isDev ? 'http://127.0.0.1:8000' : 'http://47.82.90.240/doc-api'

const isLoading = ref(false)
const username = ref('')
const password = ref('')

const handleLogin = async () => {
    if (!username.value || !password.value) {
        uni.showToast({ title: '请输入用户名和密码', icon: 'none' })
        return
    }
	isLoading.value = true;
	uni.request({
		url:`${BASE_URL}/auth/login`,
		method:'POST',
		data:{username:username.value,password:password.value},
		success:(res) => {
			if(res.statusCode===200){
				// 把token存在本地
				uni.setStorageSync('docmind_token',res.data.token)
				uni.setStorageSync('docmind_username',res.data.username)
				// 跳转到主页
				uni.reLaunch({
					url:'/pages/index/index'
				})
			}else{
				uni.showToast({
					title:res.data.detail || '登录失败',icon:'none'
				})
			}
			isLoading.value = false;
		},
		fail:() => {
			 uni.showToast({ title: '网络错误', icon: 'none' })
			 isLoading.value = false;
		},
	})
}

const handleRegister = async () => {
    uni.navigateTo({ url: '/pages/register/register' })
}
</script>

<style scoped lang="scss">
.container {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 100vh;
    padding: 0 40px;
    background-color: #f5f7fa;

    .title {
        font-size: 32px;
        font-weight: bold;
        color: #1F3864;
        margin-bottom: 8px;
    }

    .subtitle {
        font-size: 14px;
        color: #999;
        margin-bottom: 60px;
    }

    .form {
        width: 100%;

        .input {
            width: 100%;
            height: 48px;
            background-color: #ffffff;
            border-radius: 8px;
            padding: 0 16px;
            margin-bottom: 16px;
            font-size: 14px;
            box-sizing: border-box;
            border: 1px solid #eee;
        }

        .btn {
            width: 100%;
            height: 48px;
            background-color: #4080ff;
            color: #ffffff;
            border-radius: 8px;
            font-size: 16px;
            margin-bottom: 12px;
            border: none;
        }

        .btn-register {
            background-color: #ffffff;
            color: #4080ff;
            border: 1px solid #4080ff;
        }
    }
}
</style>