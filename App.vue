<script setup>
import { onLaunch, onShow, onHide } from '@dcloudio/uni-app';

// #ifdef H5
/**
 * 高德地图 H5 端安全密钥配置
 * 注意：必须在地图组件加载前设置，且必须与 manifest.json 中的 Key 配对使用
 */
if (typeof window !== 'undefined') {
    window._AMapSecurityConfig = {
        securityJsCode: '2e0e83b6343b9d5093f2eaf66ae973ad钥', // 👈 替换为你从高德后台申请的“安全密钥”
    }
}
// #endif

onLaunch(() => {
    console.log('App Launch - 聊城数字游民平台已启动');
});

onShow(() => {
    console.log('App Show - 应用进入前台');
});

onHide(() => {
    console.log('App Hide - 应用进入后台');
});
</script>

<style lang="scss">
/* 每个页面公共css */

/* 建议在这里添加一些基础的全局样式，防止地图容器失效 */
page {
    width: 100%;
    height: 100%;
    margin: 0;
    padding: 0;
    display: flex;
    flex-direction: column;
}

/* 统一滚动条样式（可选） */
::-webkit-scrollbar {
    display: none;
    width: 0 !important;
    height: 0 !important;
    -webkit-appearance: none;
    background: transparent;
}
</style>