<script setup>
import { ref, onMounted } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus';
import { useRouter } from 'vue-router'

let router = useRouter()

const loginName = ref('')
//定义钩子函数, 获取登录用户名
onMounted(() => {
  //获取登录用户名
  let loginUser = JSON.parse(localStorage.getItem('loginUser'))
  if (loginUser) {
    loginName.value = loginUser.name
  }
})

const logout = () => {
  //弹出确认框, 如果确认, 则退出登录, 跳转到登录页面
  ElMessageBox.confirm('确认退出登录吗?', '提示', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'warning'
  }).then(() => {//确认, 则清空登录信息
    ElMessage.success('退出登录成功')
    localStorage.removeItem('loginUser')
    router.push('/login')//跳转到登录页面
  })
}
</script>

<template>
  <div class="common-layout">
    <el-container>
      <!-- Header 区域 -->
      <el-header class="header">
        <span class="title">智教云</span>
        <span class="right_tool">
          <a href="">
            <el-icon><EditPen /></el-icon> 修改密码 &nbsp;&nbsp;&nbsp; |  &nbsp;&nbsp;&nbsp;
          </a>
          <a href="javascript:void(0)" @click="logout">
            <el-icon><SwitchButton /></el-icon> 退出登录 【{{ loginName }}】
          </a>
        </span>
      </el-header>
      
      <el-container>
        <!-- 左侧菜单 -->
        <el-aside width="200px" class="aside">

          <el-menu router>
            <!-- 首页菜单 -->
            <el-menu-item index="/index">
              <el-icon><Promotion /></el-icon> 首页
            </el-menu-item>
            
            <!-- 班级管理菜单 -->
            <el-sub-menu index="/manage">
              <template #title>
                <el-icon><Menu /></el-icon> 班级学员管理
              </template>
              <el-menu-item index="/clazz">
                <el-icon><HomeFilled /></el-icon>班级管理
              </el-menu-item>
              <el-menu-item index="/stu">
                <el-icon><UserFilled /></el-icon>学员管理
              </el-menu-item>
            </el-sub-menu>
            
            <!-- 系统信息管理 -->
            <el-sub-menu index="/system">
              <template #title>
                <el-icon><Tools /></el-icon>系统信息管理
              </template>
              <el-menu-item index="/dept">
                <el-icon><HelpFilled /></el-icon>部门管理
              </el-menu-item>
              <el-menu-item index="/emp">
                <el-icon><Avatar /></el-icon>员工管理
              </el-menu-item>
            </el-sub-menu>

            <!-- 数据统计管理 -->
            <el-sub-menu index="/report">
              <template #title>
                <el-icon><Histogram /></el-icon>数据统计管理
              </template>
              <el-menu-item index="/report/emp">
                <el-icon><InfoFilled /></el-icon>员工信息统计
              </el-menu-item>
              <el-menu-item index="/report/stu">
                <el-icon><Share /></el-icon>学员信息统计
              </el-menu-item>
              <el-menu-item index="/log">
                <el-icon><Document /></el-icon>日志信息统计
              </el-menu-item>
            </el-sub-menu>
          </el-menu>
        </el-aside>
        
        <!-- 主展示区域 -->
        <el-main>
          <router-view></router-view>
        </el-main>
      </el-container>
    </el-container>
  </div>
</template>

<style scoped>
.common-layout {
  height: 100vh;
  display: flex;
  flex-direction: column;
}

/* 顶部导航栏样式 */
.header {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  height: 64px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 24px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  z-index: 100;
}

.title {
  color: white;
  font-size: 24px;
  font-weight: 600;
  letter-spacing: 1px;
  display: flex;
  align-items: center;
  gap: 8px;
}

.title::before {
  content: '';
  display: inline-block;
  width: 32px;
  height: 32px;
  background: url(../../assets/logo.png) no-repeat center;
  background-size: contain;
}

/* 右侧工具栏 */
.right_tool {
  display: flex;
  align-items: center;
  gap: 16px;
  color: white;
}

.right_tool a {
  color: white;
  text-decoration: none;
  display: flex;
  align-items: center;
  gap: 4px;
  padding: 6px 12px;
  border-radius: 4px;
  transition: all 0.3s ease;
}

.right_tool a:hover {
  background: rgba(255, 255, 255, 0.1);
  transform: translateY(-1px);
}

.right_tool .el-icon {
  font-size: 16px;
}

/* 主体容器 */
.el-container {
  flex: 1;
  display: flex;
  height: calc(100vh - 64px);
}

/* 侧边栏样式 */
.aside {
  width: 220px;
  background: white;
  border-right: 1px solid #e4e7ed;
  overflow-y: auto;
  transition: all 0.3s ease;
  box-shadow: 2px 0 8px rgba(0, 0, 0, 0.05);
}

/* 菜单样式增强 */
.el-menu {
  border-right: none;
  height: 100%;
  background: transparent;
}

.el-menu-item,
.el-sub-menu__title {
  height: 50px;
  line-height: 50px;
  padding: 0 24px;
  font-size: 14px;
  transition: all 0.3s ease;
  border-radius: 0 25px 25px 0;
  margin: 4px 0;
}

.el-menu-item:hover,
.el-sub-menu__title:hover {
  background: #f0f9ff;
  color: #409eff;
}

.el-menu-item.is-active {
  background: linear-gradient(90deg, #e6f7ff 0%, #f0f9ff 100%);
  color: #409eff;
  font-weight: 500;
  border-right: 3px solid #409eff;
}

.el-menu-item .el-icon,
.el-sub-menu__title .el-icon {
  font-size: 18px;
  margin-right: 12px;
}

.el-sub-menu .el-menu {
  background: white;
}

.el-sub-menu .el-menu-item {
  padding-left: 56px;
  border-radius: 0;
}

/* 主内容区域 */
.el-main {
  padding: 24px;
  background: #f5f7fa;
  overflow-y: auto;
}

/* 响应式适配 */
@media (max-width: 1200px) {
  .aside {
    width: 200px;
  }
  
  .title {
    font-size: 20px;
  }
}

@media (max-width: 768px) {
  .aside {
    position: fixed;
    left: -220px;
    z-index: 99;
  }
  
  .aside.show {
    left: 0;
  }
  
  .el-main {
    padding: 16px;
  }
}
</style>