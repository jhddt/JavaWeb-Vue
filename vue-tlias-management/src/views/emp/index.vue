<script setup>
import { ref, watch, onMounted } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus'
import { queryPageApi, addEmpApi, queryInfoApi, updateEmpApi, deleteEmpApi} from '@/api/emp'
import { queryAllApi as queryAllDeptApi } from '@/api/dept'

//职位列表数据
const jobs = ref([{ name: '班主任', value: 1 },{ name: '讲师', value: 2 },{ name: '学工主管', value: 3 },{ name: '教研主管', value: 4 },{ name: '咨询师', value: 5 },{ name: '其他', value: 6 }])
//性别列表数据
const genders = ref([{ name: '男', value: 1 }, { name: '女', value: 2 }])
//部门列表数据
const deptList = ref([])

const searchEmp = ref({
  name: '',
  gender: '',
  date: [],
  begin: '',
  end: ''
})

//侦听searchEmp中的date属性
watch(
  () => searchEmp.value.date,
  (newValue, oldValue) => {
     if(newValue.length == 2){
      searchEmp.value.begin = newValue[0]
      searchEmp.value.end = newValue[1]
     }else {
      searchEmp.value.begin = ''
      searchEmp.value.end = ''
     }
  }
)

//声明token
const token = ref('')

onMounted(async () => {
  handleSearch()

  //加载所有部门数据
  const result = await queryAllDeptApi();
  if(result.code){
    deptList.value = result.data
  }

  //加载localStorage存储的员工登录信息
  const loginUser =  localStorage.getItem('loginUser');
  if(loginUser){
    token.value = JSON.parse(loginUser).token
  }
})

//查询员工
const handleSearch = async () => {
  console.log('Search:', searchEmp.value)
  const result = await queryPageApi(searchEmp.value.name, searchEmp.value.gender, searchEmp.value.begin, searchEmp.value.end, currentPage.value, pageSize.value);
  if(result.code){
    empList.value = result.data.rows
    total.value = result.data.total
  }
}

const handleReset = () => {
  // 清空表单
  searchEmp.value = {
    name: '',
    gender: '',
    date: [],
    begin: '',
    end: ''
  }
  handleSearch()
}

// 示例数据
const empList = ref([])

// 分页配置
const currentPage = ref(1)
const pageSize = ref(10)
const total = ref(0)

// 职位转换函数
const getJobTitle = (job) => {
  switch (job) {
    case 1:
      return '班主任'
    case 2:
      return '讲师'
    case 3:
      return '学工主管'
    case 4:
      return '教研主管'
    case 5:
      return '咨询师'
    default:
      return '其他'
  }
}

// 分页处理
const handleSizeChange = (val) => {
  pageSize.value = val
  handleSearch()
}
const handleCurrentChange = (val) => {
  currentPage.value = val
  handleSearch()
}

// 操作处理
const handleEdit = async (id) => {
  console.log('Edit:', id)
  const result = await queryInfoApi(id);
  if(result.code){
    dialogVisible.value = true
    dialogTitle.value = '修改员工'
    employee.value = result.data

    //处理返回的工作经历信息
    let exprList = employee.value.exprList;
    if(exprList && exprList.length > 0){
      exprList.forEach(expr => {
        expr.exprDate = [expr.begin, expr.end]
      })
    }
  }
}

// 删除单个员工
const handleDelete = async (id) => {
  //弹出一个确认框, 如果确认, 就删除;
  ElMessageBox.confirm('确定删除该员工吗?', '提示', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'warning'
  }).then(async () => {
    // 删除员工
    const result = await deleteEmpApi(id);
    if(result.code){
      ElMessage.success('删除员工成功')
      handleSearch()
    }else{
      ElMessage.error(result.msg)
    }
  })
}

//新增员工
const addEmp = () => {
  dialogVisible.value = true
  dialogTitle.value = '新增员工'
  employee.value = {
    username: '',
    name: '',
    gender: '',
    phone: '',
    job: '',
    salary: '',
    deptId: '',
    entryDate: '',
    image: '',
    exprList: []
  }
  employeeFormRef.value.resetFields()
}


