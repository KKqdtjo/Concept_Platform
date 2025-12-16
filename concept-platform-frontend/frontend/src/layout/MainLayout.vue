<template>
  <div class="common-layout">
    <el-container>
      <el-aside width="200px">
        <div class="logo">概念验证平台</div>
        <el-menu
          :default-active="activeMenu"
          class="el-menu-vertical-demo"
          router
        >
           <!-- APPLICANT (申报人) 菜单 -->
          <template v-if="userRole === 'APPLICANT' || userRole === 'student'">
             <el-menu-item index="/my-projects">
              <el-icon><document /></el-icon>
              <span>我的项目</span>
            </el-menu-item>
             <el-menu-item index="/project/add">
              <el-icon><plus /></el-icon>
              <span>新建申报</span>
            </el-menu-item>
          </template>

          <!-- ADMIN (管理员) 菜单 -->
          <template v-if="userRole === 'ADMIN'">
             <el-menu-item index="/audit-projects">
              <el-icon><finished /></el-icon>
              <span>项目审核</span>
            </el-menu-item>
          </template>

          <!-- EXPERT (专家) 菜单 -->
          <template v-if="userRole === 'EXPERT'">
             <el-menu-item index="/expert-reviews">
              <el-icon><chat-dot-square /></el-icon>
              <span>评审任务</span>
            </el-menu-item>
          </template>
        </el-menu>
      </el-aside>
      <el-container>
        <el-header>
          <div class="header-content">
            <span>当前用户: {{ username }} ({{ userRole }})</span>
            <el-button link @click="logout">退出登录</el-button>
          </div>
        </el-header>
        <el-main>
          <router-view />
        </el-main>
      </el-container>
    </el-container>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRouter, useRoute } from 'vue-router'
import { Document, Plus, Finished, ChatDotSquare } from '@element-plus/icons-vue'

const router = useRouter()
const route = useRoute()

const user = ref({})
const username = computed(() => user.value.username || 'Unknown')
const userRole = computed(() => user.value.role || 'GUEST') // 榛樿 GUEST

const activeMenu = computed(() => route.path)

onMounted(() => {
  const storedUser = localStorage.getItem('user')
  if (storedUser) {
    try {
      user.value = JSON.parse(storedUser)
      // 鍏煎涔嬪墠鐨勬ā鎷熺櫥褰曢€昏緫锛屽鏋滄病鏈� role 榛樿缁欎竴涓紝鎴栬€呯‘淇濈櫥褰曟椂鍐欏叆 role
      if (!user.value.role && user.value.username === 'student') {
        user.value.role = 'APPLICANT'
      }
    } catch (e) {
      console.error('Failed to parse user info', e)
    }
  }
})

const logout = () => {
  localStorage.removeItem('user')
  router.push('/login')
}
</script>

<style scoped>
.common-layout{height:100vh}
.el-container{height:100%}

/* 侧边栏 / 头部 / 主区域 使用玻璃质感面板 */
.el-aside{
  background: linear-gradient(180deg, rgba(10,10,16,0.6), rgba(18,20,32,0.45));
  border-right: 1px solid rgba(255,255,255,0.03);
  backdrop-filter: blur(10px);
  padding-top:8px;
}
.logo{
  height:64px;line-height:64px;text-align:center;font-size:15px;font-weight:700;color:var(--accent);border-bottom:1px solid rgba(255,255,255,0.03);letter-spacing:0.6px
}
.el-header{
  background: linear-gradient(180deg, rgba(10,12,20,0.22), transparent);
  border-bottom:1px solid rgba(255,255,255,0.02);
  display:flex;justify-content:flex-end;align-items:center;padding:8px 20px
}
.header-content{display:flex;gap:12px;align-items:center;color:rgba(235,245,255,0.9)}
.el-main{padding:20px}

/* 使用科技风标题 */
.logo, .card-header{font-family:'Orbitron','Rajdhani','Exo 2','Poppins','Microsoft YaHei',sans-serif}

/* 面板基类，应用到内部视图 */
.el-main .panel{background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));border:1px solid rgba(255,255,255,0.04);border-radius:10px;padding:18px}

/* 菜单样式优化，使用高性能 transform 交互 */
::v-deep .el-menu{
  background:transparent;border-right:0;color:rgba(210,230,255,0.8)
}
::v-deep .el-menu-item{
  transition:transform var(--smooth-time) cubic-bezier(.2,.9,.25,1),color var(--smooth-time);
  border-radius:8px;margin:6px 8px;padding:10px 12px
}
::v-deep .el-menu-item:hover{transform:translateX(6px);background:linear-gradient(90deg, rgba(0,208,255,0.04), rgba(124,77,255,0.03));color:var(--accent)}
::v-deep .el-menu-item.is-active{background:linear-gradient(90deg, rgba(0,208,255,0.06), rgba(124,77,255,0.04));color:var(--accent)}

/* 调整按钮样式 */
::v-deep .el-button--link{color:rgba(200,230,255,0.9)}

</style>

