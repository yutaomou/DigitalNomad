<template>
	<view class="container">
		<view class="header">
			<view class="brand">
				<text class="title">聊城数字游民服务平台</text>
				<text class="subtitle">两河明珠 · 江北水城</text>
			</view>
		</view>

		<view class="role-tabs">
			<view 
				v-for="item in roles" 
				:key="item.id"
				:class="['tab-item', activeRole === item.id ? 'active' : '']"
				@tap="switchRole(item.id)"
			>
				{{ item.name }}
			</view>
		</view>

		<view class="form-box">
			<view class="input-item">
				<input type="number" v-model="formData.phone" placeholder="请输入手机号" maxlength="11" />
			</view>
			<view class="input-item">
				<input type="password" v-model="formData.password" placeholder="请输入密码" />
			</view>
			
			<view v-if="!isLogin">
				<view v-if="activeRole === 'nomad'" class="input-item">
					<input type="text" v-model="formData.job" placeholder="您的职业 (如: 开发、设计)" />
				</view>
				<view v-if="activeRole === 'merchant'" class="input-item">
					<picker @change="onCategoryChange" :range="categories">
						<view class="picker-text">{{ formData.category || '请选择经营类目' }}</view>
					</picker>
				</view>
			</view>

			<button class="submit-btn" @tap="handleSubmit">
				{{ isLogin ? '立即登录' : '提交注册' }}
			</button>
			
			<view class="switch-mode" @tap="isLogin = !isLogin">
				{{ isLogin ? '没有账号？立即注册' : '已有账号？返回登录' }}
			</view>
		</view>

		<view v-if="activeRole === 'nomad'" class="eco-section">
			<view class="divider"><text>外部平台一键认证</text></view>
			<view class="platform-list">
				<view class="p-item" @tap="handleThirdParty('猪八戒网')">
					<image src="/static/zhubajie.png" mode="aspectFit" /><text>猪八戒网</text>
				</view>
				<view class="p-item" @tap="handleThirdParty('站酷')">
					<image src="/static/zhanku.png" mode="aspectFit" /><text>站酷</text>
				</view>
			</view>
		</view>

		<view class="footer-info">
			<text>{{ footerTips[activeRole] }}</text>
			<view class="path-tag">虚拟存储节点: {{ JSON_FILES[activeRole] }}</view>
		</view>
	</view>
</template>

<script setup>
import { ref, reactive, onMounted } from 'vue';

// 1. 模拟文件路径配置
const JSON_FILES = {
	nomad: '/user/nomads.json',
	merchant: '/user/merchants.json',
	gov: '/user/admins.json'
};

const isLogin = ref(true);
const activeRole = ref('nomad'); 
const roles = [
	{ id: 'nomad', name: '数字游民' },
	{ id: 'merchant', name: '合作商户' },
	{ id: 'gov', name: '政府/管理方' }
];
const categories = ['餐饮美食', '精品民宿', '文旅景区', '休闲娱乐'];
const footerTips = {
	nomad: '🎁 认证即领 7 天免费共享工位',
	merchant: '📈 接入平台，获取数字游民消费画像',
	gov: '📊 实时看板：掌握全市数据要素动态'
};

const formData = reactive({
	phone: '',
	password: '',
	job: '',
	category: ''
});

onMounted(() => {
	// 初始化虚拟存储环境
	Object.values(JSON_FILES).forEach(path => {
		if (!uni.getStorageSync(path)) {
			uni.setStorageSync(path, []);
		}
	});
	
	// 预置政府管理员数据
	const adminPath = JSON_FILES.gov;
	const admins = uni.getStorageSync(adminPath) || [];
	if (admins.length === 0) {
		admins.push({ phone: '13800000000', password: 'admin', role: 'gov' });
		uni.setStorageSync(adminPath, admins);
	}
});

const switchRole = (role) => {
	activeRole.value = role;
	isLogin.value = true;
};

const onCategoryChange = (e) => {
	formData.category = categories[e.detail.value];
};

/**
 * 提交逻辑：处理登录比对及注册写入
 */
