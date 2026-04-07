<template>
  <div class="main-layout">
    <!-- <div class="background-mask"></div> -->
 
    <!-- <div class="click-zone left-zone" @click="goToForm2">
      <div class="hover-tip">前往 故障记录表</div> 
    </div> -->

    <!-- <div class="click-zone right-zone" @click="goToForm1">
       <div class="hover-tip">前往 检修记录表</div>
    </div> -->

    <div class="side-column left-col">
      <div class="sub-panel overview-panel">
        
        <div class="panel-header-new">
          <div class="title-text-new">
            <span class="label">当前项目：</span>
            <span class="value">{{ isOverview ? "全国总览" : currentData.name }}</span>
          </div>
          <div class="status-dots">
            <span class="dot active"></span>
            <span class="dot"></span>
          </div>
        </div>

        <div class="stat-grid-clean">
          <div class="click-zone left-zone" style="width: 100%" @click="goToForm2">
            <div class="hover-tip">前往 故障记录表</div>
          </div>

          <div class="grid-item-clean">
            <div class="icon-box-clean">
              <svg viewBox="0 0 24 24"><path d="M12 2c-4 0-8 .5-8 4v9.5C4 17.43 5.57 19 7.5 19L6 20.5v.5h12v-.5L16.5 19c1.93 0 3.5-1.57 3.5-3.5V6c0-3.5-4-4-8-4zM7.5 17c-.83 0-1.5-.67-1.5-1.5S6.67 14 7.5 14s1.5.67 1.5 1.5S8.33 17 7.5 17zm3.5-7H6V6h5v4zm2 0V6h5v4h-5zm3.5 7c-.83 0-1.5-.67-1.5-1.5s.67-1.5 1.5-1.5 1.5.67 1.5 1.5-.67 1.5-1.5 1.5z"/></svg>
            </div>
            <div class="data-box-clean">
              <div class="num text-blue tech-font">{{ currentData.total }}</div>
              <div class="txt">列车总数</div>
            </div>
          </div>

          <div class="grid-item-clean">
            <div class="icon-box-clean">
              <svg viewBox="0 0 24 24"><path d="M12 12c2.21 0 4-1.79 4-4s-1.79-4-4-4-4 1.79-4 4 1.79 4 4 4zm0 2c-2.67 0-8 1.34-8 4v2h16v-2c0-2.66-5.33-4-8-4z"/></svg>
            </div>
            <div class="data-box-clean">
              <div class="num tech-font">{{ currentData.online }}</div>
              <div class="txt">人车比</div>
            </div>
          </div>

          <div class="grid-item-clean">
            <div class="icon-box-clean">
              <svg viewBox="0 0 24 24"><path d="M19 3H5c-1.1 0-2 .9-2 2v14c0 1.1.9 2 2 2h14c1.1 0 2-.9 2-2V5c0-1.1-.9-2-2-2zm-2.09 13.5l-1.41 1.41-8.5-8.5 1.41-1.41 8.5 8.5zM8.5 11c-1.38 0-2.5-1.12-2.5-2.5S7.12 6 8.5 6 11 7.12 11 8.5 9.88 11 8.5 11zm7 7c-1.38 0-2.5-1.12-2.5-2.5S14.12 13 15.5 13s2.5 1.12 2.5 2.5-1.12 2.5-2.5 2.5z"/></svg>
            </div>
            <div class="data-box-clean">
              <div class="num tech-font">{{ currentData.person }}</div>
              <div class="txt">人员数量</div>
            </div>
          </div>

          <div class="grid-item-clean">
            <div class="icon-box-clean">
              <svg viewBox="0 0 24 24"><path d="M20 4H4c-1.1 0-1.99.89-1.99 2L2 18c0 1.11.89 2 2 2h16c1.11 0 2-.89 2-2V6c0-1.11-.89-2-2-2zm0 14H4V6h16v12zM6 10h2v2H6zm0 4h8v2H6zm10 0h2v2h-2zm-6-4h8v2h-8z"/></svg>
            </div>
            <div class="data-box-clean">
              <div class="num tech-font">{{ currentData.certify }}</div>
              <div class="txt">持证人数</div>
            </div>
          </div>

        </div>
      </div>

      
      <div class="sub-panel list-panel">
        <div class="panel-title">| 项目列表</div>
        <div class="scroll-container" ref="listContainerRef">
          <div
            class="project-row"
            @click="goToProject(item, index)"
            v-for="(item, index) in projectList"
            :key="index"
            :class="{ 'active-row': currentIndex === index }"
          >
            <span class="p-name">{{ item.name }}</span>

            <div class="glow-effect" v-if="currentIndex === index"></div>
          </div>
        </div>
      </div>
    </div>

    <!-- <div class="center-column" >
      <MyChart 
        :all-points="allPoints"
        :target-coords="currentData.coords" 
        style="flex: 1; width: 100%; min-height: 0;" 
      />
    </div> -->

    <div class="center-column">
      <MyChart
        :all-points="allPoints"
        :target-coords="currentData.coords"
        :active-name="isOverview ? null : currentData.name"
        style="flex: 1; width: 100%; min-height: 0"
      />
    </div>

    <div class="side-column right-col">
      <div class="sub-panel top-panel">
        <div class="panel-title">| 故障监控</div>
        <!-- <div class="fault-summary">
          <div class="f-box">
            <span>今日新增</span>
            <span class="num-warn">{{ currentData.fault.new }}</span>
          </div>
          <div class="f-box">
            <span>今日闭环</span>
            <span class="num-safe">{{ currentData.fault.closed }}</span>
          </div>
        </div> -->
        <div class="chart-box" ref="faultPieRef"></div>
      </div>

      <div class="sub-panel mid-panel">
        <div class="panel-title">| 30天故障趋势</div>
        <div class="chart-box" ref="trendLineRef"></div>
      </div>

      <div class="sub-panel bot-panel">
        <div class="panel-title">| 维保服务评价</div>
        <div class="chart-box" ref="evalRadarRef"></div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted, nextTick, watch } from "vue";
