<template>
  <n-space vertical>
    <n-h1>工具</n-h1>
    <n-card title="数据库维护">
      <n-space vertical>
        <n-alert type="warning" title="SQLite 修复">
          如果遇到 "database disk image is malformed" 错误，点击下方按钮尝试修复。此操作会对数据库执行 VACUUM，重建数据库文件。
        </n-alert>
        <n-button type="warning" :loading="repairing" @click="handleRepair">
          修复数据库
        </n-button>
        <n-alert v-if="repairResult" :type="repairResult.success ? 'success' : 'error'" :title="repairResult.success ? '修复完成' : '修复失败'">
          <p v-for="(msg, db) in repairResult.results" :key="db">{{ db }}: {{ msg }}</p>
        </n-alert>
      </n-space>
    </n-card>
  </n-space>
</template>

<script setup>
import {ref} from 'vue'
import {NSpace, NH1, NCard, NButton, NAlert, useMessage} from 'naive-ui'
import axios from 'axios'
import {getAPISRV} from '@/global.js'
import {getToken} from '@/auth.js'

const message = useMessage()
const repairing = ref(false)
const repairResult = ref(null)

async function handleRepair() {
  const pwd = window.prompt('请输入密码进行验证：')
  if (!pwd) return

  repairing.value = true
  repairResult.value = null
  try {
    const resp = await axios.post(`${getAPISRV()}/web/database/repair`, null, {
      headers: {Authorization: `Bearer ${getToken()}`, 'X-Verify-Password': pwd}
    })
    repairResult.value = {success: true, results: resp.data.results}
    message.success('数据库修复完成')
  } catch (e) {
    repairResult.value = {success: false, results: {error: e?.response?.data?.detail || '修复失败'}}
    message.error('修复失败')
  } finally {
    repairing.value = false
  }
}
</script>
