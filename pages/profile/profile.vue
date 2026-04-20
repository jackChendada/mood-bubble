<template>
	<view class="profile-page">
		<!-- 用户信息卡片 -->
		<view class="user-card">
			<view class="avatar">
				<text class="avatar-emoji">🫧</text>
			</view>
			<text class="username">泡泡达人</text>
			<text class="user-id">ID: {{ userId }}</text>
		</view>

		<!-- 成就系统 -->
		<view class="achievements">
			<text class="section-title">我的成就</text>
			<view class="achievement-grid">
				<view v-for="(achievement, index) in achievements" :key="index" class="achievement-item" :class="{ unlocked: achievement.unlocked }">
					<text class="achievement-icon">{{ achievement.icon }}</text>
					<text class="achievement-name">{{ achievement.name }}</text>
					<text class="achievement-desc">{{ achievement.desc }}</text>
				</view>
			</view>
		</view>

		<!-- 统计数据 -->
		<view class="statistics">
			<text class="section-title">数据统计</text>
			<view class="stat-grid">
				<view class="stat-card">
					<text class="stat-value">{{ totalBubbles }}</text>
					<text class="stat-label">总泡泡数</text>
				</view>
				<view class="stat-card">
					<text class="stat-value">{{ longestStreak }}</text>
					<text class="stat-label">最长连续</text>
				</view>
				<view class="stat-card">
					<text class="stat-value">{{ favoriteMood }}</text>
					<text class="stat-label">最常心情</text>
				</view>
				<view class="stat-card">
					<text class="stat-value">{{ gameHighScore }}</text>
					<text class="stat-label">最高分</text>
				</view>
			</view>
		</view>

		<!-- 设置选项 -->
		<view class="settings">
			<text class="section-title">设置</text>
			<view class="setting-list">
				<view class="setting-item" @tap="clearData">
					<text class="setting-label">清除数据</text>
					<text class="setting-arrow">›</text>
				</view>
				<view class="setting-item" @tap="shareApp">
					<text class="setting-label">分享应用</text>
					<text class="setting-arrow">›</text>
				</view>
				<view class="setting-item" @tap="showAbout">
					<text class="setting-label">关于</text>
					<text class="setting-arrow">›</text>
				</view>
			</view>
		</view>

		<!-- 鸿蒙特色功能 -->
		<view class="harmony-features">
			<text class="section-title">鸿蒙特色</text>
			<view class="feature-list">
				<view class="feature-item">
					<text class="feature-icon">📱</text>
					<view class="feature-info">
						<text class="feature-name">多设备同步</text>
						<text class="feature-desc">手机、平板、手表数据互通</text>
					</view>
				</view>
				<view class="feature-item">
					<text class="feature-icon">🔔</text>
					<view class="feature-info">
						<text class="feature-name">服务卡片</text>
						<text class="feature-desc">桌面快速查看心情</text>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			userId: 'BP' + Math.random().toString(36).substr(2, 6).toUpperCase(),
			totalBubbles: 0,
			longestStreak: 0,
			favoriteMood: '😊',
			gameHighScore: 0,
			achievements: [
				{ icon: '🌟', name: '初次吹泡', desc: '创建第一个泡泡', unlocked: true },
				{ icon: '🔥', name: '七天连续', desc: '连续记录7天', unlocked: false },
				{ icon: '💯', name: '百分泡泡', desc: '创建100个泡泡', unlocked: false },
				{ icon: '🎮', name: '游戏高手', desc: '游戏得分超过500', unlocked: false },
				{ icon: '📅', name: '月度达人', desc: '单月记录30天', unlocked: false },
				{ icon: '🏆', name: '情绪大师', desc: '解锁所有成就', unlocked: false }
			]
		};
	},
	onLoad() {
		this.loadStatistics();
		this.checkAchievements();
	},
	onShow() {
		this.loadStatistics();
		this.checkAchievements();
	},
	methods: {
		loadStatistics() {
			const bubbles = uni.getStorageSync('bubbles') || [];
			console.log(bubbles);
			this.totalBubbles = bubbles.length;

			// 计算最常心情
			if (bubbles.length > 0) {
				const moodCounts = {};
				bubbles.forEach((b) => {
					moodCounts[b.mood] = (moodCounts[b.mood] || 0) + 1;
				});

				const moodEmojis = {
					happy: '😊',
					sad: '😢',
					angry: '😠',
					calm: '😌',
					excited: '🤩'
				};

				const dominantMood = Object.entries(moodCounts).sort((a, b) => b[1] - a[1])[0][0];
				this.favoriteMood = moodEmojis[dominantMood] || '😊';
			}

			// 加载游戏最高分
			this.gameHighScore = uni.getStorageSync('gameHighScore') || 0;
		},

		checkAchievements() {
			const bubbles = uni.getStorageSync('bubbles') || [];

			// 检查七天连续成就
			if (this.longestStreak >= 7) {
				this.achievements[1].unlocked = true;
			}

			// 检查百分泡泡成就
			if (bubbles.length >= 100) {
				this.achievements[2].unlocked = true;
			}
		},

		clearData() {
			uni.showModal({
				title: '确认清除',
				content: '这将删除所有泡泡记录，确定吗？',
				success: (res) => {
					if (res.confirm) {
						uni.removeStorageSync('bubbles');
						uni.showToast({
							title: '已清除',
							icon: 'success'
						});
						this.loadStatistics();
					}
				}
			});
		},

		shareApp() {
			uni.share({
				provider: 'weixin',
				type: 0,
				title: '情绪泡泡 - 记录你的每一天',
				summary: '一个超有趣的情绪管理APP',
				imageUrl: '',
				success: () => {
					uni.showToast({
						title: '分享成功',
						icon: 'success'
					});
				}
			});
		},

		showAbout() {
			uni.showModal({
				title: '关于情绪泡泡',
				content: '版本: 1.0.0\n\n一个创新的情绪管理与互动娱乐应用\n专为鸿蒙系统设计',
				showCancel: false
			});
		}
	}
};
</script>

