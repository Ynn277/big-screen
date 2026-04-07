<!-- <template>
  <div class="chart-container">
    <div ref="chartRef" style="width: 100%; height: 100%"></div>
  </div>
</template> -->

<template>
  <div class="chart-container" style="position: relative; width: 100%; height: 100%;">
    <div ref="chartRef" style="width: 100%; height: 100%"></div>

    <div v-if="showCustomTip" class="custom-center-tooltip">
      <div class="tooltip-header">{{ customTipData.name }}</div>
      <div class="tooltip-body">
        列车总数：<span class="val">{{ customTipData.total || '--' }}</span><br/>
        人员数量：<span class="val">{{ customTipData.person || '--' }}</span><br/>
        人车比：&nbsp;&nbsp;&nbsp;&nbsp;<span class="val">{{ customTipData.online || '--' }}</span><br/>
        持证人数：<span class="val">{{ customTipData.certify || '--' }}</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch, nextTick, shallowRef } from "vue";
import * as echarts from "echarts";
import "echarts-gl";
import earthImg from "../assets/img/earth.jpg";
import earthMouthImg from "../assets/img/earth-mouth.jpg";

// 在 script 顶部定义两个响应式变量（如果你还没引入 ref，记得在顶部 import { ref } from 'vue'）
const showCustomTip = ref(false); // 控制弹窗显示
const customTipData = ref({});    // 存储当前弹窗的数据

// 替换掉之前的 watch
watch(() => props.activeName, (newName) => {
  // 如果是全国总览状态 (没有 activeName)，关闭弹窗
  if (!newName) {
    showCustomTip.value = false;
    return;
  }
  
  // 在你的所有点数据中，找到当前高亮的那一条
  const activeItem = props.allPoints.find(item => item.name === newName);
  
  if (activeItem) {
    // 把数据赋给我们的自定义弹窗，并显示出来
    customTipData.value = activeItem;
    showCustomTip.value = true;
  } else {
    showCustomTip.value = false;
  }
});

const props = defineProps({
  targetCoords: { type: Array, default: null }, // [经度, 纬度] 或 null(总览)
  activeName: { type: String, default: null }, // 接收当前高亮的项目名称
  allPoints: { type: Array, default: () => [] }, // 所有点数据
});

const chartRef = ref(null);
const myChart = shallowRef(null);
// 记录上一次的坐标，用于总览模式时“锚定”视角，防止地球乱飞
const INITIAL_COORDS = [113.2644, 23.1291];
let lastCoords = [113.2644, 23.1291];
let isUserInteracting = false; // 简单的交互锁

onMounted(() => {
  nextTick(initChart);
  window.addEventListener("resize", handleResize);
});


onUnmounted(() => {
  try {
    window.removeEventListener("resize", handleResize);
    // 【修复】：先判断 myChart.value 是否存在，再判断 dispose 方法
    if (myChart.value && typeof myChart.value.dispose === "function") {
      myChart.value.dispose();
    }
  } catch (error) {
    console.error("销毁图表实例时发生错误:", error);
  }
});

// === 核心逻辑 1：监听坐标变化，驱动视图 ===
watch(
  () => props.targetCoords,
  (newCoords) => {
    // 如果有新坐标，更新“上一次坐标”记录
    if (newCoords) {
      lastCoords = newCoords;
    }
    // 执行视图更新动画
    updateView(newCoords);
  }
);

// 全局变量，用于存储转换后的线条数据 (确保这个变量在 initChart 外面定义)
let chinaLinesData = [];

