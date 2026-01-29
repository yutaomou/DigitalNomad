<template>
  <view class="page-container">
    <view class="tabs">
      <view class="tab-item" :class="{ active: currentTab === 0 }" @click="currentTab = 0">游民认证</view>
      <view class="tab-item" :class="{ active: currentTab === 1 }" @click="currentTab = 1">商户入驻 ({{ shopList.length }})</view>
    </view>

    <scroll-view scroll-y class="list-box">
      <view v-if="currentTab === 1">
        <view class="audit-card" v-for="(item, i) in shopList" :key="item._id">
          <view class="header">
            <text class="title">{{ item.shop_name }}</text>
            <text class="tag orange">{{ item.category }}</text>
          </view>
          <view class="info-row">
            <text class="label">申请人手机：</text><text class="val">{{ item.phone || '未记录' }}</text>
          </view>
          <view class="info-row">
            <text class="label">资质凭证：</text>
            <text class="link" @tap="previewImg(item.license_url)">点击预览营业执照</text>
          </view>
          <view class="btn-row">
            <button class="btn reject" @tap="doAudit(item._id, 2)">驳回</button>
            <button class="btn pass" @tap="doAudit(item._id, 1)">批准入驻</button>
          </view>
        </view>
        <view v-if="shopList.length === 0" class="empty">暂无待处理申请</view>
      </view>
      <view v-else class="empty">游民自动认证系统运行中</view>
    </scroll-view>
  </view>
</template>

<script setup>
import { ref } from 'vue';
import { onShow } from '@dcloudio/uni-app'; // 必须引入 onShow
const db = uniCloud.database();
const currentTab = ref(1);
const shopList = ref([]);

// 重点：使用 onShow 替代 onMounted，保证页面每次显示都刷新数据
onShow(() => {
  loadAuditData();
});

const loadAuditData = async () => {
  console.log("开始请求数据库...");
  try {
    const res = await db.collection('merchants')
      .where({ status: 0 }) // 如果数据库里存的是字符串，这里改成 '0' 试试
      .get();
    
    console.log("数据库返回结果：", res);
    
    if (res.result.data) {
      shopList.value = res.result.data;
      console.log("成功获取到商户数量：", shopList.value.length);
    }
  } catch (e) {
    console.error("数据库查询报错：", e);
    uni.showModal({
      title: '查询失败',
      content: e.message
    });
  }
};

const doAudit = async (id, targetStatus) => {
  const title = targetStatus === 1 ? '确认批准该商户入驻？' : '确认驳回该申请？';
  uni.showModal({
    title,
    success: async (res) => {
      if (res.confirm) {
        uni.showLoading({ title: '处理中' });
        await db.collection('merchants').doc(id).update({ status: targetStatus });
        uni.hideLoading();
        uni.showToast({ title: '处理完成' });
        loadAuditData();
      }
    }
  });
};

const previewImg = (url) => { if(url) uni.previewImage({ urls: [url] }); };
</script>

<style lang="scss" scoped>
/* 样式保持不变 */
.page-container { min-height: 100vh; background: #f5f5f5; }
.tabs { display: flex; background: #fff; padding: 20rpx 0; position: sticky; top: 0; z-index: 10;
  .tab-item { flex: 1; text-align: center; font-size: 28rpx; color: #666; padding-bottom: 20rpx;
    &.active { color: #007AFF; font-weight: bold; border-bottom: 4rpx solid #007AFF; }
  }
}
.list-box { padding: 20rpx; }
.audit-card { background: #fff; border-radius: 16rpx; padding: 30rpx; margin-bottom: 20rpx;
  .header { display: flex; justify-content: space-between; margin-bottom: 20rpx;
    .title { font-size: 32rpx; font-weight: bold; }
    .tag { font-size: 22rpx; background: #fff7e6; color: #faad14; padding: 4rpx 12rpx; border-radius: 8rpx; }
  }
  .info-row { font-size: 26rpx; margin-bottom: 12rpx; display: flex;
    .label { color: #999; width: 160rpx; }
    .val { color: #333; }
    .link { color: #007AFF; text-decoration: underline; }
  }
  .btn-row { display: flex; gap: 20rpx; margin-top: 30rpx;
    .btn { flex: 1; height: 72rpx; line-height: 72rpx; font-size: 28rpx; border-radius: 36rpx; margin: 0;
      &.reject { background: #fff1f0; color: #f5222d; } &.pass { background: #007AFF; color: #fff; }
    }
  }
}
.empty { text-align: center; color: #999; padding-top: 100rpx; font-size: 26rpx; }
</style>