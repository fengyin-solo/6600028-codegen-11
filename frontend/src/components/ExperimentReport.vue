<script setup lang="ts">
import { ref, computed } from 'vue'
import { useFluidStore } from '../store/fluid'
import type { SimParams } from '../types'

const store = useFluidStore()
const showReport = ref(false)
const studentName = ref('')
const experimentConclusion = ref('')

const paramLabels: Record<keyof SimParams, string> = {
  gravity: '重力加速度',
  viscosity: '粘性系数',
  restDensity: '静止密度',
  gasConstant: '气体常数',
  smoothingRadius: '光滑半径',
  particleMass: '粒子质量',
  dt: '时间步长',
  damping: '阻尼系数',
  boundaryStiffness: '边界刚度',
}

const paramUnits: Record<keyof SimParams, string> = {
  gravity: 'm/s²',
  viscosity: 'Pa·s',
  restDensity: 'kg/m³',
  gasConstant: 'J/(kg·K)',
  smoothingRadius: 'px',
  particleMass: 'kg',
  dt: 's',
  damping: '',
  boundaryStiffness: 'N/m',
}

const reportText = computed(() => {
  const date = new Date().toLocaleString('zh-CN')
  const lines: string[] = []

  lines.push('═'.repeat(50))
  lines.push('          流体力学 SPH 模拟实验报告')
  lines.push('═'.repeat(50))
  lines.push('')

  if (studentName.value) {
    lines.push(`学生姓名：${studentName.value}`)
  }
  lines.push(`实验日期：${date}`)
  lines.push('')

  lines.push('─'.repeat(50))
  lines.push('一、实验场景')
  lines.push('─'.repeat(50))
  lines.push(`场景名称：${store.currentPreset.label}`)
  lines.push(`场景类型：${store.currentPreset.initialConfig}`)
  lines.push(`场景描述：${store.currentPreset.description}`)
  lines.push(`粒子数量：${store.particleCount}`)
  lines.push('')

  lines.push('─'.repeat(50))
  lines.push('二、模拟参数')
  lines.push('─'.repeat(50))
  const params = store.params
  ;(Object.keys(params) as Array<keyof SimParams>).forEach((key) => {
    const label = paramLabels[key] || key
    const unit = paramUnits[key] || ''
    const value = params[key].toFixed(4)
    lines.push(`  ${label}：${value} ${unit}`.trim())
  })
  lines.push('')

  lines.push('─'.repeat(50))
  lines.push('三、运行结果')
  lines.push('─'.repeat(50))
  lines.push(`  运行帧数：${store.frameCount}`)
  lines.push(`  实时 FPS：${store.fps}`)
  lines.push(`  粒子总数：${store.particleArray.length}`)
  lines.push(`  平均密度：${store.avgDensity.toFixed(2)} kg/m³`)
  lines.push(`  最大速度：${store.maxVelocity.toFixed(2)} m/s`)
  lines.push('')

  if (experimentConclusion.value) {
    lines.push('─'.repeat(50))
    lines.push('四、实验结论')
    lines.push('─'.repeat(50))
    lines.push(experimentConclusion.value)
    lines.push('')
  }

  lines.push('═'.repeat(50))
  lines.push('  报告生成时间：' + date)
  lines.push('═'.repeat(50))

  return lines.join('\n')
})

function toggleReport() {
  showReport.value = !showReport.value
}

function copyReport() {
  navigator.clipboard.writeText(reportText.value).then(() => {
    alert('实验报告已复制到剪贴板')
  })
}

