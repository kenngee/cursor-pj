<template>
	<view>
		<view class="header">日记详情</view>
		<!-- 日记内容展示 -->
		<view class="content">
			<view><text>{{ diary.title }}</text></view>
			<view><text>{{ diary.content }}</text></view>
			<view><text>{{ formatTimestamp(diary.createtime) }}</text></view>
			<view><text>{{ diary.emotion }}</text></view>
		</view>

		<!-- 编辑和删除按钮 -->
		<button @tap="editDiary">编辑</button>
		<button @tap="deleteDiary">删除</button>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				diary: {} // 当前日记对象
			}
		},
		onLoad(options) {
			// 获取传递的日记 ID 并获取对应的日记数据
			const diaryId = options.id;
			const storedDiaries = uni.getStorageSync('diaries') || [];
			this.diary = storedDiaries.find(d => d.id === diaryId) || {}; // 查找对应的日记
		},
		methods: {
			// 编辑日记
			editDiary() {
				uni.navigateTo({
					url: `/pages/edit/edit?id=${this.diary.id}`
				});
			},

			// 删除日记
			deleteDiary() {
				uni.showModal({
					title: '确认删除',
					content: '你确定要删除这篇日记吗？',
					success: (res) => {
						if (res.confirm) {
							const storedDiaries = uni.getStorageSync('diaries') || [];
							const index = storedDiaries.findIndex(d => d.id === this.diary.id);
							if (index !== -1) {
								storedDiaries.splice(index, 1);
								uni.setStorageSync('diaries', storedDiaries);

								uni.showToast({
									title: '删除成功',
									icon: 'success'
								});

								// 返回上一页（列表页）
								setTimeout(() => {
									uni.navigateBack();
								}, 600); // 等待提示框结束后再返回
							} else {
								uni.showToast({
									title: '删除失败',
									icon: 'none'
								});
							}
						}
					}
				});
			},
			formatTimestamp(timestamp) {
				let date = new Date(timestamp);
				return date.toLocaleString('zh-CN', {
					year: 'numeric',
					month: '2-digit',
					day: '2-digit',
					hour: '2-digit',
					minute: '2-digit',
					second: '2-digit',
					hour12: false
				}).replace(/\//g, '-').replace(',', ''); // 移除默认逗号
			},
			// **更新详情页的数据**
			updateDiaryData(title, content, emotion) {
				this.diary.title = title;
				this.diary.content = content;
				this.diary.emotion = emotion;
				console.log('更新详情页数据', this.diary);
			}
		}
	}
</script>

<style>
	.header {
		font-size: 24px;
		text-align: center;
		margin-top: 20px;
	}

	.content {
		margin-top: 20px;
	}

	button {
		margin-top: 10px;
		background-color: #4CAF50;
		color: white;
	}
</style>