const handleSubmit = () => {
	if (!formData.phone || !formData.password) {
		return uni.showToast({ title: '请填写完整', icon: 'none' });
	}

	const targetPath = JSON_FILES[activeRole.value];
	let userList = uni.getStorageSync(targetPath) || [];

	if (isLogin.value) {
		// --- 登录逻辑 ---
		const user = userList.find(u => u.phone === formData.phone && u.password === formData.password);
		if (user) {
			uni.showToast({ title: '登录成功', icon: 'success' });
			
			// 关键：存储当前登录的用户 Session 信息
			uni.setStorageSync('current_user_session', user); 
			
			// 关键：跳转至 pages/view/HomeView.vue
			setTimeout(() => {
				uni.navigateTo({
					url: '/pages/view/HomeView'
				});
			}, 1000);
		} else {
			uni.showToast({ title: '账号或密码错误', icon: 'none' });
		}
	} else {
		// --- 注册逻辑 ---
		if (userList.some(u => u.phone === formData.phone)) {
			return uni.showToast({ title: '该号码已注册', icon: 'none' });
		}

		const newUser = {
			phone: formData.phone,
			password: formData.password,
			role: activeRole.value,
			regSource: targetPath, 
			createTime: new Date().getTime(),
			...(activeRole.value === 'nomad' && { job: formData.job }),
			...(activeRole.value === 'merchant' && { category: formData.category })
		};

		userList.push(newUser);
		uni.setStorageSync(targetPath, userList);
		
		uni.showToast({ title: '注册并写入成功' });
		isLogin.value = true;
	}
};

const handleThirdParty = (name) => {
	uni.showModal({
		title: '生态认证',
		content: `是否同步您在${name}的信用分及技能认证数据？`,
		success: (res) => { if (res.confirm) uni.showToast({ title: '同步成功', icon: 'success' }); }
	});
};
</script>

<style lang="scss">
.container {
	padding: 40rpx; min-height: 100vh; background: linear-gradient(180deg, #F8FBFF 0%, #FFFFFF 100%);
	.header { padding: 80rpx 0 40rpx; text-align: center;
		.title { font-size: 44rpx; font-weight: bold; color: #1a1a1a; display: block; }
		.subtitle { font-size: 26rpx; color: #007aff; margin-top: 10rpx; letter-spacing: 4rpx; }
	}
	.role-tabs { display: flex; background: #f0f2f5; border-radius: 60rpx; margin-bottom: 50rpx; padding: 6rpx;
		.tab-item { flex: 1; text-align: center; padding: 20rpx 0; font-size: 28rpx; color: #666; transition: 0.3s;
			&.active { background: #007aff; color: #fff; border-radius: 60rpx; font-weight: bold; box-shadow: 0 4rpx 15rpx rgba(0,122,255,0.2); }
		}
	}
	.form-box {
		.input-item { background: #fff; border-radius: 20rpx; padding: 26rpx 34rpx; margin-bottom: 25rpx; box-shadow: 0 4rpx 15rpx rgba(0,0,0,0.04);
			input, .picker-text { font-size: 30rpx; }
		}
		.submit-btn { background: #007aff; color: #fff; border-radius: 60rpx; margin-top: 40rpx; height: 100rpx; line-height: 100rpx; font-weight: bold; }
		.switch-mode { text-align: center; margin-top: 30rpx; color: #666; font-size: 26rpx; text-decoration: underline; }
	}
	.eco-section { 
		margin-top: 60rpx; 
		.divider { 
			text-align: center; position: relative; border-bottom: 1px solid #eee; height: 1rpx; margin-bottom: 40rpx; 
			text { position: absolute; left: 50%; top: -15rpx; transform: translateX(-50%); background: #fdfdff; padding: 0 20rpx; font-size: 24rpx; color: #ccc; } 
		}
		.platform-list { 
			display: flex; justify-content: center; gap: 80rpx; 
			.p-item { 
				display: flex; flex-direction: column; align-items: center; 
				image { width: 80rpx; height: 80rpx; margin-bottom: 10rpx; border-radius: 12rpx; } 
				text { font-size: 24rpx; color: #888; } 
			} 
		}
	}
	.footer-info { 
		position: fixed; bottom: 60rpx; left: 0; right: 0; text-align: center; font-size: 24rpx; color: #007aff; 
		.path-tag { font-size: 20rpx; color: #bbb; margin-top: 10rpx; }
	}
}
</style>