<template>
	<view class="container pb-24">
		<view class="header-bg">
			<view class="header-content">
				<view class="user-info">
					<view>
						<text class="greeting">Hi, {{ userInfo.phone || '游民朋友' }}</text>
						<view class="status-bar">
							<text class="status-tag">在聊 {{ daysInLiao }} 天</text>
							<text class="status-tag role-tag">{{ roleName }}</text>
						</view>
					</view>
					<view class="level-badge">LV.{{ userLevel }}</view>
				</view>
			</view>
			<view class="card-wrapper">
				<view class="id-card">
					<text class="card-title">聊城数字通行证</text>
					<text class="card-no">NO. {{ userInfo.phone ? 'LC' + userInfo.phone.slice(-4) : '******' }}</text>
					<view class="card-footer">
						<text>身份确认：已认证</text>
						<text class="verify-icon">✓</text>
					</view>
				</view>
			</view>
		</view>

		<view class="grid-box">
			<view class="grid-item" @tap="navigateTo('/pages/view/MapView')">
				<view class="icon-wrap bg-blue"><text class="iconfont">📍</text></view>
				<text class="grid-text">社区地图</text>
			</view>
			<view class="grid-item" @tap="navigateTo('/pages/view/TreasureView')">
				<view class="icon-wrap bg-orange"><text class="iconfont">🛏️</text></view>
				<text class="grid-text">寻找宝藏</text>
			</view>
			<view class="grid-item" @tap="navigateTo('/pages/view/AllianceView')">
				<view class="icon-wrap bg-purple"><text class="iconfont">📑</text></view>
				<text class="grid-text">生态联盟</text>
			</view>
			<view class="grid-item" @tap="navigateTo('/pages/view/AllianceView')">
				<view class="icon-wrap bg-green"><text class="iconfont">🌐</text></view>
				<text class="grid-text">加入联盟</text>
			</view>
		</view>

		<view v-if="userInfo.role === 'nomad'" class="task-section">
			<view class="section-title">
				<text class="bolt-icon">⚡</text>
				<text class="title-text">生态任务池</text>
			</view>
			
			<view v-for="task in mockTasks" :key="task.id" class="task-card">
				<view class="platform-tag">{{ task.platform }}</view>
				<view class="task-header">
					<text class="task-name">{{ task.title }}</text>
					<text class="task-price">¥{{ task.price }}</text>
				</view>
				<view class="task-footer">
					<view class="tags">
						<text v-for="tag in task.tags" :key="tag" class="tag-item">{{ tag }}</text>
					</view>
					<button class="apply-btn" @tap="handleApply(task.platform)">一键接单</button>
				</view>
			</view>
		</view>
		
		<view class="logout-box">
			<button class="logout-btn" @tap="handleLogout">退出登录</button>
		</view>
	</view>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue';

const userInfo = ref({});
const daysInLiao = ref(7);
const userLevel = ref('黄金会员');

const mockTasks = [
	{ id: 1, title: '聊城古城区民宿宣传画册设计', price: '1200', platform: '站酷', tags: ['设计', '远程'] },
	{ id: 2, title: '数字游民社区小程序功能开发', price: '5000', platform: '猪八戒网', tags: ['开发', '后端'] }
];

const roleName = computed(() => {
	const map = { nomad: '数字游民', merchant: '合作商户', gov: '管理方' };
	return map[userInfo.role] || '游客';
});

onMounted(() => {
	const savedUser = uni.getStorageSync('current_user_session');
	if (!savedUser) {
		uni.reLaunch({ url: '/pages/login/login' });
	} else {
		userInfo.value = savedUser;
	}
});

const navigateTo = (path) => {
	uni.navigateTo({ url: path });
};

const handleApply = (platform) => {
	uni.showModal({
		title: '接单确认',
		content: `申请已提交至${platform}，请等待审核。`,
		showCancel: false
	});
};

