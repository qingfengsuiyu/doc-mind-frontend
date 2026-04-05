<template>
	<view class="outer">
		<view class="top">
			论文助手
		</view>
		<view class="content">
			<scroll-view scroll-y="true" class="scroll" :scroll-into-view="scrollToId">
				<view class="msg-list">
					<view v-for="(item,index) in dialogHistory" :key="index"
						:id="index === dialogHistory.length - 1 ? 'msg-last' : ''"
						:class="['msg', item.role === 'user' ? 'msg-user' : 'msg-ai']">
						{{ item.content }}
					</view>
				</view>
			</scroll-view>
		</view>
		<view class="bottom">
			<button @click="handleUpdate" class="upload-btn">📎</button>
			<input class="inp" type="text" v-model="inputValue" placeholder="输入问题..." />
			<button class="send-btn" @click="handleSend">发送</button>
		</view>
	</view>
</template>


<script setup>
	import {
		ref,
		nextTick
	} from 'vue'
	const dialogHistory = ref([])
	const inputValue = ref('')
	const isLoading = ref(false)
	const isDocEmpty = ref(true)
	const scrollToId = ref('')

	// 每次消息更新后，滚动到底部
	const scrollToBottom = () => {
		scrollToId.value = ''
		nextTick(() => {
			scrollToId.value = 'msg-last'
		})
	}

	const askQuestion = (question) => {
		return new Promise((resolve, reject) => {
			uni.request({
				url: 'http://127.0.0.1:8000/ask',
				method: 'POST',
				header: {
					'Content-Type': 'application/json'
				},
				data: {
					question
				},
				success: (res) => resolve(res.data),
				fail: (err) => reject(err)
			})
		})
	}

	const handleSend = async (value) => {
		// 空内容,直接返回
		if (!inputValue.value) return
		
		// 没有文件也返回
		if (isDocEmpty.value) {
		    uni.showToast({ title: '请先上传文档', icon: 'none' })
		    return
		}

		const question = inputValue.value

		// 把用户消息加入对话历史
		dialogHistory.value.push({
			role: 'user',
			content: inputValue.value
		})
		scrollToBottom()

		// 清空输入框
		inputValue.value = ''

		// 开始和后端交互
		isLoading.value = true
		try {
			const data = await askQuestion(question)
			dialogHistory.value.push({
				role: 'ai',
				content: data.answer
			})
			scrollToBottom()
		} catch (e) {
			dialogHistory.value.push({
				role: 'ai',
				content: '请求失败，请重试'
			})
		} finally {
			isLoading.value = false

		}

	}

	const handleUpdate = () => {
		uni.chooseFile({
			count: 1,
			type: 'file',
			extension: ['.pdf'],
			success: (res) => {
				const file = res.tempFiles[0]
				uploadFile(file)
			}
		})
	}

	const uploadFile = (file) => {
		isLoading.value = true
		uni.uploadFile({
			url: 'http://127.0.0.1:8000/upload',
			filePath: file.path,
			name: 'file',
			success: (res) => {
				const data = JSON.parse(res.data)
				isDocEmpty.value = false
				uni.showToast({
					title: '上传成功',
					icon: 'success'
				})
			},
			fail: () => {
				uni.showToast({
					title: '上传失败',
					icon: 'error'
				})
			},
			complete: () => {
				isLoading.value = false
			}

		})
	}
</script>

<style lang="scss">
	page {
		height: 100%;
		overflow: hidden; // ← 加这个
	}

	.outer {
		display: flex;
		flex-direction: column;
		height: 100%;

		.top {
			height: 50px;
			width: 100vw;
			background-color: #ffffff;
			border-bottom: 1px solid #eee;
			display: flex;
			align-items: center;
			justify-content: center;
			font-weight: bold;
		}

		.content {
			flex: 1;
			overflow: hidden;

			.scroll {
				height: 100%;
				background-color: #f5f5f5;
				box-sizing: border-box;
				padding: 16px;

				.msg-list {
					display: flex;
					flex-direction: column;
				}
			}

		}

		.bottom {
			display: flex;
			flex-direction: row;
			align-items: center;
			padding: 8px 12px;
			gap: 8px;
			background-color: #ffffff;
			border-top: 1px solid #eee;

			.upload-btn {
				width: 40px;
				height: 40px;
				font-size: 20px;
				display: flex;
				align-items: center;
				justify-content: center;
				background: none;
				border: none;
				padding: 0;
			}

			.inp {
				flex: 1;
				height: 40px;
				background-color: #f5f5f5;
				border-radius: 20px;
				padding: 0 16px;
				font-size: 14px;
			}

			.send-btn {
				width: 60px;
				height: 40px;
				background-color: #4080ff;
				color: #ffffff;
				border-radius: 20px;
				font-size: 14px;
				display: flex;
				align-items: center;
				justify-content: center;
				border: none;
			}
		}
	}

	.msg {
		margin-bottom: 12px;
		padding: 10px 14px;
		border-radius: 12px;
		max-width: 70%;
		font-size: 14px;
		line-height: 1.5;
		width: fit-content;
	}

	.msg-user {
		background-color: #4080ff;
		color: #ffffff;
		margin-left: auto; // 靠右
		margin-bottom: 12px;
	}

	.msg-ai {
		background-color: #ffffff;
		color: #333333;
		margin-right: auto;
		margin-bottom: 12px;
	}
</style>