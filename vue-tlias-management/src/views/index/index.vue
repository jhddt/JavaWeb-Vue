<script setup>
import { onMounted, ref, computed } from 'vue'
import * as echarts from 'echarts'
import { queryEmpJobDataApi, queryEmpGenderDataApi } from '@/api/report'
import { queryStudentCountDataApi, queryStudentDegreeDataApi } from '@/api/report'
import { queryPageApi } from '@/api/log'
import { queryAllApi as queryAllClazzApi } from '@/api/clazz'

// 日志数据
let logData = ref([])

// 分页组件
const pagination = ref({currentPage: 1, pageSize: 15, total: 0})

// 关键指标数据
const keyMetrics = ref({
  totalEmployees: 0,
  totalStudents: 0,
  totalClasses: 0,
  activeUsers: 0
})

// 渐变色配置
const gradientColors = {
  primary: ['#007aff', '#00c6ff'],
  secondary: ['#5856d6', '#af52de'],
  success: ['#34c759', '#30b0c7'],
  warning: ['#ff9500', '#ff2d55'],
  info: ['#5ac8fa', '#5856d6']
}

// 格式化数字显示
const formatNumber = (num) => {
  if (num >= 10000) {
    return (num / 10000).toFixed(1) + '万'
  }
  return num.toString()
}

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
    
    // 初始化图表主题
    initEchartsTheme()
  })
  
  // 初始化ECharts主题
  const initEchartsTheme = () => {
    echarts.registerTheme('dashboard', {
      color: ['#007aff', '#5856d6', '#34c759', '#ff9500', '#ff2d55', '#5ac8fa'],
      backgroundColor: 'transparent',
      textStyle: {
        color: '#1c1c1e'
      },
      title: {
        textStyle: {
          color: '#1c1c1e',
          fontWeight: '600'
        }
      },
      grid: {
        left: '3%',
        right: '4%',
        bottom: '3%',
        containLabel: true
      },
      tooltip: {
        backgroundColor: 'rgba(255, 255, 255, 0.9)',
        borderColor: '#e5e5ea',
        borderWidth: 1,
        borderRadius: 8,
        textStyle: {
          color: '#1c1c1e'
        },
        shadowBlur: 8,
        shadowColor: 'rgba(0, 0, 0, 0.1)'
      }
    })
  }

// 加载员工职位统计图表
  const loadEmpJobChart = async () => {
    let result = await queryEmpJobDataApi();
    let jobList = result.data.jobList;
    let dataList = result.data.dataList;
    
    // 计算总员工数
    keyMetrics.value.totalEmployees = dataList.reduce((sum, count) => sum + count, 0)
    
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
    
    // 设置班级总数
    keyMetrics.value.totalClasses = allClazzList.length
    
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
    
    // 计算总学员数
    keyMetrics.value.totalStudents = dataList.reduce((sum, count) => sum + count, 0)
    
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
    var myChart = echarts.init(document.getElementById('empJobChart'), 'dashboard');
    myChart.setOption({
      title: {
        text: '员工职位统计',
        left: 'center',
        textStyle: { fontSize: 16, fontWeight: 600 }
      },
      tooltip: {
        trigger: 'axis',
        axisPointer: {
          type: 'shadow'
        },
        formatter: '{b}: {c} 人'
      },
      grid: {
        left: '5%',
        right: '5%',
        bottom: '15%',
        top: '20%',
        containLabel: true
      },
      xAxis: {
        data: jobList,
        axisLine: { show: true },
        axisTick: { show: false },
        axisLabel: {
          color: '#6e6e73',
          interval: 0,
          rotate: 45,
          fontSize: 12
        }
      },
      yAxis: {
        axisLine: { show: false },
        axisTick: { show: false },
        splitLine: {
          lineStyle: {
            color: '#f2f2f7'
          }
        },
        axisLabel: {
          color: '#6e6e73'
        }
      },
      series: [{
        name: '人数',
        type: 'bar',
        data: dataList,
        barWidth: '60%',
        animationDelay: (idx) => idx * 100,
        itemStyle: {
          color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
            { offset: 0, color: gradientColors.primary[0] },
            { offset: 1, color: gradientColors.primary[1] }
          ]),
          borderRadius: [4, 4, 0, 0]
        },
        emphasis: {
          itemStyle: {
            color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
              { offset: 0, color: gradientColors.secondary[0] },
              { offset: 1, color: gradientColors.secondary[1] }
            ])
          }
        }
      }],
      animationEasing: 'elasticOut',
      animationDelayUpdate: (idx) => idx * 5
    });
    
    // 响应式调整
    window.addEventListener('resize', () => myChart.resize());
  }

