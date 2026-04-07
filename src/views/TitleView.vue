<script setup>
import crrcLogo from '@/assets/img/crrc.png';
</script>

<template>
  <div class="header-container">
    
    <div class="top-metal-beam">
      <div class="metal-details">
        <i></i><i></i><i></i>
        <span class="stiffener"></span>
        <span class="stiffener"></span>
        <i></i><i></i><i></i>
      </div>
    </div>

    <div class="header-side">
      <dv-decoration-10 style="width:100%; height:3px; opacity: 0.3;" />
    </div>

    <div class="header-center-wrapper">
      <div class="header-center">
        <div class="zebra-stripes left-zebra"></div>

        <div class="title-content">
          <img :src="crrcLogo" class="crrc-logo" alt="CRRC Logo">
          <div class="title-text">中车株机车辆运维总览图</div>
        </div>

        <div class="zebra-stripes right-zebra"></div>
      </div>
      
      <div class="bottom-sharp-line"></div>
    </div>

    <div class="header-side">
      <dv-decoration-10 style="width:100%; height:3px; transform: rotateY(180deg); opacity: 0.3;" />
    </div>

  </div>
</template>

<style scoped>
/* ================= 全局容器：垂直居中 ================= */
.header-container {
  width: 100%;
  height: 80px; 
  display: flex;
  justify-content: space-between;
  align-items: center; /* 🌟 核心：确保所有元素垂直中心对齐 */
  position: relative;
  background: radial-gradient(100% 100% at 50% 0%, rgba(6, 14, 28, 0.8) 0%, transparent 60%);
}

/* ================= 顶部金属横梁：硬朗材质 ================= */
.top-metal-beam {
  position: absolute;
  top: 0; left: 0;
  width: 100%; height: 6px;
  /* 深灰蓝金属渐变 */
  background: linear-gradient(90deg, transparent 0%, #0d1522 15%, #1e2a3b 50%, #0d1522 85%, transparent 100%);
  border-bottom: 1px solid rgba(0, 228, 255, 0.4);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.9);
  z-index: 10;
}

.metal-details {
  display: flex; justify-content: center; align-items: center; gap: 25px; height: 100%;
}
.metal-details i { width: 3px; height: 3px; background: rgba(255, 255, 255, 0.2); border-radius: 50%; }
.metal-details .stiffener { width: 20px; height: 2px; background: rgba(0, 228, 255, 0.2); border-radius: 2px; }

/* ================= 标题台主体：黑曜石质感 ================= */
.header-center-wrapper {
  position: relative;
  width: 48%;
  height: 72px; /* 标题台高度 */
  display: flex;
  justify-content: center;
  align-items: center; /* 🌟 核心：文字垂直居中 */
}

.header-center {
  width: 100%; height: 100%;
  display: flex; justify-content: space-between; align-items: center;
  
  /* 极致压暗：黑蓝色钛金渐变 */
  background: linear-gradient(180deg, #182638 0%, #0a1320 15%, #050b14 85%, #02050a 100%);
  clip-path: polygon(0 0, 100% 0, 95% 100%, 5% 100%);
  
  /* 🌟 核心：物理倒角高光 (Pixel-Perfect Highlight) */
  box-shadow: 
    inset 0 1px 0px rgba(255, 255, 255, 0.25), /* 顶部 1px 锋利银边 */
    inset 0 2px 10px rgba(0, 228, 255, 0.05),  /* 内部氛围蓝光 */
    inset 0 -8px 20px rgba(0, 0, 0, 0.9);      /* 底部重投影 */
    
  border-top: 1px solid #365275;
  position: relative;
  overflow: hidden; 
}

/* 🌟 增加微观拉丝暗纹，消除“糊”感 */
.header-center::before {
  content: ""; position: absolute; inset: 0;
  background: repeating-linear-gradient(0deg, transparent, transparent 2px, rgba(255, 255, 255, 0.02) 2px, rgba(255, 255, 255, 0.02) 4px);
  pointer-events: none;
  mix-blend-mode: overlay;
}


.title-content {
  display: flex;
  align-items: center; /* 垂直居中 */
  justify-content: center;
  gap: 18px; /* 图标与文字之间的间距 */
  z-index: 2;
  white-space: nowrap; 
}
.crrc-logo {
  height: 50px; /* 略小于 46px 的字号，视觉上更协调 */
  width: auto;
  /* 增加轻微的投影，使其在深色底座上更有立体感 */
  object-fit: contain;
}

/* ================= 铬金属文字特效 ================= */
.title-text {
  font-family: 'almm', 'Microsoft YaHei', sans-serif !important;
  font-size: 40px; /* 增大字号，压住全场 */
  font-weight: 900;
  letter-spacing: 6px;
  line-height: 1; /* 🌟 核心：消除默认行高导致的偏置 */
  
  /* 模拟物理金属圆柱体的高反光渐变 */
  background: linear-gradient(
    to bottom, 
    #ffffff 0%,    /* 顶高光 */
    #a5c2e3 45%,   /* 环境色反光 */
    /* #4a6583 50%,   金属转折暗带 (核心：深浅交界要锐利) */
    #c6e4ff 55%,   /* 下受光面 */
    #ffffff 100%   /* 底边缘反光 */
  );
  
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  color: transparent;

  /* 锐利黑色钢印投影 */
  filter: drop-shadow(0px 3px 2px rgba(0, 0, 0, 0.9)) 
          drop-shadow(0px 0px 8px rgba(0, 228, 255, 0.2));
}

/* ================= 辅助装饰细节 ================= */
.header-side {
  width: 25%; height: 60px; display: flex; align-items: center; padding: 0 20px;
}

.zebra-stripes { width: 60px; height: 10px; margin: 0 30px; opacity: 0.15; }
.left-zebra { background: repeating-linear-gradient(45deg, transparent, transparent 3px, #00e4ff 3px, #00e4ff 5px); }
.right-zebra { background: repeating-linear-gradient(-45deg, transparent, transparent 3px, #00e4ff 3px, #00e4ff 5px); }

.bottom-sharp-line {
  position: absolute; bottom: 0; left: 5.2%; right: 5.2%; height: 1.5px;
  background: linear-gradient(90deg, transparent 0%, rgba(0, 228, 255, 0.4) 20%, #ffffff 50%, rgba(0, 228, 255, 0.4) 80%, transparent 100%);
  box-shadow: 0 0 5px #00e4ff;
  z-index: 1;
}
</style>