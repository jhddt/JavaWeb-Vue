<script setup>
import { ref, reactive, onMounted } from 'vue'
import { loginApi } from '@/api/login'
import { ElMessage } from 'element-plus'
import { useRouter } from 'vue-router'
// 导入Element Plus图标
import { User, Lock } from '@element-plus/icons-vue'

// 表单数据使用reactive以便更好的类型支持
const loginForm = reactive({ username: '', password: '' })
const router = useRouter()
const loading = ref(false) // 添加加载状态
const rememberMe = ref(false) // 记住我选项
const formRules = reactive({
  username: [{ required: true, message: '请输入用户名', trigger: 'blur' }],
  password: [{ required: true, message: '请输入密码', trigger: 'blur' }]
})

// 添加Enter键登录功能
const handleKeyPress = (e) => {
  if (e.key === 'Enter') {
    login()
  }
}

// 登录函数优化 - 添加加载状态和错误处理
const login = async () => {
  if (!loginForm.username || !loginForm.password) {
    ElMessage.warning('请输入用户名和密码')
    return
  }
  
  loading.value = true
  try {
    const result = await loginApi(loginForm) // 移除错误的.value
    if (result.code) { // 登录成功
      ElMessage.success('登录成功')
      localStorage.setItem('loginUser', JSON.stringify(result.data))
      // 添加延迟提升体验
      setTimeout(() => {
        router.push('/') // 跳转
      }, 500)
    } else {
      ElMessage.error(result.msg || '登录失败，请检查用户名和密码')
    }
  } catch (error) {
    ElMessage.error('登录失败，请重试')
    console.error('Login error:', error)
  } finally {
    loading.value = false
  }
}

// 重置函数
const resetForm = () => {
  Object.keys(loginForm).forEach(key => {
    loginForm[key] = ''
  })
}

// 页面加载时聚焦到用户名输入框
onMounted(() => {
  setTimeout(() => {
    const usernameInput = document.querySelector('input[placeholder="请输入用户名"]')
    if (usernameInput) {
      usernameInput.focus()
    }
  }, 300)
})

</script>

<template>
  <div id="container">
    <!-- 添加背景装饰元素，增强毛玻璃效果 -->
    <div class="background-decorations">
      <div class="bg-circle bg-circle-1"></div>
      <div class="bg-circle bg-circle-2"></div>
      <div class="bg-circle bg-circle-3"></div>
    </div>
    
    <!-- 登录表单 - 应用毛玻璃效果 -->
    <div class="login-form ios-blur-strong">
      <el-form :model="loginForm" :rules="formRules" ref="loginFormRef">
        <!-- 标题区域添加Logo -->
        <div class="title-container">
          <img class="logo" src="@/assets/生成得智教云系统 logo.png" alt="智教云系统Logo">
          <h1 class="title">智教云</h1>
          <p class="subtitle">智慧教育管理平台</p>
        </div>
        
        <el-form-item label="用户名" prop="username">
          <el-input 
            v-model="loginForm.username" 
            placeholder="请输入用户名" 
            @keyup.enter="handleKeyPress"
            :prefix-icon="User"
            class="login-input"
          ></el-input>
        </el-form-item>
        
        <el-form-item label="密码" prop="password">
          <el-input 
            type="password" 
            v-model="loginForm.password" 
            placeholder="请输入密码" 
            @keyup.enter="handleKeyPress"
            :prefix-icon="Lock"
            class="login-input"
          ></el-input>
        </el-form-item>

        <el-form-item>
          <!-- 单独用div包裹按钮并居中 -->
          <div class="button-container">
            <div class="button-group">
              <el-button 
                class="button"
                type="primary" 
                @click="login"
                :loading="loading"
              >
                <template v-if="!loading">登 录</template>
                <template v-else>登录中...</template>
              </el-button>
              <el-button 
                class="button" 
                @click="resetForm"
              >
                重 置
              </el-button>
            </div>
          </div>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<style scoped>
