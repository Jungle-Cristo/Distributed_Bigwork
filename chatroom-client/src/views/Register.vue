<template>
  <div class="register-container">
    <div class="particles-bg">
      <canvas ref="canvas" class="particles-canvas"></canvas>
    </div>

    <div class="content-wrapper">
      <div class="register-card glass-effect">
        <div class="card-header">
          <div class="logo-wrapper">
            <div class="logo">
              <el-icon :size="40" color="white"><Plus /></el-icon>
            </div>
          </div>
          <h1 class="title">创建账号</h1>
          <p class="subtitle">加入我们，开始新的社交之旅</p>
        </div>

        <el-form :model="form" :rules="rules" ref="formRef" label-width="0">
          <el-form-item prop="nickname">
            <div class="input-group">
              <el-icon class="input-icon"><UserFilled /></el-icon>
              <el-input v-model="form.nickname" placeholder="昵称" size="large" />
            </div>
          </el-form-item>
          <el-form-item prop="password">
            <div class="input-group">
              <el-icon class="input-icon"><Lock /></el-icon>
              <el-input v-model="form.password" type="password" placeholder="密码（至少6位）" size="large"
                show-password />
            </div>
          </el-form-item>
          <el-form-item prop="confirmPassword">
            <div class="input-group">
              <el-icon class="input-icon"><Lock /></el-icon>
              <el-input v-model="form.confirmPassword" type="password" placeholder="确认密码" size="large"
                show-password />
            </div>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" size="large" class="register-btn" @click="handleRegister" :loading="loading">
              <span v-if="!loading">注册</span>
              <span v-else>注册中...</span>
            </el-button>
          </el-form-item>
        </el-form>

        <div class="footer">
          <span>已有账号？</span>
          <router-link to="/login" class="login-link">返回登录</router-link>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { reactive, ref, onMounted, onUnmounted } from 'vue'
import { useRouter } from 'vue-router'
import { useUserStore } from '../store/user'
import { ElMessage } from 'element-plus'
import { UserFilled, Lock, Plus } from '@element-plus/icons-vue'

const router = useRouter()
const userStore = useUserStore()
const formRef = ref(null)
const loading = ref(false)
const canvas = ref(null)
let animationId = null

const form = reactive({
  nickname: '',
  password: '',
  confirmPassword: ''
})

const validateConfirm = (rule, value, callback) => {
  if (value !== form.password) {
    callback(new Error('两次密码输入不一致'))
  } else {
    callback()
  }
}

const rules = {
  nickname: [
    { required: true, message: '请输入昵称', trigger: 'blur' },
    { min: 1, max: 50, message: '昵称长度为1-50个字符', trigger: 'blur' }
  ],
  password: [
    { required: true, message: '请输入密码', trigger: 'blur' },
    { min: 6, message: '密码至少6位', trigger: 'blur' }
  ],
  confirmPassword: [
    { required: true, message: '请确认密码', trigger: 'blur' },
    { validator: validateConfirm, trigger: 'blur' }
  ]
}

async function handleRegister() {
  const valid = await formRef.value.validate().catch(() => false)
  if (!valid) return

  loading.value = true
  try {
    const result = await userStore.register({
      password: form.password,
      nickname: form.nickname
    })
    ElMessage.success('注册成功！您的账号是：' + result.user.username + '，请牢记')
    router.push('/chat')
  } catch (e) {
    // Error handled by interceptor
  } finally {
    loading.value = false
  }
}

function initParticles() {
  const ctx = canvas.value.getContext('2d')
  canvas.value.width = window.innerWidth
  canvas.value.height = window.innerHeight

  const particles = []
  const particleCount = 80

  for (let i = 0; i < particleCount; i++) {
    particles.push({
      x: Math.random() * canvas.value.width,
      y: Math.random() * canvas.value.height,
      size: Math.random() * 2 + 1,
      speedX: (Math.random() - 0.5) * 0.5,
      speedY: (Math.random() - 0.5) * 0.5,
      opacity: Math.random() * 0.5 + 0.2
    })
  }

  function animate() {
    ctx.clearRect(0, 0, canvas.value.width, canvas.value.height)

    particles.forEach((particle, index) => {
      particle.x += particle.speedX
      particle.y += particle.speedY

      if (particle.x < 0 || particle.x > canvas.value.width) particle.speedX *= -1
      if (particle.y < 0 || particle.y > canvas.value.height) particle.speedY *= -1

      ctx.beginPath()
      ctx.arc(particle.x, particle.y, particle.size, 0, Math.PI * 2)
      ctx.fillStyle = `rgba(29, 29, 31, ${particle.opacity})`
      ctx.fill()

      for (let j = index + 1; j < particles.length; j++) {
        const dx = particles[j].x - particle.x
        const dy = particles[j].y - particle.y
        const distance = Math.sqrt(dx * dx + dy * dy)

        if (distance < 120) {
          ctx.beginPath()
          ctx.moveTo(particle.x, particle.y)
          ctx.lineTo(particles[j].x, particles[j].y)
          ctx.strokeStyle = `rgba(29, 29, 31, ${0.15 * (1 - distance / 120)})`
          ctx.lineWidth = 1
          ctx.stroke()
        }
      }
    })

    animationId = requestAnimationFrame(animate)
  }

  animate()

  const handleResize = () => {
    canvas.value.width = window.innerWidth
    canvas.value.height = window.innerHeight
  }

  window.addEventListener('resize', handleResize)

  onUnmounted(() => {
    window.removeEventListener('resize', handleResize)
    if (animationId) {
      cancelAnimationFrame(animationId)
    }
  })
}

