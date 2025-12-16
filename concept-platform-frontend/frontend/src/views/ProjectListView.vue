<template>
  <div class="project-list-container fade-in-up">
    <el-card class="panel">
      <template #header>
        <div class="card-header">
          <span>椤圭洰鍒楄〃</span>
          <div>
            <el-button type="primary" size="small" @click="$router.push('/project/add')">鏂板椤圭洰</el-button>
            <el-button type="success" size="small" @click="refreshList">鍒锋柊</el-button>
          </div>
        </div>
      </template>

      <el-table :data="tableData" v-loading="loading" style="width: 100%" stripe>
        <el-table-column prop="projectName" label="椤圭洰鍚嶇О" min-width="150" />
        <el-table-column prop="techDomain" label="鎶€鏈鍩�" width="150" />
        <el-table-column prop="status" label="褰撳墠鐘舵€�" width="120">
          <template #default="scope">
            <el-tag :type="getStatusType(scope.row.status)">
              {{ getStatusLabel(scope.row.status) }}
            </el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="createdAt" label="鍒涘缓鏃堕棿" width="180">
          <template #default="scope">
            {{ scope.row.createdAt ? scope.row.createdAt.replace('T', ' ') : '' }}
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { getList } from '@/api/project'
import { ElMessage } from 'element-plus'

const tableData = ref([])
const loading = ref(false)

const getStatusType = (status) => {
  // 0-鑽夌ǹ鏄伆鑹�(info)锛�1-寰呭垵瀹℃槸钃濊壊(primary)
  const map = {
    0: 'info',
    1: 'primary',
    2: 'warning', // 璇勫涓�
    3: 'success', // 宸插叆搴�
    9: 'danger'   // 宸查┏鍥�
  }
  return map[status] || 'info'
}

const getStatusLabel = (status) => {
  const map = {
    0: '草稿',
    1: '待初审',
    2: '待评审',
    3: '已入档',
    9: '已退回'
  }
  return map[status] || '未知状态'
}

const fetchData = async () => {
  loading.value = true
  try {
    const res = await getList({})
    // 鍏煎澶勭悊锛氬鏋� res 鏄暟缁勫垯鐩存帴浣跨敤锛屽鏋滄槸瀵硅薄涓旀湁 list 灞炴€у垯浣跨敤 list锛屽惁鍒欎负绌烘暟缁�
    // 娉ㄦ剰锛歳equest.js 鎷︽埅鍣ㄥ凡缁忓鐞嗕簡 res.data 鐨勮В鏋� (杩斿洖 res.data.data)
    // 鍋囪鍚庣鏍囧噯杩斿洖 data: [] 鎴� data: { list: [], total: 0 }
    if (Array.isArray(res)) {
      tableData.value = res
    } else if (res && Array.isArray(res.list)) {
      tableData.value = res.list
    } else {
      tableData.value = []
    }
  } catch (error) {
    console.error('获取项目列表失败', error)
  } finally {
    loading.value = false
  }
}

const refreshList = () => {
  fetchData()
}

onMounted(() => {
  fetchData()
})
</script>

<style scoped>
.project-list-container {
  padding: 20px;
  max-width: 1200px;
  margin: 0 auto;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

<style scoped>
.project-list-container{padding:20px;max-width:1200px;margin:0 auto}
.card-header{display:flex;justify-content:space-between;align-items:center;color:rgba(235,245,255,0.95)}

/* 表格更多透明化，hover 高亮 */
::v-deep .el-table{background:transparent}
::v-deep .el-table .el-table__row:hover{background:linear-gradient(90deg, rgba(0,208,255,0.03), rgba(124,77,255,0.02))}

@media (max-width:800px){.project-list-container{padding:12px}}
</style>