/* iOS 26 风格基础样式重置 */
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

#container {
  height: 100vh;
  background-color: #f5f5f7;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Display', 'PingFang SC', 'Helvetica Neue', sans-serif;
  overflow: hidden;
}

/* iOS 风格登录卡片 - 适配毛玻璃效果 */
.login-form {
  max-width: 360px;
  width: 100%;
  padding: 32px;
  margin: 0 20px;
  border-radius: 22px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.08);
  transition: all 0.3s cubic-bezier(0.25, 0.1, 0.25, 1);
  position: relative;
  z-index: 2;
  overflow: hidden;
}

.login-form:hover {
  transform: translateY(-2px);
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.1);
}

/* 标题容器样式 */
.title-container {
  text-align: center;
  margin-bottom: 36px;
}

/* Logo样式 */
.logo {
  width: 64px;
  height: 64px;
  border-radius: 16px;
  margin: 0 auto 16px;
  box-shadow: 0 4px 20px rgba(102, 126, 234, 0.3);
  object-fit: contain;
}

/* iOS 风格标题样式 */
.title {
  font-size: 28px;
  text-align: center;
  margin-bottom: 8px;
  font-weight: 700;
  color: #1c1c1e;
  letter-spacing: -0.5px;
}

.subtitle {
  font-size: 14px;
  color: #8e8e93;
  text-align: center;
  font-weight: 400;
}

/* 记住我和忘记密码样式 */
.remember-me {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 24px;
}

.forgot-password {
  color: #007aff;
  text-decoration: none;
  font-size: 14px;
  transition: opacity 0.2s ease;
}

.forgot-password:hover {
  opacity: 0.8;
  text-decoration: underline;
}

/* iOS 风格表单样式 */
.el-form {
  width: 100%;
}

.el-form-item {
  margin-bottom: 24px;
  position: relative;
}

.el-form-item__label {
  font-size: 14px;
  font-weight: 500;
  color: #8e8e93;
  padding: 0;
  line-height: 20px;
  width: 100%;
  display: block;
  margin-bottom: 8px;
}

.el-input {
  width: 100%;
}

/* 确保登录表单输入框等宽的样式 */
.login-input {
  width: 100% !important;
}

.el-input__wrapper {
    border-radius: 12px !important;
    transition: all 0.2s ease;
    border: 1px solid #e5e5ea !important;
    height: 46px;
    background-color: #f5f5f7 !important; /* iOS风格浅灰色背景 */
    position: relative;
    z-index: 1; /* 确保输入框内容在最上层 */
  }

  .el-input__wrapper:hover {
    border-color: #007aff !important;
    box-shadow: 0 0 0 3px rgba(0, 122, 255, 0.1) !important;
    background-color: #f9f9fa !important;
  }

  .el-input__wrapper.is-focus {
    box-shadow: 0 0 0 3px rgba(0, 122, 255, 0.2) !important;
    border-color: #007aff !important;
    background-color: white !important;
  }

/* iOS 风格按钮样式 */
.el-button {
  height: 46px;
  font-size: 16px;
  font-weight: 600;
  border-radius: 12px !important;
  transition: all 0.2s ease;
  background-color: #007aff !important;
  border: none !important;
  color: white !important;
  letter-spacing: 0.5px;
  box-shadow: 0 2px 8px rgba(0, 122, 255, 0.3);
}

/* 按钮容器 - 确保整个按钮组居中 */
.button-container {
  display: flex;
  justify-content: center;
  width: 100%;
}

/* 按钮组样式 - 确保登录和重置按钮平级 */
.button-group {
  display: flex;
  gap: 12px;
  margin-top: 8px;
  align-items: stretch;
  width: 100%;
  max-width: 400px; /* 设置最大宽度 */
}

.button-group .el-button {
  flex: 1; /* 两个按钮完全平级，平均分配宽度 */
  height: 46px;
  display: flex;
  align-items: center;
  justify-content: center;
}

