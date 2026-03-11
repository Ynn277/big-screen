<template>
  <!-- <dv-full-screen-container> 
    <dv-loading v-if="loading">正在加载...</dv-loading>
    <div v-else style="display: flex; flex-direction: column; height: 100%">
      <TitleView style="flex: 0 0 80px;"/>
      <MainView style="flex: 1; height: 100%;"/>
      <FooterView style="flex: 0 0 200px;"/> -->
    <!-- </div>
  </dv-full-screen-container> -->

<v-scale-screen width="1920" height="1080" :fullScreen="true" :wrapperStyle="{ backgroundColor: '#000000' }" :boxStyle="{ backgroundColor: '#000000' }">
    
    <div class="content bg"> <dv-loading v-if="loading">正在加载...</dv-loading>
      
      <div v-else class="app-layout">
        <TitleView class="header-section"/>
        <MainView class="main-section"/>
        </div>

    </div>

  </v-scale-screen>
</template>


<script setup>
import {ref,onMounted} from 'vue';
import VScaleScreen from 'v-scale-screen'
import TitleView from './TitleView.vue'
import MainView from './MainView.vue'

let loading = ref(true);

onMounted(()=>{
  setTimeout(()=>{
    loading.value = false;
  },1000)
})


</script>

<style scoped>
/* 让内容区域铺满 */
.content.bg {
  width: 100%;
  height: 100%;
}

/* 父容器开启相对定位 */
.app-layout {
  position: relative;
  width: 100%;
  height: 100%;
  overflow: hidden;
}

/* 标题层：绝对定位，悬浮在最上方 */
.header-section {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 80px; /* 和你 title.vue 里的高度保持一致 */
  z-index: 999; /* 保证不被地球遮挡 */
  
  /* 关键：因为整个顶部是 100% 宽度悬浮的，防止透明区域挡住底下地球的鼠标操作（比如旋转缩放） */
  pointer-events: none; 
}

/* 让标题里面具体的可见元素恢复点击交互（如果有按钮的话） */
.header-section :deep(*) {
  pointer-events: auto;
}

/* 地球层：绝对定位，铺满全屏，垫在底层 */
.main-section {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 1; /* 层级低于标题 */
}
</style>


