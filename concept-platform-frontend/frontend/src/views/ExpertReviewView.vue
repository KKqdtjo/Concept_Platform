<template>
  <div class="expert-reviews-container fade-in-up">
    <el-card class="panel">
      <template #header>
        <div class="card-header">
          <span>评审任务</span>
          <el-button type="primary" size="small" @click="fetchData">鍒锋柊</el-button>
        </div>
      </template>

      <el-tabs v-model="activeTab" class="review-tabs">
        <!-- 待评审 Tab -->
        <el-tab-pane label="待评审" name="pending">
          <el-table :data="pendingList" v-loading="loading" style="width: 100%" stripe>
            <el-table-column prop="projectName" label="项目名称" min-width="150" />
            <el-table-column label="申报材料" width="120">
              <template #default="scope">
                <el-link 
                  v-if="scope.row.attachmentUrl" 
                  :href="getFullUrl(scope.row.attachmentUrl)" 
                  target="_blank" 
                  type="primary"
                >
                  查看附件
                </el-link>
                <span v-else style="color: #909399; font-size: 12px;">鏃犻檮浠�</span>
              </template>
            </el-table-column>
            <el-table-column prop="techDomain" label="技术领域" width="150" />
            <el-table-column prop="deadline" label="截止时间" width="180">
               <template #default="scope">
                {{ scope.row.deadline ? scope.row.deadline.replace('T', ' ') : '-' }}
              </template>
            </el-table-column>
            <el-table-column label="操作" width="120" fixed="right">
              <template #default="scope">
                <el-button type="primary" size="small" @click="handleReview(scope.row)">璇勫</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>

        <!-- 宸茶瘎瀹� Tab -->
        <el-tab-pane label="已评审" name="reviewed">
          <el-table :data="reviewedList" v-loading="loading" style="width: 100%" stripe>
            <el-table-column prop="projectName" label="项目名称" min-width="150" />
            <el-table-column prop="techDomain" label="技术领域" width="150" />
            <el-table-column prop="score" label="评审分" width="100" />
            <el-table-column prop="reviewTime" label="评审时间" width="180">
              <template #default="scope">
                {{ scope.row.reviewTime ? scope.row.reviewTime.replace('T', ' ').substring(0, 19) : '-' }}
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>

    <!-- 评审弹窗 -->
    <el-dialog v-model="dialogVisible" title="项目评审" width="600px" @close="resetForm">
      <el-descriptions title="项目信息" :column="1" border>
        <el-descriptions-item label="项目名称">{{ currentProject.projectName }}</el-descriptions-item>
        <el-descriptions-item label="技术领域">{{ currentProject.techDomain }}</el-descriptions-item>
        <el-descriptions-item label="项目简介">{{ currentProject.description }}</el-descriptions-item>
      </el-descriptions>
      
      <el-divider />

      <el-form :model="form" ref="formRef" :rules="rules" label-width="80px" style="margin-top: 20px;">
        <el-form-item label="评审分" prop="score">
          <el-input-number v-model="form.score" :min="0" :max="100" />
        </el-form-item>
        <el-form-item label="评审意见" prop="comment">
          <el-input v-model="form.comment" type="textarea" rows="4" placeholder="请输入评审意见" />
        </el-form-item>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="dialogVisible = false">取消</el-button>
          <el-button type="primary" @click="submit" :loading="submitting">提交</el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted, computed } from 'vue'
import { getReviewList, submitReview } from '@/api/review'
import { ElMessage } from 'element-plus'

const loading = ref(false)
const dialogVisible = ref(false)
const submitting = ref(false)
const activeTab = ref('pending')
const allReviews = ref([])
const currentProject = ref({})
const formRef = ref(null)

const form = reactive({
  reviewId: null, // 璇勫浠诲姟ID
  projectId: null,
  score: 80,
  comment: ''
})

const rules = {
  score: [{ required: true, message: '请输入评审分', trigger: 'blur' }],
  comment: [{ required: true, message: '请输入评审意见', trigger: 'blur' }]
}

