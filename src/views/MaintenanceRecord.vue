<template>
  <div class="maintenance-record-page">
    <div class="header-bar">
      <h2>检修记录报表</h2>
      <div class="actions">
        <input 
          v-model="searchQuery" 
          type="text" 
          placeholder="搜索车号、修程、班组等..." 
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
// 引入你刚刚准备好的检修记录 JSON 数据
import maintenanceDataJSON from '../assets/data/maintenance_data.json';

const allData = ref(maintenanceDataJSON);
const searchQuery = ref('');
const currentPage = ref(1);
const pageSize = ref(10); 

// 搜索过滤
const filteredData = computed(() => {
  if (!searchQuery.value.trim()) return allData.value;
  const query = searchQuery.value.toLowerCase();
  return allData.value.filter(row => {
    const rowValues = Object.values(row).join(' ').toLowerCase();
    return rowValues.includes(query);
  });
});

// 计算总页数
const totalPages = computed(() => {
  return Math.ceil(filteredData.value.length / pageSize.value) || 1;
});

// 当前页切片
const paginatedData = computed(() => {
  const start = (currentPage.value - 1) * pageSize.value;
  const end = start + pageSize.value;
  return filteredData.value.slice(start, end);
});

watch(searchQuery, () => {
  currentPage.value = 1;
});

const goBack = () => {
  window.close(); 
};
</script>

<style scoped>
/* 包含了完美的滚动条+吸顶方案 */
.maintenance-record-page { 
  padding: 20px; 
  background-color: #f5f7fa; 
  height: 100vh; 
  box-sizing: border-box;
  color: #333; 
  font-family: Arial, sans-serif; 
  display: flex;
  flex-direction: column; 
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
  flex-shrink: 0; 
}

.header-bar h2 { margin: 0; color: #67c23a; /* 给检修表换了个稍微不同的主色调：绿色 */ }
.actions { display: flex; gap: 15px; }
.search-input { padding: 8px 15px; width: 300px; border: 1px solid #dcdfe6; border-radius: 4px; outline: none; transition: border-color 0.2s; }
.search-input:focus { border-color: #67c23a; }
.back-btn { background: #f56c6c; color: white; border: none; padding: 8px 15px; border-radius: 4px; cursor: pointer; }
.back-btn:hover { background: #f78989; }

.table-container { 
  background: #fff; 
  border-radius: 8px; 
  box-shadow: 0 2px 12px 0 rgba(0,0,0,0.1); 
  margin-bottom: 20px; 
  flex: 1; 
  min-height: 0; 
  overflow-x: auto; 
  overflow-y: auto; 
}

table { 
  width: 100%; 
  border-collapse: collapse; 
  white-space: nowrap; 
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
  position: sticky; 
  top: 0; 
  z-index: 10; 
  box-shadow: 0 1px 0 #ebeef5; 
}

tr:hover { background-color: #f5f7fa; }

.empty-state { text-align: center; padding: 50px; color: #909399; }

.pagination { 
  display: flex; 
  align-items: center; 
  justify-content: center; 
  gap: 15px; 
  flex-shrink: 0; 
  padding-bottom: 10px;
}

button { padding: 8px 15px; border: 1px solid #dcdfe6; background: #fff; border-radius: 4px; cursor: pointer; }
button:disabled { background: #f5f7fa; color: #c0c4cc; cursor: not-allowed; }
button:not(:disabled):hover { color: #67c23a; border-color: #a4da89; background-color: #f0f9eb; }
select { padding: 8px; border-radius: 4px; border: 1px solid #dcdfe6; }
</style>