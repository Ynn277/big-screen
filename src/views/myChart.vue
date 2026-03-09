<template>
  <div class="chart-container">
    <div ref="chartRef" style="width: 100%; height: 100%"></div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch, nextTick, shallowRef } from 'vue';
import * as echarts from 'echarts';
import 'echarts-gl';

const props = defineProps({
  targetCoords: { type: Array, default: null }, // [经度, 纬度] 或 null(总览)
  activeName: { type: String, default: null }, // 接收当前高亮的项目名称
  allPoints: { type: Array, default: () => [] } // 所有点数据
})

const chartRef = ref(null);
const myChart = shallowRef(null);
// 记录上一次的坐标，用于总览模式时“锚定”视角，防止地球乱飞
let lastCoords = [113.2644, 23.1291]; 
let isUserInteracting = false; // 简单的交互锁

onMounted(() => {
  nextTick(initChart);
  window.addEventListener('resize', handleResize);
});

// onUnmounted(() => {
//   try {
//     window.removeEventListener('resize', handleResize);
//     // 最安全的检查方式：确保myChart存在，myChart.value存在，且具有dispose方法
//     if (typeof myChart !== 'undefined' && myChart !== null && 
//         typeof myChart.value !== 'undefined' && myChart.value !== null &&
//         typeof myChart.value.dispose === 'function') {
//       myChart.value.dispose();
//       // 释放引用
//       myChart.value = null;
//     }
//   } catch (error) {
//     console.error('销毁图表实例时发生错误:', error);
//   }
// });

onUnmounted(() => {
  try {
    window.removeEventListener('resize', handleResize);
    // 【修复】：先判断 myChart.value 是否存在，再判断 dispose 方法
    if (myChart.value && typeof myChart.value.dispose === 'function') {
      myChart.value.dispose();
    }
  } catch (error) {
    console.error('销毁图表实例时发生错误:', error);
  }
});

// === 核心逻辑 1：监听坐标变化，驱动视图 ===
watch(() => props.targetCoords, (newCoords) => {
  // 如果有新坐标，更新“上一次坐标”记录
  if (newCoords) {
    lastCoords = newCoords;
  }
  // 执行视图更新动画
  updateView(newCoords);
});

// 全局变量，用于存储转换后的线条数据 (确保这个变量在 initChart 外面定义)
let chinaLinesData = []; 

const initChart = async () => {
  if (!chartRef.value) return;

  // === 步骤 1：使用本地地图数据 ===
  try {
    // 直接导入本地地图文件
    import('../assets/json/china-boundary.json').then((chinaJsonModule) => {
      // 确保 chinaJsonModule 和 chinaJsonModule.default 存在
      if (chinaJsonModule && (chinaJsonModule.default || chinaJsonModule)) {
        // 兼容处理：有的打包环境可能是 default，有的是直接导出
        const chinaJson = chinaJsonModule.default || chinaJsonModule;
        
        // 检查 chinaJson 的结构是否正确
        if (chinaJson && typeof chinaJson === 'object' && Array.isArray(chinaJson.features)) {
          
          // 注册地图 (虽然 lines3D 不直接用它，但留着也没坏处)
          echarts.registerMap('china', chinaJson);
          
          // =========================================================
          // 【核心融合点】：在这里调用数据转换函数
          // 把 GeoJSON 转换成 lines3D 需要的线条数组
          // =========================================================
          chinaLinesData = convertGeoJSONToLines(chinaJson);
          console.log('本地地图数据加载并转换成功，线条数量:', chinaLinesData.length);
          
          // 初始化 ECharts
          renderGlobe();
          
        } else {
          console.error('本地地图数据格式错误:', chinaJson);
          // 即使地图数据有问题，也尝试渲染地球，只是没有中国地图层
          renderGlobe();
        }
      } else {
        console.error('本地地图数据模块加载失败:', chinaJsonModule);
        // 即使地图数据加载失败，也尝试渲染地球
        renderGlobe();
      }
    }).catch((err) => {
      console.error('本地地图数据导入失败:', err);
      // 捕获异常后，仍然尝试渲染地球
      renderGlobe();
    });
    
  } catch (err) {
    console.error('地图数据处理失败:', err);
    // 捕获异常后，仍然尝试渲染地球
    renderGlobe();
  }
}



