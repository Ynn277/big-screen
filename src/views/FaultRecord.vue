<template>
  <div class="fault-record-page">
    <div class="header-bar">
      <h2>故障记录报表</h2>
      <div class="actions">
        <input 
          v-model="searchQuery" 
          type="text" 
          placeholder="搜索工单号、部件、报修人员等..." 
          class="search-input"
        />
        <button class="back-btn" @click="goBack">返回大屏首页</button>
      </div>
    </div>

    <div class="table-container">
      <table v-if="paginatedData.length > 0">
        <thead>
          <tr>
            <th v-for="(val, key) in paginatedData[0]" :key="key">{{ key }}</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(row, index) in paginatedData" :key="index">
            <td v-for="(val, key) in row" :key="key">{{ val }}</td>
          </tr>
        </tbody>
      </table>
      <div v-else class="empty-state">
        未搜索到匹配的数据
      </div>
    </div>

    <div class="pagination" v-if="filteredData.length > 0">
      <button :disabled="currentPage === 1" @click="currentPage--">上一页</button>
      <span class="page-info">
        第 {{ currentPage }} 页 / 共 {{ totalPages }} 页 
        (共 {{ filteredData.length }} 条记录)
      </span>
      <button :disabled="currentPage === totalPages" @click="currentPage++">下一页</button>
      
      <select v-model="pageSize" @change="currentPage = 1">
        <option :value="10">10 条/页</option>
        <option :value="20">20 条/页</option>
        <option :value="50">50 条/页</option>
        <option :value="100">100 条/页</option>
      </select>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue';
// 直接引入本地的 JSON 数据文件
import faultDataJSON from '../assets/data/fault_data.json';

// 初始化数据
const allData = ref(faultDataJSON); // 将引入的 JSON 赋值给响应式变量
const searchQuery = ref('');
const currentPage = ref(1);
const pageSize = ref(10); // 默认每页显示10条

// 计算属性：根据搜索词过滤后的数据
const filteredData = computed(() => {
  if (!searchQuery.value.trim()) {
    return allData.value;
  }
  const query = searchQuery.value.toLowerCase();
  return allData.value.filter(row => {
    // 将对象每一行的值拼接成一个大字符串进行模糊搜索
    const rowValues = Object.values(row).join(' ').toLowerCase();
    return rowValues.includes(query);
  });
});

// 计算属性：总页数
const totalPages = computed(() => {
  return Math.ceil(filteredData.value.length / pageSize.value) || 1;
});

// 计算属性：当前页应该切片展示的数据
const paginatedData = computed(() => {
  const start = (currentPage.value - 1) * pageSize.value;
  const end = start + pageSize.value;
  return filteredData.value.slice(start, end);
});

// 当搜索词变化时，自动将页码重置为第一页
watch(searchQuery, () => {
  currentPage.value = 1;
});

// 返回大屏首页逻辑
const goBack = () => {
  window.close(); // 如果是新标签页打开的，直接关闭当前页
  // 如果你想在同一个页面跳转，可以使用： window.location.hash = ''
};
</script>

<style scoped>
.fault-record-page { 
  padding: 20px; 
  background-color: #f5f7fa; 
  height: 100vh; /* 限制整个页面高度就是屏幕高度 */
  box-sizing: border-box;
  color: #333; 
  font-family: Arial, sans-serif; 
  display: flex;
  flex-direction: column; /* 让内部元素竖向排列 */
}

.header-bar { 
  display: flex; 
  justify-content: space-between; 
  align-items: center; 
  margin-bottom: 20px; 
  background: #fff; 
  padding: 15px 20px; 
  border-radius: 8px; 
  box-shadow: 0 2px 12px 0 rgba(0,0,0,0.1); 
  flex-shrink: 0; /* 防止头部被压缩 */
}

.header-bar h2 { margin: 0; color: #409eff; }
.actions { display: flex; gap: 15px; }
.search-input { padding: 8px 15px; width: 300px; border: 1px solid #dcdfe6; border-radius: 4px; outline: none; transition: border-color 0.2s; }
.search-input:focus { border-color: #409eff; }
.back-btn { background: #f56c6c; color: white; border: none; padding: 8px 15px; border-radius: 4px; cursor: pointer; }
.back-btn:hover { background: #f78989; }

/* ================= 核心滚动区域修改 ================= */
.table-container { 
  background: #fff; 
  border-radius: 8px; 
  box-shadow: 0 2px 12px 0 rgba(0,0,0,0.1); 
  margin-bottom: 20px; 
  
  /* 关键：让表格容器占据剩下的所有空间，并出现上下左右滚动条 */
  flex: 1; 
  min-height: 0; /* 必须加这个，防止 flex 子项溢出 */
  overflow-x: auto; /* 左右滚动条 */
  overflow-y: auto; /* 上下滚动条 */
}

table { 
  width: 100%; 
  border-collapse: collapse; 
  white-space: nowrap; /* 强制不换行，撑开左右滚动 */
}

th, td { 
  padding: 12px 15px; 
  border-bottom: 1px solid #ebeef5; 
  text-align: left; 
  font-size: 14px; 
}

th { 
  background-color: #f5f7fa; 
  color: #909399; 
  font-weight: bold; 
  
  /* 关键：表头吸顶效果 */
  position: sticky; 
  top: 0; 
  z-index: 10; 
  /* 吸顶时为了防止穿透，加上底边框或下阴影 */
  box-shadow: 0 1px 0 #ebeef5; 
}

tr:hover { background-color: #f5f7fa; }

.empty-state { text-align: center; padding: 50px; color: #909399; }

/* 分页区域 */
.pagination { 
  display: flex; 
  align-items: center; 
  justify-content: center; 
  gap: 15px; 
  flex-shrink: 0; /* 防止分页被压缩 */
  padding-bottom: 10px;
}

button { padding: 8px 15px; border: 1px solid #dcdfe6; background: #fff; border-radius: 4px; cursor: pointer; }
button:disabled { background: #f5f7fa; color: #c0c4cc; cursor: not-allowed; }
button:not(:disabled):hover { color: #409eff; border-color: #c6e2ff; background-color: #ecf5ff; }
select { padding: 8px; border-radius: 4px; border: 1px solid #dcdfe6; }
</style>