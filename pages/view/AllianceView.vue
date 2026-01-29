<template>
  <view class="page-container">
    <view class="alliance-header">
      <view class="header-content">
        <text class="title">数字游民生态联盟</text>
        <text class="subtitle">链接全球资源 · 共享聊城权益</text>
        
        <view class="stats-row">
          <view class="stat-item">
            <text class="num">{{ connectedCount }}</text>
            <text class="label">已联通平台</text>
          </view>
          <view class="stat-item">
            <text class="num">{{ totalCreditScore }}</text>
            <text class="label">生态信用分</text>
          </view>
          <view class="stat-item">
            <text class="num">{{ projectCount }}</text>
            <text class="label">共创项目池</text>
          </view>
        </view>
      </view>
      <image class="bg-shape" src="/static/icon/logo.png" mode="aspectFit"></image>
    </view>

    <view class="section-box">
      <view class="section-title">
        <text class="t-text">平台权益互通</text>
        <text class="t-tip">绑定同步信用与接单数据</text>
      </view>
      
      <view class="platform-list">
        <view class="platform-card" v-for="(item, index) in platforms" :key="index">
          <view class="card-left">
            <view class="p-icon" :class="item.styleClass">{{ item.iconText }}</view>
            <view class="p-info">
              <text class="p-name">{{ item.name }}</text>
              <text class="p-desc" v-if="!item.isBound">{{ item.desc }}</text>
              <view class="p-status-row" v-else>
                <text class="status-tag">已认证</text>
                <text class="sync-info">信用分贡献 +{{ item.score }}</text>
              </view>
            </view>
          </view>
          <button 
            class="action-btn" 
            :class="{ bound: item.isBound }"
            :disabled="item.loading" 
            @click="handleBind(item)"
          >
            {{ item.loading ? '同步中...' : (item.isBound ? '管理' : '去绑定') }}
          </button>
        </view>
      </view>
    </view>

    <view class="section-box">
      <view class="section-title">
        <text class="t-text">联盟社区</text>
        <text class="t-tip">持聊城Pass享联盟折扣</text>
      </view>

      <scroll-view scroll-x class="community-scroll">
        <view class="comm-card" v-for="(comm, index) in communities" :key="index" @click="checkCommunity(comm)">
          <view class="img-box" :class="comm.bgClass">
            <text class="loc-badge">{{ comm.location }}</text>
          </view>
          <view class="comm-info">
            <text class="c-name">{{ comm.name }}</text>
            <view class="c-tags">
              <text class="tag-discount">聊城客享8折</text>
            </view>
            <view class="c-action">
              <text class="price">¥{{ comm.price }}<text class="unit">/晚</text></text>
              <text class="btn-text">预订 ></text>
            </view>
          </view>
        </view>
      </scroll-view>
    </view>

    <view class="section-box">
      <view class="section-title">
        <text class="t-text">共创实验室</text>
        <text class="t-tip">参与IP孵化赢奖金</text>
      </view>
      
      <view class="project-list">
        <view class="proj-item" v-for="(proj, index) in projects" :key="index">
          <view class="proj-header">
            <text class="tag" :class="proj.typeClass">{{ proj.type }}</text>
            <text class="hot-fire">🔥 {{ proj.hot }}</text>
          </view>
          <text class="proj-title">{{ proj.title }}</text>
          <text class="proj-desc">{{ proj.desc }}</text>
          <view class="proj-footer">
            <view class="partners">
              <text class="p-label">合作方：</text>
              <text class="p-val">{{ proj.partner }}</text>
            </view>
            <button class="join-btn" @click="joinProject(proj)">立即报名</button>
          </view>
        </view>
      </view>
    </view>
    
    <view class="footer-space"></view>
  </view>
</template>

<script setup>
import { reactive, computed } from 'vue';

// --- 响应式数据源 ---

const platforms = reactive([
  { 
    name: '猪八戒网', 
    desc: '免押金入驻，同步等级', 
    iconText: '猪', 
    styleClass: 'bg-orange', 
    isBound: false, 
    score: 0,       // 初始分数
    potentialScore: 350, // 绑定后可获得的分数
    loading: false 
  },
  { 
    name: '站酷 ZCOOL', 
    desc: '参与设计大赛，IP共创', 
    iconText: '酷', 
    styleClass: 'bg-yellow', 
    isBound: true,  // 默认已绑定一个
    score: 780, 
    potentialScore: 780,
    loading: false
  },
  { 
    name: '一品威客', 
    desc: '优先派单权益', 
    iconText: '威', 
    styleClass: 'bg-blue', 
    isBound: false, 
    score: 0,
    potentialScore: 420,
    loading: false
  }
]);

