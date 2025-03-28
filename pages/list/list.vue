<template>
	<view>
		<view class="header">日记列表</view>
		<!-- 分类标签 -->
		<view class="tags">
			<text @tap="filterDiaries('all')">全部</text>
			<text @tap="filterDiaries('😊')">好心情</text>
			<text @tap="filterDiaries('😢')">坏心情</text>
		</view>

		<!-- 日记列表 -->
		<view class="diary-list">
			<view v-for="(diary, index) in filteredDiaries" :key="index" class="diary-item" @tap="goToDetail(diary.id)">
				<view><text>{{ diary.title }}</text></view>
				<view><text>{{ diary.content }}</text></view>
				<view><text>{{ formatTimestamp(diary.createtime) }}</text></view>
				<view><text>{{ diary.emotion }}</text></view>
				<view><button @tap.stop="confirmDelete(diary.id)">删除</button></view>
			</view>
		</view>

		<!-- 功能按钮 -->
		<button @tap="showOptions">功能</button>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				diaries: [], // 所有日记
				filteredDiaries: [], // 筛选后的日记
				filterCriteria: 'all' // 当前筛选的心情标签,
			};
		},
		onShow() {
			// 获取所有日记
			const storedDiaries = uni.getStorageSync('diaries') || [];
			this.diaries = storedDiaries;
			this.filteredDiaries = this.diaries; // 默认显示所有日记
		},
		methods: {
			// 点击日记项跳转到详情页
			goToDetail(diaryId) {
				uni.navigateTo({
					url: `/pages/detail/detail?id=${diaryId}` // 传递日记的 id
				});
			},
			// 删除单个日记
			deleteDiary(diaryId) {
				let diaries = uni.getStorageSync('diaries') || []
				const index = diaries.findIndex(diary => diary.id === diaryId)
				if (index !== -1) {
					diaries.splice(index, 1)
					uni.setStorageSync('diaries', diaries)
					this.filteredDiaries = diaries // ✅ 关键！同步更新页面数据
					uni.showToast({
						title: '删除成功',
						icon: 'success'
					})

				} else {
					uni.showToast({
						title: '删除失败',
						icon: 'none'
					})
				}
			},

			// 全选或删除所有日记
			showOptions() {
				// 示例功能，后续可以拓展
				const action = prompt('请输入操作：全选或删除所有');
				if (action === '全选') {
					console.log('选中所有日记');
				} else if (action === '删除所有') {
					this.diaries = []; // 清空所有日记
					this.updateStorage();
				}
			},

			// 按心情筛选日记
			filterDiaries(mood) {
				if (mood === 'all') {
					this.filteredDiaries = this.diaries;
				} else {
					this.filteredDiaries = this.diaries.filter(diary => diary.emotion === mood);
				}
			},

			// 更新本地缓存
			updateStorage() {
				uni.setStorageSync('diaries', this.diaries);
				this.filteredDiaries = this.diaries; // 更新筛选后的列表
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
			// 删除确认逻辑
			confirmDelete(diaryId) {
				uni.showModal({
					title: '确认删除',
					content: '确定要删除这篇日记吗？',
					success: (res) => {
						if (res.confirm) {
							this.deleteDiary(diaryId)
						}
					}
				})
			},
		}
	}
</script>

<style>
	.header {
		font-size: 24px;
		text-align: center;
		margin-top: 20px;
	}

	.tags {
		display: flex;
		justify-content: space-around;
		margin-top: 20px;
	}

	.diary-list {
		margin-top: 40px;
	}

	.diary-item {
		padding: 10px;
		border-bottom: 1px solid #ccc;
	}

	button {
		margin-top: 10px;
		background-color: red;
		color: white;
	}
</style>