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
  background: linear-gradient(135deg, #f8fafc 0%, #f1f5f9 50%, #e2e8f0 100%);
  background-image: 
    radial-gradient(circle at 20% 30%, rgba(99, 102, 241, 0.08) 0%, transparent 50%),
    radial-gradient(circle at 80% 70%, rgba(16, 185, 129, 0.08) 0%, transparent 50%),
    radial-gradient(circle at 40% 80%, rgba(245, 158, 11, 0.05) 0%, transparent 50%);
  position: relative;
  overflow: hidden;
}

.common-layout::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: 
    linear-gradient(45deg, transparent 30%, rgba(255, 255, 255, 0.1) 50%, transparent 70%),
    linear-gradient(-45deg, transparent 30%, rgba(255, 255, 255, 0.05) 50%, transparent 70%);
  pointer-events: none;
  z-index: 0;
}

/* 现代化顶部导航栏样式 */
.header {
  height: 70px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 32px;
  z-index: 100;
  position: relative;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.9) 0%, rgba(255, 255, 255, 0.7) 100%);
  backdrop-filter: blur(20px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.2);
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.title {
  color: #1e293b;
  font-size: 24px;
  font-weight: 800;
  letter-spacing: -0.5px;
  display: flex;
  align-items: center;
  gap: 12px;
  background: linear-gradient(135deg, #6366f1 0%, #8b5cf6 50%, #06b6d4 100%);
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  position: relative;
}

.title::before {
  content: '';
  display: inline-block;
  width: 36px;
  height: 36px;
  background: linear-gradient(135deg, #6366f1 0%, #8b5cf6 100%);
  border-radius: 12px;
  background-image: url(../../assets/logo.png);
  background-size: 24px 24px;
  background-repeat: no-repeat;
  background-position: center;
  box-shadow: 0 4px 12px rgba(99, 102, 241, 0.3);
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.title:hover::before {
  transform: scale(1.1) rotate(5deg);
  box-shadow: 0 6px 20px rgba(99, 102, 241, 0.4);
}

/* 现代化右侧工具栏 */
.right_tool {
  display: flex;
  gap: 16px;
  color: #1e293b;
}

.right_tool a {
  color: #6366f1;
  text-decoration: none;
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 12px 20px;
  border-radius: 16px;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  font-size: 14px;
  background: linear-gradient(135deg, rgba(99, 102, 241, 0.1) 0%, rgba(139, 92, 246, 0.1) 100%);
  font-weight: 600;
  border: 1px solid rgba(99, 102, 241, 0.2);
  position: relative;
  overflow: hidden;
}

.right_tool a::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.3), transparent);
  transition: left 0.5s ease;
}

.right_tool a:hover::before {
  left: 100%;
}

.right_tool a:hover {
  background: linear-gradient(135deg, #6366f1 0%, #8b5cf6 100%);
  color: white;
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(99, 102, 241, 0.3);
  border-color: transparent;
}

.right_tool .el-icon {
  font-size: 18px;
  transition: transform 0.3s ease;
}

.right_tool a:hover .el-icon {
  transform: scale(1.1);
}

/* 桌面端优化的主体容器 */
.el-container {
  flex: 1;
  display: flex;
  height: calc(100vh - 60px);
  overflow: hidden;
}

/* 现代化侧边栏样式 */
.aside {
  width: 260px;
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.95) 0%, rgba(255, 255, 255, 0.8) 100%);
  backdrop-filter: blur(20px);
  border-right: 1px solid rgba(255, 255, 255, 0.2);
  overflow-y: auto;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  box-shadow: 4px 0 20px rgba(0, 0, 0, 0.08);
  flex-shrink: 0;
  position: relative;
}

.aside::before {
  content: '';
  position: absolute;
  top: 0;
  right: 0;
  width: 1px;
  height: 100%;
  background: linear-gradient(180deg, transparent, rgba(99, 102, 241, 0.3), transparent);
}

/* 现代化菜单样式系统 */
.el-menu {
  border-right: none;
  height: 100%;
  background: transparent;
  padding: 20px 0;
}

.el-menu-item,
.el-sub-menu__title {
  height: 52px;
  line-height: 52px;
  padding: 0 24px;
  font-size: 15px;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  border-radius: 0 16px 16px 0;
  margin: 4px 0;
  color: #374151;
  font-weight: 500;
  position: relative;
  overflow: hidden;
}

.el-menu-item::before,
.el-sub-menu__title::before {
  content: '';
  position: absolute;
  left: 0;
  top: 0;
  width: 4px;
  height: 100%;
  background: linear-gradient(135deg, #6366f1, #8b5cf6);
  transform: scaleY(0);
  transition: transform 0.3s ease;
  border-radius: 0 2px 2px 0;
}

.el-menu-item:hover,
.el-sub-menu__title:hover {
  background: linear-gradient(135deg, rgba(99, 102, 241, 0.08) 0%, rgba(139, 92, 246, 0.08) 100%) !important;
  color: #6366f1 !important;
  padding-left: 28px;
  transform: translateX(4px);
}

.el-menu-item:hover::before,
.el-sub-menu__title:hover::before {
  transform: scaleY(1);
}

.el-menu-item.is-active {
  background: linear-gradient(135deg, rgba(99, 102, 241, 0.15) 0%, rgba(139, 92, 246, 0.15) 100%) !important;
  color: #6366f1 !important;
  font-weight: 700 !important;
  border-right: 4px solid #6366f1;
  box-shadow: 0 4px 12px rgba(99, 102, 241, 0.2);
  transform: translateX(4px);
}

.el-menu-item.is-active::before {
  transform: scaleY(1);
}

.el-menu-item .el-icon,
.el-sub-menu__title .el-icon {
  font-size: 20px;
  margin-right: 16px;
  color: #9ca3af;
  transition: all 0.3s ease;
}

.el-menu-item.is-active .el-icon,
.el-sub-menu__title:hover .el-icon {
  color: #6366f1;
  transform: scale(1.1);
}

.el-sub-menu .el-menu {
  background: linear-gradient(135deg, rgba(249, 250, 251, 0.8) 0%, rgba(243, 244, 246, 0.8) 100%);
  padding: 0;
  border-radius: 0 16px 16px 0;
  margin: 4px 0;
}

.el-sub-menu .el-menu-item {
  padding-left: 60px;
  font-size: 14px;
  height: 44px;
  line-height: 44px;
}

/* 现代化主内容区域 */
.el-main {
  padding: 32px;
  background: transparent;
  overflow-y: auto;
  flex: 1;
  max-width: 100%;
  position: relative;
  z-index: 1;
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

/* 现代化滚动毛玻璃效果 */
.scrolled-blur {
  backdrop-filter: blur(30px) !important;
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.95) 0%, rgba(255, 255, 255, 0.85) 100%) !important;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.12) !important;
  border-bottom: 1px solid rgba(255, 255, 255, 0.3) !important;
  transform: translateY(-2px) !important;
}
</style>