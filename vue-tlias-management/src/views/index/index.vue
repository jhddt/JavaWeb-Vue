<script setup>
import { onMounted, ref } from 'vue'
import * as echarts from 'echarts'
import { queryEmpJobDataApi, queryEmpGenderDataApi } from '@/api/report'
import { queryStudentCountDataApi, queryStudentDegreeDataApi } from '@/api/report'
import { queryPageApi } from '@/api/log'
import { queryAllApi as queryAllClazzApi } from '@/api/clazz'

// 日志数据
let logData = ref([])

// 分页组件
const pagination = ref({currentPage: 1, pageSize: 15, total: 0})

// 钩子函数 - 加载所有统计数据
onMounted(() => {
  // 加载员工统计图表
  loadEmpJobChart()
  loadEmpGenderChart()
  
  // 加载学员统计图表
  loadStudentCountChart()
  loadStudentDegreeChart()
  
  // 加载日志数据
  loadLogData()
})

// 加载员工职位统计图表
const loadEmpJobChart = async () => {
  let result = await queryEmpJobDataApi();
  let jobList = result.data.jobList;
  let dataList = result.data.dataList;
  initEmpJobChart(jobList, dataList)
}

// 加载员工性别统计图表
const loadEmpGenderChart = async () => {
  let result = await queryEmpGenderDataApi();
  initEmpGenderChart(result.data)
}

// 加载班级人数统计图表
const loadStudentCountChart = async () => {
  // 获取所有班级列表
  let allClazzResult = await queryAllClazzApi();
  let allClazzList = [];
  if(allClazzResult.code) {
    allClazzList = allClazzResult.data;
  }
  
  // 获取班级人数统计数据
  let result = await queryStudentCountDataApi();
  let countMap = new Map();
  
  // 创建班级人数映射
  if(result.data && result.data.clazzList && result.data.dataList) {
    for(let i = 0; i < result.data.clazzList.length; i++) {
      countMap.set(result.data.clazzList[i], result.data.dataList[i]);
    }
  }
  
  // 构建完整的班级列表和数据列表
  let clazzList = [];
  let dataList = [];
  
  // 遍历所有班级，确保没有人的班级也显示
  allClazzList.forEach(clazz => {
    clazzList.push(clazz.name);
    // 如果该班级没有人，则显示0
    dataList.push(countMap.get(clazz.name) || 0);
  });
  
  initStudentCountChart(clazzList, dataList)
}

// 加载学员学历统计图表
const loadStudentDegreeChart = async () => {
  let result = await queryStudentDegreeDataApi();
  initStudentDegreeChart(result.data)
}

// 加载日志数据
const loadLogData = async () => {
  const result = await queryPageApi(pagination.value.currentPage, pagination.value.pageSize);
  if(result.code) {
    logData.value = result.data.rows
    pagination.value.total = result.data.total
  }
}

// 处理日志分页
const handleSizeChange = (pageSize) => {
  pagination.value.pageSize = pageSize
  loadLogData()
}

const handleCurrentChange = (page) => {
  pagination.value.currentPage = page
  loadLogData()
}

// 员工职位统计图表初始化
function initEmpJobChart(jobList, dataList) {
  var myChart = echarts.init(document.getElementById('empJobChart'));
  myChart.setOption({
    title: {
      text: '员工职位统计',
      textStyle: { fontSize: 16 }
    },
    tooltip: {},
    xAxis: { data: jobList },
    yAxis: {},
    series: [{
      name: '人数',
      type: 'bar',
      data: dataList,
      itemStyle: {
        color: new echarts.graphic.LinearGradient(0, 0, 1, 1, [
          { offset: 0, color: '#ffbf61' },
          { offset: 1, color: '#dd5f85' }
        ])
      }
    }]
  });
  
  // 响应式调整
  window.addEventListener('resize', () => myChart.resize());
}

// 员工性别统计图表初始化
function initEmpGenderChart(genderDataList) {
  var myChart = echarts.init(document.getElementById('empGenderChart'));
  myChart.setOption({
    title: {
      text: '员工性别统计',
      textStyle: { fontSize: 16 }
    },
    tooltip: { trigger: 'item' },
    series: [{
      name: '性别',
      type: 'pie',
      radius: ['40%', '70%'],
      itemStyle: {
        borderRadius: 5,
        borderColor: '#fff',
        borderWidth: 2
      },
      data: genderDataList
    }]
  });
  
  window.addEventListener('resize', () => myChart.resize());
}

// 班级人数统计图表初始化
function initStudentCountChart(clazzList, dataList) {
  var myChart = echarts.init(document.getElementById('studentCountChart'));
  myChart.setOption({
    title: {
      text: '班级人数统计',
      textStyle: { fontSize: 16 }
    },
    grid: {
      left: '3%',
      right: '4%',
      bottom: '15%',  // 增加底部空间以显示旋转后的标签
      containLabel: true
    },
    tooltip: {},
    xAxis: {
      data: clazzList,
      axisLabel: {
        interval: 0,  // 强制显示所有标签
        rotate: 45,   // 旋转标签45度，避免重叠
        color: '#333', // 设置标签颜色
        fontSize: 12  // 设置标签字体大小
      }
    },
    yAxis: {},
    series: [{
      name: '人数',
      type: 'bar',
      data: dataList,
      itemStyle: {
        color: new echarts.graphic.LinearGradient(0, 0, 1, 1, [
          { offset: 0, color: '#00c6ff' },
          { offset: 1, color: '#0072ff' }
        ])
      }
    }]
  });
  
  window.addEventListener('resize', () => myChart.resize());
}

