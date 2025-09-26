<script setup>
import { ref, onMounted } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus'
import { queryAllApi, addDeptApi, queryInfoApi, updateDeptApi, deleteDeptApi } from '@/api/dept'

// 声明列表展示数据
let tableData = ref([])

// 动态加载数据 - 查询部门
const queryAll = async () => {
  const result = await queryAllApi()
  tableData.value = result.data
}

// 钩子函数
onMounted(() => {
  queryAll()
})

const formTitle = ref('')
//新增部门
const add = () => {
  formTitle.value = '新增部门'
  showDialog.value = true
  deptForm.value = {name: ''}
}

// 编辑部门 - 根据ID查询回显数据
const handleEdit = async (row) => {
  console.log(`Edit item with ID ${row.id}`);
  formTitle.value = '修改部门'
  showDialog.value = true
  deptForm.value = {name: ''}

  const result = await queryInfoApi(row.id)
  if(result.code){
    deptForm.value = result.data
  }
};

// 删除部门 - 根据ID删除部门
const handleDelete = (row) => {
  console.log(`Delete item with ID ${row.id}`);
  //删除部门时, 需要弹出一个确认框, 如果是确认, 则删除部门
  ElMessageBox.confirm('此操作将永久删除该部门, 是否继续?', '提示', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'warning'
  }).then(async () => {
    // 删除部门
    const result = await deleteDeptApi(row.id)
    if(result.code){
      ElMessage.success('删除成功')
      queryAll()
    }
  })
};

// 新增部门对话框的状态
const showDialog = ref(false)
// 表单数据
const deptForm = ref({name: ''})
// 表单验证规则
const formRules = ref({
  name: [
    { required: true, message: '请输入部门名称', trigger: 'blur' },
    { min: 2, max: 10, message: '长度在 2 到 10 个字符', trigger: 'blur' }
  ]
})

// 表单引用
const deptFormRef = ref(null)

// 重置表单
const resetForm = () => {
  deptFormRef.value.resetFields()
}

// 提交表单
const save = async () => {
  await deptFormRef.value.validate(async valid => {
    if (!valid) return
    // 提交表单
    let result = null;
    if(deptForm.value.id){
      result = await updateDeptApi(deptForm.value) // 修改
    }else {
      result = await addDeptApi(deptForm.value) // 新增
    } 
    if(result.code){
      ElMessage.success('操作成功')
      // 关闭对话框
      showDialog.value = false
      // 重置表单
      resetForm()
      // 重新加载数据
      queryAll()
    }else {
      ElMessage.error(result.msg)
    }
  })
}
</script>

<template>
  <div class="page-container">
    <h1>部门管理</h1>

    <!-- 按钮靠页面右侧显示 -->
    <el-button type="primary" @click="add()" style="float: right;"> + 新增部门</el-button> <br><br>

    <!-- 数据展示表格 -->
    <el-table :data="tableData" border style="width: 100%;">
      <el-table-column type="index" label="序号" width="100" align="center"/>
      <el-table-column prop="name" label="部门名称" width="300" align="center"/>
      <el-table-column prop="updateTime" label="最后修改时间" width="300" align="center"/>
      <el-table-column fixed="right" label="操作" align="center">
        <template #default="{ row }">
          <el-button size="small" @click="handleEdit(row)">修改</el-button>
          <el-button size="small" type="danger" @click="handleDelete(row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- 新增部门的对话框 -->
    <el-dialog v-model="showDialog" :title="formTitle" width="30%" @close="resetForm">
      <el-form :model="deptForm" :rules="formRules" ref="deptFormRef">
        <el-form-item label="部门名称" prop="name" label-width="80px">
          <el-input v-model="deptForm.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>

      <template #footer>
        <span class="dialog-footer">
          <el-button @click="showDialog = false">取消</el-button>
          <el-button type="primary" @click="save">确定</el-button>
        </span>
      </template>
    </el-dialog>
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

/* 按钮样式 */
.el-button {
  border-radius: 6px;
  transition: all 0.3s ease;
}

.el-button:hover {
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(64, 158, 255, 0.2);
}

.el-button + .el-button {
  margin-left: 10px;
}

/* 表格样式 */
.el-table {
  background-color: white;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.06);
}

.el-table thead {
  background-color: #f8f9fa;
}

.el-table th {
  font-weight: 600;
  color: #606266;
  background-color: #f8f9fa !important;
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

/* 对话框样式 */
.el-dialog {
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
}

.el-dialog__header {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 16px 20px;
}

.el-dialog__title {
  color: white;
  font-size: 16px;
  font-weight: 600;
}

.el-dialog__body {
  padding: 20px;
  max-height: 60vh;
  overflow-y: auto;
}

.el-dialog__footer {
  padding: 16px 20px;
  background-color: #f8f9fa;
  border-top: 1px solid #e9ecef;
}

/* 表单样式增强 */
.el-input {
  border-radius: 4px;
  transition: all 0.3s ease;
}

.el-input:hover {
  border-color: #c6e2ff;
}

.el-input:focus-within {
  border-color: #409eff;
  box-shadow: 0 0 0 2px rgba(64, 158, 255, 0.2);
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
@media (max-width: 768px) {
  .page-container {
    padding: 10px;
  }
  
  h1 {
    font-size: 20px;
  }
  
  .el-table {
    font-size: 13px;
  }
  
  .el-dialog {
    width: 90% !important;
    margin: 5vh auto !important;
  }
}
</style>