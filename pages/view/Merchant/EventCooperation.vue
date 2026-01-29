<template>
	<view class="container">
		<view class="box">
			<view class="line">
				<text class="t">地图寻宝点位</text>
				<switch color="#3b82f6" @change="e => active = e.detail.value" />
			</view>
			<view v-if="active" class="gift">
				<text class="l">打卡礼品描述</text>
				<input v-model="gift" placeholder="如：免费特调一小杯" />
			</view>
		</view>
		<view class="box mt">
			<view class="line">
				<text class="t">社群活动承接</text>
				<switch color="#10b981" />
			</view>
			<text class="d">开启后将获得“金牌合作商”标识，优先推送团建需求</text>
		</view>
		<button class="btn" @tap="save">保存设置</button>
	</view>
</template>

<script setup>
import { ref } from 'vue';
const active = ref(false);
const gift = ref('');
const save = () => uni.showToast({ title: '已同步至地图' });
</script>

<style lang="scss">
.container { padding: 30rpx; background: #f8fafc; min-height: 100vh; }
.box { background: #fff; padding: 30rpx; border-radius: 20rpx;
	.line { display: flex; justify-content: space-between; align-items: center; .t { font-weight: bold; font-size: 32rpx; } }
	.gift { margin-top: 30rpx; .l { font-size: 24rpx; color: #666; } input { border-bottom: 1rpx solid #eee; padding: 15rpx 0; } }
	.d { font-size: 22rpx; color: #999; margin-top: 10rpx; display: block; }
}
.mt { margin-top: 30rpx; }
.btn { background: #3b82f6; color: #fff; margin-top: 50rpx; border-radius: 50rpx; }
</style>