/* 确保重置按钮样式与登录按钮一致 */
.button-group .el-button[type="info"] {
  background-color: #f5f5f7 !important;
  color: #1c1c1e !important;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.button-group .el-button[type="info"]:hover {
  background-color: #e5e5ea !important;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.15);
}

/* 输入框图标样式优化 */
.el-input .el-input__prefix {
  left: 12px;
}

/* 复选框样式优化 */
.el-checkbox__label {
  color: #8e8e93;
  font-size: 14px;
}

/* 加载状态样式 */
.el-button--loading .el-button__loading-icon {
  margin-right: 4px;
}

/* 动画效果增强 */
@keyframes pulse-subtle {
  0%, 100% {
    opacity: 0.7;
  }
  50% {
    opacity: 0.9;
  }
}

.logo {
  animation: pulse-subtle 3s ease-in-out infinite;
}

/* 表单元素的动画效果 */
.el-form-item {
  animation: fadeIn 0.3s ease-out;
}

/* 顺序动画 */
.el-form-item:nth-child(1) { animation-delay: 0.1s; }
.el-form-item:nth-child(2) { animation-delay: 0.2s; }
.el-form-item:nth-child(3) { animation-delay: 0.3s; }
.el-form-item:nth-child(4) { animation-delay: 0.4s; }

.el-button:hover {
  transform: translateY(-1px);
  box-shadow: 0 4px 16px rgba(0, 122, 255, 0.4);
  color: white !important;
}

.el-button:active {
  transform: translateY(0);
}

.el-button:disabled {
  background-color: #c7c7cc !important;
  box-shadow: none !important;
}

/* iOS 风格链接样式 */
.link {
  color: #007aff;
  text-decoration: none;
  font-size: 14px;
  transition: color 0.2s ease;
}

.link:hover {
  color: #0056cc;
  text-decoration: none;
}

/* iOS 风格错误提示 */
.el-form-item__error {
  color: #ff3b30;
  font-size: 12px;
  line-height: 1;
  padding-top: 4px;
}

/* iOS 风格动画效果 */
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.login-form {
  animation: fadeIn 0.5s ease-out;
}

/* iOS 风格响应式设计 */
@media (max-width: 768px) {
  .login-form {
    padding: 24px;
    margin: 0 16px;
    border-radius: 20px;
  }
  
  .title {
    font-size: 24px;
    margin-bottom: 28px;
  }
  
  .el-form-item {
    margin-bottom: 20px;
  }
  
  .el-button {
    height: 44px;
  }
}

/* iOS 风格辅助文字 */
.helper-text {
  color: #8e8e93;
  font-size: 13px;
  text-align: center;
  margin-top: 16px;
  opacity: 0.8;
}

/* 背景装饰元素样式 - 增强毛玻璃效果的视觉体验 */
.background-decorations {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 1;
  overflow: hidden;
}

.bg-circle {
  position: absolute;
  border-radius: 50%;
  filter: blur(100px);
  opacity: 0.4;
  transform: scale(1);
  animation: pulse 8s infinite ease-in-out;
}

.bg-circle-1 {
  width: 600px;
  height: 600px;
  top: -300px;
  left: -200px;
  background: linear-gradient(135deg, #007aff, #5ac8fa);
  animation-delay: 0s;
}

.bg-circle-2 {
  width: 500px;
  height: 500px;
  bottom: -200px;
  right: -150px;
  background: linear-gradient(135deg, #34c759, #30d158);
  animation-delay: 2s;
}

.bg-circle-3 {
  width: 400px;
  height: 400px;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: linear-gradient(135deg, #ff2d55, #ff3b30);
  animation-delay: 4s;
}

@keyframes pulse {
  0%, 100% {
    transform: scale(1);
    opacity: 0.4;
  }
  50% {
    transform: scale(1.1);
    opacity: 0.5;
  }
}
</style>