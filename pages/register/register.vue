<template>
	<view class="container">
		<view class="title">账号注册</view>
		<view class="subtitle">AI 知识库问答系统</view>

		<view class="form">
			<input class="input" v-model="username" placeholder="用户名" />
			<input class="input" v-model="password" placeholder="密码" type="password" />
			<input class="input" v-model="re_password" placeholder="确认密码" type="password" />
			<button class="btn" @click="handleRegister" :disabled="isLoading">{{ isLoading ? '注册中...' : '注册' }}</button>
			<button class="btn btn-register" @click="handleLogin">返回登录</button>
		</view>
	</view>
</template>

<script setup>
	import {
		ref
	} from 'vue'
	
	const isLoading = ref(false)
	const isDev = process.env.NODE_ENV === 'development'
	const BASE_URL = isDev ? 'http://127.0.0.1:8000' : 'https://xiaoyihao.top/doc-api'

	const username = ref('')
	const password = ref('')
	const re_password = ref('')

	const handleRegister = async () => {
		if (!username.value || !password.value) {
			uni.showToast({
				title: '请输入用户名和密码',
				icon: 'none'
			})
			return
		}
		if (password.value !== re_password.value) {
			uni.showToast({
				title: '两次密码输入不一致',
				icon: 'none'
			})
			return
		}
		isLoading.value = true
		uni.request({
			url: `${BASE_URL}/auth/register`,
			method: 'POST',
			data: {
				username: username.value,
				password: password.value
			},
			success: (res) => {
				if (res.statusCode === 200) {
					uni.showToast({
						title: '注册成功，请登录',
						icon: 'success'
					})
					setTimeout(() => {
						uni.navigateBack()
					}, 1500) // 等提示显示完再跳转
				} else {
					uni.showToast({
						title: res.data.detail || '注册失败',
						icon: 'none'
					})
				}
				isLoading.value = false
			},
			fail: () => {
				uni.showToast({
					title: '网络错误',
					icon: 'none'
				})
				isLoading.value = false
			}
		})
	}

	const handleLogin = async () => {
		 uni.reLaunch({ url: '/pages/login/login' })
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