<script setup>
import { onMounted, ref, watch } from 'vue'
import { queryPageApi , addApi, queryInfoApi, updateApi, deleteApi} from '@/api/clazz'
import { queryAllApi as queryAllEmpApi } from '@/api/emp'
import { ElMessage, ElMessageBox } from 'element-plus'

//学科列表数据
const subjects = ref([{ name: 'Java', value: 1 },{ name: '前端', value: 2 },{ name: '大数据', value: 3 },{ name: 'Python', value: 4 },{ name: 'Go', value: 5 },{ name: '嵌入式', value: 6 }])
//搜索表单对象
let searchClazz = ref({begin:'', end:'', date:[], name:''})
//列表展示数据
let tableData = ref([])

//钩子函数 - 页面加载时触发
onMounted(() => {
  queryPage()
  queryAllEmp()
})

//所有的员工数据
let emps = ref([])
//加载所有的员工数据
const queryAllEmp = async () => {
  let result = await queryAllEmpApi()
  if(result.code) {
    emps.value = result.data
  }
}

//分页组件
const pagination = ref({currentPage: 1, pageSize: 10, total: 0})
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
  const result = await queryPageApi(
      searchClazz.value.begin,
      searchClazz.value.end,
      searchClazz.value.name,
      pagination.value.currentPage,
      pagination.value.pageSize
  );

  if(result.code) {
    tableData.value = result.data.rows
    pagination.value.total = result.data.total
  }
}

//清空搜索栏
const clear = () => {
  searchClazz.value = {begin:'', end:'', date:[], name:''}
  queryPage()
}

//监听searchEmp的date属性
watch(() => searchClazz.value.date, (newVal, oldVal) => {
  console.log(`newVal : ${newVal} ; oldVal: ${oldVal} `)
  if(newVal.length > 0) {
    console.log('-----------');
    searchClazz.value.begin = newVal[0]
    searchClazz.value.end = newVal[1]
  }else {
    console.log('==========');
    searchClazz.value.begin = ''
    searchClazz.value.end = ''
  }
})



//----------- 新增 / 修改 ---------------------------
let dialogFormVisible = ref(false) //控制新增/修改的对话框的显示与隐藏
let labelWidth = ref(80) //form表单label的宽度
let formTitle = ref('') //表单的标题
//员工对象-表单数据绑定
let clazz = ref({
  id: '',
  name: '',
  room: '',
  beginDate: '',
  endDate: '',
  subject: '',
  masterId: ''
})

//清空表单
const clearClazz = () => {
  clazz.value = {
    id: '',
    name: '',
    room: '',
    beginDate: '',
    endDate: '',
    subject: '',
    masterId: ''
  }
}

//新增班级
const addClazz = () => {
  dialogFormVisible.value = true
  formTitle.value = '新增班级'
  clearClazz()
}

//修改班级
const updateClazz = async (id) => {
  clearClazz()
  dialogFormVisible.value = true
  let result = await queryInfoApi(id)
  if(result.code){
    clazz.value = result.data
  }
}

//表单校验规则
const clazzFormRef = ref()
const rules = ref({
  name: [
    { required: true, message: '班级名称为必填项', trigger: 'blur' },
    { min: 4, max: 30, message: '用户名长度为4-30个字', trigger: 'blur' }
  ],
  room: [
    { min: 1, max: 20, message: '班级教室长度为1-20个字', trigger: 'blur' }
  ],
  beginDate: [{ required: true, message: '开课时间为必填项', trigger: 'change' }],
  endDate: [{ required: true, message: '结课时间为必填项', trigger: 'change' }],
  subject: [{ required: true, message: '学科为必填项', trigger: 'change' }]
})

//重置表单
const resetForm = (clazzForm) => {
  if (!clazzForm) return
  clazzForm.resetFields()
}

//-------------保存班级信息 
const save = (clazzForm) => {
  console.log('clazzForm: ' + clazz);
  // debugger;
  //表单校验
  if(!clazzForm) return
  clazzForm.validate(async (valid) => {
    if(valid) {
      let api 
      if(clazz.value.id) {
        api = updateApi(clazz.value)
      }else {
        api = addApi(clazz.value)
      }

      let result = await api
      if(result.code) {
        ElMessage.success('操作成功')
        dialogFormVisible.value = false
        queryPage()
      }else {
        ElMessage.error(result.msg)
      }
    }else {
      console.log('valid: '+ valid);
      
      return false
    }
  })
}


//------- 删除班级
//根据ID删除单个班级
const delById = async (id) => {
  ElMessageBox.confirm('您确认删除此数据吗?' , '删除班级', {confirmButtonText:'确认', cancelButtonText:'取消',type:'warning'})
    .then(async () => {
      let result =  await deleteApi(id)
      if(result.code) {
        ElMessage.success('删除成功')
        queryPage()
      }else {
        ElMessage.error(result.msg)
      }
    }).catch(() => {
      ElMessage.info('取消删除')
    })
}
</script>