//新增/修改表单
const employeeFormRef = ref(null)
const employee = ref({
  username: '',
  name: '',
  gender: '',
  phone: '',
  job: '',
  salary: '',
  deptId: '',
  entryDate: '',
  image: '',
  exprList: []
})

//表单校验规则
// 验证规则
const rules = ref({
  username: [
    { required: true, message: '请输入用户名', trigger: 'blur' },
    { min: 2, max: 20, message: '用户名长度应在2到20个字符之间', trigger: 'blur' }
  ],
  name: [
    { required: true, message: '请输入姓名', trigger: 'blur' },
    { min: 2, max: 10, message: '姓名长度应在2到10个字符之间', trigger: 'blur' }
  ],
  gender: [
    { required: true, message: '请选择性别', trigger: 'change' }
  ],
  phone: [
    { required: true, message: '请输入手机号', trigger: 'blur' },
    { pattern: /^1\d{10}$/, message: '请输入有效的手机号', trigger: 'blur' }
  ]
});

// 控制弹窗
const dialogVisible = ref(false)
const dialogTitle = ref('新增员工')

// 图片上传成功后触发
const handleAvatarSuccess = (response,uploadFile) => {
  employee.value.image = response.data
}
// 文件上传之前触发
const beforeAvatarUpload = (rawFile) => {
  if (rawFile.type !== 'image/jpeg' && rawFile.type !== 'image/png') {
    ElMessage.error('只支持上传图片')
    return false
  } else if (rawFile.size / 1024 / 1024 > 10) {
    ElMessage.error('只能上传10M以内图片')
    return false
  }
  return true
}


//工作经历
//动态添加工作经历 .
const addExprItem = () => {
  employee.value.exprList.push({exprDate: [], begin: '', end: '', company: '', job: ''})
}

//动态删除工作经历 .
const delExprItem = (index) => {
  employee.value.exprList.splice(index, 1)
}

//监听-employee员工对象中的工作经历数据
watch(employee, (newVal, oldVal) => {
  if(employee.value.exprList) {
    employee.value.exprList.forEach(expr => {
      expr.begin = expr.exprDate[0]
      expr.end = expr.exprDate[1]
    })
  }
}, {deep: true})


//保存员工信息
const save = async () => {
  employeeFormRef.value.validate(async valid => {
    if(valid){ // 校验通过
      let result ;
      if(employee.value.id){ //存在ID, 修改
        result = await updateEmpApi(employee.value);
      }else { //不存在ID, 新增
        result = await addEmpApi(employee.value);
      }
      if(result.code){
        ElMessage.success('新增员工成功')
        dialogVisible.value = false
        handleSearch()
      }else {
        ElMessage.error(result.msg)
      }
    }
  })
}

// 存储选中的 ID
const selectedIds = ref([]);

// 处理复选框选择变化的函数
function handleSelectionChange(selection) {
  const ids = selection.map(item => item.id);
  selectedIds.value = ids;
}

//批量删除
const deleteByIds = async () => {
  //弹出一个确认框, 如果确认, 就删除;
  ElMessageBox.confirm('确定删除选中员工吗?', '提示', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'warning'
  }).then(async () => {
    // 删除员工
    const result = await deleteEmpApi(selectedIds.value);
    if(result.code){
      ElMessage.success('删除员工成功')
      handleSearch()
    }else{
      ElMessage.error(result.msg)
    }
  })
}

</script>