const communities = reactive([
  { name: '大理 NCC 社区', location: '云南·大理', price: 120, bgClass: 'bg-dali' },
  { name: 'DNA 数字游民公社', location: '浙江·安吉', price: 150, bgClass: 'bg-anji' },
  { name: 'Seada 海岛社区', location: '海南·万宁', price: 180, bgClass: 'bg-wanning' }
]);

const projects = reactive([
  { 
    title: '“运河印象”非遗文创设计大赛', 
    desc: '设计阿胶包装或运河周边，入选作品将投入生产并获得销售分成。',
    type: '设计竞赛',
    typeClass: 'tag-purple',
    partner: '站酷 x 东阿阿胶',
    hot: '2.3w围观'
  },
  { 
    title: '聊城文旅短视频星探计划', 
    desc: '拍摄聊城古城Vlog，优质内容将获得流量扶持及现金奖励。',
    type: '内容创作',
    typeClass: 'tag-red',
    partner: '抖音 x 文旅局',
    hot: '1.5w参与'
  }
]);

// --- 响应式计算属性 (Computed) ---
// 这里的变量会随着 platforms 和 projects 的变化自动更新，无需手动维护

// 1. 计算已连接的平台数量
const connectedCount = computed(() => {
  return platforms.filter(p => p.isBound).length;
});

// 2. 计算总生态信用分 (累加所有已绑定平台的分数)
const totalCreditScore = computed(() => {
  return platforms.reduce((sum, p) => sum + p.score, 0);
});

// 3. 计算项目总数
const projectCount = computed(() => {
  return projects.length;
});


// --- 方法逻辑 ---

const handleBind = (item) => {
  // 如果已绑定，进入管理模式（这里简化为解绑演示）
  if (item.isBound) {
    uni.showActionSheet({
      itemList: ['更新数据', '解除绑定'],
      success: (res) => {
        if(res.tapIndex === 1) {
          // 响应式更新：直接修改数据，界面头部数字会自动减少
          item.isBound = false;
          item.score = 0; 
          uni.showToast({ title: '已解绑', icon: 'none' });
        }
      }
    });
    return;
  }

  // 开始绑定流程
  item.loading = true; // 触发 UI loading 状态
  
  setTimeout(() => {
    item.loading = false;
    item.isBound = true;
    // 响应式更新：赋予分数，界面头部总分会自动增加
    item.score = item.potentialScore; 
    
    uni.showToast({ title: '绑定成功，信用分已同步', icon: 'success' });
  }, 1200);
};

const checkCommunity = (comm) => {
  uni.showModal({
    title: '权益核验',
    content: `系统检测到您是聊城认证数字游民，预订 ${comm.name} 可享 8 折优惠。`,
    confirmText: '去预订',
    cancelText: '关闭'
  });
};

const joinProject = (proj) => {
  uni.showToast({ title: '报名通道即将开启', icon: 'none' });
};
</script>

<style lang="scss" scoped>
/* 样式与之前保持一致，微调了部分细节以适应新逻辑 */
.page-container {
  min-height: 100vh;
  background-color: #F4F6F9;
  padding-bottom: 40rpx;
}

