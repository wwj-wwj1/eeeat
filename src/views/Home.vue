<script setup lang="ts">
import { ref ,onMounted, watch} from 'vue'
import { Plus, Delete} from '@element-plus/icons-vue'

const input = ref('')
const candidates = ref<string[]>([])
const selected = ref('')
const dialogVisible = ref(false)

function removeCandidate(index: number) {
  candidates.value.splice(index, 1)
}

onMounted(() => {
  const saved = localStorage.getItem('candidates')
  if (saved) {
    candidates.value = JSON.parse(saved)
  }
})

watch(candidates, (val) => {
  localStorage.setItem('candidates', JSON.stringify(val))
}, { deep: true })

function addCandidate() {
  if (input.value.trim()) {
    candidates.value.push(input.value.trim())
    input.value = ''
  }
}

function pickRandom() {
  if (candidates.value.length) {
    const idx = Math.floor(Math.random() * candidates.value.length)
    selected.value = candidates.value[idx]
  } else {
    selected.value = ''
  }
}

function openDialog() {
  pickRandom()
  dialogVisible.value = true
}

function goToMeituan() {
  if (selected.value) {
    const keyword = encodeURIComponent(selected.value)
    // 美团 App 的 URL Scheme（部分安卓/iOS支持）
    const appUrl = `meituanwaimai://waimai.meituan.com/search?keyword=${keyword}`
    // 网页端兜底
    const webUrl = `https://www.meituan.com/s/${keyword}`

    // 尝试唤起 App，延迟后跳转网页
    const iframe = document.createElement('iframe')
    iframe.style.display = 'none'
    iframe.src = appUrl
    document.body.appendChild(iframe)
    setTimeout(() => {
      window.open(webUrl, '_blank')
      document.body.removeChild(iframe)
    }, 1200)
  }
}
</script>

<template>
<div style="display: flex; flex-direction: column; width: 100%;">
<!--  输入区-->
  <div style="display: flex; align-items: center;width: 360px;">
<!--    输入框-->
    <div style="margin-right: 28px;">
      <el-input
          v-model="input"
          style="width: 300px"
          placeholder="我想吃~~~~~"
          size="large"
          clearable
          @keyup.enter="addCandidate"
      />
    </div>
<!--    add按钮-->
    <div style="margin-left: auto;">
      <el-button :icon="Plus" circle @click="addCandidate" id="add"/>
    </div>
  </div>
<!--待选区-->
  <div style="margin-top: 20px;
  width: 360px;
  min-height: 200px;
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  border: 1px solid #ccc;
  border-radius: 8px;
  padding: 12px 8px;
 position: relative;">
<!--选项-->
    <el-button
        v-for="(item, idx) in candidates"
        :key="item"
        round
        type="warning"
        @click="removeCandidate(idx)"
    >
      {{ item }}
    </el-button>
<!--    删除-->
    <el-button
        v-if="candidates.length"
        type="danger"
        :icon="Delete"
        circle
        style="position: absolute; right: 12px; bottom: 12px;"
        @click="candidates = []"
    />

  </div>
<!--  随机-->
  <div style="margin-top: 12px;display: flex; justify-content: center;">
    <el-button
        round
        type="warning"
        size="large"
        :disabled="!candidates.length"
        @click="openDialog"
        style="width: 300px; "
    >我要吃!</el-button>
  </div>
<!-- 结果区-->
  <el-dialog
      v-model="dialogVisible"
      width="400px"
      align-center
      :show-close="false"
      style="text-align: center;"
  >
    <div style="font-size: 28px; margin: 40px 0 32px 0; color: rgb(234,101,17);">
      {{ selected }}
    </div>
    <div style="display: flex; flex-direction: column; gap: 16px; margin-bottom: 12px;">
      <div style="display: flex; justify-content: center;">
      <el-button type="warning" size="large" id="go"
                 style="height:60px;width:240px"
                 @click="goToMeituan">去吃!</el-button>
      </div>
      <div style="display: flex; gap: 16px; justify-content: center;">
      <el-button type="success" size="large" @click="pickRandom" id="no">不要这个</el-button>
      <el-button type="info" size="large" @click="dialogVisible = false" id="off">关闭</el-button>
      </div>
    </div>
  </el-dialog>
</div>
</template>

<style scoped>

</style>