import * as echarts from "echarts";
import MyChart from "./myChart.vue";

// 跳转到检修记录表 (右边)

const goToForm1 = () => {
  // 拼上 #/maintenance 路由，在新标签页打开
  const targetUrl =
    window.location.origin + window.location.pathname + "#/maintenance";
  window.open(targetUrl, "_blank");
};

// 跳转到表单 2 (左边)
const goToForm2 = () => {
  // 获取当前系统的主域名，并拼上 #/fault 路由，在新标签页打开
  const targetUrl =
    window.location.origin + window.location.pathname + "#/fault";
  window.open(targetUrl, "_blank");
};

const projectList = ref([]);

// ==========================================
// 2. 逻辑处理区域
// ==========================================

// 状态管理
const currentIndex = ref(-1); // -1: 总览, 0~N: 具体项目
const isOverview = computed(() => currentIndex.value === -1);
const listContainerRef = ref(null);
let timer = null;



// --- 自动计算“全国总览”数据 ---
const overviewData = computed(() => {
  // 防御性判断
  if (projectList.value.length === 0) return {}

  let totalVehicles = 0
  let totalOnline = 0
  let totalPerson = 0
  let totalCertify = 0
  
  let validOnlineCount = 0 // 用于记录有效的人车比数据条数，用来算平均值

  let faultNew = 0
  let faultClosed = 0
  let faultDistSum = [0,0,0,0,0]
  let trendSum = new Array(30).fill(0)
  let evalSum = [0,0,0,0,0]

  projectList.value.forEach(p => {
    // 🌟 修改：只有当值不是 '--' 时，才进行数字累加
    if (p.total !== '--') totalVehicles += p.total
    if (p.person !== '--') totalPerson += p.person
    if (p.certify !== '--') totalCertify += p.certify 
    
    if (p.online !== '--') {
      totalOnline += p.online
      validOnlineCount++ // 记录有几条有效的人车比数据
    }
    
    faultNew += (p.fault.new || 0)
    faultClosed += (p.fault.closed || 0)
    
    p.fault.distribution.forEach((v, i) => faultDistSum[i] += v)
    p.trend30Days.forEach((v, i) => trendSum[i] += v)
    p.evaluation.forEach((v, i) => evalSum[i] += v)
  })

  const avgEval = evalSum.map(v => Math.round(v / projectList.value.length))

  return {
    name: '全国总览',
    coords: null, 
    opTime: '-',
    total: totalVehicles,
    // 🌟 修改：如果没有有效人车比，直接返回 '--'，否则计算平均
    online: validOnlineCount > 0 ? (totalOnline / validOnlineCount).toFixed(2) : '--',
    person: totalPerson,
    certify: totalCertify,
    fault: { new: faultNew, closed: faultClosed, distribution: faultDistSum },
    trend30Days: trendSum,
    evaluation: avgEval
  }
})