const initChart = async () => {
  
  if (!chartRef.value) return;

  // === 步骤 1：使用本地地图数据 ===
  try {
    // 直接导入本地地图文件
    import("../assets/json/china-boundary.json")
      .then((chinaJsonModule) => {
        // 确保 chinaJsonModule 和 chinaJsonModule.default 存在
        if (chinaJsonModule && (chinaJsonModule.default || chinaJsonModule)) {
          // 兼容处理：有的打包环境可能是 default，有的是直接导出
          const chinaJson = chinaJsonModule.default || chinaJsonModule;

          // 检查 chinaJson 的结构是否正确
          if (
            chinaJson &&
            typeof chinaJson === "object" &&
            Array.isArray(chinaJson.features)
          ) {
            // 注册地图 (虽然 lines3D 不直接用它，但留着也没坏处)
            echarts.registerMap("china", chinaJson);

            // =========================================================
            // 【核心融合点】：在这里调用数据转换函数
            // 把 GeoJSON 转换成 lines3D 需要的线条数组
            // =========================================================
            chinaLinesData = convertGeoJSONToLines(chinaJson);
            console.log(
              "本地地图数据加载并转换成功，线条数量:",
              chinaLinesData.length
            );

            // 初始化 ECharts
            renderGlobe();
          } else {
            console.error("本地地图数据格式错误:", chinaJson);
            // 即使地图数据有问题，也尝试渲染地球，只是没有中国地图层
            renderGlobe();
          }
        } else {
          console.error("本地地图数据模块加载失败:", chinaJsonModule);
          // 即使地图数据加载失败，也尝试渲染地球
          renderGlobe();
        }
      })
      .catch((err) => {
        console.error("本地地图数据导入失败:", err);
        // 捕获异常后，仍然尝试渲染地球
        renderGlobe();
      });
  } catch (err) {
    console.error("地图数据处理失败:", err);
    // 捕获异常后，仍然尝试渲染地球
    renderGlobe();
  }
};


// 辅助函数：将 GeoJSON 解析为线条，并加入【人工校准】逻辑
const convertGeoJSONToLines = (geoJson) => {
  const lines = [];
  if (!geoJson || !geoJson.features) return lines;

  // ================= 校准参数 (在这里修改) =================
  // 如果线条偏左，就加大经度 (正数向右，负数向左)
  // 如果线条偏上，就减小纬度 (正数向上，负数向下)
  const lngOffset = -0.1; // <--- 试着改这个 (例如 0.5 或 -0.5)
  const latOffset = 0.1; // <--- 试着改这个
  const lineAltitude = 1; // <--- 线条悬浮高度 (建议 0.5 ~ 2.0)
  // ========================================================

  geoJson.features.forEach((feature) => {
    const geometry = feature.geometry;
    if (!geometry) return;

    let rings = [];
    if (geometry.type === "Polygon") {
      rings = geometry.coordinates;
    } else if (geometry.type === "MultiPolygon") {
      geometry.coordinates.forEach((polygon) => {
        rings = rings.concat(polygon);
      });
    }

    rings.forEach((ring) => {
      // 1. 预处理：应用偏移量并计算新边界
      // 我们把校准后的坐标存到一个新数组里，用来做后续判断和渲染
      const adjustedRing = ring.map((coord) => {
        return [
          coord[0] + lngOffset, // 经度校准
          coord[1] + latOffset, // 纬度校准
          lineAltitude, // 加上高度，防止被地球地形遮挡
        ];
      });

      // 2. 基于【校准后】的坐标计算特征
      let minLng = 180,
        maxLng = -180;
      let minLat = 90,
        maxLat = -90;

      adjustedRing.forEach((coord) => {
        const [lng, lat] = coord;
        if (lng < minLng) minLng = lng;
        if (lng > maxLng) maxLng = lng;
        if (lat < minLat) minLat = lat;
        if (lat > maxLat) maxLat = lat;
      });
      const pointCount = adjustedRing.length;

      // === 3. 黑名单逻辑 (沿用之前的完美逻辑) ===

      // A. 剔除台湾右侧红框 (注意：如果加了偏移，这里判断也要相应微调，不过通常0.几度的偏移不影响这个粗略判断)
      if (minLng > 122.5 + lngOffset) return;

      // B. 剔除南海大矩形框
      if (minLat < 15 + latOffset && maxLat > 20 + latOffset) return;

      // C. 剔除南海南部碎块
      if (maxLat < 18 + latOffset) return;

      // === 4. 白名单逻辑 ===

      // D. 大陆海岸线保留
      if (pointCount > 100) {
        lines.push({ coords: adjustedRing }); // 注意：push的是校准后的 coordinates
        return;
      }

      // E. 台湾/海南保留
      const isTaiwan =
        minLng > 119 && maxLng < 123 && minLat > 21 && maxLat < 26;
      const isHainan =
        minLng > 108 && maxLng < 112 && minLat > 18 && maxLat < 21;

      if (isTaiwan || isHainan) {
        lines.push({ coords: adjustedRing });
        return;
      }

      // === 5. 幸存者 ===
      // F. 沿海小岛保留
      if (pointCount > 10) {
        lines.push({ coords: adjustedRing });
      }
    });
  });

  return lines;
};

