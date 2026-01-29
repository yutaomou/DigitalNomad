<template>
	<view class="stats">
		<view class="board">
			<view class="item"><text class="v">{{ list.length }}</text><text class="l">总核销(人)</text></view>
			<view class="item"><text class="v">￥{{ revenue }}</text><text class="l">总营收(元)</text></view>
		</view>
		<view class="list">
			<view class="title">核销记录流水</view>
			<view class="row" v-for="(o, i) in list" :key="i">
				<view>
					<text class="id">游民 #{{ o._id.slice(-4) }}</text>
					<text class="time">2026-01-29</text>
				</view>
				<text class="price">￥{{ o.final_amount }}</text>
			</view>
		</view>
	</view>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue';
const list = ref([]);
const revenue = computed(() => list.value.reduce((s, i) => s + i.final_amount, 0).toFixed(2));

onMounted(async () => {
	const db = uniCloud.database();
	const user = uni.getStorageSync('current_user_session');
	const res = await db.collection('orders').where({ merchant_id: user._id }).get();
	list.value = res.result.data;
});
</script>

<style lang="scss">
.stats { padding: 30rpx; background: #f8fafc; min-height: 100vh; }
.board { background: #1e293b; color: #fff; padding: 50rpx; border-radius: 30rpx; display: flex; 
	.item { flex: 1; text-align: center; .v { font-size: 40rpx; font-weight: bold; display: block; } .l { font-size: 22rpx; opacity: 0.6; } }
}
.list { margin-top: 40rpx; .title { font-weight: bold; margin-bottom: 20rpx; }
	.row { background: #fff; padding: 25rpx; border-radius: 15rpx; margin-bottom: 15rpx; display: flex; justify-content: space-between; align-items: center;
		.id { font-size: 26rpx; display: block; } .time { font-size: 20rpx; color: #999; } .price { color: #10b981; font-weight: bold; }
	}
}
</style>