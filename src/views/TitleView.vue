

<template>
  <div class="header-container">
    
    <div class="header-side">
      <dv-decoration-10 style="width:100%; height:5px;" />
    </div>

    <div class="header-center">
      <dv-decoration-8 class="deco-bracket" :color="['#568aea', '#568aea']" />
      
      <div class="title-text">中车株机车辆运维总览图</div>
      
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
/* 1. 整体容器：保持完全透明，让两侧装饰线自然悬空 */
.header-container {
  width: 100%;
  height: 80px; 
  display: flex;
  justify-content: space-between;
  align-items: center; 
  padding: 0 20px;
  background-color: transparent; /* 确保这里没有任何背景色，完全透出地球 */
}

/* 3. 中间标题区域：改为清透的毛玻璃蒙版 */
.header-center {
  width: 45%; 
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  
  /* === 关键修改：换成清透背景 === */
  background: rgba(200, 200, 200, 0.08); /* 极度透明的浅灰白，和左右面板呼应 */
  backdrop-filter: blur(3px);            /* 轻微模糊背后的地球轮廓 */
  -webkit-backdrop-filter: blur(3px);
  
  /* 底部保留一条青色线压住阵脚，加上微弱的内发光 */
  border-bottom: 2px solid rgba(0, 228, 255, 0.4);
  box-shadow: inset 0 -10px 20px rgba(0, 228, 255, 0.05);
  
  /* 保留这个倒梯形切角，很有科技感（如果你更喜欢普通的矩形，可以把这一行删掉） */
  clip-path: polygon(0 0, 100% 0, 95% 100%, 5% 100%);
}

/* 标题文字样式：确保在透明背景下依然清晰 */
.title-text {
  font-size: 38px;
  font-weight: bold;
  margin: 0 15px; 
  white-space: nowrap; 
  letter-spacing: 4px; 
  
  color: #ffffff;
  /* 增强文字阴影，防止透明背景导致文字和地球底图顺色 */
  text-shadow: 0 4px 6px rgba(0, 0, 0, 0.9), 0 0 15px rgba(0, 228, 255, 0.8);
}

/* 2. 两侧线条区域 */
.header-side {
  width: 40%;
  height: 100%;
  display: flex;
  align-items: center; 
}


/* 4. 中间两侧的装饰括号 */
.deco-bracket {
  width: 100px; 
  height: 50px;
}



/* 响应式微调：屏幕较宽时，装饰线可以长一点 */
@media screen and (min-width: 1600px) {
  .deco-bracket {
    width: 150px; 
  }
}
</style>


