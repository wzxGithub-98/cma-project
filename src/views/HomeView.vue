<script setup lang="ts">
import * as XLSX from 'xlsx'

let cmaData = []
let clkData = []

const uniqueObjArr = (list, key) => {
  return list.filter((item, index) => {
    return list.findIndex((t) => t[key] === item[key]) === index
  })
}

const readExcel = (file, sheetIndex, callback) => {
  const reader = new FileReader()
  reader.onload = (e) => {
    const data = e.target.result
    const workbook = XLSX.read(data, {
      type: 'binary',
    })
    const sheet = workbook.Sheets[workbook.SheetNames[sheetIndex]]
    const sheetData = XLSX.utils.sheet_to_json(sheet)
    callback(sheetData)
  }
  reader.readAsArrayBuffer(file)
}

const onCma = (e) => {
  const cmaFile = e.target.files[0]
  cmaData = []
  readExcel(cmaFile, 0, (data) => {
    data.forEach((item, index) => {
      let objName = '标准编号(含年号)'
      if (item[objName] && !item['备注']?.includes('废止')) {
        cmaData.push(item[objName])
      }
    })
    console.log('cmaData', cmaData)
  })
}

const onClk = (e) => {
  const clkFile = e.target.files[0]
  clkData = []
  readExcel(clkFile, 0, (data) => {
    data.forEach((item, index) => {
      if (index >= 2 && item['__EMPTY_2'] && item['__EMPTY_6']) {
        clkData.push({ code: item['__EMPTY_2'], inLib: item['__EMPTY_6'] })
      }
    })
    clkData = [...uniqueObjArr(clkData, 'code')].map((item, index) => {
      return { id: index + 1, ...item }
    })
    console.log('clkData', clkData)
  })
}

const diffExcel = (cma, clk) => {
  let diff = []
  let id = 0;
  clk.forEach((item, index) => {
    if (cma.includes(item.code)) {
      item.inLib = '是'
    } else {
      item.inLib = '否'
    }
    if (item.code === clkData[index].code && item.inLib !== clkData[index].inLib) {
      diff.push({...item, id: ++id})
    }
  })
  return diff
}

const getDiffUrl = (diff) => {
  const diffJSON = JSON.stringify(diff, null, 2)
  const blob = new Blob([diffJSON], {
    type: 'application/json',
  })
  const url = URL.createObjectURL(blob)
  const link = document.createElement('a')
  link.href = url
  link.download = 'diff.json'
  link.click()
  URL.revokeObjectURL(url)
}

const onClick = () => {
  let diffData = diffExcel([...cmaData], JSON.parse(JSON.stringify(clkData)))
  console.log('diffData', diffData)
  getDiffUrl(diffData);
}
</script>

<template>
  <main>
    <label>请选择CMA能力项目库文件：</label>
    <input type="file" ref="cmaFile" @change="onCma" />
    <br />
    <label>请选择莱恩CMA资质能力认证项目表：</label>
    <input type="file" ref="clkFile" @change="onClk" />

    <br />
    <button type="button" @click="onClick">开始比对</button>
  </main>
</template>
