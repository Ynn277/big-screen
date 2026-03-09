

<template>
  <div class="header-container">
    
    <div class="header-side">
      <dv-decoration-10 style="width:100%; height:5px;" />
    </div>

    <div class="header-center">
      <dv-decoration-8 class="deco-bracket" :color="['#568aea', '#568aea']" />
      
      <div class="title-text">全球智能维护系统</div>
      
      <dv-decoration-8 :reverse="true" class="deco-bracket" :color="['#568aea', '#568aea']" />
    </div>

    <div class="header-side">
      <dv-decoration-10 style="width:100%; height:5px; transform: rotateY(180deg);" />
    </div>

  </div>
</template>

<script setup>
// 无需特殊逻辑，主要靠 CSS 布局
</script>

<style scoped>
/* 1. 整体容器布局 */
.header-container {
  width: 100%;
  height: 80px; /* 设定一个固定高度 */
  display: flex;
  justify-content: space-between;
  align-items: center; /* 垂直居中 */
  padding: 0 20px;
  background-color: transparent; /* 保持透明，由父级背景决定 */
}

/* 2. 两侧线条区域 (各占 25%) */
.header-side {
  width: 40%;
  height: 100%;
  display: flex;
  align-items: center; /* 让线条垂直居中 */
}

/* 3. 中间标题区域 (占 50%) */
.header-center {
  width: 50%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}

/* 4. 中间两侧的装饰括号 */
.deco-bracket {
  width: 100px; /* 调整宽度，原 300px 太宽导致挤压文字 */
  height: 50px;
}

/* 5. 标题文字样式 (重点优化) */
.title-text {
  font-size: 32px;
  font-weight: bold;
  color: #fff;
  margin: 0 15px; /* 文字和括号的间距 */
  white-space: nowrap; /* 强制不换行 */
  letter-spacing: 4px; /* 字间距，增加大气感 */
  
  /* 科技感渐变文字 */
  background: linear-gradient(to bottom, #ffffff 0%, #85c6ff 100%);
  -webkit-background-clip: text;
  color: transparent;
  
  /* 文字发光阴影 */
  text-shadow: 0 0 10px rgba(0, 161, 255, 0.5);
}

/* 响应式微调：屏幕较宽时，装饰线可以长一点 */
@media screen and (min-width: 1600px) {
  .deco-bracket {
    width: 150px; 
  }
}
</style>


