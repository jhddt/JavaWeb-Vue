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
/*页面整体布局 - 增强版 */
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
  justify-content: flex-end;
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
}

.el-table tr:hover > td {
  background-color: #f9f9fc !important;
}

/* 表格行动画效果 */
.el-table__row {
  transition: all 0.3s ease;
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
.el-input {
  border-radius: 12px;
  transition: all 0.3s ease;
  border: 1px solid #e5e5ea;
  height: 44px;
  background-color: white;
}

.el-input:hover {
  border-color: #c6e2ff;
  box-shadow: 0 2px 8px rgba(0, 122, 255, 0.05);
}

.el-input:focus-within {
  border-color: #007aff;
  box-shadow: 0 0 0 3px rgba(0, 122, 255, 0.15);
}

/* iOS 26 风格表单标签 */
.el-form-item__label {
  font-weight: 500;
  color: #8e8e93;
  font-size: 14px;
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

/* 其他样式增强 */
.dialog-footer {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
}
</style>