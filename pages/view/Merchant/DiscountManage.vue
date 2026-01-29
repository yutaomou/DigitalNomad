<template>
	<view class="container">
		<view class="card">
			<view class="rule-row" v-for="(r, i) in rules" :key="i">
				<text>满</text>
				<input type="number" v-model="r.days" />
				<text>天，享受</text>
				<input type="digit" v-model="r.off" />
				<text>折</text>
				<text class="del" @tap="rules.splice(i,1)">×</text>
			</view>
			<button class="add" @tap="rules.push({days:0,off:10})">+ 添加规则</button>
			<button class="save" @tap="save">保存规则</button>
		</view>
	</view>
</template>

<script setup>
import { ref } from 'vue';
const rules = ref([{ days: 1, off: 9.5 }, { days: 30, off: 8.5 }]);
const save = () => uni.showToast({ title: '保存成功' });
</script>

<style lang="scss">
.container { padding: 30rpx; }
.card { background: #fff; padding: 30rpx; border-radius: 20rpx; box-shadow: 0 4rpx 20rpx rgba(0,0,0,0.05); }
.rule-row { display: flex; align-items: center; gap: 15rpx; margin-bottom: 30rpx;
	input { border: 1rpx solid #eee; width: 80rpx; text-align: center; height: 60rpx; border-radius: 10rpx; }
	.del { color: #ff4d4f; margin-left: auto; font-size: 40rpx; }
}
.add { background: #f0f7ff; color: #3b82f6; font-size: 26rpx; margin-bottom: 20rpx; }
.save { background: #3b82f6; color: #fff; border-radius: 50rpx; }
</style>