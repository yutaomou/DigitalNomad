<template>
  <view class="gov-dashboard">
    <view class="header-section">
      <view class="nav-bar">
        <view>
          <text class="app-title">聊城数字游民大脑</text>
          <text class="app-subtitle">CITY BRAIN OPERATION CENTER</text>
        </view>
        <text class="date-tag">2026.01.29</text>
      </view>

      <scroll-view scroll-x class="kpi-scroll" show-scrollbar="false">
        <view class="kpi-card blue">
          <view class="kpi-icon">👥</view>
          <view class="kpi-info">
            <text class="label">在聊游民总数</text>
            <text class="num">1,208</text>
            <view class="trend">环比 +12% <text class="up">▲</text></view>
          </view>
        </view>
        <view class="kpi-card purple">
          <view class="kpi-icon">💰</view>
          <view class="kpi-info">
            <text class="label">带动文旅消费</text>
            <text class="num">85.4w</text>
            <view class="trend">环比 +8.5% <text class="up">▲</text></view>
          </view>
        </view>
        <view class="kpi-card orange">
          <view class="kpi-icon">🏠</view>
          <view class="kpi-info">
            <text class="label">社区入住率</text>
            <text class="num">78%</text>
            <view class="trend">剩余工位 45</view>
          </view>
        </view>
      </scroll-view>
    </view>

    <view class="chart-container">
      
      <view class="chart-card half">
        <view class="card-head">游民画像分布</view>
        <view class="donut-chart-box">
          <svg viewBox="0 0 100 100" class="svg-donut">
            <circle cx="50" cy="50" r="40" fill="none" stroke="#f0f0f0" stroke-width="12" />
            <circle 
              v-for="(item, index) in pieData" 
              :key="index"
              cx="50" cy="50" r="40" 
              fill="none" 
              :stroke="item.color" 
              stroke-width="12"
              :stroke-dasharray="`${item.percent * 2.51} 251`"
              :stroke-dashoffset="calculateOffset(index)"
              stroke-linecap="round"
              transform="rotate(-90 50 50)"
            />
          </svg>
          <view class="center-text">
            <text class="main">TOP1</text>
            <text class="sub">程序员</text>
          </view>
        </view>
        <view class="legend-box">
          <view class="legend-item" v-for="(item, i) in pieData" :key="i">
            <view class="dot" :style="{ background: item.color }"></view>
            <text class="name">{{ item.name }}</text>
            <text class="val">{{ item.percent }}%</text>
          </view>
        </view>
      </view>

      <view class="chart-card half">
        <view class="card-head">资源负载 TOP3</view>
        <view class="bar-chart-box">
          <view class="bar-item" v-for="(item, i) in resourceData" :key="i">
            <view class="bar-info">
              <text class="b-name">{{ item.name }}</text>
              <text class="b-val" :class="item.status">{{ item.value }}%</text>
            </view>
            <view class="track">
              <view class="fill" :style="{ width: item.value + '%', background: item.color }"></view>
            </view>
          </view>
        </view>
      </view>

      <view class="chart-card full">
        <view class="card-head">近半年消费趋势 (万元)</view>
        <view class="line-chart-box">
          <svg viewBox="0 0 300 100" class="svg-line" preserveAspectRatio="none">
            <line x1="0" y1="25" x2="300" y2="25" stroke="#eee" stroke-width="1" />
            <line x1="0" y1="50" x2="300" y2="50" stroke="#eee" stroke-width="1" />
            <line x1="0" y1="75" x2="300" y2="75" stroke="#eee" stroke-width="1" />
            <polyline 
              fill="none" 
              stroke="#007AFF" 
              stroke-width="3" 
              :points="trendPoints"
              stroke-linejoin="round"
            />
            <polygon 
              fill="rgba(0,122,255,0.1)" 
              :points="`0,100 ${trendPoints} 300,100`" 
            />
          </svg>
          <view class="x-axis">
            <text v-for="(m, i) in months" :key="i">{{ m }}</text>
          </view>
        </view>
      </view>
    </view>

    <view class="action-grid">
      <view class="grid-item" @click="navTo('AuditView')">
        <view class="icon-wrap color-1">📋</view>
        <text>合规审核</text>
        <view class="badge">12</view>
      </view>
      <view class="grid-item" @click="navTo('EcoView')">
        <view class="icon-wrap color-2">🤝</view>
        <text>生态监控</text>
      </view>
      <view class="grid-item" @click="navTo('ActivityView')">
        <view class="icon-wrap color-3">🚩</view>
        <text>活动配置</text>
      </view>
    </view>
  </view>