// --- 获取当前展示的数据 ---
const currentData = computed(() => {
  if (projectList.value.length === 0) return {}; // 防御性判断
  if (isOverview.value) return overviewData.value;
  return projectList.value[currentIndex.value];
});

// --- 地图坐标点 (智能分散 + 防闪烁处理) ---
const allPoints = computed(() => {
  const groups = {};

  // 注意：这里改成了 projectList.value.forEach
  projectList.value.forEach((item, index) => {
    if (!item.coords || !Array.isArray(item.coords) || item.coords.length < 2)
      return;
    const key = item.coords.join(",");
    if (!groups[key]) groups[key] = [];
    groups[key].push({ ...item, _originalIndex: index });
  });

  const result = [];
  Object.keys(groups).forEach((key) => {
    const items = groups[key];
    const count = items.length;
    const [centerLng, centerLat] = key.split(",").map(Number);

    items.forEach((item, i) => {
      let finalLng = centerLng;
      let finalLat = centerLat;

      if (count > 1) {
        const angle = (i / count) * Math.PI * 2;
        const radius = 0.15;
        finalLng = centerLng + Math.cos(angle) * radius;
        finalLat = centerLat + Math.sin(angle) * radius;
      }
      const altitude = 5.2 + i * 0.05;
      result.push({
        name: item.name,
        value: [finalLng, finalLat, altitude],
        total: item.total,
        person: item.person,
        online: item.online,
        certify: item.certify,
        _originalIndex: item._originalIndex,
      });
    });
  });
  return result;
});

// --- 辅助颜色函数 ---
const getRateColor = (val) => {
  if (val >= 98) return "c-green";
  if (val >= 90) return "c-blue";
  return "c-yellow";
};

// ==========================================
// 3. ECharts 图表逻辑
// ==========================================
const faultPieRef = ref(null);
const trendLineRef = ref(null);
const evalRadarRef = ref(null);
let pieChart = null,
  lineChart = null,
  radarChart = null;

