<template>
	<view class="outer">
		<view class="top">
			知识库问答助手
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

	const askQuestion = (question,history) => {
		return new Promise((resolve, reject) => {
			uni.request({
				url: 'http://127.0.0.1:8000/ask/stream',
				method: 'POST',
				header: {
					'Content-Type': 'application/json'
				},
				data: {
					question,history
				},
				success: (res) => resolve(res.data),
				fail: (err) => reject(err)
			})
		})
	}

	const handleSend = async () => {
	    if (!inputValue.value) return
	    if (isDocEmpty.value) {
	        uni.showToast({ title: '请先上传文档', icon: 'none' })
	        return
	    }
	
	    const question = inputValue.value
	    // ← 先保存当前历史（不含这次提问）
	    const history = [...dialogHistory.value]
	
	    dialogHistory.value.push({ role: 'user', content: question })
	    scrollToBottom()
	    inputValue.value = ''
	
	    isLoading.value = true
	    try {
	        // ← 传 history 而不是 dialogHistory.value
			await askStream(question, history)
	        // dialogHistory.value.push({ role: 'ai', content: data.answer })
	        // scrollToBottom()
	    } catch (e) {
	        dialogHistory.value.push({ role: 'ai', content: '请求失败，请重试' })
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
	
	const askStream = async (question, history) => {
	    // 先在对话历史里加一条空的 AI 消息
	    dialogHistory.value.push({ role: 'ai', content: '' })
	    const aiIndex = dialogHistory.value.length - 1
	    scrollToBottom()
	
	    const response = await fetch('http://127.0.0.1:8000/ask/stream', {
	        method: 'POST',
	        headers: { 'Content-Type': 'application/json' },
	        body: JSON.stringify({ question, history })
	    })
	
	    // 拿到流读取器
	    const reader = response.body.getReader()
	    const decoder = new TextDecoder()
	
	    while (true) {
	        const { done, value } = await reader.read()
	        if (done) break
	
	        // 把二进制数据转成字符串
	        const text = decoder.decode(value)
	        
	        // 按行分割，处理每一条 SSE 数据
	        const lines = text.split('\n')
	        for (const line of lines) {
	            if (!line.startsWith('data: ')) continue
	            const data = line.slice(6)  // 去掉 "data: " 前缀
	            if (data === '[DONE]') break
	            
	            try {
	                const parsed = JSON.parse(data)
	                // 实时拼接到 AI 消息上
	                dialogHistory.value[aiIndex].content += parsed.text
	                scrollToBottom()
	            } catch(e) {}
	        }
	    }
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