<template>
  <div class="page-container">
    <h1>员工管理</h1> <br>
    <el-form :inline="true" :model="searchEmp">
      <el-form-item label="姓名">
        <el-input v-model="searchEmp.name" placeholder="请输入员工姓名"></el-input>
      </el-form-item>

      <el-form-item label="性别">
        <el-select v-model="searchEmp.gender" placeholder="请选择">
          <el-option label="男" value="1"></el-option>
          <el-option label="女" value="2"></el-option>
        </el-select>
      </el-form-item>

      <el-form-item label="入职日期">
        <el-date-picker
          v-model="searchEmp.date"
          type="daterange"
          range-separator="至"
          start-placeholder="开始日期"
          end-placeholder="结束日期"
          value-format="YYYY-MM-DD"
        ></el-date-picker>
      </el-form-item>

      <el-form-item>
        <el-button type="primary" @click="handleSearch">查询</el-button>
        <el-button @click="handleReset">清空</el-button>
      </el-form-item>
    </el-form>


    <el-button type="primary" @click="addEmp"> + 新增员工</el-button>
    <el-button type="danger" @click="deleteByIds"> - 批量删除</el-button>
    <br><br>

    <!-- 表格 -->
    <el-table :data="empList" border style="width: 100%" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center"></el-table-column>
      <el-table-column prop="name" label="姓名" width="120" align="center"></el-table-column>
      <el-table-column label="性别" width="80" align="center">
        <template #default="scope" >
          {{ scope.row.gender == 1 ? '男' : '女' }}
        </template>
      </el-table-column>
    <el-table-column label="头像" width="170" align="center">
      <template #default="scope" >
        <img :src="scope.row.image" alt="Avatar" class="avatar" />
      </template>
    </el-table-column>
    <el-table-column prop="deptName" label="部门名称" width="170" align="center"></el-table-column>
    <el-table-column label="职位" width="120" align="center">
      <template #default="scope">
        {{ getJobTitle(scope.row.job) }}
      </template>
    </el-table-column>
    <el-table-column prop="entryDate" label="入职日期" width="180" align="center"></el-table-column>
    <el-table-column prop="updateTime" label="最后操作时间" width="210" align="center"></el-table-column>
    <el-table-column label="操作" fixed="right" align="center">
      <template #default="scope">
        <el-button size="small" type="primary" @click="handleEdit(scope.row.id)">编辑</el-button>
        <el-button size="small" type="danger" @click="handleDelete(scope.row.id)">删除</el-button>
      </template>
    </el-table-column>
  </el-table>
  <br>

  <!-- 分页 -->
  <el-pagination
    @size-change="handleSizeChange"
    @current-change="handleCurrentChange"
    :current-page="currentPage"
    :page-sizes="[10, 20, 30, 40]"
    :page-size="pageSize"
    layout="total, sizes, prev, pager, next, jumper"
    :total="total"
  >
  </el-pagination>

  <!-- 新增/修改员工的对话框 -->
  <el-dialog v-model="dialogVisible" :title="dialogTitle">
      <el-form ref="employeeFormRef" :model="employee" :rules="rules" label-width="80px">
        <!-- 基本信息 -->
        <!-- 第一行 -->
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="用户名" prop="username">
              <el-input v-model="employee.username" placeholder="请输入员工用户名，2-20个字"></el-input>
            </el-form-item>
          </el-col>
          
          <el-col :span="12">
            <el-form-item label="姓名" prop="name">
              <el-input v-model="employee.name" placeholder="请输入员工姓名，2-10个字"></el-input>
            </el-form-item>
          </el-col>
        </el-row>

        <!-- 第二行 -->
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="性别" prop="gender">
              <el-select v-model="employee.gender" placeholder="请选择性别" style="width: 100%;">
                <el-option v-for="gender in genders" :key="gender.name" :label="gender.name" :value="gender.value"></el-option>
              </el-select>
            </el-form-item>
          </el-col>

          <el-col :span="12">
            <el-form-item label="手机号" prop="phone">
              <el-input v-model="employee.phone" placeholder="请输入员工手机号"></el-input>
            </el-form-item>
          </el-col>
        </el-row>

        <!-- 第三行 -->
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="职位">
              <el-select v-model="employee.job" placeholder="请选择职位" style="width: 100%;">
                <el-option v-for="job in jobs" :key="job.name" :label="job.name" :value="job.value"></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="薪资" prop="salary">
              <el-input v-model="employee.salary" placeholder="请输入员工薪资"></el-input>
            </el-form-item>
          </el-col>
        </el-row>
        
        <!-- 第四行 -->
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="所属部门">
              <el-select v-model="employee.deptId" placeholder="请选择部门" style="width: 100%;">
                <el-option v-for="dept in deptList" :key="dept.id" :label="dept.name" :value="dept.id"></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="入职日期">
              <el-date-picker v-model="employee.entryDate" type="date" style="width: 100%;" placeholder="选择日期" format="YYYY-MM-DD" value-format="YYYY-MM-DD"></el-date-picker>
            </el-form-item>
          </el-col>
        </el-row>

        <!-- 第五行 -->
        <el-row :gutter="20">
          <el-col :span="24">
            <el-form-item label="头像">
              <el-upload
                class="avatar-uploader"
                action="/api/upload"
                :headers="{'token': token}"
                :show-file-list="false"
                :on-success="handleAvatarSuccess"
                :before-upload="beforeAvatarUpload"
                >
                <img v-if="employee.image" :src="employee.image" class="avatar" />
                <el-icon v-else class="avatar-uploader-icon"><Plus /></el-icon>
              </el-upload>
            </el-form-item>
          </el-col>
        </el-row>
        

        <!-- 工作经历 -->
        <!-- 第六行 -->
        <el-row :gutter="10">
          <el-col :span="24">
            <el-form-item label="工作经历">
              <el-button type="success" size="small" @click="addExprItem">+ 添加工作经历</el-button>
            </el-form-item>
          </el-col>
        </el-row>
        
        <!-- 第七行 ...  工作经历 -->
        <el-row :gutter="3" v-for="(expr, index) in employee.exprList">
          <el-col :span="10">
            <el-form-item size="small" label="时间" label-width="80px">
              <el-date-picker type="daterange" v-model="expr.exprDate" range-separator="至" start-placeholder="开始日期" end-placeholder="结束日期" format="YYYY-MM-DD" value-format="YYYY-MM-DD" ></el-date-picker>
            </el-form-item>
          </el-col>

          <el-col :span="6">
            <el-form-item size="small" label="公司" label-width="60px">
              <el-input placeholder="请输入公司名称" v-model="expr.company"></el-input>
            </el-form-item>
          </el-col>

          <el-col :span="6">
            <el-form-item size="small" label="职位" label-width="60px">
              <el-input placeholder="请输入职位" v-model="expr.job"></el-input>
            </el-form-item>
          </el-col>

          <el-col :span="2">
            <el-form-item size="small" label-width="0px">
              <el-button type="danger" @click="delExprItem(index)">- 删除</el-button>
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>

      <!-- 底部按钮 -->
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="dialogVisible = false">取消</el-button>
          <el-button type="primary" @click="save">保存</el-button>
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
.el-form {
  background-color: white;
  padding: 20px;
  border-radius: 16px;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.05);
  margin-bottom: 20px;
  border: 1px solid #e5e5ea;
  transition: box-shadow 0.3s ease;
}