const updateCharts = () => {
  const data = currentData.value;
  const techFont = "'hst', 'ysbt', 'almm'";

  // 1. 饼图 (故障分布)
  if (pieChart) {
    const colors = ["#3aa1ff", "#fbd437", "#4ecb73", "#975fe5", "#f2637b"];
    // 或者更接近你截图的配色：
    const targetColors = [
      "#4593fc",
      "#fddb5a",
      "#54d8b9",
      "#9be564",
      "#f59056",
    ];
    pieChart.setOption({
      tooltip: { trigger: "item",textStyle : {fontFamily :techFont}  },
      legend: {
        orient: "vertical",
        right: "5%", // 靠右距离
        top: "center", // 垂直居中
        itemGap: 15, // 图例项间距
        textStyle: {
          color: "#dbeeff", // 文字颜色
          fontSize: 12,
          fontFamily: techFont,
        },
        itemWidth: 10, // 图例图标宽度
        itemHeight: 10,
      },
      color: targetColors,
      series: [
        {
          name: "故障分布",
          type: "pie",
          // 调整大小：内径45%，外径65%
          radius: ["45%", "65%"],
          // 关键：将圆心向左移，给右侧图例腾地儿
          center: ["35%", "50%"],
          itemStyle: {
            borderRadius: 0,
            // borderColor: '#0b1628', // 加个深色描边让扇区有间隔感
            // borderWidth: 2
            borderWidth: 0,
            borderColor: "transparent",
          },
          label: { show: false }, // 饼图上不显示连线文字
          labelLine: { show: false },
          data: [
            // 注意：这里的数据依然读取自 data.fault.distribution
            // 如果你想模拟截图的文字（如"XX监控"），可以修改 name 字段
            { value: data.fault.distribution[0], name: "分析系统" },
            { value: data.fault.distribution[1], name: "轨旁系统" },
            { value: data.fault.distribution[2], name: "普查整改" },
            { value: data.fault.distribution[3], name: "人工提报" },
            { value: data.fault.distribution[4], name: "其他" },
          ],
        },
      ],
    });
  }

  // 2. 折线图 (30天趋势)
  if (lineChart) {
    lineChart.setOption({
      tooltip: { trigger: "axis",textStyle : {fontFamily :techFont}  },
      grid: { top: 10, right: 10, bottom: 20, left: 30 },
      xAxis: { type: "category", show: false, boundaryGap: false },
      yAxis: {
        type: "value",
        splitLine: { show: false },
        axisLabel: {
          color: "#b0cbe9",
          
        },
      },
      series: [
        {
          data: data.trend30Days,
          type: "line",
          smooth: true,
          areaStyle: {
            color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
              { offset: 0, color: "rgba(0, 228, 255, 0.5)" },
              { offset: 1, color: "rgba(0, 228, 255, 0)" },
            ]),
          },
          itemStyle: { color: "#00e4ff" },
        },
      ],
    });
  }

  // 3. 雷达图 (评价)
  if (radarChart) {
    radarChart.setOption({
      tooltip: { textStyle : {fontFamily :techFont}  },
      radar: {
        indicator: [
          { name: "安全", max: 100 },
          { name: "质量", max: 100 },
          { name: "生产", max: 100 },
          { name: "技术", max: 100 },
          { name: "资产", max: 100 },
        ],
        center: ['50%', '50%'],
        radius: '65%',
        splitArea: { areaStyle: { color: ['rgba(0,228,255,0.1)', 'rgba(0,228,255,0)'] } },
        axisLine: { lineStyle: { color: 'rgba(255,255,255,0.3)' } },
        splitLine: { lineStyle: { color: 'rgba(255,255,255,0.1)' } },
        name: { textStyle: {fontSize: 15,color: '#ffffff',fontFamily: techFont} }
      },
      series: [
        {
          type: "radar",
          data: [
            {
              value: data.evaluation,
              name: "维保评价",
              itemStyle: { color: "#00ff72" },
              areaStyle: { opacity: 0.3 },
            },
          ],
        },
      ],
    });
  }
};


// 修改后的永无止境轮播逻辑
const startCarousel = () => {
  if (timer) clearInterval(timer);

  timer = setInterval(() => {
    // 1. 获取项目总数
    const totalCount = projectList.value.length;

    // 如果没有数据，则不执行轮播
    if (totalCount === 0) return;

    // 2. 计算下一个索引
    // 逻辑：-1(总览) -> 0 -> 1 -> ... -> N-1 -> 回到 -1
    let next = currentIndex.value + 1;

    if (next >= totalCount) {
      next = -1; // 回到全国总览
    }

    currentIndex.value = next;

    // 3. 列表滚动跟随 (增强健壮性)
    if (listContainerRef.value) {
      if (currentIndex.value === -1) {
        // 回到总览时，滚动条置顶
        listContainerRef.value.scrollTo({ top: 0, behavior: "smooth" });
      } else {
        // 假设每行高度约为 45px (根据 css padding 和 line-height 计算)
        listContainerRef.value.scrollTo({
          top: currentIndex.value * 45,
          behavior: "smooth",
        });
      }
    }
  }, 5000); // 每 5 秒切换一次
};

// 点击跳转到对应轮播项目
const goToProject = (item, index) => {
  currentIndex.value = index;
  if (item.name.indexOf("十一号线") !== -1) {
    window.open(`http://${window.location.hostname}:9810`, "_blank");
  }
};
watch(currentIndex, updateCharts);