<style scoped>
.profile-page {
	min-height: 100vh;
	background: linear-gradient(180deg, #f5f7fa 0%, #e8eaf6 100%);
	padding: 40rpx;
	padding-bottom: calc(40rpx + env(safe-area-inset-bottom));
}

.user-card {
	margin-top: 30rpx;
	background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
	border-radius: 30rpx;
	padding: 60rpx 40rpx;
	text-align: center;
	margin-bottom: 40rpx;
	box-shadow: 0 8rpx 30rpx rgba(102, 126, 234, 0.3);
}

.avatar {
	width: 120rpx;
	height: 120rpx;
	background: rgba(255, 255, 255, 0.2);
	border-radius: 50%;
	margin: 0 auto 20rpx;
	display: flex;
	align-items: center;
	justify-content: center;
}

.avatar-emoji {
	font-size: 60rpx;
}

.username {
	font-size: 40rpx;
	font-weight: bold;
	color: #fff;
	display: block;
	margin-bottom: 10rpx;
}

.user-id {
	font-size: 24rpx;
	color: rgba(255, 255, 255, 0.8);
}

.section-title {
	font-size: 32rpx;
	font-weight: bold;
	color: #333;
	display: block;
	margin-bottom: 20rpx;
}

.achievements {
	background: #fff;
	border-radius: 30rpx;
	padding: 30rpx;
	margin-bottom: 30rpx;
	box-shadow: 0 4rpx 20rpx rgba(0, 0, 0, 0.08);
}

.achievement-grid {
	display: grid;
	grid-template-columns: repeat(3, 1fr);
	gap: 20rpx;
}

.achievement-item {
	display: flex;
	flex-direction: column;
	align-items: center;
	padding: 20rpx;
	background: #f5f7fa;
	border-radius: 20rpx;
	opacity: 0.5;
}

.achievement-item.unlocked {
	background: linear-gradient(135deg, #ffe5e5 0%, #fff0f0 100%);
	opacity: 1;
}

.achievement-icon {
	font-size: 48rpx;
	margin-bottom: 10rpx;
}

.achievement-name {
	font-size: 24rpx;
	font-weight: bold;
	color: #333;
	margin-bottom: 5rpx;
}

.achievement-desc {
	font-size: 20rpx;
	color: #999;
	text-align: center;
}

.statistics {
	background: #fff;
	border-radius: 30rpx;
	padding: 30rpx;
	margin-bottom: 30rpx;
	box-shadow: 0 4rpx 20rpx rgba(0, 0, 0, 0.08);
}

.stat-grid {
	display: grid;
	grid-template-columns: repeat(2, 1fr);
	gap: 20rpx;
}

.stat-card {
	background: linear-gradient(135deg, #f5f7fa 0%, #e8eaf6 100%);
	border-radius: 20rpx;
	padding: 30rpx;
	text-align: center;
}

.stat-value {
	font-size: 48rpx;
	font-weight: bold;
	color: #667eea;
	display: block;
	margin-bottom: 10rpx;
}

.stat-label {
	font-size: 24rpx;
	color: #666;
}

.settings {
	background: #fff;
	border-radius: 30rpx;
	padding: 30rpx;
	margin-bottom: 30rpx;
	box-shadow: 0 4rpx 20rpx rgba(0, 0, 0, 0.08);
}

.setting-list {
	display: flex;
	flex-direction: column;
}

.setting-item {
	display: flex;
	justify-content: space-between;
	align-items: center;
	padding: 25rpx 0;
	border-bottom: 1rpx solid #f5f7fa;
}

.setting-item:last-child {
	border-bottom: none;
}

.setting-label {
	font-size: 28rpx;
	color: #333;
}

.setting-arrow {
	font-size: 40rpx;
	color: #999;
}

.harmony-features {
	background: #fff;
	border-radius: 30rpx;
	padding: 30rpx;
	box-shadow: 0 4rpx 20rpx rgba(0, 0, 0, 0.08);
}

.feature-list {
	display: flex;
	flex-direction: column;
	gap: 20rpx;
}

.feature-item {
	display: flex;
	gap: 20rpx;
	padding: 20rpx;
	background: linear-gradient(135deg, #e3f2fd 0%, #bbdefb 100%);
	border-radius: 20rpx;
}

.feature-icon {
	font-size: 48rpx;
}

.feature-info {
	flex: 1;
	display: flex;
	flex-direction: column;
	justify-content: center;
}

.feature-name {
	font-size: 28rpx;
	font-weight: bold;
	color: #333;
	margin-bottom: 5rpx;
}

.feature-desc {
	font-size: 24rpx;
	color: #666;
}
</style>
