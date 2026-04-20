<template>
	<view class="game-page">
		<view class="game-header">
			<text class="game-title">泡泡乐园</text>
			<view class="score-board">
				<text class="score">得分: {{ score }}</text>
				<text class="level">关卡: {{ level }}</text>
			</view>
		</view>

		<view class="game-area" @touchstart="handleTap">
			<view
				v-for="(target, index) in targets"
				:key="target.id"
				class="target"
				:style="{
					left: target.x + 'px',
					top: target.y + 'px',
					width: target.size + 'px',
					height: target.size + 'px',
					background: target.color
				}"
				@tap="hitTarget(index)"
			>
				<text class="target-emoji">{{ target.emoji }}</text>
			</view>
		</view>

		<view class="game-controls">
			<button class="start-btn" @tap="startGame" v-if="!isPlaying">开始游戏</button>
			<button class="pause-btn" @tap="pauseGame" v-else>
				{{ isPaused ? '继续' : '暂停' }}
			</button>
		</view>

		<view class="tips">
			<text>点击出现的泡泡得分，避开黑色炸弹！</text>
		</view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			score: 0,
			level: 1,
			isPlaying: false,
			isPaused: false,
			targets: [],
			gameTimer: null,
			targetId: 0,
			targetTypes: [
				{ emoji: '⭐', color: 'linear-gradient(135deg, #FFD93D 0%, #FFA502 100%)', points: 10 },
				{ emoji: '💖', color: 'linear-gradient(135deg, #FF6B9D 0%, #FF8E53 100%)', points: 15 },
				{ emoji: '🌟', color: 'linear-gradient(135deg, #A29BFE 0%, #6C5CE7 100%)', points: 20 },
				{ emoji: '💣', color: 'linear-gradient(135deg, #636E72 0%, #2D3436 100%)', points: -30, isBomb: true }
			]
		};
	},
	methods: {
		startGame() {
			this.isPlaying = true;
			this.isPaused = false;
			this.score = 0;
			this.level = 1;
			this.targets = [];
			this.spawnTarget();
		},

		pauseGame() {
			this.isPaused = !this.isPaused;
			if (!this.isPaused) {
				this.spawnTarget();
			}
		},

		spawnTarget() {
			if (!this.isPlaying || this.isPaused) return;

			const containerWidth = uni.getSystemInfoSync().windowWidth;
			const containerHeight = uni.getSystemInfoSync().windowHeight - 300;

			const typeIndex = Math.random() > 0.8 ? 3 : Math.floor(Math.random() * 3);
			const type = this.targetTypes[typeIndex];

			const target = {
				id: ++this.targetId,
				x: Math.random() * (containerWidth - 100),
				y: Math.random() * (containerHeight - 100),
				size: 80 + Math.random() * 40,
				color: type.color,
				emoji: type.emoji,
				points: type.points,
				isBomb: type.isBomb || false,
				createdAt: Date.now()
			};

			this.targets.push(target);

			// 根据等级调整生成速度
			const spawnInterval = Math.max(500, 1500 - this.level * 100);
			this.gameTimer = setTimeout(() => this.spawnTarget(), spawnInterval);

			// 自动移除旧目标
			setTimeout(() => {
				const index = this.targets.findIndex((t) => t.id === target.id);
				if (index !== -1) {
					this.targets.splice(index, 1);
				}
			}, 3000);
		},

		hitTarget(index) {
			const target = this.targets[index];

			if (target.isBomb) {
				this.score += target.points;
				uni.showToast({
					title: '踩到炸弹了！',
					icon: 'none'
				});
			} else {
				this.score += target.points;

				// 升级检测
				if (this.score >= this.level * 100) {
					this.level++;
					uni.showToast({
						title: `升级到第${this.level}关！`,
						icon: 'success'
					});
				}
			}

			this.targets.splice(index, 1);
		},

		handleTap(e) {
			// 可以添加更多交互
		}
	},
	onUnload() {
		if (this.gameTimer) {
			clearTimeout(this.gameTimer);
		}
	}
};
</script>

<style scoped>
.game-page {
	min-height: 100vh;
	background: linear-gradient(180deg, #667eea 0%, #764ba2 100%);
	padding: 40rpx;
}

.game-header {
	display: flex;
	justify-content: space-between;
	align-items: center;
	margin-bottom: 40rpx;
	padding-top: 30rpx;
}

.game-title {
	font-size: 48rpx;
	font-weight: bold;
	color: #fff;
}

.score-board {
	display: flex;
	gap: 30rpx;
}

.score,
.level {
	font-size: 32rpx;
	color: #fff;
	background: rgba(255, 255, 255, 0.2);
	padding: 10rpx 20rpx;
	border-radius: 20rpx;
}

.game-area {
	height: 60vh;
	background: rgba(255, 255, 255, 0.1);
	border-radius: 30rpx;
	position: relative;
	overflow: hidden;
	margin-bottom: 40rpx;
}

.target {
	position: absolute;
	border-radius: 50%;
	display: flex;
	align-items: center;
	justify-content: center;
	animation: pulse 1s ease-in-out infinite;
	box-shadow: 0 8rpx 24rpx rgba(0, 0, 0, 0.2);
}

.target-emoji {
	font-size: 48rpx;
}

@keyframes pulse {
	0%,
	100% {
		transform: scale(1);
	}
	50% {
		transform: scale(1.1);
	}
}

.game-controls {
	display: flex;
	justify-content: center;
	margin-bottom: 30rpx;
}

.start-btn,
.pause-btn {
	background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
	color: #fff;
	border-radius: 30rpx;
	padding: 20rpx 60rpx;
	font-size: 32rpx;
	font-weight: bold;
	border: none;
}

.tips {
	text-align: center;
	color: rgba(255, 255, 255, 0.8);
	font-size: 28rpx;
}
</style>