onMounted(async () => {
  try {
    const timestamp = new Date().getTime()
    const res = await fetch(`/linedata.json?t=${timestamp}`) 
    
    if (!res.ok) {
      throw new Error('无法读取数据文件')
    }
    
    const rawData = await res.json()

    // 🌟 新增：数据容错校验函数
    const formatData = (val) => {
      if (val === null || val === undefined || val === '') return '--';
      if (isNaN(Number(val))) return '--';
      return Number(val);
    };

    // 2. 格式化数据
    projectList.value = rawData.map(item => {
      return {
        name: item.name || '未命名线路',
        city: item.city || '未知',
        coords: item.coords ? item.coords.split(',').map(Number) : [116.40, 39.90], 
        
        // 🌟 修改：应用校验函数处理核心指标
        total: formatData(item.total),
        person: formatData(item.person),
        certify: formatData(item.certify),
        online: formatData(item.online),
        
        // 解析故障分布
        fault: { 
          distribution: item.fault_dist ? item.fault_dist.split(',').map(Number) : [0, 0, 0, 0, 0] 
        },
        evaluation: item.evaluation ? item.evaluation.split(',').map(Number) : [0, 0, 0, 0, 0],
        trend30Days: item.trend30Days ? item.trend30Days.split(',').map(Number) : new Array(30).fill(0)
      }
    })

    // 3. 数据处理完毕后，再去初始化 ECharts 和轮播
    nextTick(() => {
      pieChart = echarts.init(faultPieRef.value);
      lineChart = echarts.init(trendLineRef.value);
      radarChart = echarts.init(evalRadarRef.value);
      initChartsOptions();
      updateCharts();
      startCarousel();
    });
  } catch (error) {
    console.error("获取或解析大屏数据失败:", error);
    // 这里可以加一个 alert 或者让 projectList 赋一个默认的空假数据以防页面崩溃
  }
});

onUnmounted(() => {
  if (timer) clearInterval(timer);
  pieChart?.dispose();
  lineChart?.dispose();
  radarChart?.dispose();
});

const initChartsOptions = () => {
  // 可以在这里设置一些静态的 Option 基础配置
};
</script>

<style scoped>




.click-zone {
  position: absolute;
  top: 0;
  bottom: 0;
  width: 25%; /* 宽度设为 20%~30%，留出中间给地球交互 */
  z-index: 9999; /* 确保在最上层，能被点到 */
  cursor: pointer; /* 鼠标移上去变小手 */

  /* 开发调试时可以把颜色打开，上线前改成 transparent */
  /* background: rgba(255, 0, 0, 0.2); */
  background: transparent;

  /* 简单的 Flex 布局用于显示提示文字 */
  display: flex;
  align-items: center;
  justify-content: center;
}

/* 左侧定位 */
.left-zone {
  left: 0;
}

/* 右侧定位 */
.right-zone {
  right: 0;
}

/* 鼠标悬停时的提示文字样式 */
.hover-tip {
  color: #fff;
  font-size: 20px;
  font-weight: bold;
  opacity: 0; /* 默认隐藏 */
  transition: opacity 0.3s;
  background: rgba(0, 0, 0, 0.6);
  padding: 10px 20px;
  border-radius: 8px;
  border: 1px solid #00e4ff;
}

/* 鼠标悬停在热区时，显示文字 */
.click-zone:hover .hover-tip {
  opacity: 1;
}
/* 保持原有布局结构 */
.main-layout {
  position: relative;
  width: 100%;
  height: 100%;
  overflow: hidden;
  color: #fff;
  background: transparent;
}
.center-column {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 1;
}

/* .center-column::after {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5); 
  pointer-events: none;
  z-index: 2;
} */

/* 优化后的 .center-column::after */
.center-column::after {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  
  /* 🌟 修改点 1：将 background 从纯黑改为径向渐变 */
  /* 中心透明(让地球亮出来)，四周深蓝黑(让文字清晰) */
  background: radial-gradient(
    circle at center, 
    rgba(5, 15, 30, 0) 0%,      /* 中心完全透明 */
    rgba(5, 15, 30, 0.3) 50%,    /* 中间适度压暗 */
    rgba(1, 4, 10, 0.9) 100%     /* 边缘极暗，产生深空感 */
  );

  /* 🌟 修改点 2：添加离线物理暗纹 (提升工业质感) */
  /* 利用重复渐变产生微弱的横向扫描线，不需要任何外部图片 */
  background-image: 
    radial-gradient(circle at center, rgba(5, 15, 30, 0) 0%, rgba(1, 4, 10, 0.9) 100%),
    repeating-linear-gradient(0deg, transparent, transparent 2px, rgba(255, 255, 255, 0.01) 2px, rgba(255, 255, 255, 0.01) 3px);
  background-blend-mode: overlay;

  pointer-events: none;
  z-index: 2;
}


