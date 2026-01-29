<template>
	<view class="apply-container">
		<view class="form-card">
			<view class="header-tip">
				<text class="title">商户入驻申请</text>
				<text class="sub">请填写真实信息，审核通过后即可发布优惠</text>
			</view>

			<view class="label">店铺名称</view>
			<input class="input" v-model="form.shop_name" placeholder="请输入营业执照上的名称" />
			
			<view class="label">服务类目</view>
			<picker @change="catChange" :range="cats">
				<view class="picker-box">
					<text :class="{ 'placeholder': !form.category }">
						{{ form.category || '点击选择类目' }}
					</text>
					<text class="arrow">▼</text>
				</view>
			</picker>
			
			<view class="label">营业执照 (清晰照片)</view>
			<view class="upload-area" @tap="chooseAndUpload">
				<image v-if="tempImgPath" :src="tempImgPath" mode="aspectFill" class="preview-img"></image>
				<view v-else class="upload-placeholder">
					<text class="plus">+</text>
					<text class="txt">点击上传</text>
				</view>
			</view>
			
			<button class="submit-btn" :loading="isSubmitting" @tap="submit">提交审核</button>
			<text class="footer-note">预计 24 小时内完成人工审核</text>
		</view>
	</view>
</template>

<script setup>
import { ref, reactive } from 'vue';

const db = uniCloud.database();
const cats = ['餐饮', '住宿', '文旅', '休闲'];
const isSubmitting = ref(false);
const tempImgPath = ref(''); // 用于页面临时展示

const form = reactive({
	shop_name: '',
	category: '',
	license_url: '' // 最终存储到数据库的云地址
});

// 选择类目
const catChange = (e) => {
	form.category = cats[e.detail.value];
};

// 选择并准备上传图片
const chooseAndUpload = () => {
	uni.chooseImage({
		count: 1,
		sizeType: ['compressed'],
		success: (res) => {
			tempImgPath.value = res.tempFilePaths[0];
		}
	});
};

// 提交申请逻辑
const submit = async () => {
	// 1. 基础校验
	if (!form.shop_name || !form.category) {
		return uni.showToast({ title: '请填写完整信息', icon: 'none' });
	}
	if (!tempImgPath.value) {
		return uni.showToast({ title: '请上传营业执照', icon: 'none' });
	}

	isSubmitting.value = true;
	uni.showLoading({ title: '正在提交...' });

	try {
		// 2. 上传图片到云存储
		const uploadRes = await uniCloud.uploadFile({
			filePath: tempImgPath.value,
			cloudPath: `merchants/license_${Date.now()}_${Math.floor(Math.random()*1000)}.jpg`
		});
		
		form.license_url = uploadRes.fileID;

		// 3. 写入数据库 merchants 表
		const savedUser = uni.getStorageSync('current_user_session');
		await db.collection('merchants').add({
			shop_name: form.shop_name,
			category: form.category,
			license_url: form.license_url,
			phone: savedUser ? savedUser.phone : '',
			status: 0, // 核心：0 表示审核中
			is_priority: false,
			create_time: Date.now()
		});

		uni.hideLoading();
		uni.showToast({ title: '提交成功' });
		
		// 4. 跳转回上一页或进度页
		setTimeout(() => {
			uni.navigateBack();
		}, 1500);

	} catch (e) {
		uni.hideLoading();
		console.error('提交报错：', e);
		uni.showModal({
			title: '提交失败',
			content: e.message || '网络异常',
			showCancel: false
		});
	} finally {
		isSubmitting.value = false;
	}
};
</script>

<style lang="scss" scoped>
.apply-container {
	padding: 40rpx;
	background: #f1f5f9;
	min-height: 100vh;
}

.form-card {
	background: #ffffff;
	padding: 40rpx;
	border-radius: 32rpx;
	box-shadow: 0 4rpx 20rpx rgba(0, 0, 0, 0.05);

	.header-tip {
		margin-bottom: 40rpx;
		.title { font-size: 40rpx; font-weight: bold; color: #1e293b; display: block; }
		.sub { font-size: 24rpx; color: #94a3b8; margin-top: 8rpx; display: block; }
	}

	.label {
		font-size: 28rpx;
		color: #475569;
		margin-top: 40rpx;
		margin-bottom: 16rpx;
		font-weight: 600;
	}

	.input {
		background: #f8fafc;
		height: 90rpx;
		padding: 0 30rpx;
		border-radius: 16rpx;
		font-size: 30rpx;
		border: 1rpx solid #e2e8f0;
	}

	.picker-box {
		background: #f8fafc;
		height: 90rpx;
		padding: 0 30rpx;
		border-radius: 16rpx;
		display: flex;
		justify-content: space-between;
		align-items: center;
		border: 1rpx solid #e2e8f0;
		font-size: 30rpx;
		.placeholder { color: #94a3b8; }
		.arrow { font-size: 20rpx; color: #cbd5e1; }
	}

	.upload-area {
		width: 100%;
		height: 340rpx;
		background: #f8fafc;
		border: 2rpx dashed #cbd5e1;
		border-radius: 20rpx;
		display: flex;
		justify-content: center;
		align-items: center;
		overflow: hidden;
		
		.preview-img { width: 100%; height: 100%; }
		
		.upload-placeholder {
			text-align: center;
			.plus { font-size: 60rpx; color: #94a3b8; display: block; line-height: 1; }
			.txt { font-size: 24rpx; color: #94a3b8; margin-top: 10rpx; }
		}
	}

	.submit-btn {
		background: #3b82f6;
		color: #ffffff;
		margin-top: 60rpx;
		height: 100rpx;
		line-height: 100rpx;
		border-radius: 50rpx;
		font-weight: bold;
		box-shadow: 0 8rpx 20rpx rgba(59, 130, 246, 0.3);
		&::after { border: none; }
	}

	.footer-note {
		display: block;
		text-align: center;
		font-size: 22rpx;
		color: #94a3b8;
		margin-top: 30rpx;
	}
}
</style>