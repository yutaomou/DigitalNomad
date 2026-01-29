<template>
  <view class="page-container">
    <view class="page-header">
      <text class="title">生态合作监控</text>
      <text class="sub">实时大数据汇总</text>
    </view>

    <view class="card stats-grid">
      <view class="stat-item">
        <text class="num">{{ summary.merchants }}</text>
        <text class="label">入驻商户</text>
      </view>
      <view class="stat-item">
        <text class="num">¥{{ summary.revenue }}</text>
        <text class="label">全城流水</text>
      </view>
      <view class="stat-item">
        <text class="num">{{ summary.activities }}</text>
        <text class="label">活动总数</text>
      </view>
    </view>

    <view class="card">
      <view class="card-title">系统接口状态</view>
      <view class="status-line">
        <text>高德地图 SDK</text>
        <text class="badge">RUNNING</text>
      </view>
      <view class="status-line">
        <text>uniCloud 数据库</text>
        <text class="badge">CONNECTED</text>
      </view>
    </view>
  </view>
</template>

<script setup>
import { reactive } from 'vue';
import { onShow } from '@dcloudio/uni-app';
const db = uniCloud.database();
const summary = reactive({ merchants: 0, revenue: 0, activities: 0 });

onShow(() => {
  fetchData();
});

const fetchData = async () => {
  // 统计已通过商户
  const mRes = await db.collection('merchants').where({ status: 1 }).count();
  summary.merchants = mRes.result.total;
  
  // 统计活动总数
  const aRes = await db.collection('activities').count();
  summary.activities = aRes.result.total;

  // 统计流水 (从订单表)
  const oRes = await db.collection('orders').get();
  summary.revenue = oRes.result.data.reduce((s, i) => s + (Number(i.final_amount) || 0), 0).toFixed(2);
};
</script>

<style lang="scss" scoped>
.page-container { padding: 30rpx; background: #f0f2f5; min-height: 100vh; }
.page-header { margin-bottom: 40rpx; .title { font-size: 36rpx; font-weight: bold; } .sub { font-size: 24rpx; color: #999; } }
.card { background: #fff; border-radius: 24rpx; padding: 30rpx; margin-bottom: 30rpx;
  &.stats-grid { display: flex; justify-content: space-around;
    .stat-item { text-align: center; .num { font-size: 36rpx; font-weight: bold; color: #007AFF; display: block; } .label { font-size: 22rpx; color: #999; } }
  }
  .card-title { font-size: 28rpx; font-weight: bold; margin-bottom: 20rpx; }
}
.status-line { display: flex; justify-content: space-between; padding: 15rpx 0; border-bottom: 1rpx solid #f5f5f5; font-size: 26rpx;
  .badge { color: #52c41a; font-weight: bold; font-size: 22rpx; }
}
</style>