.side-column {
  position: absolute;
  top: 80px;
  bottom: 20px;
  width: 26%;
  z-index: 10;
  display: flex;
  flex-direction: column;
  gap: 20px; /* 控制上下模块之间的间距，20px 能透出不错的地球背景 */

  /* === 关键：去掉这里的 background、blur 和 border，让容器完全透明 === */
  background: transparent;
  border: none;
  box-shadow: none;
  padding: 0; /* 内边距也去掉，移交给子模块 */

  /* 容器设置为穿透，这样鼠标点击模块空隙时，可以拖拽背后的地球 */
  pointer-events: none;

  /* text-shadow: 0 2px 2px #000000, 0 0 8px rgba(0, 0, 0, 0.8); */
}

/* 使用 :deep 穿透 Vue 的作用域限制 */


.left-col {
  left: 20px;
}
.right-col {
  right: 20px;
}

/* 通用面板样式 */
/* .sub-panel { pointer-events: auto; padding: 10px; display: flex; flex-direction: column; background: transparent; } */
/* 通用面板样式 */


/* 修改 MainView.vue 中的 .sub-panel */
.sub-panel {
  pointer-events: auto;
  display: flex;
  flex-direction: column;
  padding: 15px; 
  
  /* ================= 核心修改区域 ================= */
  
  /* 1. 变成圆角 */
  border-radius: 8px !important;
  
  /* 2. 半透明深色背景 (让底下的地图能透出一点点颜色) */
  background: rgba(10, 25, 45, 0.6) !important; 
  
  /* 3. 核心：毛玻璃滤镜 (模糊背后的地图) */
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px); /* 兼容不同浏览器 */
  
  /* 4. 纤细的发光边框 (代替你原来死板的粗实线) */
  border: 1px solid rgba(0, 228, 255, 0.2) !important;
  
  /* 5. 阴影提升立体感：内侧微亮发光 + 外侧深色投影浮空感 */
  box-shadow: 
    inset 0 0 20px rgba(0, 228, 255, 0.05),
    0 8px 20px rgba(0, 0, 0, 0.6) !important;
    
  /* ============================================== */
}
.panel-title {
  /* 保持你已经配好的工业字体 */
  font-family: 'almm', 'Microsoft YaHei', sans-serif !important;
  font-size: 26px;
  font-weight: 900; /* 调至最粗，增加金属受光面积 */
  margin-bottom: 15px;
  padding-left: 12px;
  letter-spacing: 2px; /* 增加字间距，复刻机甲风的疏朗感 */
  
  /* 1. 核心：金属圆柱体反射渐变 (针对 26px 字号优化) */
  background: linear-gradient(
    to bottom, 
    #ffffff 0%,    /* 顶部高光 */
    #a5c2e3 45%,   /* 上部反光区 */
    /* #4a6583 50%,   中间金属转折暗带 (核心：深浅交界要锐利) */
    #c6e4ff 55%,   /* 下部受光区 */
    #ffffff 100%   /* 底部边缘亮线 */
  );
  
  /* 2. 文字裁剪：将渐变色填充进文字 */
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  color: transparent;

  /* 3. 拒绝漫射模糊，改用锐利的物理重投影 + 微弱的青色冷光 */
  /* drop-shadow 比 text-shadow 在裁剪模式下兼容性更好 */
  /* filter: drop-shadow(0px 2px 2px rgba(0, 0, 0, 0.9)) 
          drop-shadow(0px 0px 4px rgba(0, 228, 255, 0.2)); */

  /* 4. 优化左侧竖线：将其从纯色改为具有发光感的激光条 */
  border-left: 4px solid #00e4ff;
  /* box-shadow: -4px 0 10px rgba(0, 228, 255, 0.4);  */
  
  /* 解决某些浏览器下背景裁剪可能溢出的问题 */
  
}