</template>

<script setup>
import { ref, computed } from 'vue';

// --- 数据配置 ---

// 1. 饼图数据 (游民职业)
const pieData = ref([
  { name: '开发', percent: 45, color: '#007AFF' },
  { name: '设计', percent: 30, color: '#34C759' },
  { name: '自媒体', percent: 25, color: '#FF9500' }
]);

// 2. 资源负载数据
const resourceData = ref([
  { name: '古城共享工位', value: 92, color: '#FF3B30', status: 'high' },
  { name: '东昌湖民宿', value: 75, color: '#FF9500', status: 'mid' },
  { name: '会议中心', value: 40, color: '#34C759', status: 'low' }
]);

// 3. 折线图数据 (近6个月消费)
const months = ['8月', '9月', '10月', '11月', '12月', '1月'];
const values = [30, 45, 40, 60, 85, 95]; // 0-100 的相对值

// 计算 SVG 圆环偏移量 (实现首尾相接)
const calculateOffset = (index) => {
  let prevPercent = 0;
  for(let i=0; i<index; i++) {
    prevPercent += pieData.value[i].percent;
  }
  // 251 是 r=40 时的周长 (2 * PI * 40 ≈ 251)
  return -prevPercent * 2.51; 
};

// 计算折线图 Points 字符串
const trendPoints = computed(() => {
  // SVG 宽 300, 高 100
  const stepX = 300 / (values.length - 1);
  return values.map((val, index) => {
    const x = index * stepX;
    const y = 100 - val; // 坐标系Y轴向下，所以用100减
    return `${x},${y}`;
  }).join(' ');
});

// --- 路由跳转 ---
const navTo = (page) => {
  uni.navigateTo({ url: `/pages/gov/${page}` });
};

const showTip = () => {
  uni.showToast({ title: '报表已生成至邮箱', icon: 'success' });
};
</script>

<style lang="scss" scoped>
.gov-dashboard {
  min-height: 100vh;
  background-color: #F5F7FA;
  padding-bottom: 40rpx;
}

