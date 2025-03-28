<template>
	<view>
		<view class="custom-navbar">
			<u-icon name="arrow-left" size="38" @tap="onBackConfirm"></u-icon>
			<view class="header">{{ diaryId ? '编辑日记' : '写日记' }}</view>
		</view>
		<!-- 标题输入框 -->
		<input v-model="title" placeholder="请输入标题" />
		<!-- 正文输入框 -->
		<textarea v-model="content" placeholder="请输入内容"></textarea>
		<!-- 心情选择 -->
		<view class="mood-selector">
			<text>选择心情：</text>
			<view class="mood-options">
				<button v-for="(mood, index) in moods" :key="index"
					:class="['mood-btn', { selected: selectedMood === mood }]" @tap="selectMood(mood)">
					{{ mood }}
				</button>
			</view>
			<text>已选择：{{ selectedMood }}</text>
		</view>
		<!-- 发送按钮 -->
		<button @tap="saveDiary" :disabled="!hasChanged" class="save-btn">保存</button>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				title: '',
				content: '',
				selectedMood: '',
				moods: ['😊', '😢', '😡', '😐'],
				diaryId: null,
				originalData: {} // ✅ 用于记录初始值
			}
		},
		onLoad(options) {
			this.originalData = {
				title: '',
				content: '',
				emotion: ''
			};

			if (options.id) {
				this.diaryId = options.id; // ✅ 将 ID 存入 data 中，便于后续保存时判断
				const diaries = uni.getStorageSync('diaries') || [];
				const diary = diaries.find(d => d.id === options.id); // ✅ 根据 ID 查找原日记
				if (diary) {
					// ✅ 回显原有数据
					this.title = diary.title;
					this.content = diary.content;
					this.selectedMood = diary.emotion;

					// ✅ 存储初始数据副本
					this.originalData = {
						title: diary.title,
						content: diary.content,
						emotion: diary.emotion
					};
				}
			}
		},
		computed: {
			// ✅ 计算属性：判断是否有任何改动
			hasChanged() {
				return (
					this.title !== this.originalData.title ||
					this.content !== this.originalData.content ||
					this.selectedMood !== this.originalData.emotion
				);
			}
		},
		methods: {
			// 选择心情
			selectMood(mood) {
				this.selectedMood = mood;
			},

			//保存日记
			saveDiary() {
				// 检查必填项是否为空
				if (!this.title || !this.content || !this.selectedMood) {
					uni.showToast({
						title: '请填写完整信息',
						icon: 'none'
					});
					return;
				}
				const diaries = uni.getStorageSync('diaries') || [];
				if (this.diaryId) {
					const index = diaries.findIndex(d => d.id === this.diaryId);
					if (index !== -1) {
						diaries[index] = {
							...diaries[index], // 保留原 ID 不变
							title: this.title,
							content: this.content,
							emotion: this.selectedMood,
							edittime: new Date().getTime()
						};
					}
				} else {
					const newDiary = {
						id: new Date().getTime().toString(), // 使用时间戳生成唯一 ID
						title: this.title,
						content: this.content,
						emotion: this.selectedMood,
						createtime: new Date().getTime()
					};

					// 保存到本地缓存

					diaries.unshift(newDiary); // 新日记添加到最前面
					// 更新昨日日记
					uni.setStorageSync('yesterdayDiary', newDiary);
				}
				uni.setStorageSync('diaries', diaries);

				if (this.diaryId) {
					uni.navigateBack({
						delta: 1, // 返回1级页面
						success: () => {
							const pages = getCurrentPages(); // 获取当前页面栈
							console.log('当前页面栈:', pages); // 打印页面栈
							const prevPage = pages[pages.length - 1].$vm; // 获取上一页
							console.log('上一页:', prevPage); // 检查上一页是否是详情页

							if (prevPage && typeof prevPage.updateDiaryData === 'function') {
								// 确保上一页有 updateDiaryData 方法
								prevPage.updateDiaryData(this.title, this.content, this.selectedMood);
							} else {
								console.error('上一页没有 updateDiaryData 方法');
							}
						}
					});
				} else {
					uni.navigateBack();
				}
			},
			onBackConfirm() {
				if (this.hasChanged) {
					uni.showModal({
						title: '提示',
						content: '你有未保存的修改，是否放弃？',
						success: (res) => {
							if (res.confirm) {
								uni.navigateBack();
							}
						}
					});
				} else {
					uni.navigateBack();
				}
			}
		}
	}
</script>

<style lang="scss">
	.edit-page {
		padding: 20rpx;
	}

	.header {
		font-size: 36rpx;
		font-weight: bold;
		text-align: center;
		margin-bottom: 30rpx;
	}

	.input {
		width: 100%;
		border: 1rpx solid #ccc;
		border-radius: 10rpx;
		padding: 20rpx;
		margin-bottom: 30rpx;
	}

	.textarea {
		width: 100%;
		height: 200rpx;
		border: 1rpx solid #ccc;
		border-radius: 10rpx;
		padding: 20rpx;
		margin-bottom: 30rpx;
	}

	.mood-selector {
		margin-bottom: 30rpx;
	}

	.mood-options {
		display: flex;
		justify-content: space-around;
		margin-top: 10rpx;
	}

	.mood-btn {
		padding: 15rpx 30rpx;
		border: 1rpx solid #ccc;
		border-radius: 10rpx;
		background-color: #f5f5f5;
	}

	.mood-btn.selected {
		background-color: #4CAF50;
		color: white;
		border-color: #4CAF50;
	}

	.save-btn {
		width: 100%;
		padding: 20rpx;
		background-color: #007AFF;
		color: white;
		border-radius: 10rpx;
		font-size: 30rpx;
	}

	.custom-navbar {
		padding-top: var(--status-bar-height);
		height: calc(88rpx + var(--status-bar-height));
		display: flex;
		align-items: center;
		background-color: #ffffff;
		border-bottom: 1rpx solid #eee;
		padding-left: 20rpx;
	}

	.title {
		font-size: 32rpx;
		font-weight: bold;
		margin-left: 20rpx;
	}
</style>