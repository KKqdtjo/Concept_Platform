<template>
  <div class="login-container fade-in-up">
    <div class="login-decor">
      <div class="geo-shape shape-a"></div>
      <div class="geo-shape shape-b"></div>
    </div>
    <el-card class="login-card panel">
      <template #header>
        <div class="card-header">
          <span>登录</span>
        </div>
      </template>
      <el-form :model="form" :rules="rules" ref="loginFormRef" label-width="60px">
        <el-form-item label="账号" prop="username">
          <el-input v-model="form.username" placeholder="请输入账号" />
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="form.password" type="password" placeholder="请输入密码" show-password @keyup.enter="handleLogin" />
        </el-form-item>
        <el-form-item>
          <div style="width: 100%;">
            <el-button type="primary" class="login-button btn-tech" :loading="loading" @click="handleLogin">登录</el-button>
            <div style="text-align: right; margin-top: 10px;">
              <el-link type="primary" @click="handleRegister">没有账号？立即注册</el-link>
            </div>
          </div>
        </el-form-item>
      </el-form>
    </el-card>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'
import { useRouter } from 'vue-router'
import { login } from '@/api/user'
import { ElMessage } from 'element-plus'

const router = useRouter()
const loginFormRef = ref(null)
const loading = ref(false)

const form = reactive({
  username: '',
  password: ''
})

const rules = {
  username: [{ required: true, message: '请输入账号', trigger: 'blur' }],
  password: [{ required: true, message: '请输入密码', trigger: 'blur' }]
}

const handleRegister = () => {
  console.log('跳转注册页面...') // 调试用
  router.push('/register')
}

const handleLogin = async () => {
  if (!loginFormRef.value) return
  
  await loginFormRef.value.validate(async (valid) => {
    if (valid) {
      loading.value = true
      
      try {
        const res = await login(form)
        
        // 1. 鎵撳嵃鍚庣杩斿洖鐨勬暟鎹紝渚夸簬璋冭瘯
        console.log('Login API Response:', res)

        // 2. 鏁版嵁澶勭悊涓庢槧灏�
        // 纭繚 user 瀵硅薄涓寘鍚� userId
        const userObj = { ...res }
        
        // 鏄犲皠 user_id -> userId
        if (userObj.user_id && !userObj.userId) {
          userObj.userId = userObj.user_id
        }
        // 鏄犲皠 id -> userId (濡傛灉鍚庣杩斿洖鐨勬槸 id)
        if (userObj.id && !userObj.userId) {
          userObj.userId = userObj.id
        }

        // 3. 瀛樺叆 localStorage
        localStorage.setItem('user', JSON.stringify(userObj)) 
        
        ElMessage.success('登录成功')
        router.push('/')
      } catch (error) {
        console.error('登录失败', error)
      } finally {
        loading.value = false
      }
    }
  })
}
</script>



<style scoped>
.login-container{display:flex;align-items:flex-start;justify-content:center;min-height:calc(100vh - 0px);padding:20px 20px 40px;position:relative;background:transparent}
.login-decor{position:absolute;inset:0;z-index:0;pointer-events:none}
.login-decor .geo-shape{position:absolute;opacity:0.14;filter:blur(6px);transform:translateZ(0)}
.login-decor .shape-a{width:220px;height:220px;right:6%;top:8%;background:linear-gradient(135deg,var(--accent-2),transparent);clip-path:polygon(50% 0,100% 38%,82% 100%,18% 100%,0 38%)}
.login-decor .shape-b{width:140px;height:140px;left:8%;bottom:6%;background:linear-gradient(225deg,var(--accent),transparent);border-radius:50%}

.login-card{width:420px;padding:12px 24px 20px 24px;position:relative;z-index:2;margin-top:-72px;transform:translateY(-8px)}
.card-header{font-weight:700;text-align:center;color:var(--accent);font-size:20px;margin-bottom:6px}
.login-button{width:100%}

/* 把表单上移 */
.login-card ::v-deep .el-form{margin-top:-10px}

/* Input glow and underline like CSDN tech login */
::v-deep .el-input__inner,
::v-deep input.el-input__inner,
::v-deep textarea.el-textarea__inner{
  background: rgba(255,255,255,0.06);
  border:1px solid rgba(0,0,0,0.08);
  color:#000 !important;
  transition:box-shadow var(--smooth-time),border-color var(--smooth-time);
  border-radius:8px;
}
::v-deep .el-input__inner::placeholder,
::v-deep input.el-input__inner::placeholder,
::v-deep textarea.el-textarea__inner::placeholder{
  color: rgba(0,0,0,0.45) !important;
}
::v-deep .el-input__inner:focus,
::v-deep input.el-input__inner:focus,
::v-deep textarea.el-textarea__inner:focus{outline:none;border-color:var(--accent);box-shadow:0 6px 24px rgba(0,243,255,0.06),0 0 8px rgba(0,243,255,0.04) inset}

/* link style */
::v-deep .el-link{color:var(--accent);opacity:0.95}

/* reduce card background to glass look */
.panel{padding:22px}

/* small devices */
@media (max-width:480px){.login-card{width:92vw;padding:12px;margin-top:-24px;transform:none}}

</style>