const handleLogout = () => {
	uni.removeStorageSync('current_user_session');
	uni.reLaunch({ url: '/pages/login/login' });
};
</script>

<style lang="scss">
/* 保持你提供的原有样式... */
.container { min-height: 100vh; background-color: #f7f8fa; }
.header-bg { background: linear-gradient(to bottom right, #4f46e5, #3b82f6); padding: 80rpx 40rpx 140rpx; border-bottom-left-radius: 80rpx; border-bottom-right-radius: 80rpx; position: relative; }
.user-info { display: flex; justify-content: space-between; color: #fff; }
.greeting { font-size: 40rpx; font-weight: bold; }
.status-bar { display: flex; gap: 15rpx; margin-top: 10rpx; }
.status-tag { background: rgba(255, 255, 255, 0.2); font-size: 22rpx; padding: 4rpx 16rpx; border-radius: 8rpx; }
.level-badge { background-color: #fbbf24; color: #312e81; font-size: 22rpx; font-weight: bold; padding: 6rpx 20rpx; border-radius: 100rpx; }
.card-wrapper { position: absolute; bottom: -40rpx; left: 40rpx; right: 40rpx; }
.id-card { background: #1e293b; height: 180rpx; border-radius: 24rpx; padding: 30rpx; color: #fff; display: flex; flex-direction: column; justify-content: space-between;
	.card-title { font-size: 24rpx; opacity: 0.6; }
	.card-no { font-size: 32rpx; letter-spacing: 4rpx; }
	.card-footer { display: flex; justify-content: space-between; font-size: 22rpx; color: #fbbf24; }
}
.grid-box { display: flex; justify-content: space-around; padding: 80rpx 20rpx 40rpx;
	.grid-item { display: flex; flex-direction: column; align-items: center; gap: 12rpx; &.disabled { opacity: 0.4; } }
	.icon-wrap { width: 100rpx; height: 100rpx; border-radius: 30rpx; display: flex; align-items: center; justify-content: center;
		.iconfont { font-size: 40rpx; }
	}
	.bg-blue { background-color: #3b82f6; color: #fff; }
	.bg-orange { background-color: #fb923c; color: #fff; }
	.bg-purple { background-color: #a855f7; color: #fff; }
	.bg-green { background-color: #22c55e; color: #fff; }
	.grid-text { font-size: 24rpx; color: #4b5563; font-weight: 500; }
}
.task-section { padding: 0 40rpx;
	.section-title { display: flex; align-items: center; gap: 10rpx; margin-bottom: 30rpx; .bolt-icon { color: #fbbf24; } .title-text { font-weight: bold; font-size: 32rpx; } }
}
.task-card { background-color: #fff; border-radius: 24rpx; padding: 30rpx; margin-bottom: 24rpx; position: relative;
	.platform-tag { position: absolute; right: 0; top: 0; background-color: #f3f4f6; font-size: 20rpx; color: #9ca3af; padding: 4rpx 16rpx; border-bottom-left-radius: 20rpx; }
	.task-header { display: flex; justify-content: space-between; margin-bottom: 20rpx; .task-name { font-size: 28rpx; font-weight: bold; } .task-price { color: #ef4444; font-weight: bold; } }
	.task-footer { display: flex; justify-content: space-between; align-items: center;
		.tag-item { background-color: #eff6ff; color: #3b82f6; font-size: 20rpx; padding: 2rpx 12rpx; border-radius: 6rpx; margin-right: 10rpx; }
		.apply-btn { margin: 0; background-color: #111827; color: #fff; font-size: 24rpx; padding: 0 24rpx; height: 54rpx; line-height: 54rpx; border-radius: 100rpx; }
	}
}
.logout-box { padding: 40rpx; .logout-btn { background: #fff; color: #ff4d4f; border: 1px solid #ff4d4f; font-size: 28rpx; border-radius: 20rpx; } }
</style>
