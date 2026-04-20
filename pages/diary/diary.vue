<template>
  <view class="diary-page">
    <view class="diary-header">
      <text class="diary-title">心情日记</text>
      <text class="diary-subtitle">记录每一天的情绪轨迹</text>
    </view>

    <!-- 情绪日历 -->
    <view class="calendar-section">
      <view class="month-nav">
        <text class="arrow" @tap="prevMonth">‹</text>
        <text class="month-text">{{ currentMonth }}</text>
        <text class="arrow" @tap="nextMonth">›</text>
      </view>

      <view class="calendar-grid">
        <view class="weekday" v-for="day in weekdays" :key="day">
          {{ day }}
        </view>
        <view
          v-for="(date, index) in calendarDates"
          :key="index"
          class="date-cell"
          :class="{
            today: date.isToday,
            'has-mood': date.mood,
          }"
          @tap="showDayDetail(date)"
        >
          <text class="date-num">{{ date.day }}</text>
          <text class="date-emoji" v-if="date.mood">{{ date.mood }}</text>
        </view>
      </view>
    </view>

    <!-- 情绪统计图表 -->
    <view class="stats-section">
      <text class="section-title">本周情绪分布</text>
      <view class="mood-bars">
        <view
          v-for="(item, index) in moodStats"
          :key="index"
          class="mood-bar-item"
        >
          <text class="bar-emoji">{{ item.emoji }}</text>
          <view class="bar-container">
            <view
              class="bar-fill"
              :style="{
                width: item.percentage + '%',
                background: item.color,
              }"
            ></view>
          </view>
          <text class="bar-count">{{ item.count }}</text>
        </view>
      </view>
    </view>

    <!-- 最近记录 -->
    <view class="recent-section">
      <text class="section-title">最近心情</text>
      <view class="mood-list">
        <view
          v-for="(record, index) in recentRecords"
          :key="index"
          class="mood-card"
        >
          <view class="mood-icon">{{ record.emoji }}</view>
          <view class="mood-content">
            <text class="mood-time">{{ record.time }}</text>
            <text class="mood-text">{{ record.text }}</text>
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
      currentDate: new Date(),
      weekdays: ["日", "一", "二", "三", "四", "五", "六"],
      calendarDates: [],
      moodStats: [],
      recentRecords: [],
    };
  },
  computed: {
    currentMonth() {
      const year = this.currentDate.getFullYear();
      const month = this.currentDate.getMonth() + 1;
      return `${year}年${month}月`;
    },
  },
  onLoad() {
    this.generateCalendar();
    this.loadRecentRecords();
    this.loadMoodStats();
  },
  onShow() {
    this.generateCalendar();
    this.loadRecentRecords();
    this.loadMoodStats();
  },
  methods: {
    loadMoodStats() {
      const bubbles = uni.getStorageSync("bubbles") || [];

      const now = new Date();
      const oneWeekAgo = new Date(now.getTime() - 7 * 24 * 60 * 60 * 1000);

      const weekBubbles = bubbles.filter((b) => {
        const bubbleDate = new Date(b.createdAt);
        return bubbleDate >= oneWeekAgo && bubbleDate <= now;
      });

      const moodCounts = {
        happy: 0,
        sad: 0,
        angry: 0,
        calm: 0,
        excited: 0,
      };

      weekBubbles.forEach((b) => {
        if (moodCounts[b.mood] !== undefined) {
          moodCounts[b.mood]++;
        }
      });

      const totalCount = weekBubbles.length || 1;

      const moodConfig = {
        happy: { emoji: "😊", color: "#FFD93D" },
        sad: { emoji: "😢", color: "#74B9FF" },
        angry: { emoji: "😠", color: "#FF7675" },
        calm: { emoji: "😌", color: "#A29BFE" },
        excited: { emoji: "🤩", color: "#FD79A8" },
      };

      this.moodStats = Object.entries(moodCounts)
        .filter(([_, count]) => count > 0)
        .map(([mood, count]) => ({
          emoji: moodConfig[mood].emoji,
          color: moodConfig[mood].color,
          count: count,
          percentage: Math.round((count / totalCount) * 100),
        }))
        .sort((a, b) => b.count - a.count);

      if (this.moodStats.length === 0) {
        this.moodStats = [
          { emoji: "😊", color: "#FFD93D", count: 0, percentage: 0 },
        ];
      }
    },
    generateCalendar() {
      const year = this.currentDate.getFullYear();
      const month = this.currentDate.getMonth();

      const firstDay = new Date(year, month, 1).getDay();
      const daysInMonth = new Date(year, month + 1, 0).getDate();
      const today = new Date();

      const dates = [];

      // 填充上月日期
      for (let i = 0; i < firstDay; i++) {
        dates.push({ day: "", mood: null });
      }

      // 填充本月日期
      for (let day = 1; day <= daysInMonth; day++) {
        const date = new Date(year, month, day);
        const isToday = date.toDateString() === today.toDateString();

        // 从存储中获取该日期的情绪
        const mood = this.getMoodForDate(date);

        dates.push({
          day,
          mood,
          isToday,
          date,
        });
      }

      this.calendarDates = dates;
    },

    getMoodForDate(date) {
      const bubbles = uni.getStorageSync("bubbles") || [];
      const dateStr = date.toDateString();

      const dayBubbles = bubbles.filter(
        (b) => new Date(b.createdAt).toDateString() === dateStr,
      );

      if (dayBubbles.length === 0) return null;

      // 返回最常见的情绪
      const moodCounts = {};
      dayBubbles.forEach((b) => {
        moodCounts[b.mood] = (moodCounts[b.mood] || 0) + 1;
      });

      const dominantMood = Object.entries(moodCounts).sort(
        (a, b) => b[1] - a[1],
      )[0][0];

      const moodEmojis = {
        happy: "😊",
        sad: "😢",
        angry: "😠",
        calm: "😌",
        excited: "🤩",
      };

      return moodEmojis[dominantMood] || "😊";
    },

    prevMonth() {
      const newDate = new Date(this.currentDate);
      newDate.setMonth(newDate.getMonth() - 1);
      this.currentDate = newDate;
      this.generateCalendar();
    },

    nextMonth() {
      const newDate = new Date(this.currentDate);
      newDate.setMonth(newDate.getMonth() + 1);
      this.currentDate = newDate;
      this.generateCalendar();
    },

    showDayDetail(date) {
      if (!date.mood) return;

      const bubbles = uni.getStorageSync("bubbles") || [];
      const dateStr = date.date.toDateString();
      const dayBubbles = bubbles.filter(
        (b) => new Date(b.createdAt).toDateString() === dateStr,
      );

      const texts = dayBubbles.map((b) => b.text).join("\n");

      uni.showModal({
        title: `${date.day}日的心情`,
        content: texts || "这一天很平静",
        showCancel: false,
      });
    },

    loadRecentRecords() {
      const bubbles = uni.getStorageSync("bubbles") || [];
      const moodEmojis = {
        happy: "😊",
        sad: "😢",
        angry: "😠",
        calm: "😌",
        excited: "🤩",
      };

      this.recentRecords = bubbles
        .slice(-5)
        .reverse()
        .map((b) => ({
          emoji: moodEmojis[b.mood] || "😊",
          time: this.formatTime(b.createdAt),
          text: b.text,
        }));
    },

    formatTime(timestamp) {
      const date = new Date(timestamp);
      const month = date.getMonth() + 1;
      const day = date.getDate();
      const hour = date.getHours();
      const minute = date.getMinutes();

      return `${month}/${day} ${hour.toString().padStart(2, "0")}:${minute.toString().padStart(2, "0")}`;
    },
  },
};
</script>