const renderGlobe = () => {
  myChart.value = echarts.init(chartRef.value);

  // 添加鼠标交互监听（防止用户拖拽时被代码强制拉回）
  myChart.value.getZr().on("mousedown", () => {
    isUserInteracting = true;
    myChart.value.setOption({
      globe: { viewControl: { targetCoord: null, autoRotate: false } },
    });
  });
  myChart.value.getZr().on("mouseup", () => {
    setTimeout(() => {
      isUserInteracting = false;
    }, 1000);
  });
 const techFont = "'hst', 'ysbt', 'almm'";
  // myChart.vue 内部配置示例
const option = {
    
    

    backgroundColor: "rgba(0,0,0,0)",
    globe: {
      // 优化配置，增加真实感同时保持稳定
      baseTexture: earthImg, // 使用地球纹理
      heightTexture: earthMouthImg, // 恢复高度纹理，增加真实感
      displacementScale: 0.01, // 极低的位移缩放，避免过度变形
      // shading: 'lambert', // 恢复lambert渲染模式，增加真实光照效果
      shading: "color", // 平面渲染模式，增加简单性
      // baseColor: '#1a5cb0',
      environment: "none",

      top: "middle",
      left: "center",
      globeRadius: 50,
      // 启用简单的后期特效，增加真实感
      postEffect: {
        enable: false,
        // enable: true,
        // bloom: { enable: true, bloomIntensity: 0.05 } // 极低的辉光强度
      },

      light: {
        main: {
          intensity: 10,
          shadow: false,
          alpha: 30, // 调整光源位置
          beta: 40,
        },
        ambient: { intensity: 10 }, // 合理的环境光
        ambientCubemap: {
          texture: "",
          diffuseIntensity: 0.5,
          specularIntensity: 0.5,
        },
      }, // 优化光照，增加真实感

      // === 初始视角配置 ===
      viewControl: {
        // autoRotate: true,
        autoRotateSpeed: -10,
        targetCoord: lastCoords, // 初始锁定
        distance: 300, // 增加初始距离，确保观察清晰
        minDistance: 60,
        maxDistance: 301, // 进一步增加最小距离，防止过近观察
        animationDurationUpdate: 1000,
        easing: "cubicInOut",
        minAlpha: 10,
        maxAlpha: 50,
      },
      geo: {
    itemStyle: {
      areaColor: 'rgba(7, 21, 57, 0.8)', // 使用深蓝黑色背景
      borderColor: 'rgba(0, 228, 255, 0.2)', // 弱化省份边界线
    }
  },

    },
    
    tooltip: {
    show: false,
    
  },
  emphasis: {
    label: {
      show: false,
      formatter: '', // 强制让它没内容可显
      opacity: 0
    },
    trigger: 'item',
    renderMode: 'html',
    appendToBody: true,        // 核心：直接挂载到 body，突破所有层级
    // alwaysShowContent: true,   // 核心：让它一直亮着，不随鼠标移开而消失
    // 关键：把侧边栏的 毛玻璃、边框、阴影 样式原封不动地复制到这里！
    textStyle : {fontFamily :techFont} ,
    extraCssText: 'background: rgba(200, 200, 200, 0.08) !important; backdrop-filter: blur(3px) !important; -webkit-backdrop-filter: blur(3px) !important; border: 1px solid rgba(0, 228, 255, 0.3) !important; box-shadow: inset 0 0 15px rgba(0, 228, 255, 0.1), 0 5px 15px rgba(0, 0, 0, 0.4) !important; border-radius: 8px !important; z-index: 9999 !important; pointer-events: none;',
    formatter: function(params) {
      if(!params.data) return '';
      // 这里的 HTML 结构高度还原原型图的设计
      return `
        <div style="padding: 5px 10px; min-width: 140px;">
          <div style="color: #00e4ff; font-size: 18px; font-weight: bold; margin-bottom: 8px; border-bottom: 1px solid rgba(0,228,255,0.3); padding-bottom: 5px;">
            ${params.data.name}
          </div>
          <div style="color: #b0cbe9; font-size: 14px; line-height: 2;">
            列车总数：<span style="color:#fff; font-weight:bold; float:right;">${params.data.total || '--'}</span><br/>
            人员数量：<span style="color:#fff; font-weight:bold; float:right;">${params.data.person || '--'}</span><br/>
            人车比：&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#fff; font-weight:bold; float:right;">${params.data.online || '--'}</span><br/>
            持证人数：<span style="color:#fff; font-weight:bold; float:right;">${params.data.certify || '--'}</span>
          </div>
        </div>
      `;
    }
  },

    series: [
      // === 添加散点层用于显示标记点和标签 ===
      {
        type: "scatter3D",
        coordinateSystem: "globe",
        blendMode: "source-over", 
        symbol: "circle", // 改为圆圈
        symbolSize: 25, 
        zlevel: 100, 
        z: 1000, 

        itemStyle: { 
          color: "#33DFFF", 
          opacity: 0.9,
          borderWidth: 4,
          borderColor: "rgba(51, 223, 255, 0.4)" 
        },

        

      label: {
        show: false,
      },
      

        data: [], 
        animation: false 
      },

      {
        type: "scatter3D",
        coordinateSystem: "globe",
        blendMode: "source-over", 
        symbol: "circle",
        symbolSize: 25, 
        zlevel: 99, 
        itemStyle: { 
          color: "#33DFFF", 
          opacity: 0.6, 
          borderWidth: 4, 
          borderColor: "rgba(51, 223, 255, 0.3)" 
        },
        label: { show: false },
        data: [], // 初始为空
        animation: false 
      },
      
    ],
  };

  myChart.value.setOption(option);
  updateView(props.targetCoords);
};