// 员工性别统计图表初始化
  function initEmpGenderChart(genderDataList) {
    var myChart = echarts.init(document.getElementById('empGenderChart'), 'dashboard');
    myChart.setOption({
      title: {
        text: '员工性别统计',
        left: 'center',
        textStyle: { fontSize: 16, fontWeight: 600 }
      },
      tooltip: {
        trigger: 'item',
        formatter: '{b}: {c} 人 ({d}%)'
      },
      legend: {
        bottom: '10%',
        left: 'center',
        textStyle: {
          color: '#6e6e73'
        }
      },
      series: [{
        name: '性别',
        type: 'pie',
        radius: ['45%', '75%'],
        center: ['50%', '50%'],
        avoidLabelOverlap: false,
        itemStyle: {
          borderRadius: 8,
          borderColor: '#ffffff',
          borderWidth: 2,
          shadowBlur: 4,
          shadowOffsetX: 0,
          shadowColor: 'rgba(0, 0, 0, 0.1)'
        },
        label: {
          show: false,
          position: 'center'
        },
        emphasis: {
          label: {
            show: true,
            fontSize: 18,
            fontWeight: 'bold',
            formatter: '{b}\n{d}%'
          }
        },
        labelLine: {
          show: false
        },
        data: genderDataList,
        animationType: 'scale',
        animationEasing: 'elasticOut'
      }]
    });
    
    window.addEventListener('resize', () => myChart.resize());
  }

// 班级人数统计图表初始化
  function initStudentCountChart(clazzList, dataList) {
    var myChart = echarts.init(document.getElementById('studentCountChart'), 'dashboard');
    myChart.setOption({
      title: {
        text: '班级人数统计',
        left: 'center',
        textStyle: { fontSize: 16, fontWeight: 600 }
      },
      tooltip: {
        trigger: 'axis',
        axisPointer: {
          type: 'shadow'
        },
        formatter: '{b}: {c} 人'
      },
      grid: {
        left: '5%',
        right: '5%',
        bottom: '15%',
        top: '20%',
        containLabel: true
      },
      xAxis: {
        data: clazzList,
        axisLine: { show: true },
        axisTick: { show: false },
        axisLabel: {
          color: '#6e6e73',
          interval: 0,
          rotate: 45,
          fontSize: 12
        }
      },
      yAxis: {
        axisLine: { show: false },
        axisTick: { show: false },
        splitLine: {
          lineStyle: {
            color: '#f2f2f7'
          }
        },
        axisLabel: {
          color: '#6e6e73'
        }
      },
      series: [{
        name: '人数',
        type: 'bar',
        data: dataList,
        barWidth: '60%',
        animationDelay: (idx) => idx * 100,
        itemStyle: {
          color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
            { offset: 0, color: gradientColors.success[0] },
            { offset: 1, color: gradientColors.success[1] }
          ]),
          borderRadius: [4, 4, 0, 0]
        },
        emphasis: {
          itemStyle: {
            color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
              { offset: 0, color: gradientColors.info[0] },
              { offset: 1, color: gradientColors.info[1] }
            ])
          }
        }
      }],
      animationEasing: 'elasticOut',
      animationDelayUpdate: (idx) => idx * 5
    });
    
    window.addEventListener('resize', () => myChart.resize());
  }

