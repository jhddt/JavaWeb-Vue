<script setup>
import { onMounted } from 'vue'
import * as echarts from 'echarts'
import { queryStudentCountDataApi, queryEmpGenderDataApi, queryStudentDegreeDataApi } from '@/api/report'

//钩子函数 - 加载报表
onMounted(() => {
  loadStudentCountChart() //加载班级人数报表
  loadDegreeChart() //加载性别统计报表
})

//获取职位统计报表
const loadStudentCountChart = async () => {
  let result = await queryStudentCountDataApi();
  let clazzList = result.data.clazzList;
  let dataList = result.data.dataList;

  initStudentCountChart(clazzList, dataList)
}

//获取性别统计报表
const loadDegreeChart = async () => {
  let result = await queryStudentDegreeDataApi();
  initDegreeChart(result.data)
}

//班级人数统计报表
function initStudentCountChart(clazzList, dataList) {
  // 基于准备好的dom，初始化echarts实例
  var myChart = echarts.init(document.getElementById('container1'));
  // 绘制图表
  myChart.setOption({
    title: {
      text: '班级人数统计',
      subText: '',
      textStyle: {
        fontSize: 20
      },
      left: 'center'
    },
    grid:{
      left: '3%',
      right: '4%',
      bottom: '3%',
      containLabel:true
    },
    tooltip: {},
    xAxis: {
      data: clazzList
    },
    yAxis: {},
    series: [
      {
        name: '人数',
        type: 'bar',
        data: dataList,
        itemStyle:{
          // 设置柱状渐变色---
          color:new echarts.graphic.LinearGradient(0, 0, 1, 1, [
            {
              offset: 0,
              color: '#ffbf61'
            },
            {
              offset: 1,
              color: '#dd5f85'
            }
          ])
        }
      }
    ]
  });
}

function initDegreeChart(degreeDataList) {
  // 基于准备好的dom，初始化echarts实例
  var myChart = echarts.init(document.getElementById('container2'));
  let option = {
    title: {
      text: '学员学历统计',
      subText: '',
      textStyle: {
        fontSize: 20
      },
      left: 'center'
    },
    grid:{
      left: '3%',
      right: '4%',
      bottom: '3%',
      containLabel:true
    },
    tooltip: {
      trigger: 'item'
    },
    legend: {
      top: '10%',
      left: 'center'
    },
    series: [
      {
        name: '学历',
        type: 'pie',
        radius: ['40%', '70%'],
        avoidLabelOverlap: false,
        top: '5%',
        itemStyle: {
          borderRadius: 5,
          borderColor: '#fff',
          borderWidth: 2
        },
        label: {
          show: false,
          position: 'center'
        },
        emphasis: {
          label: {
            show: true,
            fontSize: 20,
            fontWeight: 'bold'
          }
        },
        data: degreeDataList
      }
    ]
  };
  // 绘制图表
  myChart.setOption(option);
}

</script>

<template>
  <div class="page-container">
    <!-- 顶部标题 -->
    <div>
      <h1>学员数据报表</h1>
    </div>
    
    <!-- 图表容器 -->
    <div class="charts-container">
      <div class="chart-card">
        <div class="chart-title">班级人数统计</div>
        <div class="report_container" id="container1"></div>
      </div>
      
      <div class="chart-card">
        <div class="chart-title">学历分布统计</div>
        <div class="report_container" id="container2"></div>
      </div>
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

/* 图表容器布局 */
.charts-container {
  display: flex;
  justify-content: space-between;
  gap: 20px;
  margin-top: 20px;
}

.chart-card {
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
  padding: 20px;
  flex: 1;
  transition: all 0.3s ease;
}

.chart-card:hover {
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.08);
  transform: translateY(-2px);
}

/* 图表标题样式 */
.chart-title {
  font-size: 16px;
  font-weight: 500;
  color: #303133;
  margin-bottom: 16px;
  padding-bottom: 8px;
  border-bottom: 1px solid #f0f0f0;
}

/* 图表容器样式 */
.report_container {
  width: 100%;
  height: 400px;
  margin-top: 10px;
}

/* 移除原有的边框 */
#container1 {
  border-right: none;
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
  
  .charts-container {
    gap: 16px;
  }
  
  .chart-card {
    padding: 16px;
  }
  
  .report_container {
    height: 350px;
  }
}

@media (max-width: 768px) {
  .page-container {
    padding: 10px;
  }
  
  h1 {
    font-size: 20px;
  }
  
  .charts-container {
    flex-direction: column;
    gap: 12px;
  }
  
  .chart-card {
    padding: 12px;
  }
  
  .chart-title {
    font-size: 14px;
  }
  
  .report_container {
    height: 300px;
  }
}

/* 滚动条美化 */
.charts-container::-webkit-scrollbar {
  width: 6px;
}

.charts-container::-webkit-scrollbar-thumb {
  background-color: #dcdfe6;
  border-radius: 3px;
}

.charts-container::-webkit-scrollbar-track {
  background-color: #f0f2f5;
}
</style>