<template>
  <view class="mood-bubble">
    <!-- 顶部标题 -->
    <view class="header">
      <text class="title">情绪泡泡</text>
      <text class="subtitle">说出你的心情，让它化作泡泡飘走</text>
    </view>

    <!-- 泡泡展示区域 -->
    <view class="bubble-container" @touchstart="handleTouchStart">
      <view
        v-for="(bubble, index) in bubbles"
        :key="bubble.id"
        class="bubble"
        :style="{
          left: bubble.x + 'px',
          top: bubble.y + 'px',
          width: bubble.size + 'px',
          height: bubble.size + 'px',
          background: bubble.color,
          animationDuration: bubble.floatDuration + 's',
        }"
        @tap="popBubble(index)"
      >
        <view class="bubble-shine"></view>
        <text class="bubble-emoji">{{ bubble.emoji }}</text>
      </view>
    </view>

    <!-- 输入区域 -->
    <view class="input-section">
      <view class="mood-selector">
        <view
          v-for="(mood, index) in moodOptions"
          :key="index"
          class="mood-item"
          :class="{ active: selectedMood === mood.type }"
          @tap="selectMood(mood)"
        >
          <text class="mood-emoji">{{ mood.emoji }}</text>
          <text class="mood-label">{{ mood.label }}</text>
        </view>
      </view>

      <view class="input-box">
        <textarea
          v-model="moodText"
          placeholder="今天心情怎么样？..."
          class="mood-input"
          maxlength="100"
        />
        <button class="create-btn" @tap="createBubble" :disabled="!moodText">
          <text>吹泡泡</text>
        </button>
      </view>
    </view>

    <!-- 今日统计 -->
    <view class="stats">
      <view class="stat-item">
        <text class="stat-num">{{ todayCount }}</text>
        <text class="stat-label">今日泡泡</text>
      </view>
      <view class="stat-item">
        <text class="stat-num">{{ streakDays }}</text>
        <text class="stat-label">连续天数</text>
      </view>
      <view class="stat-item">
        <text class="stat-num">{{ totalBubbles }}</text>
        <text class="stat-label">总泡泡数</text>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      bubbles: [],
      moodText: "",
      selectedMood: "",
      todayCount: 0,
      streakDays: 0,
      totalBubbles: 0,
      moodOptions: [
        {
          type: "happy",
          emoji: "😊",
          label: "开心",
          color: "linear-gradient(135deg, #FFD93D 0%, #FFA502 100%)",
        },
        {
          type: "sad",
          emoji: "😢",
          label: "难过",
          color: "linear-gradient(135deg, #74B9FF 0%, #0984E3 100%)",
        },
        {
          type: "angry",
          emoji: "😠",
          label: "生气",
          color: "linear-gradient(135deg, #FF7675 0%, #D63031 100%)",
        },
        {
          type: "calm",
          emoji: "😌",
          label: "平静",
          color: "linear-gradient(135deg, #A29BFE 0%, #6C5CE7 100%)",
        },
        {
          type: "excited",
          emoji: "🤩",
          label: "兴奋",
          color: "linear-gradient(135deg, #FD79A8 0%, #E84393 100%)",
        },
      ],
      bubbleId: 0,
    };
  },
  onLoad() {
    this.loadStats();
    this.loadBubbles();
  },
  methods: {
    selectMood(mood) {
      this.selectedMood = mood.type;
      this.moodText = mood.label;
    },

    createBubble() {
      if (!this.moodText) return;

      const mood =
        this.moodOptions.find((m) => m.type === this.selectedMood) ||
        this.moodOptions[0];
      const containerWidth = uni.getSystemInfoSync().windowWidth;

      const newBubble = {
        id: ++this.bubbleId,
        x: Math.random() * (containerWidth - 100),
        y: uni.getSystemInfoSync().windowHeight - 300,
        size: 80 + Math.random() * 40,
        color: mood.color,
        emoji: mood.emoji,
        floatDuration: 3 + Math.random() * 2,
        text: this.moodText,
        mood: mood.type,
        createdAt: Date.now(),
      };

      this.bubbles.push(newBubble);
      this.saveBubble(newBubble);
      this.updateStats();

      // 播放音效（可选）
      this.playPopSound();

      // 清空输入
      this.moodText = "";
      this.selectedMood = "";

      // 提示
      uni.showToast({
        title: "泡泡已吹出！",
        icon: "success",
      });
    },

    popBubble(index) {
      const bubble = this.bubbles[index];

      // 显示心情建议
      this.showMoodAdvice(bubble);

      // 移除泡泡
      this.bubbles.splice(index, 1);

      // 播放爆破动画效果
      this.playPopAnimation(bubble);
    },

    showMoodAdvice(bubble) {
      const advices = {
        happy: ["保持这份快乐！", "分享你的喜悦吧！", "快乐会传染哦~"],
        sad: ["难过时会过去的", "给自己一个拥抱", "明天会更好"],
        angry: ["深呼吸，放松", "听听音乐吧", "冷静下来想想"],
        calm: ["享受这份宁静", "冥想片刻吧", "内心平和真好"],
        excited: ["太棒了！", "抓住这个机会", "能量满满！"],
      };

      const moodAdvices = advices[bubble.mood] || advices.calm;
      const advice =
        moodAdvices[Math.floor(Math.random() * moodAdvices.length)];

      uni.showModal({
        title: bubble.emoji + " 心情小贴士",
        content: advice,
        showCancel: false,
      });
    },

    handleTouchStart(e) {
      // 可以在这里添加手势交互
    },

    playPopSound() {
      // 播放泡泡音效（需要添加音频文件）
    },

    playPopAnimation(bubble) {
      // 播放爆破动画
    },

    saveBubble(bubble) {
      const bubbles = uni.getStorageSync("bubbles") || [];
      bubbles.push(bubble);
      uni.setStorageSync("bubbles", bubbles);
    },

    loadBubbles() {
      const bubbles = uni.getStorageSync("bubbles") || [];
      // 只显示最近创建的5个泡泡
      this.bubbles = bubbles.slice(-5);
    },

    updateStats() {
      const today = new Date().toDateString();
      const bubbles = uni.getStorageSync("bubbles") || [];

      this.todayCount = bubbles.filter(
        (b) => new Date(b.createdAt).toDateString() === today,
      ).length;

      this.totalBubbles = bubbles.length;

      // 计算连续天数
      this.calculateStreak(bubbles);
    },

    calculateStreak(bubbles) {
      if (bubbles.length === 0) {
        this.streakDays = 0;
        return;
      }

      const dates = [
        ...new Set(bubbles.map((b) => new Date(b.createdAt).toDateString())),
      ].sort((a, b) => new Date(b) - new Date(a));

      let streak = 1;
      for (let i = 1; i < dates.length; i++) {
        const prevDate = new Date(dates[i - 1]);
        const currDate = new Date(dates[i]);
        const diffDays = (prevDate - currDate) / (1000 * 60 * 60 * 24);

        if (diffDays === 1) {
          streak++;
        } else {
          break;
        }
      }

      this.streakDays = streak;
    },

    loadStats() {
      this.updateStats();
    },
  },
};
</script>