// 学员学历统计图表初始化
  function initStudentDegreeChart(degreeDataList) {
    var myChart = echarts.init(document.getElementById('studentDegreeChart'), 'dashboard');
    myChart.setOption({
      title: {
        text: '学员学历统计',
        left: 'center',
        textStyle: { fontSize: 16, fontWeight: 600 }
      },
      tooltip: {
        trigger: 'item',
        formatter: '{b}: {c} 人 ({d}%)'
      },
      legend: {
        bottom: '10%',
        left: 'center',
        textStyle: {
          color: '#6e6e73'
        }
      },
      series: [{
        name: '学历',
        type: 'pie',
        radius: ['45%', '75%'],
        center: ['50%', '50%'],
        avoidLabelOverlap: false,
        itemStyle: {
          borderRadius: 8,
          borderColor: '#ffffff',
          borderWidth: 2,
          shadowBlur: 4,
          shadowOffsetX: 0,
          shadowColor: 'rgba(0, 0, 0, 0.1)'
        },
        label: {
          show: false,
          position: 'center'
        },
        emphasis: {
          label: {
            show: true,
            fontSize: 18,
            fontWeight: 'bold',
            formatter: '{b}\n{d}%'
          }
        },
        labelLine: {
          show: false
        },
        data: degreeDataList,
        animationType: 'scale',
        animationEasing: 'elasticOut'
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
      <p class="subtitle">实时监控系统核心指标与数据分析</p>
    </div>
    
    <!-- 关键指标卡片 -->
    <div class="metrics-container">
      <div class="metric-card" :class="{ 'primary': true }">
        <div class="metric-icon"><UserFilled /></div>
        <div class="metric-content">
          <div class="metric-title">总员工数</div>
          <div class="metric-value">{{ formatNumber(keyMetrics.totalEmployees) }}</div>
        </div>
      </div>
      <div class="metric-card" :class="{ 'success': true }">
        <div class="metric-icon"><Users /></div>
        <div class="metric-content">
          <div class="metric-title">总学员数</div>
          <div class="metric-value">{{ formatNumber(keyMetrics.totalStudents) }}</div>
        </div>
      </div>
      <div class="metric-card" :class="{ 'info': true }">
        <div class="metric-icon"><School /></div>
        <div class="metric-content">
          <div class="metric-title">班级数量</div>
          <div class="metric-value">{{ formatNumber(keyMetrics.totalClasses) }}</div>
        </div>
      </div>
      <div class="metric-card" :class="{ 'warning': true }">
        <div class="metric-icon"><Bell /></div>
        <div class="metric-content">
          <div class="metric-title">活跃用户</div>
          <div class="metric-value">{{ formatNumber(keyMetrics.activeUsers || 0) }}</div>
        </div>
      </div>
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
        <span>系统操作日志</span>
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
/* 桌面端优化的主容器样式 */
.dashboard-container {
  padding: 24px;
  background-color: #f5f5f7;
  background-image: 
    radial-gradient(circle at 20% 30%, rgba(0, 122, 255, 0.03) 0%, transparent 40%),
    radial-gradient(circle at 80% 70%, rgba(52, 199, 89, 0.03) 0%, transparent 40%);
  min-height: 100vh;
  animation: fadeIn 0.5s ease;
}

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

/* 桌面端优化的页面标题样式 */
.page-title {
  margin-bottom: 32px;
  text-align: center;
  padding-bottom: 16px;
  border-bottom: 1px solid #e5e5ea;
}

.page-title h2 {
  color: #1c1c1e;
  font-size: 32px;
  font-weight: 700;
  margin: 0 0 8px 0;
  letter-spacing: -0.5px;
}

.page-title .subtitle {
  color: #8e8e93;
  font-size: 14px;
  margin: 0;
  font-weight: 400;
}

/* 关键指标卡片容器 */
.metrics-container {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 20px;
  margin-bottom: 24px;
  animation: slideUp 0.4s ease 0.1s both;
}

/* 关键指标卡片样式 */
.metric-card {
  background: white;
  border-radius: 16px;
  padding: 20px;
  display: flex;
  align-items: center;
  gap: 16px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
  transition: all 0.3s cubic-bezier(0.25, 0.1, 0.25, 1);
  position: relative;
  overflow: hidden;
}

.metric-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.08);
}

/* 不同颜色主题的指标卡片 */
.metric-card.primary .metric-icon {
  background: linear-gradient(135deg, #007aff, #00c6ff);
  color: white;
}

.metric-card.success .metric-icon {
  background: linear-gradient(135deg, #34c759, #30b0c7);
  color: white;
}

.metric-card.info .metric-icon {
  background: linear-gradient(135deg, #5856d6, #af52de);
  color: white;
}

.metric-card.warning .metric-icon {
  background: linear-gradient(135deg, #ff9500, #ff2d55);
  color: white;
}

/* 指标卡片图标 */
.metric-icon {
  width: 56px;
  height: 56px;
  border-radius: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 24px;
  flex-shrink: 0;
}

/* 指标卡片内容 */
.metric-content {
  flex: 1;
}

.metric-title {
  font-size: 13px;
  color: #8e8e93;
  font-weight: 500;
  margin-bottom: 4px;
}

.metric-value {
  font-size: 28px;
  color: #1c1c1e;
  font-weight: 700;
  line-height: 1.2;
}

/* 桌面端优化的卡片样式 */
.card {
  background: white;
  border-radius: 16px;
  padding: 24px;
  margin-bottom: 24px;
  transition: all 0.3s cubic-bezier(0.25, 0.1, 0.25, 1);
  position: relative;
  overflow: hidden;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
}

.card:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.08);
}

/* 桌面端优化的卡片标题样式 */
.card-title {
  display: flex;
  align-items: center;
  margin-bottom: 20px;
  font-size: 20px;
  color: #1c1c1e;
  font-weight: 600;
}

.card-title .el-icon {
  font-size: 22px;
  color: #007aff;
  margin-right: 12px;
}

/* 桌面端优化的图表容器样式 */
.chart-container {
  display: flex;
  gap: 20px;
  height: 400px;
  animation: slideUp 0.4s ease 0.1s both;
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* 桌面端优化的图表项样式 */
.chart-item {
  flex: 1;
  height: 100%;
  border-radius: 12px;
  padding: 20px;
  background: #f9f9fa;
  transition: all 0.2s ease;
}

.chart-item:hover {
  background: #fcfcfc;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
}

/* 桌面端优化的日志表格容器样式 */
.log-container {
  width: 100%;
  animation: slideUp 0.4s ease 0.2s both;
}

/* 桌面端优化的表格样式 */
.el-table {
  border-radius: 12px;
  overflow: hidden;
  border: 1px solid #e5e5ea;
  background: white;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
}

.el-table__header-wrapper th {
  background: #f9f9fa !important;
  font-weight: 600 !important;
  color: #1c1c1e !important;
  border-bottom: 1px solid #e5e5ea !important;
  height: 48px;
  font-size: 14px;
}

.el-table__body-wrapper {
  overflow-x: auto;
}

.el-table__row {
  transition: all 0.2s ease;
  height: 44px;
}

.el-table__row:hover > td {
  background: #f9f9fa !important;
}

.el-table__row.current-row > td {
  background: #e9f5ff !important;
}

/* 桌面端优化的分页样式 */
.pagination {
  margin-top: 20px;
  display: flex;
  justify-content: flex-end;
  padding: 12px 16px;
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
  border: 1px solid #e5e5ea;
  background: white;
}

/* 桌面端优化的弹出层样式 */
.el-popover {
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  border: 1px solid #e5e5ea;
  background: white;
}

/* 桌面端响应式设计 */
@media (max-width: 1200px) {
  .metrics-container {
    grid-template-columns: repeat(2, 1fr);
  }
  
  .chart-container {
    flex-direction: column;
    height: auto;
    gap: 16px;
  }
  
  .chart-item {
    height: 350px;
  }
}

/* 移除移动端响应式样式 - 仅保留桌面端优化 */
@media (max-width: 768px) {
  .metrics-container {
    grid-template-columns: 1fr;
    gap: 16px;
  }
  
  .metric-card {
    padding: 16px;
  }
  
  .chart-item {
    height: 300px;
    padding: 16px;
  }
  
  .card {
    padding: 20px;
  }
}

/* 加载状态样式 */
.loading-state {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 250px;
  color: #8e8e93;
}

/* 空状态样式 */
.empty-data {
  text-align: center;
  padding: 60px 20px;
  color: #8e8e93;
}

.empty-data .el-icon {
  font-size: 64px;
  margin-bottom: 16px;
  opacity: 0.4;
}

/* 滚动条美化 */
.log-container::-webkit-scrollbar,
.el-table__body-wrapper::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}

.log-container::-webkit-scrollbar-track,
.el-table__body-wrapper::-webkit-scrollbar-track {
  background: #f2f2f7;
  border-radius: 4px;
}

.log-container::-webkit-scrollbar-thumb,
.el-table__body-wrapper::-webkit-scrollbar-thumb {
  background: #c7c7cc;
  border-radius: 4px;
}

.log-container::-webkit-scrollbar-thumb:hover,
.el-table__body-wrapper::-webkit-scrollbar-thumb:hover {
  background: #a8a8ad;
}
</style>