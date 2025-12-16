<template>
  <div class="my-projects-container fade-in-up">
    <el-card class="panel">
      <template #header>
        <div class="card-header">
          <span>我的项目</span>
          <el-button type="primary" size="small" @click="fetchData">刷新</el-button>
        </div>
      </template>

      <el-table :data="tableData" v-loading="loading" style="width: 100%" stripe>
        <el-table-column prop="projectName" label="项目名称" min-width="150" />
        <el-table-column prop="techDomain" label="技术领域" width="150" />
        <el-table-column prop="budget" label="预算 (万元)" width="150" />
        <el-table-column prop="status" label="当前状态" width="120">
          <template #default="scope">
            <el-tag :type="getStatusType(scope.row.status)">
              {{ getStatusLabel(scope.row.status) }}
            </el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="createdAt" label="创建时间" width="180">
          <template #default="scope">
            {{ scope.row.createdAt ? scope.row.createdAt.replace('T', ' ') : '' }}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="120" fixed="right">
          <template #default="scope">
            <el-button 
              v-if="scope.row.status === 3 || scope.row.status === 9"
              type="primary" 
              link
              size="small" 
              @click="handleViewFeedback(scope.row)"
            >
              查看反馈
            </el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 鍙嶉寮圭獥 -->
    <el-dialog v-model="feedbackVisible" title="评审反馈" width="600px">
      <!-- 鎯呭喌A: 鍒濆琚嫆 -->
      <div v-if="currentProject.status === 9">
        <el-alert title="项目已被退回" type="error" :closable="false" show-icon />
        <div style="margin-top: 20px;">
          <p><strong>退回理由：</strong></p>
          <div style="background: #fef0f0; padding: 10px; border-radius: 4px; color: #f56c6c;">
            {{ currentProject.rejectReason || '暂无退回理由' }}
          </div>
        </div>
      </div>

      <!-- 鎯呭喌B: 涓撳璇勫 -->
      <div v-else-if="currentProject.status === 3">
        <div style="margin-bottom: 20px;">
          <el-statistic title="平均评审分" :value="averageScore" precision="1" />
        </div>
        
        <el-table :data="reviewList" border style="width: 100%">
          <el-table-column prop="expertName" label="专家" width="120">
             <template #default="scope">
               专家 {{ scope.$index + 1 }}
             </template>
          </el-table-column>
          <el-table-column prop="score" label="评分" width="80" />
          <el-table-column prop="comments" label="评审意见" />
        </el-table>
      </div>

      <template #footer>
        <span class="dialog-footer">
          <el-button @click="feedbackVisible = false">关闭</el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import { getMyProjectList } from '@/api/project'
import { getProjectReviews } from '@/api/review'
import { ElMessage } from 'element-plus'

const tableData = ref([])
const loading = ref(false)
const feedbackVisible = ref(false)
const currentProject = ref({})
const reviewList = ref([])

const averageScore = computed(() => {
  if (!reviewList.value || reviewList.value.length === 0) return 0
  const sum = reviewList.value.reduce((acc, curr) => acc + (curr.score || 0), 0)
  return sum / reviewList.value.length
})

const getStatusType = (status) => {
  const map = {
    0: 'info',    // 鑽夌ǹ
    1: 'primary', // 寰呭垵瀹�
    2: 'warning', // 璇勫涓�
    3: 'success', // 宸插叆搴�
    9: 'danger'   // 宸查┏鍥�
  }
  return map[status] || 'info'
}

const getStatusLabel = (status) => {
  const map = {
    0: 'info',
    1: 'primary',
    2: 'warning',
    3: 'success',
    9: 'danger'
  }
  return map[status] || 'info'
}

const fetchData = async () => {
  loading.value = true
  try {
    const userStr = localStorage.getItem('user')
    let userId = null
    if (userStr) {
      try {
        const user = JSON.parse(userStr)
        userId = user.userId || user.id || user.user_id
      } catch (e) {
        console.error('JSON Parse Error', e)
      }
    }

    if (!userId) {
      ElMessage.warning('未能获取当前用户ID，无法加载数据')
      return
    }

    const res = await getMyProjectList({ applicantId: userId })
    
    if (Array.isArray(res)) {
      tableData.value = res
    } else if (res && Array.isArray(res.list)) {
      tableData.value = res.list
    } else {
      tableData.value = []
    }
  } catch (error) {
    console.error('获取我的项目列表失败', error)
  } finally {
    loading.value = false
  }
}

const handleViewFeedback = async (row) => {
  currentProject.value = row
  feedbackVisible.value = true
  reviewList.value = [] // 閲嶇疆鍒楄〃

  if (row.status === 3) {
    // 鍙湁鐘舵€佷负3锛堝凡鍏ュ簱/璇勫缁撴潫锛夋墠鍘绘媺鍙栬瘎瀹¤鎯�
    // 鐘舵€�9锛堥┏鍥烇級鐩存帴鏄剧ず row 涓殑 rejectReason
    try {
      const res = await getProjectReviews(row.id || row.projectId)
      console.log('获取到我的评审列表:', res) // 添加日志
      if (Array.isArray(res)) {
        reviewList.value = res
      } else if (res && Array.isArray(res.list)) {
        reviewList.value = res.list
      }
    } catch (error) {
      console.error('获取评审详情失败', error)
      ElMessage.error('获取评审详情失败')
    }
  }
}

onMounted(() => {
  fetchData()
})
</script>

<style scoped>
.my-projects-container {
  padding: 20px;
}
.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

<style scoped>
.my-projects-container{padding:20px}
.card-header{display:flex;justify-content:space-between;align-items:center;color:rgba(235,245,255,0.95)}

::v-deep .el-table{background:transparent}
::v-deep .el-tag{background:rgba(255,255,255,0.03);color:var(--accent)}

</style>
