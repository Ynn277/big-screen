<template>
  <div class="main-layout">
    <!-- <div class="click-zone left-zone" @click="goToForm2">
      <div class="hover-tip">前往 故障记录表</div>
    </div> -->

    <div class="click-zone right-zone" @click="goToForm1">
      <div class="hover-tip">前往 检修记录表</div>
    </div>

    <div class="side-column left-col">
      <div class="sub-panel overview-panel">
        <div class="panel-title">
          | {{ isOverview ? "全国总览" : currentData.name }}
        </div>

        <!-- <div class="stat-row" v-if="!isOverview">
          <span class="label">运营时间：</span>
          <span class="value text-small">{{ currentData.opTime }}</span>
        </div> -->

        <div class="stat-grid">
          <div
            class="click-zone left-zone"
            style="width: 100%"
            @click="goToForm2"
          >
            <div class="hover-tip">前往 故障记录表</div>
          </div>
          <div class="grid-item">
            <div class="num highlight">{{ currentData.total }}</div>
            <div class="txt">列车总数</div>
          </div>
          <div class="grid-item">
            <div class="num">{{ currentData.online }}</div>
            <div class="txt">人车比</div>
          </div>
          <div class="grid-item">
            <div class="num">{{ currentData.person }}</div>
            <div class="txt">人员数量</div>
          </div>
          <div class="grid-item">
            <div class="num">{{ currentData.certify }}</div>
            <div class="txt">持证人数</div>
          </div>
        </div>
      </div>

      <!-- <div class="sub-panel list-panel">
        <div class="panel-title">| 项目列表</div>
        <div class="scroll-container" ref="listContainerRef">
          <div 
            class="project-row" 
            v-for="(item, index) in projectList" 
            :key="index"
            :class="{ 'active-row': currentIndex === index }"
          >
            <div class="row-info">
              <span class="p-name">{{ item.name }}</span>
              <span class="p-status">
                <span class="dot" :style="{background: item.rate > 90 ? '#00ff72' : '#ffba00'}"></span>
                {{ item.rate }}%
              </span>
            </div>
            <div class="progress-track">
              <div class="progress-bar" 
                   :style="{ 
                     width: (item.online / item.total * 100) + '%',
                     background: currentIndex === index ? 'linear-gradient(90deg, #005bea, #00c6fb)' : 'rgba(255,255,255,0.2)'
                   }">
              </div>
            </div>
          </div>
        </div>
      </div> -->
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

// //跳转
// const goToForm1 = () => {
//   // 使用 window.open 在新标签页打开，以免覆盖当前大屏
//   // 你的目标地址
//   window.open('https://frp-way.com:64662/online/cgformList/402809819bd4f946019bda5a76dd0001?pure=true', '_blank');
//   // window.open('http://localhost:3100/online/cgformList/402809819bd4f946019bda5a76dd0001?pure=true', '_blank');
// }

