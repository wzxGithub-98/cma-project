<script setup lang="ts">
import { ref, computed } from 'vue'
import * as XLSX from 'xlsx'
import { showToast, showNotify } from 'vant'

interface ClkItem {
  id: number
  code: string
  inLib: string
}

interface DiffItem extends ClkItem {}

const cmaData = ref<string[]>([])
const clkData = ref<ClkItem[]>([])
const cmaFileName = ref('')
const clkFileName = ref('')
const cmaFileList = ref<File[]>([])
const clkFileList = ref<File[]>([])
const loading = ref(false)

const cmaCount = computed(() => cmaData.value.length)
const clkCount = computed(() => clkData.value.length)
const canCompare = computed(() => cmaCount.value > 0 && clkCount.value > 0)

const uniqueObjArr = (list: ClkItem[], key: keyof ClkItem): ClkItem[] => {
  return list.filter((item, index) => {
    return list.findIndex((t) => t[key] === item[key]) === index
  })
}

const readExcel = (file: File, sheetIndex: number, callback: (data: Record<string, unknown>[]) => void) => {
  const reader = new FileReader()
  reader.onload = (e: ProgressEvent<FileReader>) => {
    const data = e.target?.result
    if (!data) {
      showToast('文件读取失败')
      return
    }
    const workbook = XLSX.read(data, { type: 'binary' })
    const sheet = workbook.Sheets[workbook.SheetNames[sheetIndex]]
    const sheetData = XLSX.utils.sheet_to_json<Record<string, unknown>>(sheet)
    callback(sheetData)
  }
  reader.readAsArrayBuffer(file)
}

const onCma = (file: File | File[]) => {
  const cmaFile = Array.isArray(file) ? file[0] : file
  if (!cmaFile) return
  cmaFileName.value = cmaFile.name
  cmaData.value = []
  readExcel(cmaFile, 0, (data) => {
    data.forEach((item) => {
      const objName = '标准编号(含年号)'
      const codeValue = item[objName] as string | undefined
      const remarkValue = item['备注'] as string | undefined
      if (codeValue && !remarkValue?.includes('废止')) {
        cmaData.value.push(codeValue)
      }
    })
    showToast(`CMA文件已加载，共${cmaData.value.length}条`)
  })
}

const onClk = (file: File | File[]) => {
  const clkFile = Array.isArray(file) ? file[0] : file
  if (!clkFile) return
  clkFileName.value = clkFile.name
  clkData.value = []
  readExcel(clkFile, 0, (data) => {
    const rawList: ClkItem[] = []
    data.forEach((item, index) => {
      const codeVal = item['__EMPTY_2'] as string | undefined
      const inLibVal = item['__EMPTY_6'] as string | undefined
      if (index >= 2 && codeVal && inLibVal) {
        rawList.push({ id: index, code: codeVal, inLib: inLibVal })
      }
    })
    clkData.value = [...uniqueObjArr(rawList, 'code')].map((item, idx) => {
      return { id: idx + 1, ...item, code: item.code, inLib: item.inLib }
    })
    showToast(`莱恩文件已加载，共${clkData.value.length}条`)
  })
}

const diffExcel = (cma: string[], clk: ClkItem[]): DiffItem[] => {
  const diff: DiffItem[] = []
  let id = 0
  clk.forEach((item, index) => {
    const original = clkData.value[index]
    if (cma.includes(item.code)) {
      item.inLib = '是'
    } else {
      item.inLib = '否'
    }
    if (original && item.code === original.code && item.inLib !== original.inLib) {
      diff.push({ ...item, id: ++id })
    }
  })
  return diff
}

const getDiffUrl = (diff: DiffItem[]) => {
  const diffJSON = JSON.stringify(diff, null, 2)
  const blob = new Blob([diffJSON], { type: 'application/json' })
  const url = URL.createObjectURL(blob)
  const link = document.createElement('a')
  link.href = url
  link.download = 'diff.json'
  link.click()
  URL.revokeObjectURL(url)
}

const onClick = () => {
  if (!canCompare.value) {
    showNotify({ type: 'warning', message: '请先上传两个文件' })
    return
  }
  loading.value = true
  try {
    const diffData = diffExcel([...cmaData.value], JSON.parse(JSON.stringify(clkData.value)))
    getDiffUrl(diffData)
    if (diffData.length > 0) {
      showNotify({ type: 'success', message: `比对完成，共${diffData.length}条差异，文件已下载` })
    } else {
      showNotify({ type: 'primary', message: '比对完成，无差异数据' })
    }
  } catch {
    showNotify({ type: 'danger', message: '比对失败，请检查文件格式' })
  } finally {
    loading.value = false
  }
}
</script>

<template>
  <div class="home-view">
    <van-nav-bar title="CMA标准比对工具" />

    <van-cell-group inset title="数据源">
      <van-cell title="CMA能力项目库" :label="cmaFileName || '请上传CMA能力项目库Excel文件'">
        <template #value>
          <van-tag v-if="cmaCount > 0" type="primary" round>{{ cmaCount }} 条</van-tag>
        </template>
        <template #right-icon>
          <van-uploader
            v-model="cmaFileList"
            :max-count="1"
            accept=".xlsx,.xls"
            :after-read="onCma"
            :preview-image="false"
          >
            <van-button size="small" type="primary" plain icon="upload">选择文件</van-button>
          </van-uploader>
        </template>
      </van-cell>

      <van-cell title="莱恩CMA资质认定项目表" :label="clkFileName || '请上传莱恩CMA资质能力认证项目表'">
        <template #value>
          <van-tag v-if="clkCount > 0" type="success" round>{{ clkCount }} 条</van-tag>
        </template>
        <template #right-icon>
          <van-uploader
            v-model="clkFileList"
            :max-count="1"
            accept=".xlsx,.xls"
            :after-read="onClk"
            :preview-image="false"
          >
            <van-button size="small" type="primary" plain icon="upload">选择文件</van-button>
          </van-uploader>
        </template>
      </van-cell>
    </van-cell-group>

    <div class="action-bar">
      <van-button
        type="primary"
        block
        round
        :disabled="!canCompare"
        :loading="loading"
        loading-text="比对中..."
        @click="onClick"
      >
        开始比对
      </van-button>
    </div>

    <van-notice-bar
      wrapable
      :scrollable="false"
      text="说明：上传CMA能力项目库和莱恩CMA资质认定项目表后，点击「开始比对」将自动比较两个文件中的标准编号差异，并下载差异结果为JSON文件。"
    />
  </div>
</template>

<style scoped>
.home-view {
  min-height: 100vh;
  background-color: #f7f8fa;

}

.action-bar {
  padding: 16px;
}

.van-cell {
  align-items: center;
}
</style>