const updateView = (coords) => {
  if (!myChart.value) return;

  // 1. 判定模式 - 安全检查coords是否存在及长度
  const isOverview = !coords || !Array.isArray(coords) || coords.length === 0;

  // 2. 视角配置
  const targetToUse = isOverview ? INITIAL_COORDS : coords;
  const viewOption = {
    autoRotate: isOverview ? true : false,
    autoRotateSpeed: -10,
    distance: isOverview ? 300 : 51,
    beta: isOverview ? 30 : 65,
    targetCoord: targetToUse,
    animationDurationUpdate: 1000,
    easing: "cubicInOut",
  };

  // 3. 准备 Globe 皮肤 (完整配置，确保不白球)
  const globeConfig = {
    baseTexture: earthImg,
    heightTexture: earthMouthImg,
    displacementScale: 0.01,
    shading: "color",
    environment: "none",
    globeRadius: 150,
    top: "middle",
    left: "center",
    light: {
      main: { intensity: 10, shadow: false },
      ambient: { intensity: 10 },
    },
    viewControl: viewOption,
  };

  // =========================================================
  // 【核心修改】数据准备阶段
  // 我们不再动态 push series，而是每次都准备好所有的数据
  // 不需要显示的图层，数据直接给空数组 []
  // =========================================================

  // A. 准备光点数据 (只在总览模式下有值)
  let haloData = [];
  let coreData = [];
  // 确保props.allPoints是数组且有数据
  if (
    isOverview &&
    Array.isArray(props.allPoints) &&
    props.allPoints.length > 0
  ) {
    haloData = props.allPoints.map((p) => ({
      name: p.name || "",
      value:
        Array.isArray(p.value) && p.value.length >= 2
          ? [p.value[0], p.value[1], 0]
          : [0, 0, 0],
    }));
    coreData = props.allPoints.map((p) => ({
      name: p.name || "",
      value:
        Array.isArray(p.value) && p.value.length >= 2
          ? [p.value[0], p.value[1], 0]
          : [0, 0, 0],
    }));
  }

  // B. 准备图标数据 (只在详情模式下有值)
  let pinData = [];
  if (!isOverview && Array.isArray(coords) && coords.length >= 2) {
    let targetPoint = null;
    // 确保props.allPoints是数组且有数据
    if (Array.isArray(props.allPoints) && props.allPoints.length > 0) {
      targetPoint = props.allPoints.find((p) => p.name === props.activeName);
      if (!targetPoint) {
        targetPoint = props.allPoints.find(
          (p) =>
            Array.isArray(p.value) &&
            p.value.length >= 2 &&
            Math.abs(p.value[0] - coords[0]) < 0.0001 &&
            Math.abs(p.value[1] - coords[1]) < 0.0001
        );
      }
    }
    if (targetPoint) {
      pinData = [targetPoint];
    } else {
      pinData = [
        {
          name: props.activeName || "目标区域",
          value: [coords[0], coords[1], 0.5],
          itemStyle: { color: "#ffcc00" },
        },
      ];
    }
  }

  let historyPinData = [];
  if (!isOverview && Array.isArray(props.allPoints)) {
    // 找到当前高亮的项目
    const targetPoint = props.allPoints.find((p) => p.name === props.activeName);
    
    // 确保找到了，并且它具有原始排序索引
    if (targetPoint && targetPoint._originalIndex !== undefined) {
      const currentIdx = targetPoint._originalIndex;
      
      // 筛选出所有 原始索引 < 当前索引 的项目作为历史足迹
      historyPinData = props.allPoints
        .filter(p => p._originalIndex !== undefined && p._originalIndex < currentIdx)
        .map(p => ({
          name: p.name,
          value: [p.value[0], p.value[1], 0.5] // 保持和主气泡一样的高度
        }));
    }
  }
  

  // =========================================================
  // 【核心修改】构建 Series 数组
  // 包含所有图层 (Halo + Core + Pin)
  // 通过 data 是否为空来控制显示/隐藏，而不是删除图层
  // =========================================================
  const seriesConfig = [
    {
      type: "lines3D",
      coordinateSystem: "globe",
      effect: {
        show: false, // 不需要光点流动，只需要常亮
      },
      blendMode: "lighter",

      // 线条样式
      lineStyle: {
        width: 2, // 线条宽度
        color: "#ffcc00", // 金色
        opacity: 1,
      },

      // 关键：把线条数据传进去
      data: chinaLinesData,

      // 让线条稍微“浮”在地球表面一点点，防止被山脉遮挡
      // polyline: true 表示数据是多段线
      polyline: true,
      zlevel: -10,
    },
    // 1. 总览光晕层 (Overview_Halo)
    {
      type: "scatter3D",
      coordinateSystem: "globe",
      name: "Overview_Halo",
      symbol: "circle",
      symbolSize: 15,
      blendMode: "lighter",
      itemStyle: { color: "#00e4ff", opacity: 0.8 },
      label: { show: false },
      silent: true,
      data: haloData, // 有数据就显示，没数据([])就自动隐藏
    },

    // 2. 总览核心层 (Overview_Core)
    {
      type: "scatter3D",
      coordinateSystem: "globe",
      name: "Overview_Core",
      symbol: "circle",
      symbolSize: 10,
      blendMode: "lighter",
      itemStyle: { color: "#ffffff", opacity: 1 },
      label: { show: false },
      silent: true,
      data: coreData, // 有数据就显示，没数据([])就自动隐藏
    },

   
    // 3. 详情图标层 (Detail_Pin)
    {
      type: "scatter3D",
      coordinateSystem: "globe",
      name: "Detail_Pin",
      symbol: "circle", // 将水滴图标 'pin' 改为圆圈 'circle' 以模拟气泡
      symbolSize: 28,   // 适当缩小尺寸
      zlevel: 100,
      symbolOffset: [0, 0], // 圆圈不需要像水滴那样偏移，使其正对坐标
      itemStyle: { 
        color: "#33DFFF", // 亮蓝色核心
        opacity: 0.9,
        borderWidth: 4,   // 加上边框模拟外围的光晕效果
        borderColor: "rgba(51, 223, 255, 0.4)" 
      },
     
      label: {
        show: false, // 关闭弹窗显示
      },
      data: pinData, // 有数据就显示，没数据([])就自动隐藏
      emphasis: {

      tooltip: {
      show: false
    },
      label: {
      show: false,
      formatter: '', // 强制让它没内容可显
      opacity: 0
    }
  },
    },

    
    

    {
      type: "scatter3D",
      coordinateSystem: "globe",
      name: "History_Pin",
      symbol: "circle",
      symbolSize: 25, 
      zlevel: 99,     // 层级略低于当前主气泡(100)，防止遮挡当前的高亮弹窗
      itemStyle: { 
        color: "#33DFFF", 
        opacity: 0.8, // 透明度设低一点，让历史点稍暗，突出当前正在看的主气泡
        borderWidth: 4,   
        borderColor: "rgba(51, 223, 255, 0.4)" 
      },
      label: { show: false }, // 🚀 关键：关闭弹窗显示
      data: historyPinData // 传入计算好的历史点数据
    }

  ];

  // 5. 应用配置
  // 【关键】：这里不需要 replaceMerge，也不需要 notMerge
  // 因为 series 结构固定不变，只是 data 变了，ECharts 处理这种情况非常稳健
  myChart.value.setOption({
    globe: globeConfig,
    series: seriesConfig,
  });
};