// // 跳转到表单 2 (左边)
// const goToForm2 = () => {
//   // 这里填入你表单2的地址，我先用百度代替演示
//   window.open('https://frp-way.com:64662/online/cgformList/402809819bd4f946019bdf39cad50004?pure=true', '_blank');
//   // window.open('http://localhost:3100/online/cgformList/402809819bd4f946019bdf39cad50004?pure=true', '_blank');
// }
// ==========================================
// 1. 用户配置区域 (在此处添加/修改线路数据)
// ==========================================
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
  // 防御性判断：如果没有数据，返回空结构
  if (projectList.value.length === 0) return {};

  let totalVehicles = 0;
  let totalOnline = 0;
  let totalPerson = 0;
  let totalCertify = 0;
  let faultNew = 0;
  let faultClosed = 0;
  let faultDistSum = [0, 0, 0, 0, 0];
  let trendSum = new Array(30).fill(0);
  let evalSum = [0, 0, 0, 0, 0];

  // 注意：这里改成了 projectList.value.forEach
  projectList.value.forEach((p) => {
    totalVehicles += p.total;
    totalOnline += p.online || 0;
    totalPerson += p.person;
    totalCertify += p.certify;

    // 如果你有 new 和 closed 字段可以在这里加，目前先给默认值0
    faultNew += p.fault.new || 0;
    faultClosed += p.fault.closed || 0;

    // 数组累加
    p.fault.distribution.forEach((v, i) => (faultDistSum[i] += v));
    p.trend30Days.forEach((v, i) => (trendSum[i] += v));
    p.evaluation.forEach((v, i) => (evalSum[i] += v));
  });

  // 评价取平均分
  const avgEval = evalSum.map((v) => Math.round(v / projectList.value.length));

  return {
    name: "全国总览",
    coords: null,
    opTime: "-",
    total: totalVehicles,
    online: (totalOnline / projectList.value.length).toFixed(2),
    person: totalPerson,
    certify: totalCertify,
    fault: { new: faultNew, closed: faultClosed, distribution: faultDistSum },
    trend30Days: trendSum,
    evaluation: avgEval,
  };
});

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
      tooltip: { trigger: "item" },
      legend: {
        orient: "vertical",
        right: "5%", // 靠右距离
        top: "center", // 垂直居中
        itemGap: 15, // 图例项间距
        textStyle: {
          color: "#dbeeff", // 文字颜色
          fontSize: 12,
          textBorderColor: "#000000", // 文字描边颜色
          textBorderWidth: 2, // 描边宽度 (加粗效果)

          textShadowColor: "#000000", // 阴影颜色
          textShadowBlur: 5, // 阴影模糊
          textShadowOffsetY: 2,
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
      tooltip: { trigger: "axis" },
      grid: { top: 10, right: 10, bottom: 20, left: 30 },
      xAxis: { type: "category", show: false, boundaryGap: false },
      yAxis: {
        type: "value",
        splitLine: { show: false },
        axisLabel: {
          color: "#b0cbe9",
          textBorderColor: "#000000",
          textBorderWidth: 2,
          textShadowColor: "#000000",
          textShadowBlur: 3,
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
      tooltip: {},
      radar: {
        indicator: [
          { name: "安全", max: 100 },
          { name: "质量", max: 100 },
          { name: "生产", max: 100 },
          { name: "技术", max: 100 },
          { name: "资产", max: 100 },
        ],
        center: ["50%", "50%"],
        radius: "65%",
        splitArea: {
          areaStyle: { color: ["rgba(0,228,255,0.1)", "rgba(0,228,255,0)"] },
        },
        axisLine: { lineStyle: { color: "rgba(255,255,255,0.3)" } },
        splitLine: { lineStyle: { color: "rgba(255,255,255,0.1)" } },
        name: {
          textStyle: {
            color: "#b0cbe9",
            textBorderColor: "#000000",
            textBorderWidth: 2,
            textShadowColor: "#000000",
            textShadowBlur: 5,
          },
        },
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

// 轮播逻辑
// const startCarousel = () => {
//   if (timer) clearInterval(timer)
//   timer = setInterval(() => {
//     let next = currentIndex.value + 1
//     if (next >= projectList.length) next = -1
//     currentIndex.value = next

//     // 列表滚动跟随
//     if (currentIndex.value >= 0 && listContainerRef.value) {
//       listContainerRef.value.scrollTo({
//         top: currentIndex.value * 60, // 假设行高60
//         behavior: 'smooth'
//       })
//     }
//   }, 5000)
// }

//
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
    // 1. 拉取 JSON 文件 (加时间戳防止浏览器缓存老数据)
    const timestamp = new Date().getTime();
    // 请确保线上环境或 public 目录下有这个 data.json 文件
    const res = await fetch(`/linedata.json?t=${timestamp}`);

    if (!res.ok) {
      throw new Error("无法读取数据文件");
    }

    const rawData = await res.json();

    // 2. 格式化数据：将在线工具转出来的字符串，切割成图表需要的数组和数字
    projectList.value = rawData.map((item) => {
      return {
        name: item.name || "未命名线路",
        city: item.city || "未知",
        // 解析坐标，将 "113.36,22.92" 转成 [113.36, 22.92]
        coords: item.coords
          ? item.coords.split(",").map(Number)
          : [116.4, 39.9],
        total: Number(item.total) || 0,
        person: Number(item.person) || 0,
        certify: Number(item.certify) || 0,
        online: Number(item.online) || 0,
        // 解析故障分布，将 "1511,41,97,7192,4688" 转成数组
        fault: {
          distribution: item.fault_dist
            ? item.fault_dist.split(",").map(Number)
            : [0, 0, 0, 0, 0],
        },
        // 解析雷达图评价
        evaluation: item.evaluation
          ? item.evaluation.split(",").map(Number)
          : [0, 0, 0, 0, 0],
        // 如果没有写30天趋势，默认塞30个0进去，防止报错
        trend30Days: item.trend30Days
          ? item.trend30Days.split(",").map(Number)
          : new Array(30).fill(0),
      };
    });

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

  text-shadow: 0 2px 2px #000000, 0 0 8px rgba(0, 0, 0, 0.8);
}

.left-col {
  left: 20px;
}
.right-col {
  right: 20px;
}

/* 通用面板样式 */
/* .sub-panel { pointer-events: auto; padding: 10px; display: flex; flex-direction: column; background: transparent; } */
/* 通用面板样式 */
.sub-panel {
  /* 模块自身恢复鼠标交互，保证图表 tooltip 和列表滚动正常 */
  pointer-events: auto;
  display: flex;
  flex-direction: column;

  /* === 关键：把之前的清透蒙版样式加到这里 === */
  background: rgba(200, 200, 200, 0.08); /* 极度透明的浅灰白 */
  backdrop-filter: blur(3px); /* 毛玻璃模糊 */
  -webkit-backdrop-filter: blur(3px);

  border: 1px solid rgba(0, 228, 255, 0.3); /* 青色科技边框 */
  box-shadow: inset 0 0 15px rgba(0, 228, 255, 0.1),
    0 5px 15px rgba(0, 0, 0, 0.4);
  border-radius: 8px; /* 给每个小模块切出圆角 */
  padding: 15px; /* 保证里面的文字和图表不会贴着边框 */
}
.panel-title {
  color: #00e4ff;
  font-size: 26px;
  font-weight: bold;
  margin-bottom: 12px;
  padding-left: 10px;
  border-left: 4px solid #00e4ff;
  /* text-shadow: 0 0 10px rgba(0, 228, 255, 0.8);  */
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

.stat-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
  padding: 0 5px;
}
/* .grid-item { background: rgba(0, 91, 234, 0.1); border-radius: 4px; padding: 10px; text-align: center; border: 1px solid rgba(0, 91, 234, 0.3); } */
.grid-item {
  /* === 修改开始 === */
  /* 原代码: background: rgba(0, 91, 234, 0.1); */
  background: transparent; /* 改为透明 */

  border-radius: 4px;
  padding: 10px;
  text-align: center;

  /* 原代码: border: 1px solid rgba(0, 91, 234, 0.3); */
  border: none; /* 去掉边框 */
  /* === 修改结束 === */
}
.grid-item .num {
  font-size: 28px;
  font-weight: bold;
  font-family: "Arial";
  color: #fff;
  text-shadow: 0 2px 2px #000000, 0 0 8px rgba(0, 0, 0, 0.8);
}
.grid-item .num.highlight {
  color: #00e4ff;
}
.grid-item .txt {
  font-size: 16px;
  color: #b0cbe9;
  margin-top: 4px;
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
/* .project-row.active-row { background: rgba(0, 228, 255, 0.15); border-color: rgba(0, 228, 255, 0.5); box-shadow: inset 0 0 10px rgba(0,228,255,0.2); } */
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