.el-form:hover {
  box-shadow: 0 6px 24px rgba(0, 0, 0, 0.08);
}

.el-form-item {
  margin-bottom: 16px;
}

.el-form-item__label {
  font-weight: 500;
  color: #8e8e93;
  font-size: 14px;
}

/* iOS 26 风格按钮样式 - 增强版 */
.el-button {
  border-radius: 12px;
  transition: all 0.3s ease;
  margin-right: 10px;
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
  height: 60px;
  padding: 0 16px;
  font-size: 14px;
  transition: all 0.3s ease;
}

.el-table tr:hover > td {
  background-color: #f9f9fc !important;
}

/* 表格内容居中对齐 */
.el-table .cell {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
}

/* iOS 26 风格滚动条美化 */
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

/* iOS 26 风格头像样式 - 增强版 */
.avatar {
  height: 48px;
  width: 48px;
  border-radius: 50%;
  object-fit: cover;
  border: 2px solid #e5e5ea;
  transition: all 0.3s ease;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.avatar:hover {
  transform: scale(1.1);
  border-color: #007aff;
  box-shadow: 0 4px 8px rgba(0, 122, 255, 0.2);
}

/* 对话框中的头像样式 */
.avatar-uploader {
  display: inline-block;
  position: relative;
}

.avatar-uploader .avatar {
  width: 96px;
  height: 96px;
  display: block;
  border-radius: 16px;
  border: 2px solid #e5e5ea;
}

.avatar-uploader .el-upload {
  border: 2px dashed #e5e5ea;
  border-radius: 16px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
  transition: all 0.3s ease;
  background-color: white;
}

.avatar-uploader .el-upload:hover {
  border-color: #007aff;
  background-color: #f2f2f7;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 122, 255, 0.15);
}

.el-icon.avatar-uploader-icon {
  font-size: 32px;
  color: #8e8e93;
  width: 96px;
  height: 96px;
  text-align: center;
  line-height: 96px;
  border: 2px dashed #e5e5ea;
  border-radius: 16px;
}

/* iOS 26 风格分页样式 - 增强版 */
.el-pagination {
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
  display: flex;
  justify-content: flex-end;
  gap: 12px;
}

/* iOS 26 风格表单样式增强 */
.el-input,
.el-select,
.el-date-picker {
  border-radius: 12px;
  transition: all 0.3s ease;
  border: 1px solid #e5e5ea;
  height: 44px;
  background-color: white;
}

.el-input:hover,
.el-select:hover,
.el-date-picker:hover {
  border-color: #c6e2ff;
  box-shadow: 0 2px 8px rgba(0, 122, 255, 0.05);
}

.el-input:focus-within,
.el-select:focus-within,
.el-date-picker:focus-within {
  border-color: #007aff;
  box-shadow: 0 0 0 3px rgba(0, 122, 255, 0.15);
}

/* iOS 26 风格工作经历样式 - 增强版 */
.expr-item {
  padding: 16px;
  background-color: #f2f2f7;
  border-radius: 16px;
  margin-bottom: 16px;
  border-left: 4px solid #007aff;
  transition: all 0.3s ease;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
}

.expr-item:hover {
  background-color: #e9f5ff;
  transform: translateX(3px);
  box-shadow: 0 4px 12px rgba(0, 122, 255, 0.1);
}

/* 工作经历容器样式 */
.expr-container {
  background-color: #f9f9fc;
  padding: 16px;
  border-radius: 16px;
  border: 1px solid #e5e5ea;
  margin-bottom: 16px;
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
  
  .el-form {
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
  
  .el-form {
    padding: 16px;
    border-radius: 12px;
  }
  
  .el-table {
    font-size: 13px;
  }
  
  .el-dialog {
    width: 95% !important;
    margin: 2vh auto !important;
    border-radius: 16px;
  }
  
  .el-form-item {
    margin-bottom: 12px;
  }
  
  .el-button {
    border-radius: 10px;
    height: 36px;
    padding: 0 14px;
    font-size: 13px;
  }
  
  .avatar {
    height: 40px;
    width: 40px;
  }
  
  .avatar-uploader .avatar {
    width: 80px;
    height: 80px;
  }
}

/* 其他样式增强 */
.dialog-footer {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
}

/* 表格选择框样式增强 */
.el-table__header-wrapper .el-checkbox__input.is-checked .el-checkbox__inner,
.el-table__header-wrapper .el-checkbox__input.is-indeterminate .el-checkbox__inner {
  background-color: #007aff;
  border-color: #007aff;
}

/* 优化滚动条样式 */
.el-dialog__body::-webkit-scrollbar {
  width: 8px;
}

.el-dialog__body::-webkit-scrollbar-thumb {
  background-color: #c7c7cc;
  border-radius: 4px;
}

.el-dialog__body::-webkit-scrollbar-track {
  background-color: #f2f2f7;
  border-radius: 4px;
}
</style>