// 学员学历统计图表初始化
function initStudentDegreeChart(degreeDataList) {
  var myChart = echarts.init(document.getElementById('studentDegreeChart'));
  myChart.setOption({
    title: {
      text: '学员学历统计',
      textStyle: { fontSize: 16 }
    },
    tooltip: { trigger: 'item' },
    series: [{
      name: '学历',
      type: 'pie',
      radius: ['40%', '70%'],
      itemStyle: {
        borderRadius: 5,
        borderColor: '#fff',
        borderWidth: 2
      },
      data: degreeDataList
    }]
  });
  
  window.addEventListener('resize', () => myChart.resize());
}
</script>

<template>
  <div class="dashboard-container">
    <!-- 页面标题 -->
    <div class="page-title">
      <h2>数据统计看板</h2>
    </div>
    
    <!-- 员工信息统计卡片 -->
    <div class="card">
      <div class="card-title">
        <el-icon><UserFilled /></el-icon>
        <span>员工信息统计</span>
      </div>
      <div class="chart-container">
        <div class="chart-item" id="empJobChart"></div>
        <div class="chart-item" id="empGenderChart"></div>
      </div>
    </div>
    
    <!-- 学员信息统计卡片 -->
    <div class="card">
      <div class="card-title">
        <el-icon><Users /></el-icon>
        <span>学员信息统计</span>
      </div>
      <div class="chart-container">
        <div class="chart-item" id="studentCountChart"></div>
        <div class="chart-item" id="studentDegreeChart"></div>
      </div>
    </div>
    
    <!-- 日志信息统计卡片 -->
    <div class="card">
      <div class="card-title">
        <el-icon><Document /></el-icon>
        <span>日志信息统计</span>
      </div>
      <div class="log-container">
        <el-table :data="logData" border style="width: 100%" size="small" :scroll="{x: 'max-content', y: 400}">
          <el-table-column prop="operateEmpName" label="操作人" align="center" width="100"/>
          <el-table-column prop="operateTime" label="操作时间" align="center" width="180"/>
          <el-table-column prop="className" label="类名" align="center"/>
          <el-table-column prop="methodName" label="方法名" align="center" width="120"/>
          <el-table-column prop="methodParams" label="请求参数" align="center" min-width="150">
            <template #default="scope">
              <el-popover effect="light" trigger="hover" placement="top" width="400">
                <template #default>
                  <div style="max-height: 300px; overflow-y: auto; word-break: break-all;">
                    {{ scope.row.methodParams || '无参数' }}
                  </div>
                </template>
                <template #reference>
                  <span v-if="!scope.row.methodParams">无参数</span>
                  <span v-else-if="scope.row.methodParams.length <= 30">{{ scope.row.methodParams }}</span>
                  <span v-else>{{ scope.row.methodParams.substring(0, 30) }}...</span>
                </template>
              </el-popover>
            </template>
          </el-table-column>
          <el-table-column prop="returnValue" label="返回参数" align="center" min-width="150">
            <template #default="scope">
              <el-popover effect="light" trigger="hover" placement="top" width="400">
                <template #default>
                  <div style="max-height: 300px; overflow-y: auto; word-break: break-all;">
                    {{ scope.row.returnValue || '无返回值' }}
                  </div>
                </template>
                <template #reference>
                  <span v-if="!scope.row.returnValue">无返回值</span>
                  <span v-else-if="scope.row.returnValue.length <= 30">{{ scope.row.returnValue }}</span>
                  <span v-else>{{ scope.row.returnValue.substring(0, 30) }}...</span>
                </template>
              </el-popover>
            </template>
          </el-table-column>
        </el-table>
        <el-pagination
          v-model:current-page="pagination.currentPage"
          v-model:page-size="pagination.pageSize"
          :page-sizes="[10, 15, 20, 50]"
          layout="total, sizes, prev, pager, next, jumper"
          :total="pagination.total"
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          class="pagination"
        />
      </div>
    </div>
  </div>
</template>

<style scoped>
.dashboard-container {
  padding: 20px;
  background-color: #f0f2f5;
  min-height: 100vh;
}

.page-title {
  margin-bottom: 20px;
}

.page-title h2 {
  color: #303133;
  font-size: 24px;
  font-weight: 500;
  margin: 0;
}

.card {
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  padding: 20px;
  margin-bottom: 20px;
}

.card-title {
  display: flex;
  align-items: center;
  margin-bottom: 15px;
  font-size: 18px;
  color: #303133;
  font-weight: 500;
}

.card-title span {
  margin-left: 8px;
}

.chart-container {
  display: flex;
  gap: 20px;
  height: 400px;
}

.chart-item {
  flex: 1;
  height: 100%;
}

.log-container {
  width: 100%;
  overflow-x: auto;
  overflow-y: auto;
}

/* 确保表格内容不换行，强制显示横向滚动条 */
.log-container .el-table__body-wrapper {
  overflow-x: auto !important;
}

/* 调整列宽，确保请求参数列有足够空间 */
.log-container .el-table__column--min-width-150 {
  min-width: 150px !important;
  white-space: nowrap;
}

.pagination {
  margin-top: 15px;
  display: flex;
  justify-content: flex-end;
}

/* 响应式布局 */
@media (max-width: 1200px) {
  .chart-container {
    flex-direction: column;
    height: auto;
  }
  
  .chart-item {
    height: 300px;
    margin-bottom: 20px;
  }
}
</style>