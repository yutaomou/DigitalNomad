<template>
	<view class="merchant-container">
		<view class="shop-card" :class="'status-bg-' + auditStatus">
			<view class="shop-main">
				<view class="avatar-box">店</view>
				<view class="shop-text">
					<text class="name">{{ shopName }}</text>
					<text class="role-tag" :class="statusClass">{{ statusText }}</text>
				</view>
			</view>
			
			<view v-if="auditStatus === 1" class="status-bar" @tap="mockVerify">
				<text>✨ 点击此处模拟一次核销 (产生测试订单数据)</text>
			</view>
			<view v-else class="status-bar disabled">
				<text>{{ auditStatus === 0 ? '⏳ 资料审核中，请稍后' : '⚠️ 请先完成入驻申请' }}</text>
			</view>
		</view>

		<view class="menu-grid">
			<view class="menu-item" @tap="navTo('MerchantApply')">
				<view class="icon-bg blue">📜</view>
				<view class="info">
					<text class="title">入驻管理</text>
					<text class="desc">查看资质及审核进度</text>
				</view>
			</view>

			<view class="menu-item" :class="{ 'lock-item': auditStatus !== 1 }" @tap="handleNav('DiscountManage')">
				<view class="icon-bg orange">💰</view>
				<view class="info">
					<text class="title">折扣权益</text>
					<text class="desc">{{ auditStatus === 1 ? '设置时长梯度折扣' : '认证后开启此功能' }}</text>
				</view>
			</view>

			<view class="menu-item" :class="{ 'lock-item': auditStatus !== 1 }" @tap="handleNav('EventCooperation')">
				<view class="icon-bg green">🏆</view>
				<view class="info">
					<text class="title">活动合作</text>
					<text class="desc">{{ auditStatus === 1 ? '寻宝点位点亮管理' : '认证后开启此功能' }}</text>
				</view>
			</view>
		</view>

		<view class="stats-preview" @tap="navTo('MerchantStats')">
			<view class="stats-header">
				<text>营收简报 (今日)</text>
				<text class="link">查看完整报表 ></text>
			</view>
			<view class="stats-body">
				<view class="stat-box">
					<text class="num">{{ todayCount }}</text>
					<text class="label">核销人数</text>
				</view>
				<view class="stat-box">
					<text class="num">￥{{ todayRevenue }}</text>
					<text class="label">预计营收</text>
				</view>
			</view>
		</view>
	</view>
</template>

<script setup>
import { ref, computed } from 'vue';
import { onShow } from '@dcloudio/uni-app';

const db = uniCloud.database();
const shopName = ref('加载中...');
const auditStatus = ref(-1); // -1:未入驻, 0:审核中, 1:已认证, 2:驳回
const todayCount = ref(0);
const todayRevenue = ref(0);
const internalMerchantId = ref(''); // 数据库分配的真正 _id

// 计算属性：映射状态文字
const statusText = computed(() => {
	const textMap = { 0: '审核中', 1: '认证商户', 2: '驳回请重试' };
	return textMap[auditStatus.value] || '未认证';
});

// 计算属性：映射标签样式
const statusClass = computed(() => {
	const classMap = { 0: 'tag-warning', 1: 'tag-success', 2: 'tag-danger' };
	return classMap[auditStatus.value] || 'tag-default';
});

// 每次进入页面刷新
onShow(() => {
	fetchMerchantInfo();
});

// 核心逻辑：从数据库获取最新商户状态
const fetchMerchantInfo = async () => {
	const user = uni.getStorageSync('current_user_session');
	if (!user || !user.phone) return;

	try {
		const res = await db.collection('merchants')
			.where({ phone: user.phone })
			.orderBy('create_time', 'desc')
			.get();

		if (res.result.data.length > 0) {
			const info = res.result.data[0];
			shopName.value = info.shop_name;
			auditStatus.value = info.status;
			internalMerchantId.value = info._id;
			
			// 如果已认证，加载今日订单统计
			if (info.status === 1) {
				loadTodayStats(info._id);
			}
		} else {
			shopName.value = '待入驻小店';
			auditStatus.value = -1;
		}
	} catch (e) {
		console.error('获取商户信息失败:', e);
	}
};

// 加载统计数据
const loadTodayStats = async (mId) => {
	try {
		const res = await db.collection('orders').where({ merchant_id: mId }).get();
		todayCount.value = res.result.data.length;
		// 累加金额
		const total = res.result.data.reduce((sum, item) => sum + (Number(item.final_amount) || 0), 0);
		todayRevenue.value = total.toFixed(2);
	} catch (e) {
		console.log('统计加载失败:', e);
	}
};

