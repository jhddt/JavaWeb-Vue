<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus';
import { useRouter } from 'vue-router'

let router = useRouter()

const loginName = ref('')

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

// 添加滚动时增强毛玻璃效果的功能
const handleScroll = () => {
  const header = document.querySelector('.header')
  const scrollY = window.scrollY
  
  // 当页面滚动超过20px时，增强毛玻璃效果
  if (scrollY > 20) {
    header.classList.add('scrolled-blur')
  } else {
    header.classList.remove('scrolled-blur')
  }
}

// 合并的onMounted钩子函数
onMounted(() => {
  //获取登录用户名
  let loginUser = JSON.parse(localStorage.getItem('loginUser'))
  if (loginUser) {
    loginName.value = loginUser.name
  }
  
  // 添加滚动监听
  window.addEventListener('scroll', handleScroll)
})

// 组件卸载时移除滚动监听
onUnmounted(() => {
  window.removeEventListener('scroll', handleScroll)
})
</script>

<template>
  <div class="common-layout">
    <el-container>
      <!-- Header 区域 - 应用毛玻璃效果 -->
      <el-header class="header header-blur">
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
        <!-- 左侧菜单 - 应用毛玻璃效果 -->
        <el-aside width="200px" class="aside card-blur">

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
  background-color: #f5f5f7;
  background-image: 
    radial-gradient(circle at 20% 30%, rgba(0, 122, 255, 0.05) 0%, transparent 40%),
    radial-gradient(circle at 80% 70%, rgba(52, 199, 89, 0.05) 0%, transparent 40%);
}

/* 桌面端优化的顶部导航栏样式 */
.header {
  height: 60px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 24px;
  z-index: 100;
  position: relative;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
  background-color: rgba(255, 255, 255, 0.85);
  backdrop-filter: blur(10px);
}

.title {
  color: #1c1c1e;
  font-size: 22px;
  font-weight: 700;
  letter-spacing: -0.5px;
  display: flex;
  align-items: center;
  gap: 10px;
}

.title::before {
  content: '';
  display: inline-block;
  width: 32px;
  height: 32px;
  background: url(../../assets/logo.png) no-repeat center;
  background-size: contain;
}

/* 桌面端优化的右侧工具栏 */
.right_tool {
  display: flex;
  gap: 12px;
  color: #1c1c1e;
}

.right_tool a {
  color: #007aff;
  text-decoration: none;
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 8px 16px;
  border-radius: 12px;
  transition: all 0.2s ease;
  font-size: 14px;
  background-color: #f2f2f7;
  font-weight: 500;
}

.right_tool a:hover {
  background-color: #e5e5ea;
  transform: translateY(-1px);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.right_tool .el-icon {
  font-size: 16px;
}

/* 桌面端优化的主体容器 */
.el-container {
  flex: 1;
  display: flex;
  height: calc(100vh - 60px);
  overflow: hidden;
}

/* 桌面端优化的侧边栏样式 */
.aside {
  width: 240px;
  background-color: #ffffff;
  border-right: 1px solid #e5e5ea;
  overflow-y: auto;
  transition: all 0.3s cubic-bezier(0.25, 0.1, 0.25, 1);
  box-shadow: 2px 0 8px rgba(0, 0, 0, 0.05);
  flex-shrink: 0;
}

/* 桌面端优化的菜单样式增强 */
.el-menu {
  border-right: none;
  height: 100%;
  background: transparent;
  padding: 16px 0;
}

.el-menu-item,
.el-sub-menu__title {
  height: 48px;
  line-height: 48px;
  padding: 0 20px;
  font-size: 15px;
  transition: all 0.2s ease;
  border-radius: 0 12px 12px 0;
  margin: 2px 0;
  color: #1c1c1e;
}

.el-menu-item:hover,
.el-sub-menu__title:hover {
  background-color: #f2f2f7 !important;
  color: #007aff !important;
  padding-left: 22px;
}

.el-menu-item.is-active {
  background-color: rgba(0, 122, 255, 0.1) !important;
  color: #007aff !important;
  font-weight: 600 !important;
  border-right: 3px solid #007aff;
  box-shadow: 0 2px 8px rgba(0, 122, 255, 0.15);
}

.el-menu-item .el-icon,
.el-sub-menu__title .el-icon {
  font-size: 18px;
  margin-right: 12px;
  color: #8e8e93;
}

.el-menu-item.is-active .el-icon,
.el-sub-menu__title:hover .el-icon {
  color: #007aff;
}

.el-sub-menu .el-menu {
  background-color: rgba(249, 249, 250, 0.8);
  padding: 0;
}

.el-sub-menu .el-menu-item {
  padding-left: 52px;
}

/* 桌面端优化的主内容区域 */
.el-main {
  padding: 24px;
  background-color: #f5f5f7;
  overflow-y: auto;
  flex: 1;
  max-width: 100%;
}

/* 移除移动端响应式样式 */
@media (max-width: 768px) {
  .aside {
    position: fixed;
    left: -240px;
    z-index: 99;
    width: 240px;
    height: calc(100vh - 60px);
    border-right: 1px solid #e5e5ea;
    box-shadow: 2px 0 10px rgba(0, 0, 0, 0.1);
  }
  
  .aside.show {
    left: 0;
  }
  
  .el-main {
    padding: 12px;
  }
  
  .header {
    padding: 0 12px;
  }
  
  .title {
    font-size: 18px;
  }
}

/* 滚动时增强毛玻璃效果的JavaScript钩子 */
.scrolled-blur {
  backdrop-filter: blur(24px) !important;
  background-color: rgba(255, 255, 255, 0.95) !important;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
}
</style>