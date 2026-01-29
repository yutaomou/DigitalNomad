<template>
	<view class="map-wrapper">
		<view class="filter-tabs">
			<view 
				v-for="tab in tabs" :key="tab.id"
				:class="['tab-item', activeTab === tab.id ? 'active' : '']"
				@tap="switchTab(tab.id)"
			>
				{{ tab.name }}
			</view>
		</view>

		<map 
			id="liaochengMap"
			class="main-map" 
			:latitude="centerPoint.latitude" 
			:longitude="centerPoint.longitude" 
			:markers="filteredMarkers"
			:polyline="activeTab === 'treasure' ? routes : []" 
			:scale="13"
			@markertap="onMarkerTap"
			show-location
		></map>

		<view class="bottom-card" v-if="selectedLoc">
			<view class="card-content">
				<view class="title-row">
					<text class="loc-name">{{ selectedLoc.title }}</text>
					<text class="category-tag">{{ activeTabName }}</text>
				</view>
				<view class="detail-info">
					<view v-if="selectedLoc.category === 'community'" class="info-column">
						<text class="info-desc">{{ selectedLoc.extra.desc || '数字游民基地' }}</text>
					</view>
					<view v-else-if="selectedLoc.category === 'culture'" class="info-text">
						🏛️ {{ selectedLoc.extra.history || '历史文化遗迹' }}
					</view>
					<view v-else class="info-text">💰 专属特惠点位</view>
				</view>
			</view>
			<view class="action-btns">
				<button class="nav-btn" @tap="startNav">导航</button>
			</view>
		</view>

		<view class="debug-tip">模式：{{ activeTabName }} | 已加载点位：{{ allMarkers.length }}</view>
	</view>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue';

// --- 配置区 ---
const AMAP_KEY = '9a14fea4802bff76c50515137cdbab90'; 
const activeTab = ref('community'); 
const selectedLoc = ref(null);
const allMarkers = ref([]);

const tabs = [
	{ id: 'community', name: '游民社区' },
	{ id: 'culture', name: '两河文旅' },
	{ id: 'shop', name: '特惠商户' },
	{ id: 'treasure', name: '寻宝路线' }
];

const centerPoint = reactive({ 
	latitude: 36.4418, 
	longitude: 115.9705 
});

// 寻宝路线容器
const routes = reactive([{
	id: 99,
	points: [], 
	color: "#3b82f6",
	width: 6,
	arrowLine: true,
	borderWidth: 2,
	borderColor: "#ffffff"
}]);

// --- 逻辑区 ---

// 1. 从云数据库获取点位
const fetchLocations = async () => {
	uni.showLoading({ title: '同步数据...' });
	try {
		const res = await uniCloud.callFunction({ name: 'get-locations' });
		if (res.result && res.result.code === 200) {
			allMarkers.value = res.result.data.map((item, index) => ({
				...item,
				id: index,
				latitude: parseFloat(item.latitude),
				longitude: parseFloat(item.longitude),
				iconPath: '/static/location.png',
				width: 30,
				height: 30,
				callout: { content: item.title, display: 'ALWAYS', padding: 5, borderRadius: 5 }
			}));
		}
	} catch (e) {
		console.error(e);
	} finally {
		uni.hideLoading();
	}
};

// 2. 调用高德API进行路径规划（避开河流走大桥）
const planRealRoute = () => {
	// 阿尔卡迪亚坐标
	const origin = "115.955458,36.430792"; 
	// 新东方广场坐标
	const destination = "115.985632,36.452815"; 
	
	uni.request({
		url: `https://restapi.amap.com/v3/direction/walking?origin=${origin}&destination=${destination}&key=${AMAP_KEY}`,
		success: (res) => {
			if (res.data.status === '1' && res.data.route.paths.length > 0) {
				const steps = res.data.route.paths[0].steps;
				let pathPoints = [];
				
				// 高德返回的数据是字符串格式，我们需要解析它
				steps.forEach(step => {
					const lineArr = step.polyline.split(';');
					lineArr.forEach(point => {
						const lnglat = point.split(',');
						pathPoints.push({
							longitude: parseFloat(lnglat[0]),
							latitude: parseFloat(lnglat[1])
						});
					});
				});
				
				// 将解析后的“贴地”点赋值给路线
				routes[0].points = pathPoints;
				console.log('路径规划成功，共计点数：', pathPoints.length);
			} else {
				uni.showToast({ title: '路径规划失败，请检查Key', icon: 'none' });
			}
		},
		fail: () => {
			uni.showToast({ title: '网络请求失败', icon: 'none' });
		}
	});
};

// 3. 标签切换逻辑
const switchTab = (id) => {
	activeTab.value = id;
	selectedLoc.value = null;
	if (id === 'treasure') {
		planRealRoute(); // 仅在切换到寻宝模式时请求API
	}
};

const filteredMarkers = computed(() => {
	if (activeTab.value === 'treasure') return allMarkers.value;
	return allMarkers.value.filter(m => m.category === activeTab.value);
});

const activeTabName = computed(() => tabs.find(t => t.id === activeTab.value)?.name);

const onMarkerTap = (e) => {
	const marker = allMarkers.value.find(m => m.id === e.detail.markerId);
	if (marker) selectedLoc.value = marker;
};

const startNav = () => {
	if (!selectedLoc.value) return;
	uni.openLocation({
		latitude: selectedLoc.value.latitude,
		longitude: selectedLoc.value.longitude,
		name: selectedLoc.value.title
	});
};

onMounted(fetchLocations);
</script>

<style lang="scss">
/* 样式与之前保持一致 */
.map-wrapper { width: 100vw; height: 100vh; position: relative; }
.main-map { width: 100%; height: 100%; }
.filter-tabs {
	position: absolute; top: 80rpx; left: 20rpx; right: 20rpx; z-index: 10;
	display: flex; background: #fff; padding: 10rpx; border-radius: 60rpx;
	box-shadow: 0 4rpx 20rpx rgba(0,0,0,0.1);
	.tab-item {
		flex: 1; text-align: center; padding: 20rpx 0; font-size: 26rpx; color: #666;
		&.active { background: #3b82f6; color: #fff; border-radius: 50rpx; font-weight: bold; }
	}
}
.bottom-card {
	position: absolute; bottom: 60rpx; left: 25rpx; right: 25rpx;
	background: #fff; border-radius: 40rpx; padding: 35rpx;
	display: flex; justify-content: space-between; align-items: center;
	box-shadow: 0 -10rpx 40rpx rgba(0,0,0,0.1); z-index: 10;
	.card-content {
		flex: 1; margin-right: 20rpx;
		.title-row {
			display: flex; align-items: center; margin-bottom: 10rpx;
			.loc-name { font-size: 34rpx; font-weight: bold; }
			.category-tag { font-size: 20rpx; background: #eef2ff; color: #3b82f6; padding: 4rpx 10rpx; margin-left: 15rpx; border-radius: 8rpx; }
		}
		.info-desc { font-size: 24rpx; color: #888; }
		.highlight { color: #ff4d4f; font-weight: bold; }
	}
	.nav-btn {
		background: #3b82f6; color: #fff; font-size: 26rpx; border-radius: 40rpx;
		height: 80rpx; line-height: 80rpx; padding: 0 40rpx; margin: 0;
	}
}
.debug-tip {
	position: absolute; bottom: 15rpx; width: 100%; text-align: center;
	font-size: 20rpx; color: rgba(0,0,0,0.3); pointer-events: none;
}
</style>
