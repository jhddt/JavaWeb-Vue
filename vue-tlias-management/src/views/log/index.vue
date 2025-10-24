<script setup>
import { onMounted, ref } from 'vue'
import { queryPageApi} from '@/api/log'

//列表展示数据
let tableData = ref([])

//钩子函数 - 页面加载时触发
onMounted(() => {
  queryPage()
})

//分页组件
const pagination = ref({currentPage: 1, pageSize: 15, total: 0})
//每页展示记录数发生变化时触发
const handleSizeChange = (pageSize) => {
  pagination.value.pageSize = pageSize
  queryPage()
}
//当前页码发生变化时触发
const handleCurrentChange = (page) => {
  pagination.value.currentPage = page
  queryPage()
}

//分页条件查询
const queryPage = async () => {
  const result = await queryPageApi(pagination.value.currentPage,pagination.value.pageSize);

  if(result.code) {
    tableData.value = result.data.rows
    pagination.value.total = result.data.total
  }
}
</script>

<template>
  <div class="page-container">
    <!-- 顶部标题 -->
    <div>
      <h1>日志管理</h1>
      <br>
    </div>
    
    <!-- 列表展示 -->
    <el-table :data="tableData" border style="width: 100%" fit size="small">
      <el-table-column prop="operateEmpName" label="操作人" align="center" width="80px"/>
      <el-table-column prop="operateTime" label="操作时间" align="center" width="150px"/>
      <el-table-column prop="className" label="类名" align="center" width="300px" />
      <el-table-column prop="methodName" label="方法名" align="center" width="100px" />
      <el-table-column prop="costTime"  label="操作耗时(ms)" align="center" width="100px"/>
      <el-table-column prop="methodParams" label="请求参数" align="center" width="280px">
        <template #default="scope">
          <el-popover effect="light" trigger="hover" placement="top" width="auto" popper-style="font-size:12px">
            <template #default>
              <div>参数: {{ scope.row.methodParams }}</div>
            </template>
            <template #reference>
              <el-tag v-if="scope.row.methodParams.length <= 30">{{ scope.row.methodParams}}</el-tag>
              <el-tag v-else>{{ scope.row.methodParams.substring(0,30) + '...' }}</el-tag>
            </template>
          </el-popover>
        </template>
      </el-table-column>
      <el-table-column prop="returnValue"  label="返回值" align="center"></el-table-column>
    </el-table>
    <br>
    
    <!-- 分页组件Pagination -->
    <div class="pagination-container">
      <el-pagination
        v-model:current-page="pagination.currentPage"
        v-model:page-size="pagination.pageSize"
        :page-sizes="[10, 20, 50, 100]"
        layout="total, sizes, prev, pager, next, jumper"
        :total="pagination.total"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
      />
    </div>
  </div>
</template>


<style scoped>
/* iOS 26 风格页面整体布局 - 增强版 */
.page-container {
  min-height: 100%;
  background-color: #f5f5f7;
  padding: 24px;
  animation: fadeIn 0.5s ease-in-out;
  position: relative;
  overflow: hidden;
}

/* 背景图案 - 微妙的装饰效果 */
.page-container::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-image: radial-gradient(circle at 20% 20%, rgba(0, 122, 255, 0.03) 0%, transparent 50%),
                    radial-gradient(circle at 80% 80%, rgba(52, 199, 89, 0.03) 0%, transparent 50%);
  pointer-events: none;
}

/* iOS 26 风格标题样式 - 增强版 */
h1 {
  color: #1c1c1e;
  font-size: 28px;
  font-weight: 600;
  margin-bottom: 24px;
  padding-bottom: 12px;
  border-bottom: 1px solid #e5e5ea;
  position: relative;
}

h1::after {
  content: '';
  position: absolute;
  bottom: -1px;
  left: 0;
  width: 60px;
  height: 3px;
  background-color: #007aff;
  border-radius: 1.5px;
}

