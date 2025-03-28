<template>
	<view>
		<!-- 顶部标题 -->
		<view class="header">我的情绪日记</view>

		<!-- 昨日日记 -->
		<view class="diary-card yesterday-card" @tap="goToDetail(yesterdayDiary.id)">
			<image :src="yesterdayDiary.image || '/static/image/img001.jpg'" mode="aspectFill" class="diary-image">
			</image>
			<view class="diary-header">
				<text class="diary-emotion">{{ yesterdayDiary.emotion }}</text>
				<view class="diary-title">{{ yesterdayDiary.title }}</view>
			</view>
			<view class="diary-content full-content">
				{{ yesterdayDiary.content }}
			</view>
			<view class="diary-info">
				<text class="diary-date date-tag">{{ formatTimestamp(yesterdayDiary.createtime) }}</text>
			</view>
		</view>

		<!-- 以往日记列表 -->
		<view class="section">
			<view class="section-title-wrapper">
				<view class="line"></view>
				<u--text text="以往日记" size="18" color="#373737" bold align="center"
					customStyle="padding: 12rpx 40rpx;"></u--text>
				<view class="line"></view>
			</view>

			<view class="diary-card" v-for="(item, index) in diaries" :key="item.id" @tap="goToDetail(item.id)">
				<view class="diary-header">
					<text class="diary-emotion">{{ item.emotion }}</text>
					<view class="diary-title">{{ item.title }}</view>
				</view>
				<view class="diary-content"><u--text :lines="2" :text="item.content"></u--text></view>
				<view class="diary-info">
					<text class="diary-date date-tag">{{ formatTimestamp(item.createtime) }}</text>
				</view>
			</view>

			<button class="more-btn" @tap="goToList">查看更多</button>
		</view>

		<!-- 浮动写日记按钮 -->
		<view class="fab" @tap="goToWriteDiary">
			<u-icon name="edit-pen" size="40" color="#fff"></u-icon>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				yesterdayDiary: null, // 改为 null，等待 onShow 中动态赋值
				diaries: [] // 以往日记列表
			}
		},
		onShow() {
			// 获取昨日日记
			const storedDiary = uni.getStorageSync('yesterdayDiary');
			if (storedDiary) {
				this.yesterdayDiary = storedDiary;
			}

			// 获取以往日记
			const storedDiaries = uni.getStorageSync('diaries') || [];
			this.diaries = storedDiaries.slice(0, 5); // 显示 5 个日记
		},
		methods: {
			goToWriteDiary() {
				uni.navigateTo({
					url: '/pages/edit/edit'
				});
			},
			goToList() {
				uni.navigateTo({
					url: '/pages/list/list'
				});
			},
			goToDetail(id) {
				uni.navigateTo({
					url: `/pages/detail/detail?id=${id}`
				});
			},
			formatTimestamp(ts) {
				const date = new Date(ts);
				const now = new Date();
				const diff = now - date;

				// 计算时间差
				const days = Math.floor(diff / (1000 * 60 * 60 * 24));
				const months = Math.floor(days / 30);
				const years = Math.floor(days / 365);

				// 今天
				if (days === 0) {
					return '今天';
				}
				// 昨天
				if (days === 1) {
					return '昨天';
				}
				// 前天
				if (days === 2) {
					return '前天';
				}
				// 3天前
				if (days === 3) {
					return '3天前';
				}
				// 1周前
				if (days === 7) {
					return '1周前';
				}
				// 1个月前
				if (months === 1) {
					return '1个月前';
				}
				// 3个月前
				if (months === 3) {
					return '3个月前';
				}
				// 半年前
				if (months === 6) {
					return '半年前';
				}
				// 1年前
				if (years === 1) {
					return '1年前';
				}

				// 其他情况显示具体日期
				return `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}-${String(date.getDate()).padStart(2, '0')}`;
			}
		}
	}
</script>

<style lang="scss">
	.header {
		font-size: 36rpx;
		font-weight: bold;
		text-align: center;
		padding: 30rpx 0;
		color: #333;
		background: linear-gradient(to right, #3c93ff, #6ab1ff);
		color: #fff;
		margin-bottom: 30rpx;
	}

	.yesterday-card {
		margin: 0 30rpx 24rpx; // ✅ 与 .diary-card 保持一致
	}

	.yesterday-diary {
		margin: 0 30rpx 30rpx;
		background: #fff;
		border-radius: 20rpx;
		padding: 30rpx;
		box-shadow: 0 4rpx 20rpx rgba(0, 0, 0, 0.08);

		image {
			width: 100%;
			height: 300rpx;
			border-radius: 12rpx;
			margin-bottom: 20rpx;
		}

		view {
			margin-bottom: 16rpx;

			text {
				font-size: 28rpx;
				color: #666;

				&:first-child {
					font-size: 32rpx;
					font-weight: bold;
					color: #333;
				}
			}
		}
	}

	.section {
		margin-top: 30rpx;
		padding: 0 30rpx;
	}

	.section-title-wrapper {
		width: 100%;
		display: flex;
		align-items: center;
		justify-content: center;
		margin-bottom: 30rpx;

		.line {
			flex: 1;
			height: 2rpx;
			background-color: #ddd;
		}
	}

	.diary-card {
		background-color: #fff;
		border-radius: 20rpx;
		padding: 30rpx;
		margin-bottom: 24rpx;
		box-shadow: 0 4rpx 20rpx rgba(0, 0, 0, 0.08);
		transition: all 0.3s ease;

		&:active {
			transform: scale(0.98);
		}
	}

	.diary-header {
		display: flex;
		align-items: center;
		margin-bottom: 16rpx;

		.diary-emotion {
			font-size: 40rpx;
			margin-right: 20rpx;
		}

		.diary-title {
			font-size: 32rpx;
			font-weight: bold;
			color: #333;
			margin-bottom: 0;
		}
	}

	.diary-content {
		font-size: 28rpx;
		color: #666;
		margin-bottom: 24rpx;
		line-height: 1.6;
	}

	.full-content {
		// ✅ 针对昨日的内容显示完整不截断
		overflow: visible;
		white-space: normal;
	}

	.diary-info {
		display: flex;
		justify-content: flex-end;
		align-items: center;
		font-size: 24rpx;
		color: #999;
	}

	.date-tag {
		display: inline-block;
		padding: 8rpx 24rpx;
		background-color: #f5f5f5;
		border-radius: 12rpx;
		color: #666;
		font-size: 24rpx;
	}

	.more-btn {
		margin: 30rpx auto;
		width: 80%;
		height: 80rpx;
		line-height: 80rpx;
		background: linear-gradient(to right, #3c93ff, #6ab1ff);
		color: #fff;
		border-radius: 40rpx;
		font-size: 28rpx;
		border: none;
		box-shadow: 0 4rpx 12rpx rgba(60, 147, 255, 0.3);

		&:active {
			transform: scale(0.98);
		}
	}

	.fab {
		position: fixed;
		bottom: 80rpx;
		right: 40rpx;
		width: 100rpx;
		height: 100rpx;
		border-radius: 50%;
		background: linear-gradient(to right, #3c93ff, #6ab1ff);
		display: flex;
		align-items: center;
		justify-content: center;
		box-shadow: 0 8rpx 16rpx rgba(0, 0, 0, 0.2);
		z-index: 999;
	}
</style>