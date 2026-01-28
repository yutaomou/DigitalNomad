<script setup>
import { onMounted, onUnmounted, ref } from 'vue'

const mapContainer = ref(null)
let mapInstance = null

// 演示数据：7个试点社区中的2个
const locations = [
  { name: "东昌湖数字游民公社", pos: [115.973381, 36.444743], type: '工位充足' },
  { name: "古城文化共创基地", pos: [115.986872, 36.456321], type: '非遗体验' }
]

const initMap = () => {
  // 动态加载高德地图脚本 (如果index.html没加，这里做兜底，建议在index.html加)
  if (!window.AMap) {
    const script = document.createElement('script')
    script.src = `https://webapi.amap.com/maps?v=2.0&key=5654de0a4038fc047f7a5aa95886bb9c` // TODO: 替换Key
    script.onload = createMap
    document.head.appendChild(script)
  } else {
    createMap()
  }
}

const createMap = () => {
  mapInstance = new window.AMap.Map(mapContainer.value, {
    zoom: 13,
    center: [115.980126, 36.450532], // 聊城中心
    viewMode: '3D'
  })

  // 添加标记
  locations.forEach(loc => {
    const markerContent = `
      <div class="flex flex-col items-center">
        <div class="w-8 h-8 bg-blue-600 rounded-full flex items-center justify-center text-white border-2 border-white shadow-lg">
          <i class="fas fa-map-marker-alt"></i>
        </div>
        <div class="bg-white text-[10px] px-2 py-0.5 rounded shadow mt-1 font-bold whitespace-nowrap">${loc.name}</div>
      </div>
    `
    const marker = new window.AMap.Marker({
      position: loc.pos,
      content: markerContent,
      offset: new window.AMap.Pixel(-16, -32)
    })
    marker.setMap(mapInstance)
  })
}

onMounted(() => {
  initMap()
})

onUnmounted(() => {
  if (mapInstance) mapInstance.destroy()
})
</script>

<template>
  <div class="h-screen w-full relative">
    <div ref="mapContainer" class="w-full h-full"></div>
    
    <div class="absolute top-4 left-4 right-4 bg-white p-3 rounded-xl shadow-lg flex items-center gap-2 z-10">
      <i class="fas fa-search text-gray-400"></i>
      <input placeholder="搜索工位 / 民宿 / 景区" class="w-full outline-none text-sm"/>
    </div>
  </div>
</template>