// 鎷嗗垎鏁版嵁
const pendingList = computed(() => {
  return allReviews.value.filter(item => item.status === 0 || !item.status) // status 0 鎴� null 涓哄緟璇勫
})

const reviewedList = computed(() => {
  return allReviews.value.filter(item => item.status === 1) // status 1 涓哄凡璇勫
})

const fetchData = async () => {
  loading.value = true
  try {
    const userStr = localStorage.getItem('user')
    let userId = null
    if (userStr) {
      const user = JSON.parse(userStr)
      // 鍏煎澶氱 ID 瀛楁鍚�
      userId = user.userId || user.user_id || user.id
    }
    
    // 浼犲叆 expertId 鑾峰彇浠诲姟鍒楄〃
    const res = await getReviewList({ expertId: userId })
    
    if (Array.isArray(res)) {
      allReviews.value = res
    } else if (res && Array.isArray(res.list)) {
      allReviews.value = res.list
    } else {
      allReviews.value = []
    }
  } catch (error) {
    console.error('获取评审列表失败', error)
  } finally {
    loading.value = false
  }
}

const handleReview = (row) => {
  currentProject.value = row
  // 鍏抽敭淇锛氭纭幏鍙栬瘎瀹′换鍔� ID
  // 浼樺厛鍙� row.reviewId锛屽鏋滄病鏈夊垯灏濊瘯鍙� row.id
  form.reviewId = row.reviewId || row.id 
  form.projectId = row.projectId
  // 纭繚绠€浠嬪瓧娈垫纭槧灏�
  // 濡傛灉鍚庣杩斿洖鐨勭畝浠嬪瓧娈靛彨 description 鎴� projectDescription锛岃鍦ㄦ澶勯€傞厤
  currentProject.value.description = row.description || row.projectDescription || '鏆傛棤绠€浠�'
  form.score = 80
  form.comment = ''
  dialogVisible.value = true
}

const getFullUrl = (url) => {
  if (!url) return ''
  if (url.match(/^https?:\/\//)) return url
  return `http://localhost:8080${url}`
}

const resetForm = () => {
  formRef.value?.resetFields()
  currentProject.value = {}
}

const submit = async () => {
  if (!formRef.value) return
  
  await formRef.value.validate(async (valid) => {
    if (valid) {
      // 鏋勯€犵鍚堝悗绔� DTO 鐨勬暟鎹粨鏋�
      const submitData = {
        reviewId: form.reviewId,
        projectId: form.projectId,
        score: form.score,
        comments: form.comment // 寮哄埗鏄犲皠锛氬墠绔� comment -> 鍚庣 comments (澶嶆暟)
      }

      // 璋冭瘯锛氭墦鍗版彁浜ゆ暟鎹�
      console.log('提交评审数据:', submitData)

      submitting.value = true
      try {
        await submitReview(submitData)
        ElMessage.success('评审提交成功')
        dialogVisible.value = false
        fetchData() // 鍒锋柊鍒楄〃
      } catch (error) {
        console.error(error)
      } finally {
        submitting.value = false
      }
    }
  })
}

onMounted(() => {
  fetchData()
})
</script>

<style scoped>
.expert-reviews-container {
  padding: 20px;
}
.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.review-tabs {
  margin-top: 10px;
}
<style scoped>
.expert-reviews-container{padding:20px}
.card-header{display:flex;justify-content:space-between;align-items:center;color:var(--hologram-white)}
.review-tabs{margin-top:10px}

::v-deep .el-table{background:transparent}
::v-deep .el-table .el-table__row:hover{background:linear-gradient(90deg, rgba(0,243,255,0.03), rgba(157,0,255,0.02))}

::v-deep .el-input__inner, ::v-deep .el-textarea__inner{background:transparent;border:1px solid rgba(255,255,255,0.04);border-radius:8px;transition:box-shadow var(--smooth-time) ease,border-color var(--smooth-time) ease}
::v-deep .el-input__inner:focus, ::v-deep .el-textarea__inner:focus{outline:none;border-color:var(--accent);box-shadow:0 8px 28px rgba(157,0,255,0.05) inset}

@media (max-width:800px){.expert-reviews-container{padding:12px}}
</style>

