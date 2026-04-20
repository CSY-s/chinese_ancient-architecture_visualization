<template>
  <section class="gallery-shell">
    <div class="gallery-toolbar">
      <button
        v-for="item in chartCatalog"
        :key="item.key"
        type="button"
        class="gallery-pill"
        :class="{ active: item.key === activeKey }"
        @click="activeKey = item.key"
      >
        {{ item.label }}
      </button>
    </div>
    <div class="gallery-layout">
      <div class="gallery-stage">
        <div ref="chartRef" class="chart"></div>
      </div>
      <aside class="gallery-note">
        <span>{{ currentChart.category }}</span>
        <h3>{{ currentChart.label }}</h3>
        <p>{{ currentChart.description }}</p>
        <ul>
          <li v-for="item in currentChart.points" :key="item">{{ item }}</li>
        </ul>
      </aside>
    </div>
  </section>
</template>

<script setup>
import { computed, onBeforeUnmount, onMounted, ref, watch } from 'vue'
import * as echarts from 'echarts'

const chartRef = ref(null)
const activeKey = ref('bullet')
let chart = null
let resizeHandler = null

const chartCatalog = [
  {
    key: 'bullet',
    label: '子弹图',
    category: '比较与排序',
    description: '适合把修缮完成度、目标值和警戒线放在同一条尺度里看。',
    points: ['同一视图里比较现状、目标与阈值', '适合看资金执行率和监测覆盖率']
  },
  {
    key: 'dumbbell',
    label: '哑铃图',
    category: '比较与排序',
    description: '适合展示修缮前后、投入与缺口、国保与县保之间的差距。',
    points: ['强调差值而不是绝对值', '很适合做层级对照']
  },
  {
    key: 'parallel',
    label: '平行坐标图',
    category: '比较与排序',
    description: '适合同时比较多维指标，比如风险、监测、数字化和投入。',
    points: ['适合多指标联合判断', '可以快速看出高风险轮廓']
  },
  {
    key: 'sunburst',
    label: '旭日图',
    category: '局部与整体',
    description: '适合展示层级嵌套关系，例如文保等级与保存状态的层层展开。',
    points: ['很适合整体到局部的结构阅读', '可用于分层样本展示']
  },
  {
    key: 'treemap',
    label: '矩形树图',
    category: '局部与整体',
    description: '适合看不同类别在总体中的占比，以及哪一类面积最大。',
    points: ['面积编码直观', '适合等级与病害并置展示']
  },
  {
    key: 'waterfall',
    label: '瀑布图',
    category: '局部与整体',
    description: '适合解释总投入如何被维修、监测、数字化和缺口逐项消耗。',
    points: ['特别适合讲资金构成', '能把“缺口”讲得更清楚']
  },
  {
    key: 'heatmap',
    label: '热力图',
    category: '分布',
    description: '适合展示病害类型和保护等级交叉后的热点强弱。',
    points: ['颜色表达密度高低', '适合看多类别交叉关系']
  },
  {
    key: 'scatter',
    label: '散点图',
    category: '相关性',
    description: '适合看投入与风险、数字化与监测之间是否存在关联。',
    points: ['适合做相关性判断', '可以同时编码点大小']
  },
  {
    key: 'sankey',
    label: '桑基图',
    category: '网络关系',
    description: '适合把病害、预警、修缮和资金流向串起来看。',
    points: ['特别适合过程流向展示', '能把多环节联动讲清楚']
  },
  {
    key: 'radialBar',
    label: '环形柱状图',
    category: '其他',
    description: '适合做紧凑型展示，把多个指标收在一个圆形面板里。',
    points: ['很适合单页看板', '视觉上更像专题展示图']
  }
]

const currentChart = computed(
  () => chartCatalog.find((item) => item.key === activeKey.value) ?? chartCatalog[0]
)