<style scoped>
.diary-page {
  min-height: 100vh;
  background: linear-gradient(180deg, #f5f7fa 0%, #e8eaf6 100%);
  padding: 40rpx;
}

.diary-header {
  margin-bottom: 40rpx;
  padding-top: 30rpx;
}

.diary-title {
  font-size: 48rpx;
  font-weight: bold;
  color: #333;
  display: block;
  margin-bottom: 12rpx;
}

.diary-subtitle {
  font-size: 28rpx;
  color: #666;
}

.calendar-section {
  background: #fff;
  border-radius: 30rpx;
  padding: 30rpx;
  margin-bottom: 30rpx;
  box-shadow: 0 4rpx 20rpx rgba(0, 0, 0, 0.08);
}

.month-nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30rpx;
}

.arrow {
  font-size: 48rpx;
  color: #667eea;
  padding: 10rpx 20rpx;
}

.month-text {
  font-size: 36rpx;
  font-weight: bold;
  color: #333;
}

.calendar-grid {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 10rpx;
}

.weekday {
  text-align: center;
  font-size: 24rpx;
  color: #999;
  padding: 10rpx 0;
}

.date-cell {
  aspect-ratio: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  border-radius: 15rpx;
  background: #f5f7fa;
  position: relative;
}

.date-cell.today {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.date-cell.today .date-num {
  color: #fff;
}

.date-cell.has-mood {
  background: linear-gradient(135deg, #ffe5e5 0%, #fff0f0 100%);
}

.date-num {
  font-size: 28rpx;
  color: #333;
  margin-bottom: 5rpx;
}

.date-emoji {
  font-size: 32rpx;
}

.stats-section {
  background: #fff;
  border-radius: 30rpx;
  padding: 30rpx;
  margin-bottom: 30rpx;
  box-shadow: 0 4rpx 20rpx rgba(0, 0, 0, 0.08);
}

.section-title {
  font-size: 32rpx;
  font-weight: bold;
  color: #333;
  display: block;
  margin-bottom: 20rpx;
}

.mood-bars {
  display: flex;
  flex-direction: column;
  gap: 20rpx;
}

.mood-bar-item {
  display: flex;
  align-items: center;
  gap: 15rpx;
}

.bar-emoji {
  font-size: 40rpx;
  width: 50rpx;
}

.bar-container {
  flex: 1;
  height: 30rpx;
  background: #f5f7fa;
  border-radius: 15rpx;
  overflow: hidden;
}

.bar-fill {
  height: 100%;
  border-radius: 15rpx;
  transition: width 0.5s ease;
}

.bar-count {
  font-size: 28rpx;
  color: #666;
  width: 40rpx;
  text-align: right;
}

.recent-section {
  background: #fff;
  border-radius: 30rpx;
  padding: 30rpx;
  box-shadow: 0 4rpx 20rpx rgba(0, 0, 0, 0.08);
}

.mood-list {
  display: flex;
  flex-direction: column;
  gap: 20rpx;
}

.mood-card {
  display: flex;
  gap: 20rpx;
  padding: 20rpx;
  background: #f5f7fa;
  border-radius: 20rpx;
}

.mood-icon {
  font-size: 56rpx;
}

.mood-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.mood-time {
  font-size: 24rpx;
  color: #999;
  margin-bottom: 8rpx;
}

.mood-text {
  font-size: 28rpx;
  color: #333;
  width: 70%;
  overflow: hidden;
  text-overflow: ellipsis;
  /* #ifndef APP-HARMONY */
  display: -webkit-box;
  -webkit-line-clamp: 4;
  -webkit-box-orient: vertical;
  /* #endif */
  /* #ifdef APP-HARMONY */
  display: -webkit-box;
  -webkit-line-clamp: 4;
  line-clamp: 4;
  box-orient: vertical;
  -webkit-box-orient: vertical;
  /* #endif */
}
</style>