onMounted(() => {
  initParticles()
})
</script>

<style scoped>
.register-container {
  height: 100vh;
  width: 100vw;
  display: flex;
  justify-content: center;
  align-items: center;
  position: relative;
  overflow: hidden;
  background: linear-gradient(180deg, #fbfbfd 0%, #f5f5f7 100%);
  margin: 0;
  padding: 0;
}

.particles-bg {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  width: 100%;
  height: 100%;
  z-index: 0;
}

.particles-canvas {
  display: block;
  width: 100%;
  height: 100%;
}

.content-wrapper {
  position: relative;
  z-index: 10;
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100%;
  margin: 0;
  padding: 0;
}

.register-card {
  width: 520px;
  max-width: 90vw;
  padding: 56px 64px;
  border-radius: var(--apple-radius-xl);
  box-shadow: var(--apple-shadow-xl);
  position: relative;
  animation: card-appear 0.6s ease-out;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  align-items: center;
}

@keyframes card-appear {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.card-header {
  text-align: center;
  margin-bottom: 40px;
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.logo-wrapper {
  margin-bottom: 20px;
  width: 100%;
  display: flex;
  justify-content: center;
}

.logo {
  width: 72px;
  height: 72px;
  background: linear-gradient(135deg, #0071e3, #5856d6);
  border-radius: var(--apple-radius-xl);
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 8px 24px rgba(0, 113, 227, 0.3);
  animation: logo-float 3s ease-in-out infinite;
}

@keyframes logo-float {
  0%, 100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-4px);
  }
}

.title {
  font-size: 32px;
  font-weight: 600;
  color: var(--apple-text-primary);
  margin: 0 0 8px 0;
  letter-spacing: -0.5px;
  width: 100%;
  text-align: center;
}

.subtitle {
  font-size: 15px;
  color: var(--apple-text-secondary);
  margin: 0;
  font-weight: 400;
  width: 100%;
  text-align: center;
}

.register-card :deep(.el-form) {
  width: 100%;
  margin: 0;
  padding: 0;
}

.register-card :deep(.el-form-item) {
  width: 100%;
  margin: 0 0 16px 0;
  padding: 0;
}

.register-card :deep(.el-form-item:last-of-type) {
  margin-bottom: 0;
}

.input-group {
  position: relative;
  width: 100%;
  margin: 0;
  padding: 0;
}

.input-icon {
  position: absolute;
  left: 16px;
  top: 50%;
  transform: translateY(-50%);
  color: var(--apple-text-secondary);
  font-size: 18px;
  z-index: 1;
}

.input-group :deep(.el-input__wrapper) {
  padding-left: 48px;
  padding-right: 16px;
  border-radius: var(--apple-radius-md);
  height: 48px;
  transition: all var(--apple-transition-fast);
  width: 100%;
  box-sizing: border-box;
}

.input-group :deep(.el-input__inner) {
  font-size: 15px;
  text-align: left;
}

.register-btn {
  width: 100%;
  height: 48px;
  border-radius: var(--apple-radius-md);
  font-size: 15px;
  font-weight: 500;
  margin: 8px 0 0 0;
  letter-spacing: 0;
}

.footer {
  text-align: center;
  color: var(--apple-text-secondary);
  margin: 28px 0 0 0;
  font-size: 14px;
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}

.login-link {
  color: var(--apple-blue);
  font-weight: 500;
  text-decoration: none;
  margin: 0 0 0 4px;
  padding: 0;
  transition: color var(--apple-transition-fast);
}

.login-link:hover {
  color: var(--apple-blue-hover);
  text-decoration: underline;
}

@media (max-width: 600px) {
  .register-card {
    width: 100%;
    padding: 48px 32px;
    border-radius: var(--apple-radius-lg);
  }
  
  .title {
    font-size: 28px;
  }
  
  .subtitle {
    font-size: 14px;
  }
}
</style>
