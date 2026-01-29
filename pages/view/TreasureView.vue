<template>
  <view class="page-container">
    <view class="assets-header">
      <view class="coin-info">
        <text class="label">当前游民币</text>
        <view class="balance">
          <text class="symbol">🪙</text>
          <text class="num">1,280</text>
        </view>
      </view>
      <button class="shop-btn">兑换商城</button>
      <image class="wave-bg" src="/static/icon/logo.png" mode="widthFix" />
    </view>

    <view class="section-container" v-if="activeTask">
      <view class="section-title">
        <text class="title-text">正在挑战</text>
        <text class="status-badge">进行中</text>
      </view>
      
      <view class="active-card">
        <view class="card-main">
          <view class="info">
            <text class="task-name">{{ activeTask.title }}</text>
            <text class="task-desc">{{ activeTask.desc }}</text>
          </view>
          <view class="reward-tag">+{{ activeTask.points }}</view>
        </view>
        
        <view class="progress-box">
          <view class="progress-bar">
            <view class="fill" :style="{ width: activeTask.progress + '%' }"></view>
          </view>
          <text class="progress-text">进度 {{ activeTask.progress }}%</text>
        </view>

        <view class="action-row">
          <button class="secondary-btn" @click="showClue">💡 查看线索</button>
          <button class="primary-btn" @click="goToMap">📍 去地图打卡</button>
        </view>
      </view>
    </view>

    <view class="section-container">
      <view class="section-title">任务广场</view>
      <scroll-view scroll-y class="task-list">
        <view class="task-item" v-for="(task, index) in taskList" :key="index">
          <view class="task-icon" :class="task.colorClass">{{ task.icon }}</view>
          <view class="task-body">
            <view class="body-top">
              <text class="t-title">{{ task.title }}</text>
              <text class="difficulty">{{ task.difficultyLabel }}</text>
            </view>
            <text class="t-desc">{{ task.subTitle }}</text>
            <text class="t-reward">🪙 {{ task.points }}</text>
          </view>
          <button class="claim-btn" :disabled="task.status !== 'open'" @click="claimTask(task)">
            {{ task.statusText }}
          </button>
        </view>
      </scroll-view>
    </view>
  </view>
</template>

<script setup>
import { reactive } from 'vue';

// 模拟正在进行的任务（这里的数据决定了点击后去哪里）
const activeTask = reactive({
  id: 101,
  title: '寻找光岳楼',
  desc: '前往光岳楼打卡，拍摄一张包含全景的照片。',
  points: 500,
  progress: 30,
  // 关键：任务的目标坐标 (光岳楼坐标)
  targetLat: 36.456013,
  targetLng: 115.986689
});

const taskList = reactive([
  { id: 1, title: '运河古码头扫描', subTitle: '拍摄码头细节', points: 200, icon: '📸', colorClass: 'blue', difficultyLabel: '简单', status: 'open', statusText: '领取' },
  { id: 2, title: '水上巴士体验', subTitle: '乘坐一次并打卡', points: 300, icon: '🚢', colorClass: 'orange', difficultyLabel: '简单', status: 'done', statusText: '已完成' }
]);

// 核心跳转逻辑
const goToMap = () => {
  if (!activeTask.targetLat || !activeTask.targetLng) return;

  // 使用 navigateTo 并在 URL 中携带参数
  // 格式: url?key=value&key2=value2
  uni.navigateTo({
    url: `/pages/view/MapView?lat=${activeTask.targetLat}&lng=${activeTask.targetLng}&name=${activeTask.title}`,
    success: () => console.log('跳转地图成功'),
    fail: (err) => console.error('跳转失败', err)
  });
};

const showClue = () => {
  uni.showModal({ title: '线索', content: '它位于古城中心，是聊城的象征。', showCancel: false });
};

const claimTask = (task) => {
  if(task.status === 'open') uni.showToast({ title: '任务领取成功', icon: 'success' });
};
</script>