// const convertGeoJSONToLines = (geoJson) => {
//   const lines = [];
//   if (!geoJson || !geoJson.features) return lines;

//   geoJson.features.forEach(feature => {
//     const geometry = feature.geometry;
//     if (!geometry) return;

//     let rings = [];
//     if (geometry.type === 'Polygon') {
//       rings = geometry.coordinates;
//     } else if (geometry.type === 'MultiPolygon') {
//       geometry.coordinates.forEach(polygon => {
//         rings = rings.concat(polygon);
//       });
//     }

//     rings.forEach(ring => {
//       // 1. 计算地理边界和特征
//       let minLng = 180, maxLng = -180;
//       let minLat = 90, maxLat = -90;
      
//       ring.forEach(coord => {
//         const [lng, lat] = coord;
//         if (lng < minLng) minLng = lng;
//         if (lng > maxLng) maxLng = lng;
//         if (lat < minLat) minLat = lat;
//         if (lat > maxLat) maxLat = lat;
//       });
//       const pointCount = ring.length;

//       // =========================================================
//       // 第一步：黑名单 (一票否决)
//       // 只要命中这些特征，直接剔除，不管点数多少，不管是不是核心
//       // =========================================================

//       // 【A. 剔除台湾右侧红框】
//       // 台湾本岛最东端大约在 122 度。
//       // 任何完全位于东经 122.5 度以东的孤立碎块，肯定是那个红框。
//       if (minLng > 122.5) return;

//       // 【B. 剔除南海大矩形框】
//       // 特征：这个框纵向跨度极大，从赤道附近(3度)一直延伸到广东附近(20多度)。
//       // 真实的省份（如广东、海南）不会跨越这么大的纬度。
//       // 逻辑：如果它同时涉足了南部深海(Lat < 15) 和 北部海域(Lat > 20)，它就是那个干扰框。
//       if (minLat < 15 && maxLat > 20) return;

//       // 【C. 剔除南海南部碎块 (九段线、小岛礁)】
//       // 海南岛最南端大约在 18 度。
//       // 凡是完全位于北纬 18 度以南的，全部剔除 (清空西沙、南沙)。
//       if (maxLat < 18) return;


//       // =========================================================
//       // 第二步：白名单 (核心保留)
//       // 能走到这里的，已经是通过了黑名单检查的安全板块
//       // =========================================================

//       // 【D. 大陆海岸线/复杂板块】
//       // 只要点数 > 100，说明是精细的陆地 (包括东部海岸线、各省边界)，保留！
//       if (pointCount > 100) {
//         lines.push({ coords: ring });
//         return;
//       }
      
//       // 【E. 台湾岛/海南岛】
//       // 双重保险，防止因为数据简化导致点数不足
//       const isTaiwan = minLng > 119 && maxLng < 122.5 && minLat > 21 && maxLat < 26;
//       const isHainan = minLng > 108 && maxLng < 112 && minLat > 18 && maxLat < 21;
      
//       if (isTaiwan || isHainan) {
//         lines.push({ coords: ring });
//         return;
//       }

//       // =========================================================
//       // 第三步：幸存者筛选
//       // 剩下的都是点数 < 100 的小碎块，且位于北纬18度以北 (广东福建沿海)。
//       // =========================================================
      
//       // 【F. 过滤极小噪点】
//       // 保留沿海小岛 (点数 > 10)，剔除剩下的微小九段线头 (点数 < 10)
//       if (pointCount > 10) {
//         lines.push({ coords: ring });
//       }
//     });
//   });
  
//   return lines;
// }

// src/views/myChart.vue