// 带有权限拦截的跳转
const handleNav = (path) => {
	if (auditStatus.value !== 1) {
		uni.showToast({ title: '商户认证通过后开启', icon: 'none' });
		return;
	}
	navTo(path);
};

const navTo = (page) => {
	uni.navigateTo({ url: `/pages/view/Merchant/${page}` });
};

// 模拟核销（生成订单数据）
const mockVerify = async () => {
	if (!internalMerchantId.value) return;
	
	uni.showLoading({ title: '正在核销...' });
	try {
		await db.collection('orders').add({
			merchant_id: internalMerchantId.value,
			nomad_id: "NOMAD_" + Math.floor(Math.random() * 10000),
			original_amount: 100,
			final_amount: 85, // 模拟打折后的金额
			create_time: Date.now()
		});
		uni.hideLoading();
		uni.showToast({ title: '核销成功' });
		loadTodayStats(internalMerchantId.value); // 刷新数据
	} catch (e) {
		uni.hideLoading();
		console.error(e);
	}
};
</script>

<style lang="scss" scoped>
.merchant-container { padding: 30rpx; background: #f8fafc; min-height: 100vh; }

/* 顶部状态卡片 */
.shop-card { 
	padding: 40rpx; 
	border-radius: 32rpx; 
	color: #ffffff; 
	transition: all 0.4s ease;
	background: linear-gradient(135deg, #64748b, #334155); // 默认灰色 (未入驻)

	&.status-bg-0 { background: linear-gradient(135deg, #f59e0b, #d97706); } // 警告黄 (审核中)
	&.status-bg-1 { background: linear-gradient(135deg, #3b82f6, #1d4ed8); } // 品牌蓝 (已通过)
	&.status-bg-2 { background: linear-gradient(135deg, #ef4444, #b91c1c); } // 危险红 (驳回)

	.shop-main { 
		display: flex; 
		align-items: center; 
		margin-bottom: 30rpx;
		.avatar-box { 
			width: 90rpx; height: 90rpx; background: rgba(255,255,255,0.25); 
			border-radius: 50%; display: flex; justify-content: center; align-items: center; 
			font-weight: bold; border: 2rpx solid #fff;
		}
		.shop-text { 
			margin-left: 24rpx; 
			.name { font-size: 36rpx; font-weight: bold; display: block; margin-bottom: 6rpx; }
		}
	}
}

.role-tag { font-size: 22rpx; padding: 4rpx 16rpx; border-radius: 100rpx; background: rgba(255,255,255,0.3); }
.tag-success { background: #10b981; }
.tag-warning { background: #fbbf24; color: #92400e; }
.tag-danger { background: #fee2e2; color: #991b1b; }

.status-bar { 
	font-size: 24rpx; background: rgba(0,0,0,0.15); 
	padding: 14rpx; border-radius: 16rpx; text-align: center;
	&.disabled { opacity: 0.6; font-style: italic; }
}

/* 菜单列表 */
.menu-grid { 
	margin-top: 40rpx;
	.menu-item { 
		background: #ffffff; padding: 32rpx; border-radius: 24rpx; 
		margin-bottom: 24rpx; display: flex; align-items: center;
		box-shadow: 0 4rpx 12rpx rgba(0,0,0,0.02);
		&.lock-item { opacity: 0.5; filter: grayscale(0.8); }
		
		.icon-bg { 
			width: 88rpx; height: 88rpx; border-radius: 20rpx; 
			display: flex; justify-content: center; align-items: center; font-size: 44rpx;
			&.blue { background: #eff6ff; } 
			&.orange { background: #fff7ed; } 
			&.green { background: #f0fdf4; }
		}
		.info { 
			margin-left: 24rpx; 
			.title { font-size: 30rpx; font-weight: bold; color: #1e293b; display: block; }
			.desc { font-size: 24rpx; color: #94a3b8; margin-top: 4rpx; }
		}
	}
}

/* 统计看板 */
.stats-preview { 
	background: #ffffff; border-radius: 24rpx; padding: 32rpx;
	box-shadow: 0 4rpx 12rpx rgba(0,0,0,0.02);
	.stats-header { 
		display: flex; justify-content: space-between; align-items: center;
		border-bottom: 1rpx solid #f1f5f9; padding-bottom: 24rpx; 
		font-weight: bold; color: #1e293b;
		.link { font-size: 24rpx; color: #3b82f6; font-weight: normal; }
	}
	.stats-body { 
		display: flex; justify-content: space-around; padding-top: 32rpx;
		.stat-box { 
			text-align: center; 
			.num { font-size: 40rpx; font-weight: bold; display: block; color: #0f172a; margin-bottom: 8rpx; }
			.label { font-size: 24rpx; color: #64748b; }
		}
	}
}
</style>