/* 顶部区域 */
.header-section {
  background: linear-gradient(135deg, #1c1e26, #2a2d3e);
  padding: 40rpx 0 20rpx 30rpx;
  border-bottom-left-radius: 40rpx;
  border-bottom-right-radius: 40rpx;
  color: #fff;
  margin-bottom: 30rpx;
}

.nav-bar {
  display: flex; justify-content: space-between; align-items: flex-start; margin-bottom: 40rpx; padding-right: 30rpx;
  .app-title { font-size: 36rpx; font-weight: bold; display: block; }
  .app-subtitle { font-size: 20rpx; opacity: 0.6; letter-spacing: 2rpx; display: block; margin-top: 6rpx; }
  .date-tag { font-size: 24rpx; background: rgba(255,255,255,0.1); padding: 4rpx 12rpx; border-radius: 20rpx; }
}

.kpi-scroll { white-space: nowrap; height: 180rpx; }
.kpi-card {
  display: inline-block; width: 280rpx; height: 160rpx; margin-right: 20rpx; padding: 24rpx; border-radius: 20rpx; vertical-align: top;
  position: relative; overflow: hidden;
  
  &.blue { background: linear-gradient(135deg, #007AFF, #00C6FF); }
  &.purple { background: linear-gradient(135deg, #5856D6, #C644FC); }
  &.orange { background: linear-gradient(135deg, #FF9500, #FFD60A); }

  .kpi-icon { position: absolute; right: -10rpx; bottom: -20rpx; font-size: 80rpx; opacity: 0.2; }
  .label { font-size: 22rpx; opacity: 0.9; display: block; margin-bottom: 10rpx; color: #fff; }
  .num { font-size: 40rpx; font-weight: bold; color: #fff; display: block; font-family: 'DIN'; }
  .trend { font-size: 20rpx; color: rgba(255,255,255,0.8); margin-top: 8rpx; .up { font-size: 16rpx; } }
}

/* 图表容器 */
.chart-container {
  padding: 0 30rpx;
  display: flex; flex-wrap: wrap; justify-content: space-between;
}

.chart-card {
  background: #fff; border-radius: 24rpx; padding: 24rpx; margin-bottom: 24rpx;
  box-shadow: 0 4rpx 20rpx rgba(0,0,0,0.03);
  
  &.half { width: 48%; }
  &.full { width: 100%; }

  .card-head { font-size: 26rpx; font-weight: bold; color: #333; margin-bottom: 20rpx; border-left: 6rpx solid #007AFF; padding-left: 12rpx; }
}

/* 环形图样式 */
.donut-chart-box {
  position: relative; width: 200rpx; height: 200rpx; margin: 0 auto 20rpx;
  .svg-donut { width: 100%; height: 100%; transform: rotate(-90deg); }
  .center-text {
    position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); text-align: center;
    .main { font-size: 24rpx; font-weight: bold; color: #007AFF; display: block; }
    .sub { font-size: 20rpx; color: #999; }
  }
}
.legend-box {
  .legend-item {
    display: flex; align-items: center; font-size: 20rpx; margin-bottom: 8rpx;
    .dot { width: 12rpx; height: 12rpx; border-radius: 50%; margin-right: 8rpx; }
    .name { flex: 1; color: #666; }
    .val { font-weight: bold; color: #333; }
  }
}

/* 柱状图样式 */
.bar-chart-box {
  .bar-item { margin-bottom: 24rpx; &:last-child { margin-bottom: 0; } }
  .bar-info { display: flex; justify-content: space-between; margin-bottom: 6rpx; font-size: 20rpx; 
    .b-name { color: #666; } .b-val { font-weight: bold; &.high { color: #FF3B30; } }
  }
  .track { height: 10rpx; background: #f0f0f0; border-radius: 5rpx; overflow: hidden; 
    .fill { height: 100%; border-radius: 5rpx; transition: width 0.5s ease; }
  }
}

/* 折线图样式 */
.line-chart-box {
  width: 100%; height: 200rpx; position: relative;
  .svg-line { width: 100%; height: 100%; overflow: visible; }
  .x-axis { 
    display: flex; justify-content: space-between; margin-top: 10rpx; 
    text { font-size: 20rpx; color: #999; }
  }
}

/* 底部操作区 */
.action-grid {
  display: flex; justify-content: space-between; padding: 0 30rpx; margin-top: 10rpx;
  .grid-item {
    width: 23%; background: #fff; border-radius: 20rpx; padding: 20rpx 0;
    display: flex; flex-direction: column; align-items: center; position: relative;
    box-shadow: 0 4rpx 12rpx rgba(0,0,0,0.03);

    .icon-wrap {
      width: 70rpx; height: 70rpx; border-radius: 20rpx; display: flex; align-items: center; justify-content: center;
      font-size: 32rpx; color: #fff; margin-bottom: 12rpx;
      &.color-1 { background: linear-gradient(135deg, #4facfe, #00f2fe); }
      &.color-2 { background: linear-gradient(135deg, #a18cd1, #fbc2eb); }
      &.color-3 { background: linear-gradient(135deg, #f6d365, #fda085); }
      &.color-4 { background: linear-gradient(135deg, #84fab0, #8fd3f4); }
    }
    text { font-size: 22rpx; color: #333; }
    .badge {
      position: absolute; top: 10rpx; right: 10rpx;
      background: #FF3B30; color: #fff; font-size: 18rpx; padding: 0 8rpx; border-radius: 10rpx;
    }
  }
}
</style>