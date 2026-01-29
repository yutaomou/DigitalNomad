<template>
  <view class="page-container">
    <view class="header-section">
      <view><text class="title">寻宝活动管理</text></view>
      <button class="add-btn" @click="openModal">+ 发布任务</button>
    </view>
    
    <scroll-view scroll-y class="task-list">
      <view class="task-card" v-for="item in tasks" :key="item._id">
        <view class="card-left">
          <text class="t-title">{{ item.title }}</text>
          <view class="t-meta">
            <text class="coord">📍 {{ item.address }}</text>
            <text class="reward">🏆 {{ item.reward }}</text>
          </view>
        </view>
        <view class="card-right" @tap="deleteTask(item._id)">
          <text class="btn-del">删除</text>
        </view>
      </view>
      <view v-if="tasks.length === 0" class="empty">暂无活动，请点击右上角发布</view>
    </scroll-view>

    <view class="modal-mask" v-if="showModal">
      <view class="modal-content">
        <view class="m-header">发布新寻宝任务</view>
        <input class="input" v-model="form.title" placeholder="任务名称 (如: 寻找光岳楼)" />
        <view class="loc-box" @tap="chooseLocation">{{ form.address || '点击选择地图坐标' }}</view>
        <input class="input" v-model="form.reward" placeholder="奖励 (如: 200能量值)" />
        <view class="modal-footer">
          <button @tap="showModal = false">取消</button>
          <button class="save" @tap="submitTask">立即发布</button>
        </view>
      </view>
    </view>
  </view>
</template>

<script setup>
import { ref, reactive } from 'vue';
import { onShow } from '@dcloudio/uni-app';
const db = uniCloud.database();
const tasks = ref([]);
const showModal = ref(false);
const form = reactive({ title: '', address: '', latitude: 0, longitude: 0, reward: '' });

onShow(() => { loadTasks(); });

const loadTasks = async () => {
  const res = await db.collection('activities').orderBy('create_time', 'desc').get();
  tasks.value = res.result.data;
};

const chooseLocation = () => {
  uni.chooseLocation({
    success: (res) => {
      form.address = res.name || res.address;
      form.latitude = res.latitude;
      form.longitude = res.longitude;
    }
  });
};

const submitTask = async () => {
  if(!form.title || !form.address) return uni.showToast({title:'请填写完整', icon:'none'});
  uni.showLoading();
  await db.collection('activities').add({ ...form, create_time: Date.now(), active: true });
  uni.hideLoading();
  showModal.value = false;
  loadTasks();
};

const deleteTask = async (id) => {
  uni.showModal({
    title: '删除任务',
    content: '确定要移除此活动吗？',
    success: async (res) => {
      if(res.confirm) {
        await db.collection('activities').doc(id).remove();
        loadTasks();
      }
    }
  });
};

const openModal = () => {
  form.title = ''; form.address = ''; form.reward = '';
  showModal.value = true;
};
</script>

<style lang="scss" scoped>
.page-container { padding: 30rpx; background: #f4f6f9; min-height: 100vh; }
.header-section { display: flex; justify-content: space-between; align-items: center; margin-bottom: 30rpx;
  .title { font-size: 34rpx; font-weight: bold; }
  .add-btn { background: #007AFF; color: #fff; font-size: 26rpx; padding: 0 30rpx; height: 64rpx; line-height: 64rpx; border-radius: 32rpx; margin: 0; }
}
.task-card { background: #fff; border-radius: 20rpx; padding: 30rpx; margin-bottom: 20rpx; display: flex; justify-content: space-between;
  .t-title { font-weight: bold; font-size: 30rpx; margin-bottom: 10rpx; display: block; }
  .t-meta { display: flex; gap: 20rpx; font-size: 22rpx; .coord { color: #007AFF; } .reward { color: #faad14; } }
  .btn-del { color: #ff3b30; font-size: 24rpx; align-self: center; border: 1rpx solid #ff3b30; padding: 4rpx 16rpx; border-radius: 10rpx; }
}
.modal-mask { position: fixed; top: 0; left: 0; right: 0; bottom: 0; background: rgba(0,0,0,0.6); display: flex; align-items: center; justify-content: center; z-index: 99;
  .modal-content { width: 85%; background: #fff; border-radius: 20rpx; padding: 40rpx;
    .m-header { font-weight: bold; margin-bottom: 30rpx; text-align: center; }
    .input, .loc-box { background: #f5f5f5; padding: 20rpx; border-radius: 10rpx; margin-bottom: 20rpx; font-size: 28rpx; }
    .loc-box { color: #007AFF; text-align: center; border: 1rpx dashed #007AFF; }
    .modal-footer { display: flex; gap: 20rpx; margin-top: 30rpx; button { flex: 1; font-size: 28rpx; } .save { background: #007AFF; color: #fff; } }
  }
}
.empty { text-align: center; color: #999; margin-top: 100rpx; font-size: 26rpx; }
</style>