/* iOS 26 风格表格样式 - 增强版 */
.el-table {
  background-color: white;
  border-radius: 16px;
  overflow: hidden;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.05);
  border: 1px solid #e5e5ea;
  transition: box-shadow 0.3s ease;
}

.el-table:hover {
  box-shadow: 0 6px 24px rgba(0, 0, 0, 0.08);
}

.el-table thead {
  background-color: #f2f2f7;
}

.el-table th {
  font-weight: 600;
  color: #8e8e93;
  background-color: #f2f2f7 !important;
  border-bottom: 1px solid #e5e5ea !important;
  height: 52px;
  font-size: 14px;
  padding: 0 16px;
}

.el-table td {
  color: #1c1c1e;
  border-bottom: 1px solid #f2f2f7 !important;
  height: 56px;
  padding: 0 16px;
  font-size: 14px;
  transition: all 0.3s ease;
}

.el-table tr:hover > td {
  background-color: #f9f9fc !important;
}

.el-table__body-wrapper {
  overflow-x: auto !important;
  overflow-y: auto;
}

/* 表格内容居中对齐 */
.el-table .cell {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
  word-break: break-word;
}

/* iOS 26 风格分页样式 - 增强版 */
.pagination-container {
  margin-top: 20px;
  text-align: right;
  padding: 16px 20px;
  background: white;
  border-radius: 16px;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.05);
  border: 1px solid #e5e5ea;
}

.el-pagination.is-background .el-pager li {
  border-radius: 8px;
  transition: all 0.3s ease;
}

.el-pagination.is-background .el-pager li:not(.disabled):hover {
  color: #007aff;
  background-color: #f2f2f7;
  transform: translateY(-1px);
}

.el-pagination.is-background .el-pager li.is-active {
  background-color: #007aff;
  color: white;
  transform: translateY(-1px);
  box-shadow: 0 2px 8px rgba(0, 122, 255, 0.3);
}

/* iOS 26 风格动画效果 */
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* 响应式适配 - 桌面端优先 */
@media (max-width: 1200px) {
  .page-container {
    padding: 20px;
  }
  
  h1 {
    font-size: 24px;
  }
  
  .el-table th, .el-table td {
    font-size: 13px;
  }
}

/* 简化移动端适配 */
@media (max-width: 768px) {
  .page-container {
    padding: 16px;
  }
  
  h1 {
    font-size: 22px;
    margin-bottom: 16px;
  }
  
  .el-table {
    font-size: 12px;
    border-radius: 12px;
  }
  
  .el-table__body-wrapper {
    overflow-x: auto !important;
  }
  
  .pagination-container {
    padding: 12px 16px;
    border-radius: 12px;
  }
}

/* 优化滚动条样式 */
.el-table__body-wrapper::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}

.el-table__body-wrapper::-webkit-scrollbar-thumb {
  background-color: #c7c7cc;
  border-radius: 4px;
  transition: background-color 0.3s ease;
}

.el-table__body-wrapper::-webkit-scrollbar-thumb:hover {
  background-color: #8e8e93;
}

.el-table__body-wrapper::-webkit-scrollbar-track {
  background-color: #f2f2f7;
  border-radius: 4px;
}

/* iOS 26 风格标签样式 */
.el-tag {
  border-radius: 12px;
  padding: 4px 12px;
  font-size: 12px;
  font-weight: 500;
  transition: all 0.3s ease;
  background-color: rgba(0, 122, 255, 0.1);
  color: #007aff;
  border: none;
}

.el-tag:hover {
  background-color: rgba(0, 122, 255, 0.15);
  transform: translateY(-1px);
}

/* iOS 26 风格弹窗样式 */
.el-popover {
  border-radius: 12px;
  box-shadow: 0 6px 24px rgba(0, 0, 0, 0.1);
  border: 1px solid #e5e5ea;
  background: white;
  font-size: 13px;
}

/* 增强参数显示区域样式 */
.el-popover .el-popover__content {
  padding: 12px 16px;
  max-width: 500px;
  word-break: break-all;
}
</style>