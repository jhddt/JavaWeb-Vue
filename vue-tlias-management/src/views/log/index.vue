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
/* 页面整体布局 */
.page-container {
  min-height: 100%;
  background-color: #f5f7fa;
  padding: 20px;
  animation: fadeIn 0.5s ease-in-out;
}

/* 标题样式 */
h1 {
  color: #409eff;
  font-size: 24px;
  font-weight: 600;
  margin-bottom: 24px;
  padding-bottom: 10px;
  border-bottom: 2px solid #e6f7ff;
  position: relative;
}

h1::after {
  content: '';
  position: absolute;
  left: 0;
  bottom: -2px;
  width: 80px;
  height: 2px;
  background: linear-gradient(90deg, #409eff, #67c23a);
}

/* 表格样式 */
.el-table {
  background-color: white;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
}

.el-table thead {
  background-color: #f8fafc;
}

.el-table th {
  font-weight: 600;
  color: #606266;
  background-color: #f8fafc !important;
  border-bottom: 1px solid #e4e7ed !important;
}

.el-table td {
  color: #303133;
  border-bottom: 1px solid #ebeef5 !important;
}

.el-table tr:hover > td {
  background-color: #f5f7fa !important;
}

.el-table__body-wrapper {
  overflow-x: auto !important;
  overflow-y: auto;
}

/* 分页样式 */
.pagination-container {
  background-color: white;
  padding: 12px 20px;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
  display: flex;
  justify-content: flex-end;
}

/* 动画效果 */
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

/* 响应式适配 */
@media (max-width: 1200px) {
  .page-container {
    padding: 16px;
  }
  
  h1 {
    font-size: 22px;
  }
  
  .el-table {
    font-size: 13px;
  }
  
  .el-table__header-wrapper th {
    font-size: 13px;
  }
}

@media (max-width: 768px) {
  .page-container {
    padding: 10px;
  }
  
  h1 {
    font-size: 20px;
  }
  
  .el-table {
    font-size: 12px;
  }
  
  .el-table__body-wrapper {
    overflow-x: auto !important;
  }
}

/* 滚动条美化 */
.el-table__body-wrapper::-webkit-scrollbar {
  width: 6px;
  height: 6px;
}

.el-table__body-wrapper::-webkit-scrollbar-thumb {
  background-color: #dcdfe6;
  border-radius: 3px;
}

.el-table__body-wrapper::-webkit-scrollbar-track {
  background-color: #f0f2f5;
}

/* 标签样式 */
.el-tag {
  border-radius: 4px;
  padding: 2px 8px;
  font-size: 12px;
  transition: all 0.3s ease;
}

/* 弹窗样式 */
.el-popover {
  border-radius: 6px;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.1);
  border: 1px solid #ebeef5;
}
</style>