.alliance-header {
  background: linear-gradient(135deg, #0052D4, #4364F7, #6FB1FC);
  padding: 40rpx 40rpx 80rpx 40rpx;
  position: relative;
  border-bottom-left-radius: 40rpx;
  border-bottom-right-radius: 40rpx;
  color: #fff;
  overflow: hidden;
  transition: all 0.3s; /* 添加过渡效果 */

  .title { font-size: 40rpx; font-weight: bold; display: block; margin-bottom: 10rpx; }
  .subtitle { font-size: 24rpx; opacity: 0.8; display: block; margin-bottom: 40rpx; }

  .stats-row {
    display: flex; justify-content: space-between;
    .stat-item {
      text-align: center;
      .num { font-size: 44rpx; font-weight: bold; font-family: 'DIN'; display: block; transition: all 0.3s; }
      .label { font-size: 22rpx; opacity: 0.7; margin-top: 6rpx; display: block; }
    }
  }

  .bg-shape {
    position: absolute; right: -40rpx; top: -20rpx; width: 300rpx; height: 300rpx; opacity: 0.1;
    transform: rotate(20deg);
  }
}

.section-box {
  margin: 30rpx 30rpx 0 30rpx;
  &:first-of-type { margin-top: -50rpx; position: relative; z-index: 10; }
  .section-title {
    display: flex; align-items: baseline; margin-bottom: 20rpx;
    .t-text { font-size: 32rpx; font-weight: bold; color: #333; margin-right: 16rpx; }
    .t-tip { font-size: 22rpx; color: #999; }
  }
}

.platform-list {
  background: #fff; border-radius: 24rpx; padding: 10rpx 30rpx;
  box-shadow: 0 4rpx 20rpx rgba(0,0,0,0.04);
}
.platform-card {
  display: flex; justify-content: space-between; align-items: center;
  padding: 30rpx 0;
  border-bottom: 1px solid #f5f5f5;
  &:last-child { border-bottom: none; }

  .card-left {
    display: flex; align-items: center;
    .p-icon {
      width: 80rpx; height: 80rpx; border-radius: 16rpx; display: flex; align-items: center; justify-content: center;
      color: #fff; font-size: 32rpx; font-weight: bold; margin-right: 20rpx;
      &.bg-orange { background: linear-gradient(to bottom right, #FF9966, #FF5E62); }
      &.bg-yellow { background: linear-gradient(to bottom right, #F6D365, #FDA085); }
      &.bg-blue { background: linear-gradient(to bottom right, #2193b0, #6dd5ed); }
    }
    .p-info {
      .p-name { font-size: 30rpx; font-weight: bold; color: #333; display: block; }
      .p-desc { font-size: 22rpx; color: #999; margin-top: 6rpx; }
      .p-status-row {
        margin-top: 6rpx;
        .status-tag { font-size: 20rpx; background: #E8F5E9; color: #4CAF50; padding: 2rpx 8rpx; border-radius: 6rpx; margin-right: 10rpx; }
        .sync-info { font-size: 20rpx; color: #007AFF; font-weight: bold;}
      }
    }
  }

  .action-btn {
    margin: 0; font-size: 24rpx; background: #007AFF; color: #fff;
    padding: 0 24rpx; height: 56rpx; line-height: 56rpx; border-radius: 28rpx;
    &.bound { background: #f0f2f5; color: #999; }
    &[disabled] { opacity: 0.6; } /* loading状态样式 */
  }
}

.community-scroll { white-space: nowrap; width: 100%; }
.comm-card {
  display: inline-block; width: 300rpx; background: #fff; border-radius: 20rpx; overflow: hidden;
  margin-right: 20rpx; box-shadow: 0 4rpx 12rpx rgba(0,0,0,0.05); vertical-align: top;
  .img-box {
    height: 180rpx; position: relative; background-color: #eee;
    &.bg-dali { background: linear-gradient(to bottom, #a1c4fd, #c2e9fb); }
    &.bg-anji { background: linear-gradient(to bottom, #d4fc79, #96e6a1); }
    &.bg-wanning { background: linear-gradient(to bottom, #89f7fe, #66a6ff); }
    .loc-badge {
      position: absolute; bottom: 10rpx; left: 10rpx;
      background: rgba(0,0,0,0.5); color: #fff; font-size: 20rpx; padding: 4rpx 10rpx; border-radius: 8rpx;
    }
  }
  .comm-info {
    padding: 20rpx;
    .c-name { font-size: 28rpx; font-weight: bold; white-space: normal; display: -webkit-box; -webkit-line-clamp: 1; overflow: hidden; }
    .c-tags { margin: 10rpx 0; }
    .tag-discount { font-size: 20rpx; color: #FF5E62; border: 1px solid #FF5E62; padding: 2rpx 6rpx; border-radius: 6rpx; }
    .c-action {
      display: flex; justify-content: space-between; align-items: center;
      .price { color: #333; font-weight: bold; font-size: 30rpx; .unit { font-size: 20rpx; font-weight: normal; } }
      .btn-text { color: #007AFF; font-size: 22rpx; }
    }
  }
}

.project-list { display: flex; flex-direction: column; gap: 20rpx; }
.proj-item {
  background: #fff; padding: 30rpx; border-radius: 20rpx; box-shadow: 0 2rpx 10rpx rgba(0,0,0,0.03);
  .proj-header {
    display: flex; justify-content: space-between; margin-bottom: 16rpx;
    .tag { font-size: 20rpx; padding: 4rpx 12rpx; border-radius: 8rpx; }
    .tag-purple { background: #F3E5F5; color: #9C27B0; }
    .tag-red { background: #FFEBEE; color: #F44336; }
    .hot-fire { font-size: 22rpx; color: #FF9800; }
  }
  .proj-title { font-size: 30rpx; font-weight: bold; color: #333; margin-bottom: 10rpx; display: block; }
  .proj-desc { font-size: 24rpx; color: #666; line-height: 1.5; display: block; margin-bottom: 20rpx; }
  .proj-footer {
    display: flex; justify-content: space-between; align-items: center; border-top: 1px solid #f5f5f5; padding-top: 20rpx;
    .partners {
      .p-label { font-size: 22rpx; color: #999; }
      .p-val { font-size: 22rpx; color: #333; font-weight: bold; }
    }
    .join-btn { margin: 0; background: #333; color: #fff; font-size: 24rpx; height: 50rpx; line-height: 50rpx; padding: 0 30rpx; border-radius: 25rpx; }
  }
}

.footer-space { height: 40rpx; }
</style>