<style lang="scss" scoped>
.page-container { min-height: 100vh; background-color: #F5F7FA; padding-bottom: 30rpx; }

/* 头部资产 */
.assets-header {
  background: linear-gradient(135deg, #2A2D3E, #1F2233); padding: 40rpx 30rpx 80rpx 30rpx;
  border-bottom-left-radius: 30rpx; border-bottom-right-radius: 30rpx;
  display: flex; justify-content: space-between; align-items: center; position: relative; overflow: hidden;
  .label { color: rgba(255,255,255,0.6); font-size: 24rpx; display: block; }
  .balance { display: flex; align-items: center; .symbol { font-size: 32rpx; margin-right: 10rpx; } .num { font-size: 48rpx; color: #FFD700; font-weight: bold; } }
  .shop-btn { background: rgba(255,255,255,0.15); color: #fff; font-size: 24rpx; border-radius: 30rpx; border: 1px solid rgba(255,255,255,0.2); margin: 0; }
  .wave-bg { position: absolute; right: -20rpx; bottom: -40rpx; width: 200rpx; opacity: 0.1; }
}

/* 正在挑战卡片 */
.section-container { padding: 0 30rpx; margin-top: 30rpx; position: relative; z-index: 2; }
.section-container:first-of-type { margin-top: -60rpx; } /* 向上偏移覆盖Header */

.section-title { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20rpx;
  .title-text { font-size: 32rpx; font-weight: bold; color: #333; }
  .status-badge { font-size: 20rpx; background: #E3F2FD; color: #007AFF; padding: 4rpx 12rpx; border-radius: 8rpx; }
}

.active-card {
  background: #fff; border-radius: 24rpx; padding: 30rpx; box-shadow: 0 8rpx 20rpx rgba(0,0,0,0.06);
  .card-main { display: flex; justify-content: space-between; margin-bottom: 20rpx; }
  .task-name { font-size: 30rpx; font-weight: bold; color: #333; display: block; }
  .task-desc { font-size: 24rpx; color: #666; margin-top: 6rpx; display: block; }
  .reward-tag { background: #FFF8E1; color: #FF8F00; font-size: 24rpx; padding: 4rpx 12rpx; border-radius: 8rpx; height: fit-content; font-weight: bold; }
  
  .progress-box { margin-bottom: 30rpx; .progress-bar { height: 12rpx; background: #eee; border-radius: 6rpx; overflow: hidden; .fill { height: 100%; background: #007AFF; } } .progress-text { font-size: 20rpx; color: #999; float: right; margin-top: 6rpx; } }
  
  .action-row { display: flex; gap: 20rpx; button { flex: 1; font-size: 26rpx; height: 70rpx; line-height: 70rpx; border-radius: 35rpx; margin: 0; } .secondary-btn { background: #F5F7FA; color: #666; } .primary-btn { background: #007AFF; color: #fff; } }
}

/* 任务列表 */
.task-item {
  background: #fff; border-radius: 20rpx; padding: 24rpx; margin-bottom: 20rpx; display: flex; align-items: center;
  .task-icon { width: 80rpx; height: 80rpx; border-radius: 16rpx; display: flex; align-items: center; justify-content: center; font-size: 36rpx; margin-right: 20rpx; &.blue { background: #E3F2FD; } &.orange { background: #FFF3E0; } }
  .task-body { flex: 1; .t-title { font-size: 28rpx; font-weight: bold; margin-right: 10rpx; } .difficulty { font-size: 20rpx; color: #4CAF50; background: #E8F5E9; padding: 2rpx 8rpx; border-radius: 6rpx; } .t-desc { font-size: 22rpx; color: #999; display: block; margin: 6rpx 0; } .t-reward { font-size: 24rpx; color: #FF8F00; font-weight: bold; } }
  .claim-btn { font-size: 24rpx; background: #333; color: #fff; height: 50rpx; line-height: 50rpx; padding: 0 24rpx; border-radius: 25rpx; margin: 0; &[disabled] { background: #eee; color: #bbb; } }
}
</style>