// 辅助函数：将 GeoJSON 解析为线条，并加入【人工校准】逻辑
const convertGeoJSONToLines = (geoJson) => {
  const lines = [];
  if (!geoJson || !geoJson.features) return lines;

  // ================= 校准参数 (在这里修改) =================
  // 如果线条偏左，就加大经度 (正数向右，负数向左)
  // 如果线条偏上，就减小纬度 (正数向上，负数向下)
  const lngOffset = -0.1;   // <--- 试着改这个 (例如 0.5 或 -0.5)
  const latOffset = 0.1;   // <--- 试着改这个
  const lineAltitude = 1; // <--- 线条悬浮高度 (建议 0.5 ~ 2.0)
  // ========================================================

  geoJson.features.forEach(feature => {
    const geometry = feature.geometry;
    if (!geometry) return;

    let rings = [];
    if (geometry.type === 'Polygon') {
      rings = geometry.coordinates;
    } else if (geometry.type === 'MultiPolygon') {
      geometry.coordinates.forEach(polygon => {
        rings = rings.concat(polygon);
      });
    }

    rings.forEach(ring => {
      // 1. 预处理：应用偏移量并计算新边界
      // 我们把校准后的坐标存到一个新数组里，用来做后续判断和渲染
      const adjustedRing = ring.map(coord => {
        return [
          coord[0] + lngOffset,  // 经度校准
          coord[1] + latOffset,  // 纬度校准
          lineAltitude           // 加上高度，防止被地球地形遮挡
        ];
      });

      // 2. 基于【校准后】的坐标计算特征
      let minLng = 180, maxLng = -180;
      let minLat = 90, maxLat = -90;
      
      adjustedRing.forEach(coord => {
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
      const isTaiwan = minLng > 119 && maxLng < 123 && minLat > 21 && maxLat < 26;
      const isHainan = minLng > 108 && maxLng < 112 && minLat > 18 && maxLat < 21;
      
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
}


const renderGlobe = () => {
  myChart.value = echarts.init(chartRef.value);

  // 添加鼠标交互监听（防止用户拖拽时被代码强制拉回）
  myChart.value.getZr().on('mousedown', () => {
    isUserInteracting = true;
    myChart.value.setOption({ globe: { viewControl: { targetCoord: null, autoRotate: false } } });
  });
  myChart.value.getZr().on('mouseup', () => {
    setTimeout(() => { isUserInteracting = false; }, 1000);
  });

  const option = {
    backgroundColor: 'rgba(0,0,0,0)',
    globe: {
      // 优化配置，增加真实感同时保持稳定
      baseTexture: '/src/assets/img/earth.jpg', // 使用地球纹理
      heightTexture: '/src/assets/img/earth-mouth.jpg', // 恢复高度纹理，增加真实感
      displacementScale: 0.01, // 极低的位移缩放，避免过度变形
      // shading: 'lambert', // 恢复lambert渲染模式，增加真实光照效果
      shading: 'color', // 平面渲染模式，增加简单性
      // baseColor: '#1a5cb0',
      environment: 'none',
      
      top: 'middle', left: 'center', globeRadius: 50,
      // 启用简单的后期特效，增加真实感
      postEffect: {
        enable:false
        // enable: true,
        // bloom: { enable: true, bloomIntensity: 0.05 } // 极低的辉光强度
      },

      light: { 
        main: { 
          intensity: 1.5, 
          shadow: false, 
          alpha: 30, // 调整光源位置
          beta: 40 
        }, 
        ambient: { intensity: 0.5 }, // 合理的环境光
        ambientCubemap: {
          texture: '',
          diffuseIntensity: 0.5,
          specularIntensity: 0.5
        } 
      }, // 优化光照，增加真实感
      
      // === 初始视角配置 ===
      viewControl: {
        // autoRotate: true,
        autoRotateSpeed: -10,
        targetCoord: lastCoords, // 初始锁定
        distance: 300,           // 增加初始距离，确保观察清晰
        minDistance: 60, maxDistance: 301, // 进一步增加最小距离，防止过近观察
        animationDurationUpdate: 1000,
        easing: 'cubicInOut',
        minAlpha: 10,  
        maxAlpha: 50
      }

      // 辉光特效：高科技发光
      // postEffect: {
      //   enable: true,
      //   bloom: { enable: true, bloomIntensity: 0.5 }
      // }
    },
    series: [
      // === 添加散点层用于显示标记点和标签 ===
      {
        type: 'scatter3D', 
        coordinateSystem: 'globe', 
        blendMode: 'source-over', // 改为source-over确保图标不被混合
        symbol: 'pin', 
        symbolSize: 60, // 图标稍微大一点 
        zlevel: 100, // 提高zlevel值，确保图标显示在最顶层
        z: 1000, // 设置z值，确保在3D空间中处于最高位置
        
        itemStyle: { color: '#ffcc00', opacity: 1 }, 
        
        // === 3. 坐标显示 (名称标签配置) === 
        label: { 
          show: true, 
          formatter: '{b}', // 显示数据项的 name (即项目名称) 
          position: 'top', 
          distance: 10,     // 文字离图标的距离 
          textStyle: { 
            color: '#fff', 
            fontSize: 24,       // 字号大一点，清晰 
            fontWeight: 'bold', 
            fontFamily: 'Microsoft YaHei', 
            // 给文字加个半透明黑底，防止被地球纹理干扰，看不清 
            backgroundColor: 'rgba(0,0,0,0.7)', 
            padding: [8, 15],   // 内边距 
            borderRadius: 6,    // 圆角 
            borderColor: '#00ffff', 
            borderWidth: 1 
          } 
        }, 
        
        data: [], // 初始为空，由 updateView 填充 
        animation: false // 禁用内部动画，防止闪烁 
      }
    ]
  };
  myChart.value.setOption(option);
  updateView(props.targetCoords);
}

// const updateView = (coords) => {
//   // 0. 安全检查
//   if (!myChart.value || isUserInteracting) return;

//   const isOverview = !coords; // coords 为 null 则是总览模式
  
//   // ==========================================
//   // 1. 视角控制器配置 (View Control)
//   // ==========================================
//   const targetToUse = isOverview ? lastCoords : coords;
  
//   const viewOption = {
//     autoRotate: isOverview, // 总览自动旋转，详情停止
//     distance: isOverview ? 200 : 120, // 距离：远 vs 近
//     targetCoord: targetToUse,
//     animationDurationUpdate: 1000,
//     easing: 'cubicInOut'
//   };

//   // ==========================================
//   // 2. 基础地图层 (始终存在)
//   // ==========================================
//   const seriesConfig = [
//     {
//       type: 'map3D',
//       coordinateSystem: 'globe',
//       map: 'china',
//       height: 5, 
//       shading: 'color', 
//       itemStyle: {
//         color: 'rgba(0, 100, 200, 0.1)', // 地图颜色
//         borderColor: '#00ffff',          // 边框颜色
//         borderWidth: 3
//       },
//       emphasis: {
//         itemStyle: { color: 'rgba(0, 255, 255, 0.3)', borderColor: '#ffffff', borderWidth: 4 }
//       },
//       label: { show: false }
//     }
//   ];

//   // ==========================================
//   // 3. 模式分叉逻辑 (互斥)
//   // ==========================================
  
//   if (isOverview) {
//     // ##############################################
//     // 模式 A: 总览模式 -> 只显示“呼吸光点”
//     // ##############################################
    
//     if (props.allPoints && props.allPoints.length > 0) {
      
//       // 光点层 1：外围光晕 (大而淡)
//       seriesConfig.push({
//         type: 'scatter3D',
//         coordinateSystem: 'globe',
//         name: 'Overview_Halo', // 给个名字方便调试
//         symbol: 'circle',
//         symbolSize: 25, 
//         blendMode: 'lighter', // 关键：光叠加模式
//         itemStyle: {
//           color: '#00e4ff', 
//           opacity: 0.4 
//         },
//         label: { show: false },
//         silent: true, // 只有总览的光点不需要点击交互
//         data: props.allPoints.map(p => ({
//           name: p.name,
//           value: [p.value[0], p.value[1], 0] // 紧贴地面
//         }))
//       });

//       // 光点层 2：核心高亮 (小而亮)
//       seriesConfig.push({
//         type: 'scatter3D',
//         coordinateSystem: 'globe',
//         name: 'Overview_Core',
//         symbol: 'circle',
//         symbolSize: 10,
//         blendMode: 'lighter',
//         itemStyle: {
//           color: '#ffffff', // 核心用白色更亮
//           opacity: 1
//         },
//         label: { show: false },
//         silent: true,
//         data: props.allPoints.map(p => ({
//           name: p.name,
//           value: [p.value[0], p.value[1], 0]
//         }))
//       });
//     }

//   } else {
//     // ##############################################
//     // 模式 B: 详情模式 -> 只显示“定位图标”
//     // (此时上面的光点代码不会执行，光点会自动消失)
//     // ##############################################
    
//     // 1. 查找目标点数据
//     let targetPoint = null;
//     if (props.allPoints && props.allPoints.length > 0) {
//       targetPoint = props.allPoints.find(p => p.name === props.activeName);
//       // 容错匹配：如果名字对不上，尝试匹配坐标
//       if (!targetPoint) {
//          targetPoint = props.allPoints.find(p => 
//            Math.abs(p.value[0] - coords[0]) < 0.0001 && 
//            Math.abs(p.value[1] - coords[1]) < 0.0001
//          );
//       }
//     }

//     // 2. 构造单个图标的数据
//     const detailData = targetPoint ? [targetPoint] : [{
//       name: props.activeName || '目标区域',
//       value: [coords[0], coords[1], 0.5],
//       itemStyle: { color: '#ffcc00' }
//     }];

//     // 3. 添加图标层
//     seriesConfig.push({
//       type: 'scatter3D',
//       coordinateSystem: 'globe',
//       name: 'Detail_Pin',
//       symbol: 'pin',
//       symbolSize: 60,
//       zlevel: 100,
//       symbolOffset: [0, '-50%'], // 针尖对准地面
      
//       label: {
//         show: true,
//         formatter: '{b}',
//         position: 'top',
//         distance: 10,
//         textStyle: {
//           color: '#fff',
//           fontSize: 24,
//           fontWeight: 'bold',
//           backgroundColor: 'rgba(0,0,0,0.7)',
//           padding: [8, 15],
//           borderRadius: 6,
//           borderColor: '#00ffff',
//           borderWidth: 1
//         }
//       },
//       itemStyle: { color: '#ffcc00', opacity: 1 },
//       data: detailData
//     });
//   }

//   // ==========================================
//   // 4. 应用配置
//   // ==========================================
//   // ECharts 会对比新旧 seriesConfig。
//   // 因为我们在 else 里没有 push 光点，ECharts 会自动移除原来的光点层。
//   myChart.value.setOption({
//     globe: { viewControl: viewOption },
//     series: seriesConfig
//   });
// }

const updateView = (coords) => {
  if (!myChart.value) return;

  // 1. 判定模式 - 安全检查coords是否存在及长度
  const isOverview = !coords || !Array.isArray(coords) || coords.length === 0;
  
  // 2. 视角配置
  const targetToUse = isOverview ? lastCoords : coords;
  const viewOption = {
    autoRotate: isOverview ? true : false,
    autoRotateSpeed: -10,
    distance: isOverview ? 300 : 51, 
    beta: isOverview ? 30 : 65,
    targetCoord: targetToUse,
    animationDurationUpdate: 1000,
    easing: 'cubicInOut'
  };

  // 3. 准备 Globe 皮肤 (完整配置，确保不白球)
  const globeConfig = {
    baseTexture: '/src/assets/img/earth.jpg', 
    heightTexture: '/src/assets/img/earth-mouth.jpg',
    displacementScale: 0.01,
    shading: 'color', 
    environment: 'none', 
    globeRadius: 150,
    top: 'middle', left: 'center',
    light: { 
      main: { intensity: 1.5, shadow: false }, 
      ambient: { intensity: 0.5 }
    },
    viewControl: viewOption
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
  if (isOverview && Array.isArray(props.allPoints) && props.allPoints.length > 0) {
    haloData = props.allPoints.map(p => ({
      name: p.name || '',
      value: Array.isArray(p.value) && p.value.length >= 2 ? [p.value[0], p.value[1], 0] : [0, 0, 0]
    }));
    coreData = props.allPoints.map(p => ({
      name: p.name || '',
      value: Array.isArray(p.value) && p.value.length >= 2 ? [p.value[0], p.value[1], 0] : [0, 0, 0]
    }));
  }

  // B. 准备图标数据 (只在详情模式下有值)
  let pinData = [];
  if (!isOverview && Array.isArray(coords) && coords.length >= 2) {
    let targetPoint = null;
    // 确保props.allPoints是数组且有数据
    if (Array.isArray(props.allPoints) && props.allPoints.length > 0) {
      targetPoint = props.allPoints.find(p => p.name === props.activeName);
      if (!targetPoint) {
         targetPoint = props.allPoints.find(p => 
           Array.isArray(p.value) && p.value.length >= 2 &&
           Math.abs(p.value[0] - coords[0]) < 0.0001 && 
           Math.abs(p.value[1] - coords[1]) < 0.0001
         );
      }
    }
    if (targetPoint) {
      pinData = [targetPoint];
    } else {
      pinData = [{
        name: props.activeName || '目标区域',
        value: [coords[0], coords[1], 0.5],
        itemStyle: { color: '#ffcc00' }
      }];
    }
  }

  // =========================================================
  // 【核心修改】构建 Series 数组
  // 包含所有图层 (Halo + Core + Pin)
  // 通过 data 是否为空来控制显示/隐藏，而不是删除图层
  // =========================================================
  const seriesConfig = [
    {
      type: 'lines3D',
      coordinateSystem: 'globe',
      effect: {
        show: false // 不需要光点流动，只需要常亮
      },
      blendMode: 'lighter',
      
      // 线条样式
      lineStyle: {
        width: 2,           // 线条宽度
        color: '#ffcc00',   // 金色
        opacity: 1
      },
      
      // 关键：把线条数据传进去
      data: chinaLinesData,
      
      // 让线条稍微“浮”在地球表面一点点，防止被山脉遮挡
      // polyline: true 表示数据是多段线
      polyline: true, 
      zlevel: -10
    },
    // 1. 总览光晕层 (Overview_Halo)
    {
      type: 'scatter3D',
      coordinateSystem: 'globe',
      name: 'Overview_Halo',
      symbol: 'circle',
      symbolSize: 15, 
      blendMode: 'lighter', 
      itemStyle: { color: '#00e4ff', opacity: 0.8 },
      label: { show: false },
      silent: true,
      data: haloData // 有数据就显示，没数据([])就自动隐藏
    },

    // 2. 总览核心层 (Overview_Core)
    {
      type: 'scatter3D',
      coordinateSystem: 'globe',
      name: 'Overview_Core',
      symbol: 'circle',
      symbolSize: 10,
      blendMode: 'lighter',
      itemStyle: { color: '#ffffff', opacity: 1 },
      label: { show: false },
      silent: true,
      data: coreData // 有数据就显示，没数据([])就自动隐藏
    },

    // 3. 详情图标层 (Detail_Pin)
    {
      type: 'scatter3D',
      coordinateSystem: 'globe',
      name: 'Detail_Pin',
      symbol: 'pin',
      symbolSize: 60,
      zlevel: 100,
      symbolOffset: [0, '-50%'], 
      label: {
        show: true,
        formatter: '{b}',
        position: 'top',
        distance: 10,
        textStyle: {
          color: '#fff', fontSize: 24, fontWeight: 'bold',
          backgroundColor: 'rgba(0,0,0,0.7)', padding: [8, 15],
          borderRadius: 6, borderColor: '#00ffff', borderWidth: 1
        }
      },
      itemStyle: { color: '#ffcc00', opacity: 1 },
      data: pinData // 有数据就显示，没数据([])就自动隐藏
    }
  ];

  // 5. 应用配置
  // 【关键】：这里不需要 replaceMerge，也不需要 notMerge
  // 因为 series 结构固定不变，只是 data 变了，ECharts 处理这种情况非常稳健
  myChart.value.setOption({
    globe: globeConfig, 
    series: seriesConfig
  });
}

const handleResize = () => {
  myChart.value && myChart.value.resize();
}
</script>

<style scoped>
.chart-container { width: 100%; height: 100%; overflow: hidden; }
</style>
