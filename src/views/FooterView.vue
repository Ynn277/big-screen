<template>
  <div class="footer-chart-wrapper">
    <!-- <div class="chart-title-bar">
       <span class="title-text">| 实时数据趋势</span>
    </div> -->
    <div ref="trendChart" class="chart-body"></div>
  </div>
</template>

<script>
import * as echarts from 'echarts';

export default {
  name: 'FooterView',
  data() {
    return { chartInstance: null };
  },
  mounted() {
    this.$nextTick(() => {
      this.initChart();
      window.addEventListener('resize', this.resizeChart);
    });
  },
  beforeUnmount() { // Vue3 写法，如果是 Vue2 用 beforeDestroy
    window.removeEventListener('resize', this.resizeChart);
    if (this.chartInstance) {
      this.chartInstance.dispose();
    }
  },
  methods: {
    initChart() {
      if (!this.$refs.trendChart) return;
      this.chartInstance = echarts.init(this.$refs.trendChart);
      
      const option = {
        backgroundColor: 'rgba(0,0,0,0)', // 透明背景
        // grid 决定了图表绘图区离边框的距离，数值太大会导致图表画不出来
        grid: { 
            top: '30px',    
            bottom: '10px',  // 底部留少一点，防止切断
            left: '40px',
            right: '20px',
            containLabel: true // 防止坐标轴标签被切断
        },
        xAxis: { 
            type: 'category', 
            data: ['10:00','11:00','12:00','13:00','14:00'], 
            axisLabel: { color: '#fff', fontSize: 10 },
            axisLine: { lineStyle: { color: '#00d8ff' } }
        },
        yAxis: { 
            type: 'value', 
            axisLabel: { color: '#fff', fontSize: 10 },
            splitLine: { show: false }
        },
        series: [{ 
            data: [120, 200, 150, 80, 70], 
            type: 'line', 
            smooth: true,
            areaStyle: { opacity: 0.3, color: '#00d8ff' },
            lineStyle: { color: '#00d8ff' }
        }]
      };
      
      this.chartInstance.setOption(option);
    },
    resizeChart() {
      if (this.chartInstance) this.chartInstance.resize();
    }
  }
}
</script>

<style scoped>
/* === 关键修改 === */
.footer-chart-wrapper {
  width: 100%;
  /* height: 100%;  <-- 【一定要删掉这一行！】 */
  /* 让 flex-basis (200px) 生效，而不是被 height: 100% 撑破屏幕 */
  
  display: flex;
  flex-direction: column;
  box-sizing: border-box;
  
  /* 加上这个可以保证内容不贴边 */
  padding-top: 10px; 
}

.chart-title-bar {
  flex: 0 0 30px; /* 给标题一个固定高度 */
  display: flex;
  align-items: center;
  padding-left: 10px;
}

.title-text {
  color: #00d8ff;
  font-weight: bold;
  font-size: 16px;
  border-left: 4px solid #00d8ff;
  padding-left: 8px;
  text-shadow: 0 0 5px rgba(0,0,0,0.5); /* 增加文字阴影，防止看不清 */
}

.chart-body {
  flex: 1;        /* 自动占据 200px 中剩下的所有高度 */
  width: 100%;
  min-height: 0;  /* 防止 flex 子项溢出 */
}
</style>