<template>
  <div class="page-container">
    <!-- 顶部标题 -->
    <div>
      <h1>班级管理</h1> <br>
    </div>

    <!-- 条件搜索表单 -->
    <el-form :inline="true" :model="searchClazz" class="search-form">
      <el-form-item label="班级名称">
        <el-input v-model="searchClazz.name" placeholder="请输入班级名称"/>
      </el-form-item>
      
      <el-form-item label="结课时间">
        <el-date-picker
          v-model="searchClazz.date"
          type="daterange"
          range-separator=" 至 "
          start-placeholder="开始时间"
          end-placeholder="结束时间"
          value-format="YYYY-MM-DD"
        />
      </el-form-item>

      <el-form-item>
        <el-button type="primary" @click="queryPage()">查询</el-button>
        <el-button type="info" @click="clear()">清空</el-button>
      </el-form-item>
    </el-form>
    
    <!-- 功能按钮 -->
    <el-button type="success" @click="addClazz();resetForm(clazzFormRef)">+ 新增班级</el-button>
    <br><br>
    
    <!-- 列表展示 -->
    <el-table :data="tableData" border style="width: 100%" fit>
      <el-table-column type="index" label="序号" width="55" align="center"/>
      <el-table-column prop="name" label="班级名称" align="center" width="200px" />
      <el-table-column prop="room" label="班级教室" align="center" width="100px"/>
      <el-table-column prop="masterName" label="班主任" align="center" width="100px"/>
      <el-table-column prop="beginDate" label="开课时间" align="center" width="150px"/>
      <el-table-column prop="endDate" label="结课时间" align="center" width="150px"/>
      <el-table-column prop="status" label="状态" align="center" width="130px" />
      <el-table-column prop="updateTime" label="最后修改时间" align="center" />
      <el-table-column label="操作" align="center">
        <template #default="scope">
          <el-button type="primary" size="small" @click="updateClazz(scope.row.id); resetForm(clazzFormRef)">编辑</el-button>
          <el-button type="danger" size="small" @click="delById(scope.row.id)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <br>

    <!-- 分页组件Pagination -->
    <div class="pagination-container">
      <el-pagination
        v-model:current-page="pagination.currentPage"
        v-model:page-size="pagination.pageSize"
        :page-sizes="[5, 10, 20, 50, 100]"
        layout="total, sizes, prev, pager, next, jumper"
        :total="pagination.total"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
      />
    </div>
  
  <!-- 新增/修改班级对话框 -->
  <el-dialog v-model="dialogFormVisible" :title="formTitle" width="35%">
    <el-form :model="clazz" ref="clazzFormRef" :rules="rules">
      <el-form-item label="班级名称" :label-width="labelWidth" prop="name">
        <el-input v-model="clazz.name" placeholder="请输入班级名称" />
      </el-form-item>

      <el-form-item label="班级教室" :label-width="labelWidth" prop="room">
        <el-input v-model="clazz.room" placeholder="请输入班级教室"/>
      </el-form-item>
      
      <el-form-item label="开课时间" :label-width="labelWidth" prop="beginDate">
        <el-date-picker v-model="clazz.beginDate" type="date" placeholder="请选择开课时间" value-format="YYYY-MM-DD" style="width: 100%;"/>
      </el-form-item>

      <el-form-item label="结课时间" :label-width="labelWidth" prop="endDate">
        <el-date-picker v-model="clazz.endDate" type="date" placeholder="请选择结课时间" value-format="YYYY-MM-DD" style="width: 100%;"/>
      </el-form-item>

      <el-form-item label="班主任" :label-width="labelWidth">
        <el-select v-model="clazz.masterId" placeholder="请选择班主任" style="width: 100%;">
          <el-option v-for="emp in emps" :label="emp.name" :value="emp.id" />
        </el-select>
      </el-form-item>

      <el-form-item label="学科" :label-width="labelWidth" prop="subject">
        <el-select v-model="clazz.subject" placeholder="请选择学科" style="width: 100%;">
          <el-option v-for="sub in subjects" :label="sub.name" :value="sub.value" />
        </el-select>
      </el-form-item>
    </el-form>

    <template #footer>
      <span class="dialog-footer">
        <el-button @click="dialogFormVisible = false; resetForm(clazzFormRef)">取消</el-button>
        <el-button type="primary" @click="save(clazzFormRef)">保存</el-button>
      </span>
    </template>
  </el-dialog>
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

/* iOS 26 风格搜索表单样式 - 增强版 */
.search-form {
  background-color: white;
  padding: 20px;
  border-radius: 16px;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.05);
  margin-bottom: 20px;
  border: 1px solid #e5e5ea;
  transition: box-shadow 0.3s ease;
}

.search-form:hover {
  box-shadow: 0 6px 24px rgba(0, 0, 0, 0.08);
}

/* iOS 26 风格按钮样式 - 增强版 */
.el-button {
  border-radius: 12px;
  transition: all 0.3s ease;
  height: 40px;
  padding: 0 18px;
  font-weight: 500;
  border: none;
}