/* 左侧：核心指标面板 */
.overview-panel {
  flex: 0 0 auto;
}
.stat-row {
  margin-bottom: 10px;
  padding-left: 10px;
  color: #b0cbe9;
  font-size: 14px;
}
.stat-row .value {
  color: #fff;
  font-weight: bold;
  margin-left: 5px;
}

/* ================= 极简统一风格 面板覆盖 ================= */
.overview-panel {
  flex: 0 0 auto;
}

/* ================= 标题行与状态指示灯 ================= */
/* ================= 标题行与状态指示灯 ================= */
.panel-header-new {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px; /* 字体变大后，底边距稍微收紧一点 */
}

/* 🌟 核心修改：完全复刻 .panel-title 的金属质感与左侧蓝条 */
.title-text-new {
  font-family: 'almm', 'Microsoft YaHei', sans-serif !important;
  font-size: 26px; /* 统一字号为 26px */
  font-weight: 900;
  letter-spacing: 2px;
  padding-left: 12px;
  border-left: 4px solid #00e4ff; /* 加上左侧的青蓝色竖线 */
  display: flex;
  align-items: center;

  /* 铬金属质感渐变 */
  background: linear-gradient(
    to bottom, 
    #ffffff 0%,    
    #a5c2e3 45%,   
    #c6e4ff 55%,   
    #ffffff 100%   
  );
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  color: transparent;
}

/* 清除原有 span 的独立颜色，强行继承父级的金属光泽 */
.title-text-new .label,
.title-text-new .value {
  font-size: inherit;
  font-weight: inherit;
}





/* 右上角两个闪烁的小绿点/蓝点 */
.status-dots {
  display: flex;
  gap: 6px;
  align-items: center;
}
.status-dots .dot {
  width: 5px;
  height: 5px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.2);
}
.status-dots .dot.active {
  background: #00e4ff; /* 原图亮起的是微弱的青绿色 */
  box-shadow: 0 0 6px #00ffc6;
  animation: pulse-dot 2s infinite;
}

@keyframes pulse-dot {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.4; }
}

/* ================= 极简无界网格 ================= */
.stat-grid-clean {
  display: grid;
  grid-template-columns: repeat(2, 1fr); /* 完美的均分两列 */
  gap: 20px 10px;                           /* 卡片之间的间距 */
  margin: 15px auto 0;
  width: 100%;
  /* position: relative; */
}

.grid-item-clean {
  display: flex;
  align-items: center;
  justify-content: center; 
  width: 100%;
}

/* 图标的深色底座 */
.icon-box-clean {
  width: 48px;
  height: 48px;
  background: rgba(30, 56, 75, 0.6); /* 截图中的深蓝灰底色 */
  border-radius: 8px;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-shrink: 0;
}

/* 图标统一样式 */
.icon-box-clean svg {
  width: 20px;
  height: 20px;
  fill: #00e4ff; /* 统一的蓝青色，没有任何渐变 */
}

/* 数据文本排版 */
.data-box-clean {
  margin-left: 12px;
  display: flex;
  flex-direction: column;
}

/* 默认数字为白色 */
.data-box-clean .num {
  font-size: 26px;
  font-weight: bold;
  color: #fff;
  line-height: 1;
  margin-bottom: 6px;
  /* 增加数字的锐利钢印投影 */
  text-shadow: 0px 2px 2px rgba(0, 0, 0, 0.8), 0 0 5px rgba(0, 228, 255, 0.3);
}

/* 精准还原截图：列车总数数字为蓝色 */
.data-box-clean .num.text-blue {
  color: #00e4ff; 
  text-shadow: 0px 2px 2px rgba(0, 0, 0, 0.8), 0 0 10px rgba(0, 228, 255, 0.6); 
}

/* 底部的小字标注 */
.data-box-clean .txt {
  font-family: 'almm', 'Microsoft YaHei', sans-serif !important;
  font-size: 13px;
  color: #89a5c3;
  letter-spacing: 1px;
}

/* 颜色类 */
.c-green {
  color: #00ff72 !important;
}
.c-blue {
  color: #00e4ff !important;
}
.c-yellow {
  color: #ffba00 !important;
}

