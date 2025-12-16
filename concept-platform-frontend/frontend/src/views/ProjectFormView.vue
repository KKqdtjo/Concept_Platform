<template>
  <div class="project-form-container fade-in-up">
    <el-card class="panel">
      <template #header>
        <div class="card-header">
          <span>项目申报</span>
          <el-button @click="$router.back()">返回</el-button>
        </div>
      </template>

      <el-form :model="form" :rules="rules" ref="formRef" label-width="100px">
        <el-form-item label="项目名称" prop="projectName">
          <el-input v-model="form.projectName" placeholder="请输入项目名称" />
        </el-form-item>

        <el-form-item label="技术领域" prop="techDomain">
          <el-input v-model="form.techDomain" placeholder="请输入技术领域" />
        </el-form-item>

        <el-form-item label="项目简介" prop="description">
          <el-input v-model="form.description" type="textarea" rows="4" placeholder="请输入项目简介" />
        </el-form-item>

        <el-form-item label="附件上传" prop="attachmentUrl">
          <!-- 
            action: 涓婁紶鎺ュ彛鍦板潃锛岀‖缂栫爜涓哄悗绔湴鍧€
            on-success: 涓婁紶鎴愬姛鍚庣殑鍥炶皟
            limit: 闄愬埗涓婁紶1涓枃浠�
          -->
          <el-upload
            class="upload-demo"
            action="http://localhost:8080/api/common/upload"
            :on-success="handleUploadSuccess"
            :on-error="handleUploadError"
            :limit="1"
            :file-list="fileList"
          >
            <el-button type="primary">点击上传</el-button>
            <template #tip>
              <div class="el-upload__tip">
                只能上传 jpg/png/pdf/doc 文件，且不超过 500kb
              </div>
            </template>
          </el-upload>
        </el-form-item>

        <el-form-item>
          <el-button type="primary" @click="handleSubmit" :loading="loading">立即申报</el-button>
          <el-button @click="resetForm">重置</el-button>
        </el-form-item>
      </el-form>
    </el-card>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'
import { useRouter } from 'vue-router'
import { addProject } from '@/api/project'
import { ElMessage } from 'element-plus'

const router = useRouter()
const formRef = ref(null)
const loading = ref(false)
const fileList = ref([])

const form = reactive({
  projectName: '',
  techDomain: '',
  description: '',
  attachmentUrl: '' // 瀛樺偍涓婁紶鎴愬姛鍚庣殑 URL
})

const rules = {
  projectName: [{ required: true, message: '请输入项目名称', trigger: 'blur' }],
  techDomain: [{ required: true, message: '请输入技术领域', trigger: 'blur' }],
  attachmentUrl: [{ required: true, message: '请上传附件', trigger: 'change' }]
}

// 涓婁紶鎴愬姛鍥炶皟
const handleUploadSuccess = (response, uploadFile) => {
  // 鍋囪鍚庣杩斿洖鏍煎紡涓� { code: 200, msg: "success", data: "http://xxx/file.pdf" }
  if (response.code === 200) {
    form.attachmentUrl = response.data
    ElMessage.success('上传成功')
  } else {
    ElMessage.error(response.msg || '上传失败')
    // 如果失败，可以清空文件列表
    fileList.value = []
  }
}

const handleUploadError = (error) => {
  console.error(error)
  ElMessage.error('上传失败，请检查网络或后端接口')
}

const handleSubmit = async () => {
  if (!formRef.value) return
  
  await formRef.value.validate(async (valid) => {
    if (valid) {
      loading.value = true
      try {
        // 1. 鑾峰彇鐢ㄦ埛淇℃伅
        const userStr = localStorage.getItem('user')
        let user = {}
        if (userStr) {
          try {
            user = JSON.parse(userStr)
          } catch (e) {
            console.error('JSON Parse Error', e)
          }
        }
        
        // 鍏抽敭鐐癸細鎵撳嵃璋冭瘯
        console.log('Current User:', user)

        // 2. 鏋勫缓鎻愪氦鏁版嵁
        const submitData = {
          projectName: form.projectName,
          description: form.description,
          techDomain: form.techDomain,
          attachmentUrl: form.attachmentUrl,
          status: 1 // 寮哄埗璁剧疆涓� 1 (寰呭垵瀹�)
        }

        // 3. 寮哄埗璧嬪€� applicantId
        // 灏濊瘯澶氱鍙兘鐨勫瓧娈靛悕
        const userId = user.userId || user.user_id || user.id

        if (userId) {
          submitData.applicantId = userId
        } else {
          // 淇濆簳绛栫暐
          submitData.applicantId = 1
          ElMessage.warning('当前未读取到用户ID，使用测试ID: 1')
        }

        // 4. 鍙戦€佽姹
        await addProject(submitData)
        ElMessage.success('项目申报成功')
        router.push('/')
      } catch (error) {
        console.error(error)
      } finally {
        loading.value = false
      }
    }
  })
}

const resetForm = () => {
  if (!formRef.value) return
  formRef.value.resetFields()
  fileList.value = []
}
</script>

<style scoped>
.project-form-container{padding:20px;max-width:800px;margin:0 auto;position:relative}
.project-decor{position:absolute;inset:0;z-index:0;pointer-events:none}
.project-decor .shape-a{position:absolute;width:160px;height:160px;right:6%;top:6%;background:linear-gradient(135deg,var(--accent-2),transparent);clip-path:polygon(50% 0,100% 38%,82% 100%,18% 100%,0 38%)}
.project-decor .shape-b{position:absolute;width:110px;height:110px;left:6%;bottom:6%;background:linear-gradient(225deg,var(--accent),transparent);border-radius:50%}
.card-header{display:flex;justify-content:space-between;align-items:center;color:rgba(235,245,255,0.95);position:relative;z-index:2}

::v-deep .el-upload__tip{color:rgba(200,220,255,0.7)}
::v-deep .el-input__inner{background:transparent;border:1px solid rgba(255,255,255,0.05);border-radius:8px}

.panel{position:relative;z-index:2}

</style>