const handleResize = () => {
  myChart.value && myChart.value.resize();
};
</script>

<style scoped>
.chart-container {
  width: 100%;
  height: 100%;
  overflow: hidden;
}
.custom-center-tooltip {
  position: absolute;
  top: 50%;
  left: 50%;
  /* 调整偏移量：向下向右偏移一点，不要正好挡住中间的那个点 */
  transform: translate(20px, -50%); 
  z-index: 9999;
  pointer-events: none; /* 穿透鼠标事件，不妨碍你拖拽地球 */
  
  /* 质感大放送：和侧边栏完全一致的 UI */
  min-width: 140px;
  padding: 10px 15px;
  background: rgba(10, 30, 50, 0.7);
  backdrop-filter: blur(8px); /* 毛玻璃 */
  -webkit-backdrop-filter: blur(8px);
  border: 1px solid rgba(0, 228, 255, 0.4);
  box-shadow: inset 0 0 15px rgba(0, 228, 255, 0.2), 
              0 5px 15px rgba(0, 0, 0, 0.6);
  border-radius: 4px;
}

.tooltip-header {
  font-family: "almm";
  color: #00e4ff;
  font-size: 16px;
  font-weight: bold;
  margin-bottom: 8px;
  padding-bottom: 5px;
  border-bottom: 1px solid rgba(0, 228, 255, 0.3);
}

.tooltip-body {
  font-family: "almm";
  color: #b0cbe9;
  font-size: 14px;
  line-height: 2.2;
}

.tooltip-body .val {
  color: #ffffff;
  font-weight: bold;
  float: right; /* 让数字靠右对齐，非常整齐 */
}

</style>



