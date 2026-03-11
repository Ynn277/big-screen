<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import HomeView from './views/HomeView.vue'
import FaultRecord from './views/FaultRecord.vue'
// 引入刚刚新建的检修组件
import MaintenanceRecord from './views/MaintenanceRecord.vue'

const currentPath = ref(window.location.hash)

const onHashChange = () => {
  currentPath.value = window.location.hash
}

onMounted(() => {
  window.addEventListener('hashchange', onHashChange)
})

onUnmounted(() => {
  window.removeEventListener('hashchange', onHashChange)
})
</script>

<template>
  <FaultRecord v-if="currentPath === '#/fault'" />
  <MaintenanceRecord v-else-if="currentPath === '#/maintenance'" />
  <HomeView v-else />
</template>

<style scoped>
</style>

<!-- <template>
  <div class="app-container">
    <div class="view-layer">
      <HomeView />
    </div>
    
    <div class="header-layer">
      <TitleView />
    </div>
  </div>
</template>

<style scoped>
.app-container {
  position: relative;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  background-color: #000; /* 兜底背景色 */
}

.view-layer {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 1; /* 底层 */
}

.header-layer {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  z-index: 99; /* 顶层悬浮 */
  pointer-events: none; /* 确保标题栏透明区域不挡住地球的鼠标操作 */
}

/* 恢复标题栏内容的交互能力 */
.header-layer :deep(.header-container) {
  pointer-events: auto;
}
</style> -->