/* 左侧：列表面板 */
/* .list-panel { flex: 1; overflow: hidden; display: flex; flex-direction: column; }
.scroll-container { flex: 1; overflow-y: hidden; display: flex; flex-direction: column; gap: 6px; }
.project-row { padding: 8px 12px; border-radius: 4px; border: 1px solid transparent; transition: all 0.3s; } */
.list-panel {
  font-family: 'almm', 'Microsoft YaHei', sans-serif !important;
  flex: 1;
  /* overflow: hidden;  */
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  /* 给列表加一点顶部内边距 */
  padding-top: 10px;
}

.scroll-container {
  flex: 1;
  overflow-y: auto; /* JS控制滚动 */
  display: flex;
  flex-direction: column;
  gap: 4px; /* 项目之间的间距 */
}

/* 针对 Webkit 浏览器（Chrome、Edge、Safari） */
.scroll-container::-webkit-scrollbar {
  display: none;
}

/* 针对 Firefox */
.scroll-container {
  scrollbar-width: none;
}

/* 针对 IE/Edge */
.scroll-container {
  -ms-overflow-style: none;
}

/* 项目行基础样式 */
.project-row {
  font-size: 17px;
  position: relative;
  padding: 10px 15px; /* 增加高度，看起来像按钮 */
  border-radius: 2px;
  /* background: rgba(255, 255, 255, 0.05); 默认微弱背景 */
  background: transparent;
  cursor: pointer;
  transition: all 0.3s ease;
  border-left: 2px solid transparent; /* 预留左侧边框位置 */
  display: flex;
  align-items: center;
}
.project-row:hover {
  background: linear-gradient(
    90deg,
    rgba(0, 228, 255, 0.3) 0%,
    rgba(0, 228, 255, 0) 100%
  ); /* 青色渐变背景 */
  border-left: 4px solid #00e4ff; /* 左侧亮条 */
  box-shadow: 0 0 10px rgba(0, 228, 255, 0.1);
}

.project-row.active-row {
  background: linear-gradient(
    90deg,
    rgba(0, 228, 255, 0.3) 0%,
    rgba(0, 228, 255, 0) 100%
  ); /* 青色渐变背景 */
  border-left: 4px solid #00e4ff; /* 左侧亮条 */
  box-shadow: 0 0 10px rgba(0, 228, 255, 0.1);
}
.row-info {
  display: flex;
  justify-content: space-between;
  margin-bottom: 5px;
  font-size: 14px;
  color: #dbeeff;
}
/* .active-row .p-name { color: #fff; text-shadow: 0 0 5px #00e4ff; font-weight: bold; } */
.active-row .p-name {
  font-size: 18px;
  color: #fff; /* 选中时文字变白 */
  /* text-shadow: 0 0 5px rgba(0, 228, 255, 0.8); 文字发光 */
  font-weight: bold;
  transform: translateX(5px); /* 选中时文字稍微右移一点，增加动感 */
}
.dot {
  display: inline-block;
  width: 6px;
  height: 6px;
  border-radius: 50%;
  margin-right: 4px;
  vertical-align: middle;
}
.progress-track {
  height: 3px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 2px;
}
.progress-bar {
  height: 100%;
  border-radius: 2px;
  transition: all 0.5s;
}

/* 右侧：图表容器 */
.top-panel,
.mid-panel,
.bot-panel {
  flex: 1;
  min-height: 0;
  display: flex;
  flex-direction: column;
}
.chart-box {
  flex: 1;
  width: 100%;
  min-height: 0;
}

.fault-summary {
  display: flex;
  justify-content: space-around;
  margin-bottom: 5px;
}
.f-box {
  text-align: center;
  font-size: 13px;
  color: #b0cbe9;
}
.num-warn {
  display: block;
  font-size: 20px;
  color: #ffba00;
  font-weight: bold;
  margin-top: 2px;
}
.num-safe {
  display: block;
  font-size: 20px;
  color: #00ff72;
  font-weight: bold;
  margin-top: 2px;
}
.glow-effect {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 1;
  background: linear-gradient(
    90deg,
    transparent,
    rgba(255, 255, 255, 0.1),
    transparent
  );
  animation: slide 2s infinite;
}
</style>