.el-button--primary {
  background-color: #007aff;
  color: white;
}

.el-button--primary:hover {
  background-color: #0062cc;
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(0, 122, 255, 0.3);
}

.el-button--danger {
  background-color: #ff3b30;
  color: white;
}

.el-button--danger:hover {
  background-color: #d30000;
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(255, 59, 48, 0.3);
}

.el-button--success {
  background-color: #34c759;
  color: white;
}

.el-button--success:hover {
  background-color: #28a745;
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(52, 199, 89, 0.3);
}

.el-button--info {
  background-color: #5ac8fa;
  color: white;
  border: none;
}

.el-button--info:hover {
  background-color: #34b7f1;
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(90, 200, 250, 0.3);
}

.el-button--default {
  background-color: white;
  color: #1c1c1e;
  border: 1px solid #e5e5ea;
}

.el-button--default:hover {
  background-color: #f2f2f7;
  border-color: #c6c6cc;
  transform: translateY(-1px);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.el-button + .el-button {
  margin-left: 12px;
}

/* 操作按钮组 */
.action-buttons {
  margin-bottom: 20px;
  display: flex;
  gap: 12px;
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
}

/* 状态标签样式 */
.status-tag {
  padding: 4px 12px;
  border-radius: 16px;
  font-size: 12px;
  font-weight: 500;
  transition: all 0.3s ease;
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

/* iOS 26 风格对话框样式 - 增强版 */
.el-dialog {
  border-radius: 20px;
  overflow: hidden;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.15);
  border: 1px solid #e5e5ea;
  background: white;
  animation: dialogFadeIn 0.3s ease-out;
}

@keyframes dialogFadeIn {
  from {
    opacity: 0;
    transform: scale(0.95) translateY(-10px);
  }
  to {
    opacity: 1;
    transform: scale(1) translateY(0);
  }
}

.el-dialog__header {
  background: white;
  color: #1c1c1e;
  padding: 20px 24px;
  border-bottom: 1px solid #e5e5ea;
}

.el-dialog__title {
  color: #1c1c1e;
  font-size: 18px;
  font-weight: 600;
}

.el-dialog__body {
  padding: 24px;
  max-height: 60vh;
  overflow-y: auto;
}

.el-dialog__footer {
  padding: 16px 24px;
  background-color: #f9f9fa;
  border-top: 1px solid #e5e5ea;
}

/* iOS 26 风格表单样式增强 */
.el-input,
.el-select,
.el-date-editor {
  border-radius: 12px;
  transition: all 0.3s ease;
  border: 1px solid #e5e5ea;
  height: 44px;
  background-color: white;
}

.el-input:hover,
.el-select:hover,
.el-date-editor:hover {
  border-color: #c6e2ff;
  box-shadow: 0 2px 8px rgba(0, 122, 255, 0.05);
}

.el-input:focus-within,
.el-select:focus-within,
.el-date-editor:focus-within {
  border-color: #007aff;
  box-shadow: 0 0 0 3px rgba(0, 122, 255, 0.15);
}

/* iOS 26 风格表单项样式 */
.el-form-item {
  margin-bottom: 16px;
}

.el-form-item__label {
  font-weight: 500;
  color: #8e8e93;
  font-size: 14px;
}

/* iOS 26 风格选择框样式 */
.el-select-dropdown {
  border-radius: 12px;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.1);
  border: 1px solid #e5e5ea;
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
  
  .search-form {
    padding: 16px;
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
  
  .search-form {
    padding: 16px;
    border-radius: 12px;
  }
  
  .el-form-item {
    margin-bottom: 12px;
  }
  
  .el-table {
    font-size: 13px;
  }
  
  .el-dialog {
    width: 95% !important;
    margin: 2vh auto !important;
    border-radius: 16px;
  }
  
  .el-button {
    border-radius: 10px;
    height: 36px;
    padding: 0 14px;
    font-size: 13px;
  }
}

/* 优化滚动条样式 */
.el-table__body-wrapper::-webkit-scrollbar,
.el-dialog__body::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}

.el-table__body-wrapper::-webkit-scrollbar-thumb,
.el-dialog__body::-webkit-scrollbar-thumb {
  background-color: #c7c7cc;
  border-radius: 4px;
  transition: background-color 0.3s ease;
}

.el-table__body-wrapper::-webkit-scrollbar-thumb:hover,
.el-dialog__body::-webkit-scrollbar-thumb:hover {
  background-color: #8e8e93;
}

.el-table__body-wrapper::-webkit-scrollbar-track,
.el-dialog__body::-webkit-scrollbar-track {
  background-color: #f2f2f7;
  border-radius: 4px;
}

/* iOS 26 风格分页按钮样式 */
.el-pagination.is-background .el-pager li:not(.disabled):hover {
  color: #007aff;
  background-color: #f2f2f7;
}

.el-pagination.is-background .el-pager li.is-active {
  background-color: #007aff;
  color: white;
}

/* iOS 26 风格对话框底部样式 */
.dialog-footer {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
}
</style>