const buildOptions = {
  bullet: () => ({
    grid: { left: 28, right: 30, top: 20, bottom: 16, containLabel: true },
    xAxis: {
      type: 'value',
      max: 100,
      splitLine: { lineStyle: { color: 'rgba(98, 138, 138, 0.12)' } },
      axisLabel: { color: 'rgb(46, 78, 78)' }
    },
    yAxis: {
      type: 'category',
      data: ['修缮完成度', '监测覆盖率', '数字建模率'],
      axisLabel: { color: 'rgb(46, 78, 78)', fontSize: 11 },
      axisTick: { show: false },
      axisLine: { show: false }
    },
    series: [
      {
        type: 'bar',
        data: [78, 73, 55],
        barWidth: 16,
        itemStyle: { color: 'rgba(214, 235, 232, 0.95)', borderRadius: 10 },
        z: 1
      },
      {
        type: 'bar',
        data: [64, 73, 55],
        barWidth: 10,
        itemStyle: { color: 'rgb(98, 138, 138)', borderRadius: 10 },
        z: 3
      },
      {
        type: 'scatter',
        symbol: 'rect',
        symbolSize: [3, 20],
        data: [[82, 0], [78, 1], [68, 2]],
        itemStyle: { color: 'rgb(201, 81, 93)' },
        z: 4
      }
    ]
  }),
  dumbbell: () => ({
    grid: { left: 36, right: 28, top: 20, bottom: 16, containLabel: true },
    xAxis: {
      type: 'value',
      axisLabel: { color: 'rgb(46, 78, 78)' },
      splitLine: { lineStyle: { color: 'rgba(98, 138, 138, 0.12)' } }
    },
    yAxis: {
      type: 'category',
      data: ['国保', '省保', '市保', '县保'],
      axisLabel: { color: 'rgb(46, 78, 78)' },
      axisTick: { show: false },
      axisLine: { show: false }
    },
    series: [
      {
        type: 'custom',
        renderItem(params, api) {
          const y = api.coord([0, params.dataIndex])[1]
          const start = api.coord([api.value(0), params.dataIndex])[0]
          const end = api.coord([api.value(1), params.dataIndex])[0]
          return {
            type: 'line',
            shape: { x1: start, y1: y, x2: end, y2: y },
            style: { stroke: 'rgba(98, 138, 138, 0.28)', lineWidth: 3 }
          }
        },
        data: [
          [72, 41],
          [64, 46],
          [53, 39],
          [41, 28]
        ]
      },
      {
        type: 'scatter',
        data: [72, 64, 53, 41],
        symbolSize: 12,
        itemStyle: { color: 'rgb(98, 138, 138)' }
      },
      {
        type: 'scatter',
        data: [41, 46, 39, 28],
        symbolSize: 12,
        itemStyle: { color: 'rgb(201, 81, 93)' }
      }
    ]
  }),
  parallel: () => ({
    parallelAxis: [
      { dim: 0, name: '病害风险', nameTextStyle: { color: 'rgb(46, 78, 78)' }, axisLabel: { color: 'rgb(46, 78, 78)' } },
      { dim: 1, name: '环境压力', nameTextStyle: { color: 'rgb(46, 78, 78)' }, axisLabel: { color: 'rgb(46, 78, 78)' } },
      { dim: 2, name: '监测覆盖', nameTextStyle: { color: 'rgb(46, 78, 78)' }, axisLabel: { color: 'rgb(46, 78, 78)' } },
      { dim: 3, name: '投入强度', nameTextStyle: { color: 'rgb(46, 78, 78)' }, axisLabel: { color: 'rgb(46, 78, 78)' } }
    ],
    parallel: {
      left: 30,
      right: 30,
      top: 28,
      bottom: 20,
      parallelAxisDefault: {
        type: 'value',
        min: 0,
        max: 100,
        axisLine: { lineStyle: { color: 'rgba(98, 138, 138, 0.26)' } }
      }
    },
    series: [
      { type: 'parallel', lineStyle: { width: 3, color: 'rgb(98, 138, 138)' }, data: [[92, 74, 82, 68], [95, 90, 78, 62], [78, 83, 69, 48]] }
    ]
  }),
  sunburst: () => ({
    series: [{
      type: 'sunburst',
      radius: ['18%', '90%'],
      sort: null,
      itemStyle: { borderWidth: 2, borderColor: 'rgba(221, 240, 238, 0.9)' },
      label: { color: 'rgb(46, 78, 78)', rotate: 'radial', fontSize: 11 },
      data: [
        {
          name: '国保',
          children: [
            { name: '完好', value: 188 },
            { name: '一般', value: 141 },
            { name: '破损', value: 82 },
            { name: '濒危', value: 28 }
          ]
        },
        {
          name: '县保',
          children: [
            { name: '完好', value: 3200 },
            { name: '一般', value: 4100 },
            { name: '破损', value: 2100 },
            { name: '濒危', value: 1100 }
          ]
        }
      ]
    }]
  }),
  treemap: () => ({
    series: [{
      type: 'treemap',
      roam: false,
      breadcrumb: { show: false },
      label: { color: 'rgb(46, 78, 78)', fontSize: 11 },
      itemStyle: {
        borderColor: 'rgba(221, 240, 238, 0.9)',
        borderWidth: 2,
        gapWidth: 2
      },
      data: [
        { name: '木构古建', value: 38 },
        { name: '石窟寺石刻', value: 26 },
        { name: '古城街区', value: 18 },
        { name: '园林建筑', value: 12 },
        { name: '近现代建筑', value: 6 }
      ]
    }]
  }),
  waterfall: () => ({
    grid: { left: 24, right: 20, top: 22, bottom: 18, containLabel: true },
    xAxis: {
      type: 'category',
      data: ['年度预算', '常规修护', '监测系统', '数字建模', '应急加固', '资金缺口'],
      axisLabel: { color: 'rgb(46, 78, 78)', fontSize: 11 }
    },
    yAxis: {
      type: 'value',
      axisLabel: { color: 'rgb(46, 78, 78)' },
      splitLine: { lineStyle: { color: 'rgba(98, 138, 138, 0.12)' } }
    },
    series: [
      {
        type: 'bar',
        stack: 'sum',
        silent: true,
        itemStyle: { color: 'transparent' },
        emphasis: { itemStyle: { color: 'transparent' } },
        data: [0, 126, 88, 63, 45, 0]
      },
      {
        type: 'bar',
        stack: 'sum',
        data: [126, -38, -25, -18, -15, -30],
        itemStyle: {
          color: (params) => params.data >= 0 ? 'rgb(98, 138, 138)' : 'rgb(201, 81, 93)',
          borderRadius: [8, 8, 0, 0]
        }
      }
    ]
  }),
  heatmap: () => ({
    grid: { left: 48, right: 20, top: 20, bottom: 24, containLabel: true },
    xAxis: {
      type: 'category',
      data: ['国保', '省保', '市保', '县保'],
      axisLabel: { color: 'rgb(46, 78, 78)' }
    },
    yAxis: {
      type: 'category',
      data: ['开裂', '漏水', '风化', '腐朽', '人为破坏'],
      axisLabel: { color: 'rgb(46, 78, 78)' }
    },
    visualMap: {
      min: 10,
      max: 90,
      show: false,
      inRange: { color: ['rgb(221, 240, 238)', 'rgb(98, 138, 138)', 'rgb(201, 81, 93)'] }
    },
    series: [{
      type: 'heatmap',
      label: { show: true, color: 'rgb(46, 78, 78)', fontSize: 10 },
      data: [
        [0, 0, 28], [1, 0, 34], [2, 0, 46], [3, 0, 72],
        [0, 1, 22], [1, 1, 38], [2, 1, 48], [3, 1, 69],
        [0, 2, 40], [1, 2, 55], [2, 2, 58], [3, 2, 61],
        [0, 3, 18], [1, 3, 25], [2, 3, 37], [3, 3, 56],
        [0, 4, 12], [1, 4, 19], [2, 4, 26], [3, 4, 42]
      ]
    }]
  }),
  scatter: () => ({
    grid: { left: 28, right: 18, top: 18, bottom: 24, containLabel: true },
    xAxis: {
      type: 'value',
      name: '投入强度',
      nameTextStyle: { color: 'rgb(46, 78, 78)' },
      axisLabel: { color: 'rgb(46, 78, 78)' },
      splitLine: { lineStyle: { color: 'rgba(98, 138, 138, 0.12)' } }
    },
    yAxis: {
      type: 'value',
      name: '风险指数',
      nameTextStyle: { color: 'rgb(46, 78, 78)' },
      axisLabel: { color: 'rgb(46, 78, 78)' },
      splitLine: { lineStyle: { color: 'rgba(98, 138, 138, 0.12)' } }
    },
    series: [{
      type: 'scatter',
      symbolSize: (val) => val[2],
      data: [
        [72, 38, 22],
        [61, 52, 18],
        [48, 68, 25],
        [36, 82, 28],
        [55, 46, 16]
      ],
      itemStyle: { color: 'rgb(98, 138, 138)' }
    }]
  }),
  sankey: () => ({
    series: [{
      type: 'sankey',
      left: 10,
      right: 10,
      top: 10,
      bottom: 10,
      lineStyle: { color: 'gradient', curveness: 0.4 },
      label: { color: 'rgb(46, 78, 78)', fontSize: 11 },
      data: [
        { name: '病害识别' },
        { name: '风险预警' },
        { name: '修缮计划' },
        { name: '数字建模' },
        { name: '资金投入' },
        { name: '应急加固' }
      ],
      links: [
        { source: '病害识别', target: '风险预警', value: 8 },
        { source: '风险预警', target: '修缮计划', value: 6 },
        { source: '风险预警', target: '应急加固', value: 3 },
        { source: '修缮计划', target: '资金投入', value: 5 },
        { source: '数字建模', target: '修缮计划', value: 4 }
      ]
    }]
  }),
  radialBar: () => ({
    angleAxis: {
      type: 'category',
      data: ['扫描', '建模', '全景', '监测'],
      axisLabel: { color: 'rgb(46, 78, 78)', fontSize: 11 }
    },
    radiusAxis: { show: false },
    polar: {},
    series: [{
      type: 'bar',
      data: [68, 55, 41, 73],
      coordinateSystem: 'polar',
      roundCap: true,
      barWidth: 18,
      itemStyle: {
        color: (params) => ['rgb(98, 138, 138)', 'rgb(125, 170, 166)', 'rgb(150, 196, 190)', 'rgb(201, 81, 93)'][params.dataIndex]
      },
      label: {
        show: true,
        position: 'middle',
        formatter: '{c}%',
        color: 'rgb(46, 78, 78)'
      }
    }]
  })
}