<style scoped>
.mood-bubble {
  min-height: 100vh;
  background: linear-gradient(180deg, #e0f7fa 0%, #fff9c4 50%, #f8bbd0 100%);
  display: flex;
  flex-direction: column;
  padding-bottom: env(safe-area-inset-bottom);
}

.header {
  padding: 80rpx 40rpx 40rpx;
  text-align: center;
}

.title {
  font-size: 56rpx;
  font-weight: bold;
  color: #ff6b9d;
  display: block;
  margin-bottom: 16rpx;
}

.subtitle {
  font-size: 28rpx;
  color: #666;
}

.bubble-container {
  flex: 1;
  position: relative;
  overflow: hidden;
  min-height: 400rpx;
}

.bubble {
  position: absolute;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  animation: float-up linear infinite;
  box-shadow:
    0 8rpx 24rpx rgba(0, 0, 0, 0.15),
    inset 0 -10rpx 20rpx rgba(255, 255, 255, 0.3);
  cursor: pointer;
  transition: transform 0.2s;
}

.bubble:active {
  transform: scale(1.2);
}

.bubble-shine {
  position: absolute;
  top: 15%;
  left: 20%;
  width: 25%;
  height: 15%;
  background: rgba(255, 255, 255, 0.6);
  border-radius: 50%;
  transform: rotate(-45deg);
}

.bubble-emoji {
  font-size: 48rpx;
  z-index: 1;
}

@keyframes float-up {
  0% {
    transform: translateY(0) translateX(0);
    opacity: 1;
  }
  50% {
    transform: translateY(-100rpx) translateX(20rpx);
  }
  100% {
    transform: translateY(-200rpx) translateX(-20rpx);
    opacity: 0.8;
  }
}

.input-section {
  padding: 40rpx;
  background: rgba(255, 255, 255, 0.9);
  backdrop-filter: blur(10rpx);
  border-radius: 40rpx 40rpx 0 0;
  margin-top: -40rpx;
}

.mood-selector {
  display: flex;
  justify-content: space-around;
  margin-bottom: 30rpx;
}

.mood-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20rpx;
  border-radius: 20rpx;
  transition: all 0.3s;
}

.mood-item.active {
  background: rgba(255, 107, 157, 0.1);
  transform: scale(1.1);
}

.mood-emoji {
  font-size: 56rpx;
  margin-bottom: 8rpx;
}

.mood-label {
  font-size: 24rpx;
  color: #666;
}

.input-box {
  display: flex;
  gap: 20rpx;
}

.mood-input {
  flex: 1;
  background: #f5f7fa;
  border-radius: 20rpx;
  padding: 24rpx;
  font-size: 28rpx;
  min-height: 80rpx;
}

.create-btn {
  background: linear-gradient(135deg, #ff6b9d 0%, #ff8e53 100%);
  color: #fff;
  border-radius: 20rpx;
  padding: 0 40rpx;
  font-size: 32rpx;
  font-weight: bold;
  border: none;
  box-shadow: 0 8rpx 24rpx rgba(255, 107, 157, 0.3);
}

.create-btn[disabled] {
  opacity: 0.5;
}

.stats {
  display: flex;
  justify-content: space-around;
  padding: 40rpx;
  background: rgba(255, 255, 255, 0.95);
}

.stat-item {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.stat-num {
  font-size: 48rpx;
  font-weight: bold;
  color: #ff6b9d;
  margin-bottom: 8rpx;
}

.stat-label {
  font-size: 24rpx;
  color: #999;
}
</style>