function downloadReport() {
  const blob = new Blob([reportText.value], { type: 'text/plain;charset=utf-8' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = `SPH实验报告_${store.currentPreset.name}_${Date.now()}.txt`
  document.body.appendChild(a)
  a.click()
  document.body.removeChild(a)
  URL.revokeObjectURL(url)
}
</script>

<template>
  <div class="w-72">
    <button
      @click="toggleReport"
      class="w-full bg-purple-600 hover:bg-purple-700 text-white py-2 px-4 rounded-lg text-sm font-medium transition flex items-center justify-center gap-2"
    >
      <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z" />
      </svg>
      实验报告
    </button>

    <div
      v-if="showReport"
      class="mt-3 bg-gray-800 rounded-lg border border-gray-700 p-4 flex flex-col gap-3 max-h-[70vh] overflow-auto"
    >
      <h3 class="text-sm font-semibold text-purple-400 flex items-center gap-2">
        <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z" />
        </svg>
        实验报告面板
      </h3>

      <div>
        <label class="text-xs text-gray-400 mb-1 block">学生姓名</label>
        <input
          v-model="studentName"
          type="text"
          placeholder="请输入姓名"
          class="w-full bg-gray-900 border border-gray-700 rounded px-2 py-1.5 text-xs text-gray-200 focus:outline-none focus:border-purple-500"
        />
      </div>

      <div class="bg-gray-900 rounded p-3">
        <h4 class="text-xs font-semibold text-gray-400 mb-2">当前场景</h4>
        <p class="text-sm text-white font-medium">{{ store.currentPreset.label }}</p>
        <p class="text-xs text-gray-500 mt-1">{{ store.currentPreset.description }}</p>
        <div class="flex items-center gap-2 mt-2 text-xs">
          <span class="text-gray-500">粒子数:</span>
          <span class="text-blue-400 font-mono">{{ store.particleCount }}</span>
        </div>
      </div>

      <div class="bg-gray-900 rounded p-3">
        <h4 class="text-xs font-semibold text-gray-400 mb-2">模拟参数</h4>
        <div class="grid grid-cols-2 gap-1.5 text-xs">
          <div class="flex justify-between">
            <span class="text-gray-500">重力</span>
            <span class="text-gray-300 font-mono">{{ store.params.gravity.toFixed(1) }}</span>
          </div>
          <div class="flex justify-between">
            <span class="text-gray-500">粘性</span>
            <span class="text-gray-300 font-mono">{{ store.params.viscosity.toFixed(1) }}</span>
          </div>
          <div class="flex justify-between">
            <span class="text-gray-500">光滑半径</span>
            <span class="text-gray-300 font-mono">{{ store.params.smoothingRadius.toFixed(0) }}</span>
          </div>
          <div class="flex justify-between">
            <span class="text-gray-500">时间步长</span>
            <span class="text-gray-300 font-mono">{{ store.params.dt.toFixed(4) }}</span>
          </div>
        </div>
      </div>

      <div class="bg-gray-900 rounded p-3">
        <h4 class="text-xs font-semibold text-gray-400 mb-2">运行结果</h4>
        <div class="grid grid-cols-2 gap-1.5 text-xs">
          <div class="flex justify-between">
            <span class="text-gray-500">FPS</span>
            <span class="text-green-400 font-mono">{{ store.fps }}</span>
          </div>
          <div class="flex justify-between">
            <span class="text-gray-500">帧数</span>
            <span class="text-gray-300 font-mono">{{ store.frameCount }}</span>
          </div>
          <div class="flex justify-between">
            <span class="text-gray-500">平均密度</span>
            <span class="text-yellow-400 font-mono">{{ store.avgDensity.toFixed(0) }}</span>
          </div>
          <div class="flex justify-between">
            <span class="text-gray-500">最大速度</span>
            <span class="text-red-400 font-mono">{{ store.maxVelocity.toFixed(1) }}</span>
          </div>
        </div>
      </div>

      <div>
        <label class="text-xs text-gray-400 mb-1 block">实验结论</label>
        <textarea
          v-model="experimentConclusion"
          placeholder="请输入实验结论和观察结果..."
          rows="3"
          class="w-full bg-gray-900 border border-gray-700 rounded px-2 py-1.5 text-xs text-gray-200 focus:outline-none focus:border-purple-500 resize-none"
        ></textarea>
      </div>

      <div class="flex gap-2">
        <button
          @click="copyReport"
          class="flex-1 bg-blue-600 hover:bg-blue-700 text-white py-1.5 rounded text-xs transition"
        >
          复制报告
        </button>
        <button
          @click="downloadReport"
          class="flex-1 bg-green-600 hover:bg-green-700 text-white py-1.5 rounded text-xs transition"
        >
          导出文件
        </button>
      </div>
    </div>
  </div>
</template>
