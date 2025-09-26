<script setup>
import { ref } from 'vue'
import { loginApi } from '@/api/login'
import { ElMessage } from 'element-plus'
import { useRouter } from 'vue-router'

let loginForm = ref({username:'', password:''})
let router = useRouter()

//登录
const login = async () => {
  const result = await loginApi(loginForm.value)
  if (result.code) {// 登录成功
    ElMessage.success('登录成功')
    localStorage.setItem('loginUser', JSON.stringify(result.data))
    router.push('/')// 跳转
  }else {
    ElMessage.error(result.msg)
  }
}

//取消
const cancel = () => {
  loginForm.value = {
    username: '',
    password: ''
  }
}

</script>

<template>
  <div id="container">
    <div class="login-form">
      <el-form label-width="80px">
        <p class="title">智能学习辅助系统</p>
        <el-form-item label="用户名" prop="username">
          <el-input v-model="loginForm.username" placeholder="请输入用户名"></el-input>
        </el-form-item>
        
        <el-form-item label="密码" prop="password">
          <el-input type="password" v-model="loginForm.password" placeholder="请输入密码"></el-input>
        </el-form-item>

        <el-form-item>
          <el-button class="button" type="primary" @click="login">登 录</el-button>
          <el-button class="button" type="info" @click="cancel">重 置</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<style scoped>
/* 基础样式重置 */
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

#container {
  height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  background-image: linear-gradient(135deg, #667eea 0%, #764ba2 100%), url('../../assets/bg1.jpg');
  background-size: cover, cover;
  background-position: center, center;
  background-blend-mode: overlay;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  overflow: hidden;
}

/* 背景装饰元素 */
#container::before {
  content: '';
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: repeating-linear-gradient(
    45deg,
    rgba(255, 255, 255, 0.05),
    rgba(255, 255, 255, 0.05) 10px,
    rgba(255, 255, 255, 0) 10px,
    rgba(255, 255, 255, 0) 20px
  );
  animation: moveBg 60s linear infinite;
  z-index: 0;
}

@keyframes moveBg {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

/* 登录卡片 */
.login-form {
  max-width: 420px;
  width: 100%;
  padding: 48px;
  margin: 0 24px;
  border-radius: 24px;
  background-color: rgba(255, 255, 255, 0.98);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(10px);
  transition: all 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
  position: relative;
  z-index: 2;
  overflow: hidden;
}

/* 卡片装饰 */
.login-form::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 6px;
  background: linear-gradient(90deg, #667eea 0%, #764ba2 100%);
}

.login-form:hover {
  transform: translateY(-8px);
  box-shadow: 0 30px 60px rgba(0, 0, 0, 0.15);
}

/* 标题样式 */
.title {
  font-size: 32px;
  font-family: 'PingFang SC', 'Microsoft YaHei', sans-serif;
  text-align: center;
  margin-bottom: 40px;
  font-weight: 700;
  color: #2c3e50;
  letter-spacing: -0.5px;
  position: relative;
}

.title::after {
  content: '';
  position: absolute;
  bottom: -12px;
  left: 50%;
  transform: translateX(-50%);
  width: 50px;
  height: 4px;
  background: linear-gradient(90deg, #667eea 0%, #764ba2 100%);
  border-radius: 2px;
}

/* 表单样式 */
.el-form {
  width: 100%;
}

.el-form-item {
  margin-bottom: 30px;
  position: relative;
}

.el-form-item__label {
  font-size: 14px;
  font-weight: 600;
  color: #555;
  padding: 0;
  line-height: 48px;
  width: 100%;
  display: block;
  margin-bottom: 8px;
}

.el-input {
  width: 100%;
}

.el-input__wrapper {
  border-radius: 12px;
  transition: all 0.3s ease;
  border: 1px solid #e1e5e9;
  height: 48px;
  background-color: #f8fafc;
}

.el-input__wrapper:hover {
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
  background-color: white;
}

.el-input__wrapper.is-focus {
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.2);
  border-color: #667eea;
  background-color: white;
}

.el-input__inner {
  font-size: 15px;
  color: #333;
  height: 48px;
  line-height: 48px;
}

/* 按钮样式 */
.button {
  margin-top: 10px;
  width: 100%;
  height: 50px;
  font-size: 16px;
  font-weight: 600;
  border-radius: 12px;
  transition: all 0.3s ease;
  border: none;
  letter-spacing: 1px;
  text-transform: uppercase;
  padding: 0;
}

/* 登录按钮 */
.el-button--primary {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  background-size: 200% auto;
  transition: all 0.4s ease;
  color: white;
}

.el-button--primary:hover {
  background-position: right center;
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(102, 126, 234, 0.4);
  color: white;
}

.el-button--primary:active {
  transform: translateY(0);
}

/* 重置按钮 */
.el-button--info {
  background-color: #f8fafc;
  color: #64748b;
  border: 1px solid #e2e8f0;
}

.el-button--info:hover {
  background-color: #eff6ff;
  color: #667eea;
  border-color: #bfdbfe;
}

/* 按钮容器 */
.el-form-item:last-child {
  display: flex !important;
  justify-content: center !important;
  gap: 20px;
  margin-top: 40px;
  width: 100%;
  padding-left: 0 !important;
  padding-right: 0 !important;
}

/* 动画效果 */
@keyframes pulse {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.02);
  }
  100% {
    transform: scale(1);
  }
}

/* 加载动画 */
.el-button--loading {
  animation: pulse 1.5s infinite;
}

/* 响应式设计 */
@media (max-width: 576px) {
  #container {
    background-position: center;
  }
  
  .login-form {
    padding: 32px 24px;
    margin: 0 16px;
    border-radius: 20px;
  }
  
  .title {
    font-size: 28px;
    margin-bottom: 32px;
  }
  
  .title::after {
    width: 40px;
    bottom: -10px;
  }
  
  .el-form-item {
    margin-bottom: 24px;
  }
  
  .el-input__wrapper {
    height: 44px;
  }
  
  .el-input__inner {
    height: 44px;
    line-height: 44px;
    font-size: 14px;
  }
  
  .button {
    height: 46px;
    font-size: 15px;
  }
  
  .el-form-item:last-child {
    margin-top: 32px;
    gap: 12px;
  }
}

/* 输入框聚焦动画 */
.el-input__wrapper:focus-within {
  transform: translateY(-1px);
}

/* 错误提示样式调整 */
.el-message {
  min-width: 300px;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}
</style>