const renderChart = () => {
  if (!chart) return
  const option = buildOptions[activeKey.value]?.() ?? {}
  chart.clear()
  chart.setOption(option, true)
}

onMounted(() => {
  chart = echarts.init(chartRef.value)
  renderChart()
  resizeHandler = () => chart?.resize()
  window.addEventListener('resize', resizeHandler)
})

watch(activeKey, () => {
  renderChart()
})

onBeforeUnmount(() => {
  if (resizeHandler) window.removeEventListener('resize', resizeHandler)
  chart?.dispose()
})
</script>

<style scoped>
.gallery-shell {
  display: grid;
  gap: 14px;
}

.gallery-toolbar {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.gallery-pill {
  border: 1px solid rgba(98, 138, 138, 0.2);
  background: rgba(214, 235, 232, 0.7);
  color: rgb(46, 78, 78);
  border-radius: 999px;
  padding: 7px 12px;
  cursor: pointer;
  font-size: 0.8rem;
}

.gallery-pill.active {
  background: rgb(98, 138, 138);
  color: white;
}

.gallery-layout {
  display: grid;
  grid-template-columns: minmax(0, 1.4fr) minmax(240px, 0.7fr);
  gap: 14px;
  min-height: 0;
}

.gallery-stage,
.gallery-note {
  border-radius: 20px;
  border: 1px solid rgba(98, 138, 138, 0.12);
  background: rgba(214, 235, 232, 0.45);
}

.gallery-stage {
  padding: 12px;
}

.gallery-note {
  padding: 16px;
}

.gallery-note span {
  display: inline-flex;
  padding: 4px 10px;
  border-radius: 999px;
  background: rgba(98, 138, 138, 0.14);
  color: rgb(98, 138, 138);
  font-size: 0.72rem;
}

.gallery-note h3 {
  margin-top: 10px;
  font-size: 1.08rem;
  color: rgb(46, 78, 78);
}

.gallery-note p {
  margin-top: 8px;
  font-size: 0.84rem;
  line-height: 1.55;
  color: rgba(46, 78, 78, 0.88);
}

.gallery-note ul {
  margin-top: 10px;
  padding-left: 18px;
  color: rgba(46, 78, 78, 0.88);
  font-size: 0.82rem;
  line-height: 1.55;
}

.chart {
  width: 100%;
  height: 430px;
}

@media (max-width: 980px) {
  .gallery-layout {
    grid-template-columns: 1fr;
  }

  .chart {
    height: 320px;
  }
}
</style>
