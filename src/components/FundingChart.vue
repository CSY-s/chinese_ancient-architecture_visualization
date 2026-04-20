<template>
  <div ref="chartRef" class="chart"></div>
</template>

<script setup>
import { onBeforeUnmount, onMounted, ref } from 'vue'
import * as echarts from 'echarts'
import { fundingMetrics } from './architectureData.js'

const emit = defineEmits(['select'])
const chartRef = ref(null)
let chart = null
let resizeHandler = null

const fundingSeries = [
  { name: '总投入', value: 126000, display: fundingMetrics.total, color: 'rgb(98, 138, 138)' },
  { name: '年均投入', value: 21000, display: fundingMetrics.annualAverage, color: 'rgb(125, 170, 166)' },
  { name: '单处平均', value: 287, display: fundingMetrics.siteAverage, color: 'rgb(150, 196, 190)' },
  { name: '资金缺口', value: 38000, display: fundingMetrics.gap, color: 'rgb(201, 81, 93)' }
]

const detailMap = {
  总投入: {
    title: '修缮总投入',
    subtitle: '当前专题口径下的累计投入规模',
    summary: '总投入用于判断保护工程总体资金强度，适合与缺口、风险状态一起看。',
    tags: ['总投入', '资金规模', '修缮资源'],
    facts: [
      { label: '当前值', value: fundingMetrics.total },
      { label: '展示口径', value: '累计投入估算值' }
    ],
    highlights: [
      '总投入越高，越能支撑系统性修缮与预防性保护。',
      '但总投入高并不代表资金已经完全充足。'
    ]
  },
  年均投入: {
    title: '年均投入',
    subtitle: '年度资金强度的简化观察值',
    summary: '年均投入更适合观察近阶段修缮节奏，可以和趋势分析一起理解。',
    tags: ['年均投入', '年度强度', '资金节奏'],
    facts: [
      { label: '当前值', value: fundingMetrics.annualAverage },
      { label: '展示含义', value: '反映年度平均投入强度' }
    ],
    highlights: [
      '年均投入能帮助判断修缮节奏是否稳定。',
      '它和总投入一起看，能更快理解投入结构。'
    ]
  },
  单处平均: {
    title: '单处平均投入',
    subtitle: '单个保护对象的平均资金强度',
    summary: '单处平均投入更适合与对象规模、病害严重度和保护等级联动解读。',
    tags: ['单处平均', '对象强度', '资源分配'],
    facts: [
      { label: '当前值', value: fundingMetrics.siteAverage },
      { label: '展示含义', value: '反映单个对象平均可获得的资金' }
    ],
    highlights: [
      '单处平均投入偏低时，精细化修缮压力会更明显。',
      '它特别适合与基层保护对象的压力一起解释。'
    ]
  },
  资金缺口: {
    title: '资金缺口',
    subtitle: '估算需求与当前投入之间的差额',
    summary: '资金缺口最适合直接服务决策判断，它说明当前投入尚不足以覆盖全部修缮与监测需求。',
    tags: ['资金缺口', '决策判断', '现实压力'],
    facts: [
      { label: '当前值', value: fundingMetrics.gap },
      { label: '展示含义', value: '缺口越高，说明优先级排序越重要' }
    ],
    highlights: [
      '资金缺口不是装饰性指标，而是最现实的约束条件。',
      '它应与破损、濒危和高风险点位同时阅读。'
    ]
  }
}

onMounted(() => {
  chart = echarts.init(chartRef.value)
  chart.setOption({
    animationDuration: 500,
    tooltip: {
      trigger: 'axis',
      axisPointer: { type: 'shadow' },
      formatter: (params) => {
        const item = fundingSeries[params[0].dataIndex]
        return `${item.name}<br/>${item.display}`
      }
    },
    grid: {
      left: 8,
      right: 16,
      top: 18,
      bottom: 6,
      containLabel: true
    },
    xAxis: {
      type: 'value',
      axisLabel: { show: false },
      axisTick: { show: false },
      axisLine: { show: false },
      splitLine: { show: false }
    },
    yAxis: {
      type: 'category',
      data: fundingSeries.map((item) => item.name),
      axisTick: { show: false },
      axisLine: { show: false },
      axisLabel: {
        color: 'rgb(46, 78, 78)',
        fontSize: 11,
        fontWeight: 600
      }
    },
    series: [
      {
        type: 'bar',
        data: fundingSeries.map((item) => ({
          value: item.value,
          itemStyle: { color: item.color, borderRadius: [0, 9, 9, 0] },
          label: {
            show: true,
            position: 'right',
            color: 'rgb(46, 78, 78)',
            fontSize: 10,
            formatter: item.display
          }
        })),
        barWidth: 16
      }
    ]
  })

  chart.on('click', (params) => {
    const detail = detailMap[params.name]
    if (detail) emit('select', detail)
  })

  resizeHandler = () => chart?.resize()
  window.addEventListener('resize', resizeHandler)
})

onBeforeUnmount(() => {
  if (resizeHandler) window.removeEventListener('resize', resizeHandler)
  chart?.dispose()
})
</script>

<style scoped>
.chart {
  width: 100%;
  height: 190px;
}
</style>
