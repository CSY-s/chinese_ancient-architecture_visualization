<template>
  <div class="page-shell">
    <header class="hero card-shell">
      <div class="hero-top">
        <div class="hero-copy">
          <span class="eyebrow">保护现状与风险预警看板</span>
          <h1>中国古建筑保护现状与风险预警看板</h1>
          <p class="hero-subtitle">数据截止 2025 年底 · 基于 439 处国保样本与本地工程支撑数据专题整理</p>
          <div class="hero-actions">
            <button type="button" class="ghost-button" @click="showGallery = true">扩展图谱</button>
          </div>
        </div>
        <div class="hero-mini-cards">
          <article class="mini-stat alert-mini" @click="openPanelZoom(alertPanel)">
            <span>核心风险</span>
            <strong>{{ dangerShare }}%</strong>
            <p>破损 + 濒危</p>
          </article>
          <article v-for="item in kpiCards" :key="item.label" class="mini-stat">
            <span>{{ item.label }}</span>
            <strong>{{ item.value }}</strong>
            <p>{{ item.note }}</p>
          </article>
        </div>
      </div>
    </header>

    <section class="dashboard-grid">
      <article class="radar-card card-shell clickable" @click="openChartZoom(chartConfigs.radar)">
        <div class="card-head">
          <div>
            <span class="mini-label">风险雷达</span>
            <h3>风险维度对比</h3>
          </div>
          <button type="button" class="ghost-button" @click.stop="openChartZoom(chartConfigs.radar)">放大图</button>
        </div>
        <RadarChart @select="openDetail" />
      </article>

      <article class="state-card card-shell">
        <div class="card-head">
          <div>
            <span class="mini-label">核心主图</span>
            <h3>保存现状比例</h3>
          </div>
          <button type="button" class="ghost-button" @click.stop="openChartZoom(chartConfigs.state)">放大图</button>
        </div>
        <div class="state-layout">
          <div class="state-legend">
            <article v-for="item in statusItems" :key="item.name" class="state-item">
              <div class="state-label">
                <span class="state-dot" :style="{ background: item.color }"></span>
                <strong>{{ item.name }}</strong>
              </div>
              <span class="state-value">{{ item.value }}%</span>
            </article>
            <div class="danger-line">⚠ 危险状态合计 {{ dangerShare }}%，其中县保对象的濒危压力更高。</div>
          </div>
          <div class="state-chart">
            <PieChart @select="handleStatusDetail" />
          </div>
        </div>
      </article>

      <article class="digital-card card-shell clickable" @click="openPanelZoom(digitalPanel)">
        <div class="card-head">
          <div>
            <span class="mini-label">数字化进度</span>
            <h3>扫描 / 建模 / 全景</h3>
          </div>
          <button type="button" class="ghost-button" @click.stop="openPanelZoom(digitalPanel)">展开</button>
        </div>
        <div class="digital-layout" @click.stop>
          <div class="digital-copy">
            <article
              v-for="item in digitalPanel.summaryCards"
              :key="item.label"
              class="digital-copy-item"
            >
              <div class="digital-copy-top">
                <strong>{{ item.label }}</strong>
                <span>{{ item.value }}</span>
              </div>
              <p>{{ item.text }}</p>
            </article>
          </div>
          <div class="digital-chart">
            <DigitalProgressChart @select="openDetail" />
          </div>
        </div>
      </article>

      <article class="monitor-card card-shell clickable" @click="openPanelZoom(monitorPanel)">
        <div class="card-head">
          <div>
            <span class="mini-label">风险预警</span>
            <h3>监测覆盖与高风险点位</h3>
          </div>
          <button type="button" class="ghost-button" @click.stop="openPanelZoom(monitorPanel)">展开</button>
        </div>
        <div class="chart-panel" @click.stop>
          <RiskAlertChart @select="openDetail" />
        </div>
      </article>

      <article class="funding-card card-shell clickable" @click="openPanelZoom(fundingPanel)">
        <div class="card-head">
          <div>
            <span class="mini-label">修缮投入</span>
            <h3>投入与缺口</h3>
          </div>
          <button type="button" class="ghost-button" @click.stop="openPanelZoom(fundingPanel)">展开</button>
        </div>
        <div class="funding-layout" @click.stop>
          <div class="funding-grid">
            <article class="funding-metric">
              <span>总投入</span>
              <strong>{{ fundingMetrics.total }}</strong>
            </article>
            <article class="funding-metric">
              <span>年均投入</span>
              <strong>{{ fundingMetrics.annualAverage }}</strong>
            </article>
            <article class="funding-metric">
              <span>单处平均</span>
              <strong>{{ fundingMetrics.siteAverage }}</strong>
            </article>
            <article class="funding-metric danger-metric">
              <span>资金缺口</span>
              <strong>{{ fundingMetrics.gap }}</strong>
            </article>
          </div>
          <div class="funding-visual">
            <FundingChart @select="openDetail" />
          </div>
        </div>
      </article>

      <article class="level-card card-shell clickable" @click="openChartZoom(chartConfigs.level)">
        <div class="card-head">
          <div>
            <span class="mini-label">层级结构</span>
            <h3>国保 / 省保 / 市保 / 县保</h3>
          </div>
          <button type="button" class="ghost-button" @click.stop="openChartZoom(chartConfigs.level)">放大图</button>
        </div>
        <BarChart @select="openDetail" />
      </article>

      <article class="trend-card card-shell clickable" @click="openChartZoom(chartConfigs.trend)">
        <div class="card-head">
          <div>
            <span class="mini-label">数字化趋势</span>
            <h3>覆盖率时间变化</h3>
          </div>
          <button type="button" class="ghost-button" @click.stop="openChartZoom(chartConfigs.trend)">放大图</button>
        </div>
        <LineChart @select="openDetail" />
      </article>

      <article class="disease-card card-shell clickable" @click="openPanelZoom(diseasePanel)">
        <div class="card-head">
          <div>
            <span class="mini-label">病害排序</span>
            <h3>严重程度优先级</h3>
          </div>
          <button type="button" class="ghost-button" @click.stop="openPanelZoom(diseasePanel)">展开</button>
        </div>
        <div class="chart-panel" @click.stop>
          <DiseaseRankingChart @select="openDetail" />
        </div>
      </article>

      <article class="findings-card card-shell clickable" @click="openPanelZoom(findingsPanel)">
        <div class="card-head">
          <div>
            <span class="mini-label">关键发现</span>
            <h3>一页结论</h3>
          </div>
          <button type="button" class="ghost-button" @click.stop="openPanelZoom(findingsPanel)">展开</button>
        </div>
        <ul class="findings-list">
          <li v-for="item in keyFindings.slice(0, 3)" :key="item">{{ item }}</li>
        </ul>
        <p class="bottom-conclusion">27% 处于破损或濒危状态，资金缺口约 3.8 亿元，高风险点位 214 个。</p>
      </article>
    </section>

    <Transition name="zoom-fade">
      <div v-if="showGallery" class="zoom-mask" @click.self="showGallery = false">
        <section class="zoom-panel card-shell gallery-panel">
          <div class="zoom-head">
            <div>
              <span class="detail-kicker">扩展图谱</span>
              <h2>更多可视化图形</h2>
              <p class="zoom-subtitle">补充比较与排序、局部与整体、分布、时间趋势、相关性和网络关系等更多图表类型。</p>
            </div>
            <button type="button" class="ghost-button" @click="showGallery = false">收起</button>
          </div>
          <ExtendedChartGallery />
        </section>
      </div>
    </Transition>

    <Transition name="zoom-fade">
      <div v-if="zoomedChart" class="zoom-mask" @click.self="closeChartZoom">
        <section class="zoom-panel card-shell chart-modal">
          <div class="zoom-head">
            <div>
              <span class="detail-kicker">图形放大</span>
              <h2>{{ zoomedChart.title }}</h2>
              <p class="zoom-subtitle">{{ zoomedChart.subtitle }}</p>
            </div>
            <button type="button" class="ghost-button" @click="closeChartZoom">收起</button>
          </div>
          <template v-if="zoomedChart.key === 'state'">
            <div class="state-zoom-layout">
              <div class="zoom-chart-stage state-chart-stage">
                <component :is="zoomedChart.component" @select="handleStatusDetail" />
              </div>
              <section v-if="activeStatePanel" class="state-side-panel">
                <div class="panel-headline">
                  <span class="detail-kicker">{{ activeStatePanel.kicker }}</span>
                  <h3>{{ activeStatePanel.title }}</h3>
                  <p>{{ activeStatePanel.subtitle }}</p>
                </div>
                <div class="zoom-note-grid single-note-grid">
                  <article v-for="card in activeStatePanel.summaryCards" :key="card.label" class="zoom-note-card">
                    <span>{{ card.label }}</span>
                    <strong>{{ card.value }}</strong>
                    <p>{{ card.text }}</p>
                  </article>
                </div>
                <div class="building-card-grid">
                  <article v-for="item in activeStatePanel.buildingCards" :key="item.name" class="building-card">
                    <span>{{ item.region }}</span>
                    <strong>{{ item.name }}</strong>
                    <em>{{ item.type }}</em>
                    <p>{{ item.text }}</p>
                  </article>
                </div>
                <div v-if="activeStatePanel.sections" class="state-section-grid">
                  <article v-for="section in activeStatePanel.sections" :key="section.title" class="zoom-section compact-section">
                    <h3>{{ section.title }}</h3>
                    <ul>
                      <li v-for="item in section.items" :key="item">{{ item }}</li>
                    </ul>
                  </article>
                </div>
              </section>
            </div>
          </template>
          <template v-else>
            <div class="zoom-chart-stage">
              <component :is="zoomedChart.component" @select="openDetail" />
            </div>
            <div class="zoom-note-grid">
              <article class="zoom-note-card">
                <span>数据来源</span>
                <strong>{{ zoomedChart.source }}</strong>
                <p>{{ zoomedChart.sourceText }}</p>
              </article>
              <article class="zoom-note-card">
                <span>分析方法</span>
                <strong>{{ zoomedChart.method }}</strong>
                <p>{{ zoomedChart.methodText }}</p>
              </article>
              <article class="zoom-note-card">
                <span>结论</span>
                <strong>{{ zoomedChart.finding }}</strong>
                <p>{{ zoomedChart.findingText }}</p>
              </article>
            </div>
            <div v-if="zoomedChart.metricCards" class="zoom-metric-grid">
              <article v-for="card in zoomedChart.metricCards" :key="card.label" class="zoom-metric-card">
                <span>{{ card.label }}</span>
                <strong>{{ card.value }}</strong>
                <p>{{ card.text }}</p>
              </article>
            </div>
            <div v-if="zoomedChart.actionCards" class="zoom-metric-grid">
              <article v-for="card in zoomedChart.actionCards" :key="card.label" class="zoom-metric-card action-card">
                <span>{{ card.label }}</span>
                <strong>{{ card.value }}</strong>
                <p>{{ card.text }}</p>
              </article>
            </div>
            <div v-if="zoomedChart.sections" class="zoom-sections detailed-sections">
              <article v-for="section in zoomedChart.sections" :key="section.title" class="zoom-section">
                <h3>{{ section.title }}</h3>
                <p>{{ section.intro }}</p>
                <ul>
                  <li v-for="item in section.items" :key="item">{{ item }}</li>
                </ul>
              </article>
            </div>
          </template>
        </section>
      </div>
    </Transition>

    <Transition name="zoom-fade">
      <div v-if="zoomedPanel" class="zoom-mask" @click.self="closePanelZoom">
        <section class="zoom-panel card-shell">
          <div class="zoom-head">
            <div>
              <span class="detail-kicker">{{ zoomedPanel.kicker }}</span>
              <h2>{{ zoomedPanel.title }}</h2>
              <p class="zoom-subtitle">{{ zoomedPanel.subtitle }}</p>
            </div>
            <button type="button" class="ghost-button" @click="closePanelZoom">收起</button>
          </div>
          <div v-if="zoomedPanel.component" class="zoom-chart-stage panel-chart-stage">
            <component :is="zoomedPanel.component" @select="openDetail" />
          </div>
          <div v-if="zoomedPanel.summaryCards" class="zoom-note-grid">
            <article v-for="card in zoomedPanel.summaryCards" :key="card.label" class="zoom-note-card">
              <span>{{ card.label }}</span>
              <strong>{{ card.value }}</strong>
              <p>{{ card.text }}</p>
            </article>
          </div>
          <div v-if="zoomedPanel.evidenceCards" class="zoom-metric-grid">
            <article v-for="card in zoomedPanel.evidenceCards" :key="card.label" class="zoom-metric-card">
              <span>{{ card.label }}</span>
              <strong>{{ card.value }}</strong>
              <p>{{ card.text }}</p>
            </article>
          </div>
          <div v-if="zoomedPanel.metrics" class="zoom-bars">
            <article v-for="item in zoomedPanel.metrics" :key="item.label" class="metric-bar-item">
              <div class="metric-top">
                <span>{{ item.label }}</span>
                <strong>{{ item.value }}</strong>
              </div>
              <div class="metric-track">
                <div class="metric-fill" :class="zoomedPanel.fillClass || 'teal-fill'" :style="{ width: item.width || `${item.value}%` }"></div>
              </div>
              <p v-if="item.text" class="metric-text">{{ item.text }}</p>
            </article>
          </div>
          <div class="zoom-sections">
            <section v-for="section in zoomedPanel.sections" :key="section.title" class="zoom-section">
              <h3>{{ section.title }}</h3>
              <p>{{ section.intro }}</p>
              <ul>
                <li v-for="item in section.items" :key="item">{{ item }}</li>
              </ul>
            </section>
          </div>
        </section>
      </div>
    </Transition>

    <Transition name="zoom-fade">
      <div v-if="selectedDetail" class="detail-toast" :class="{ pulse: detailPulse }">
        <strong>{{ selectedDetail.title }}</strong>
        <span>{{ selectedDetail.subtitle }}</span>
      </div>
    </Transition>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import BarChart from './components/BarChart.vue'
import DigitalProgressChart from './components/DigitalProgressChart.vue'
import DiseaseRankingChart from './components/DiseaseRankingChart.vue'
import ExtendedChartGallery from './components/ExtendedChartGallery.vue'
import FundingChart from './components/FundingChart.vue'
import LineChart from './components/LineChart.vue'
import PieChart from './components/PieChart.vue'
import RadarChart from './components/RadarChart.vue'
import RiskAlertChart from './components/RiskAlertChart.vue'
import {
  defaultDetail,
  digitalProgressMetrics,
  diseaseSeverityRanking,
  fundingMetrics,
  monitoringMetrics,
  overviewStats,
  ratingPieData
} from './components/architectureData.js'

const kpiCards = overviewStats.slice(0, 3)

const statusColorMap = {
  完好: '#5E9C76',
  一般: '#E6C15A',
  破损: '#E78B4D',
  濒危: '#C9515D'
}

const statusItems = ratingPieData.map((item) => ({
  ...item,
  color: statusColorMap[item.name]
}))

const damagedShare = ratingPieData.find((item) => item.name === '破损')?.value ?? 0
const endangeredShare = ratingPieData.find((item) => item.name === '濒危')?.value ?? 0
const dangerShare = damagedShare + endangeredShare
const baseSampleCount = Number(overviewStats.find((item) => item.label === '国保样本')?.value ?? 0)
const damagedCount = Math.round((baseSampleCount * damagedShare) / 100)
const endangeredCount = Math.round((baseSampleCount * endangeredShare) / 100)
const dangerCount = damagedCount + endangeredCount
const scanRate = digitalProgressMetrics.find((item) => item.label === '已三维扫描')?.value ?? 0
const modelRate = digitalProgressMetrics.find((item) => item.label === '已建模')?.value ?? 0
const panoramaRate = digitalProgressMetrics.find((item) => item.label === '已全景采集')?.value ?? 0
const scanModelGap = scanRate - modelRate
const modelPanoramaGap = modelRate - panoramaRate
const fullDigitalGap = scanRate - panoramaRate
const highRiskRatio = Math.round((monitoringMetrics.highRiskPoints / monitoringMetrics.totalPoints) * 100)
const crackShare = Math.round((monitoringMetrics.byType[0].value / monitoringMetrics.highRiskPoints) * 100)
const leakShare = Math.round((monitoringMetrics.byType[1].value / monitoringMetrics.highRiskPoints) * 100)
const totalFunding = parseFloat(fundingMetrics.total)
const fundingGapValue = parseFloat(fundingMetrics.gap)
const fundingGapRatio = Math.round((fundingGapValue / (totalFunding + fundingGapValue)) * 100)

const selectedDetail = ref(defaultDetail)
const zoomedChart = ref(null)
const zoomedPanel = ref(null)
const activeStatePanel = ref(null)
const showGallery = ref(false)
const detailPulse = ref(false)
let pulseTimer = null

const keyFindings = [
  '国保对象的修缮及时率高于省保、市保与县保，说明高等级对象更容易获得优先投入。',
  '完好 42%、一般 31%、破损 19%、濒危 8%，危险状态合计达到 27%。',
  '数字化推进中，三维扫描最快，建模和全景采集仍有明显补齐空间。',
  '高风险点位 214 个，其中开裂与漏水对应的预警需求最高。'
]

const statusPanels = {
  完好: {
    kicker: '完好状态',
    title: '完好建筑代表样本',
    subtitle: '这类对象更适合展示预防性保护、持续监测和数字建档。',
    summaryCards: [
      { label: '状态占比', value: '42%', text: '完好对象是预防性保护和常态监测的核心样本。' },
      { label: '代表建筑', value: '故宫、拙政园', text: '整体保存较好，适合展示长期维护机制。' }
    ],
    buildingCards: [
      { name: '故宫博物院', region: '北京', type: '木构宫殿群', text: '整体完好，重点部位长期监测。' },
      { name: '拙政园', region: '江苏苏州', type: '园林建筑', text: '整体良好，维护频率高。' }
    ],
    sections: [
      {
        title: '观察重点',
        items: ['完好对象最适合说明预防性保护。', '监测、建档和日常维护是这一层的核心。']
      },
      {
        title: '展示动作',
        items: ['配合监测点位图解释长期跟踪。', '配合数字化图解释扫描与建模如何服务维护。']
      }
    ]
  },
  一般: {
    kicker: '一般状态',
    title: '一般状态建筑样本',
    subtitle: '一般状态对象数量更多，是典型的持续维护压力层。',
    summaryCards: [
      { label: '状态占比', value: '31%', text: '对象已进入常态维护与局部干预阶段。' },
      { label: '代表建筑', value: '平遥古城、陈家祠', text: '适合展示街区和装饰密集型建筑的维护压力。' }
    ],
    buildingCards: [
      { name: '平遥古城', region: '山西', type: '古城街区', text: '整体保存较好，但局部存在更新压力。' },
      { name: '陈家祠', region: '广东广州', type: '宗祠建筑', text: '湿热环境下需要周期性修护。' }
    ],
    sections: [
      {
        title: '观察重点',
        items: ['一般状态对象是数量最多的维护压力层。', '通常还没到应急抢险，但已经需要规律性干预。']
      },
      {
        title: '展示动作',
        items: ['适合讲持续维护而不是单次大修。', '适合和街区管理、游客压力、湿热环境联动。']
      }
    ]
  },
  破损: {
    kicker: '破损状态',
    title: '破损建筑代表样本',
    subtitle: '破损对象最适合连接修缮投入、病害诊断和优先级判断。',
    summaryCards: [
      { label: '状态占比', value: '19%', text: '破损对象已经出现较明显的病害累积。' },
      { label: '代表建筑', value: '应县木塔、云冈石窟', text: '适合说明结构病害与风化病害的高风险压力。' }
    ],
    buildingCards: [
      { name: '应县木塔', region: '山西朔州', type: '木塔建筑', text: '结构变形与节点病害突出。' },
      { name: '云冈石窟', region: '山西大同', type: '石窟寺石刻', text: '风化、渗水和表面剥蚀明显。' }
    ],
    sections: [
      {
        title: '观察重点',
        items: ['破损对象已经有明显病害累积。', '最适合连接资金、病害和预警三张图一起看。']
      },
      {
        title: '展示动作',
        items: ['重点讲为什么要优先投入修缮。', '重点区分结构病害和风化病害。']
      }
    ]
  },
  濒危: {
    kicker: '濒危状态',
    title: '濒危建筑代表样本',
    subtitle: '濒危对象虽然比例不最高，但最能体现风险预警和快速响应的必要性。',
    summaryCards: [
      { label: '状态占比', value: '8%', text: '濒危对象是最需要高频监测与快速响应的一层。' },
      { label: '展示重点', value: '快速响应', text: '适合展示预警等级、抢险加固和应急措施。' }
    ],
    buildingCards: [
      { name: '高风险木构样本', region: '重点监测区', type: '高耸木构', text: '结构变形接近安全阈值。' },
      { name: '高风化石质样本', region: '石窟与石刻区', type: '石质遗产', text: '表层剥落和渗水叠加。' }
    ],
    sections: [
      {
        title: '观察重点',
        items: ['濒危对象比例不高，但最需要高频监测。', '这一层最能体现风险预警和快速响应。']
      },
      {
        title: '展示动作',
        items: ['配合风险预警图讲高风险点位。', '配合资金缺口图讲资源优先投向。']
      }
    ]
  }
}

const alertPanel = {
  kicker: '核心风险',
  title: '危险状态占比说明',
  subtitle: '把保存状态比例换算成样本规模后，可以直接看出风险对象已经不是零散个案，而是一组需要排序响应的工作清单。',
  fillClass: 'danger-fill',
  summaryCards: [
    { label: '破损占比', value: `${damagedShare}%`, text: '已经出现明显病害累积，需要进入修缮排序。' },
    { label: '濒危占比', value: `${endangeredShare}%`, text: '属于高频监测和快速响应对象。' },
    { label: '危险状态合计', value: `${dangerShare}%`, text: `按 ${baseSampleCount} 处样本折算，约对应 ${dangerCount} 处重点关注对象。` }
  ],
  evidenceCards: [
    { label: '折算样本量', value: `${dangerCount} 处`, text: `由 ${baseSampleCount} 处国保样本与 ${dangerShare}% 危险状态占比推算得出。` },
    { label: '结构特征', value: '破损多于濒危', text: `破损约 ${damagedCount} 处，说明当前压力主要集中在“已显性病害但尚可抢前治理”的阶段。` },
    { label: '管理含义', value: '先排序再加密监测', text: '危险状态不是展示性指标，而是后续修缮计划、监测频率和资金优先级的起点。' }
  ],
  metrics: [
    { label: '破损', value: `${damagedShare}%`, width: `${damagedShare}%`, text: '对应优先修缮与病害诊断。' },
    { label: '濒危', value: `${endangeredShare}%`, width: `${endangeredShare}%`, text: '对应抢险加固与预警加密。' },
    { label: '破损 + 濒危', value: `${dangerShare}%`, width: `${dangerShare}%`, text: '一句话说明当前有多少对象处在危险区间。' }
  ],
  sections: [
    {
      title: '样本判断',
      intro: '按照 F 型阅读顺序，最先看到的应该是风险总量，而且这组数字最好能被直接换算成“有多少处对象需要动作”。',
      items: [`危险状态合计 ${dangerShare}% 可以快速建立风险感。`, `折算后约 ${dangerCount} 处对象位于高压区，观众更容易理解问题规模。`, '这组数字最适合作为整页的一句话结论。']
    },
    {
      title: '关联判断',
      intro: '当破损规模明显高于濒危规模时，说明当前最值得投入的是“提前干预”，而不是只在极端状态下抢险。',
      items: [`破损约 ${damagedCount} 处，明显高于濒危约 ${endangeredCount} 处。`, '这意味着病害诊断、优先修缮和分级响应仍有窗口期。', '如果能把破损对象拦在升级前，后续资金效率通常更高。']
    },
    {
      title: '展示动作',
      intro: '这一页建议从风险规模切入，再引到资金缺口和高风险点位，让结论形成因果链。',
      items: ['先说高压对象有多少。', '再解释为什么资金必须优先投向危险状态。', '最后引到监测布点和快速响应逻辑。']
    }
  ]
}

const digitalPanel = {
  kicker: '数字化进度',
  title: '数字化推进情况',
  subtitle: '把扫描、建模、全景放在同一链条里看，重点不是“有没有做”，而是数据有没有顺利从采集走到可分析、可展示。',
  component: DigitalProgressChart,
  fillClass: 'teal-fill',
  summaryCards: [
    { label: '已三维扫描', value: '68%', text: '基础采集推进最快。' },
    { label: '已建模', value: '55%', text: '建模阶段仍有明显追赶空间。' },
    { label: '已全景采集', value: '41%', text: '展示与巡检档案补全相对较慢。' }
  ],
  evidenceCards: [
    { label: '扫描-建模落差', value: `${scanModelGap} 个百分点`, text: '说明底图采集已经铺开，但数据加工与模型资产生产还没完全跟上。' },
    { label: '建模-全景落差', value: `${modelPanoramaGap} 个百分点`, text: '说明成果展示层和巡检档案层仍是当前链条的末端短板。' },
    { label: '链条总缺口', value: `${fullDigitalGap} 个百分点`, text: '从采集到展示之间的落差越大，越说明数字成果还没有完全转化成可复用资产。' }
  ],
  metrics: digitalProgressMetrics.map((item) => ({
    ...item,
    value: `${item.value}%`,
    text: `${item.label}对应的当前覆盖率。`
  })),
  sections: [
    {
      title: '样本判断',
      intro: '数字化推进并不均衡，扫描明显快于建模和全景采集，这说明项目已经进入“数据加工能力”决定上限的阶段。',
      items: [`扫描率 ${scanRate}% 已经形成较稳定的底图基础。`, `建模率比扫描率低 ${scanModelGap} 个百分点，说明加工转化是主要堵点。`, `全景率仅 ${panoramaRate}% ，末端成果转译仍偏慢。`]
    },
    {
      title: '关联判断',
      intro: '数字化链条是否闭合，取决于扫描之后能否顺利进入建模和展示成果生产，因此这张图更像一条“数据转化链”而不是三个孤立百分比。',
      items: ['扫描解决“有没有底图”。', '建模解决“能不能分析”。', '全景解决“能不能展示和复核”。']
    },
    {
      title: '创新表达',
      intro: '可以把这一页理解为“数字孪生成熟度阶梯”，用链条缺口去解释后续为何要继续投向建模和成果归档。',
      items: ['先讲扫描，再讲建模，再讲全景。', '把“落差”本身当成结论，而不是只展示完成率。', '最后回到代表建筑案例，说明哪些对象已经具备深度分析条件。']
    }
  ]
}

const fundingPanel = {
  kicker: '修缮投入',
  title: '投入规模与资金缺口',
  subtitle: '这一板块不只是说明投了多少钱，而是判断投入结构能否覆盖当前风险规模，以及缺口会把哪些对象继续推向高压区。',
  component: FundingChart,
  summaryCards: [
    { label: '总投入', value: fundingMetrics.total, text: '当前专题口径下的累计投入估算。' },
    { label: '年均投入', value: fundingMetrics.annualAverage, text: '反映年度投入节奏。' },
    { label: '单处平均', value: fundingMetrics.siteAverage, text: '反映单体对象可获得的平均修缮资源。' },
    { label: '资金缺口', value: fundingMetrics.gap, text: '说明实际投入仍不足以覆盖全部需求。' }
  ],
  evidenceCards: [
    { label: '缺口占需求比', value: `${fundingGapRatio}%`, text: '把现有投入和缺口合并看，说明仍有接近四分之一的资金需求没有被覆盖。' },
    { label: '风险对应关系', value: `${dangerCount} 处高压对象`, text: '结合危险状态样本量看，当前资源仍需要非常明确的优先级分配。' },
    { label: '决策提示', value: '不能平均分配', text: '越是高风险、高病害耦合对象，越需要在资金安排中获得更高权重。' }
  ],
  metrics: [
    { label: '总投入', value: fundingMetrics.total, width: '100%', text: '作为当前板块的基准强度。' },
    { label: '年均投入', value: fundingMetrics.annualAverage, width: '67%', text: '适合观察年度修缮节奏。' },
    { label: '单处平均', value: fundingMetrics.siteAverage, width: '46%', text: '更适合与基层对象的修缮压力一起读。' },
    { label: '资金缺口', value: fundingMetrics.gap, width: '82%', text: '说明当前仍存在明显的资源缺口。' }
  ],
  sections: [
    {
      title: '样本判断',
      intro: '金额类指标更适合作为独立板块呈现，因为它承担的是资源判断，而不是覆盖率描述。',
      items: ['总投入反映整体资金强度。', `资金缺口约占总需求的 ${fundingGapRatio}% 。`, `危险状态约 ${dangerCount} 处时，资金排序就会直接影响修缮效果。`]
    },
    {
      title: '关联判断',
      intro: '看这组数时不要只看金额大小，还要看连续性、密度和缺口是否与风险规模匹配。',
      items: ['总投入说明总体规模。', '年均投入说明连续性。', '单处平均说明资源是否足够下沉到具体对象。', '资金缺口说明为什么仍需排序。']
    },
    {
      title: '创新表达',
      intro: '可以把这一页讲成“风险转预算”的桥梁页，让观众看到风险并不会自动变成资源，必须经过排序和分配机制。',
      items: ['先看危险状态占比。', '再看资金缺口。', '最后说明为什么要优先投向高风险点位和高病害耦合对象。']
    }
  ]
}

const monitorPanel = {
  kicker: '风险预警',
  title: '监测覆盖与高风险点位',
  subtitle: '这是最接近实际管理场景的一组数据，核心不是病害名称本身，而是哪些点位已经进入需要快速响应的阈值区。',
  component: RiskAlertChart,
  fillClass: 'danger-fill',
  summaryCards: [
    { label: '监测点位', value: String(monitoringMetrics.totalPoints), text: '反映整体监测覆盖规模。' },
    { label: '高风险点位', value: String(monitoringMetrics.highRiskPoints), text: '说明最需要预警响应的点位规模。' },
    { label: '覆盖率', value: `${monitoringMetrics.coverageRate}%`, text: '反映重点对象纳入监测体系的程度。' }
  ],
  evidenceCards: [
    { label: '高风险占监测比', value: `${highRiskRatio}%`, text: `在 ${monitoringMetrics.totalPoints} 个监测点位中，约每 6 个就有 1 个进入高风险区。` },
    { label: '开裂占比', value: `${crackShare}%`, text: '开裂高风险点最多，说明结构性病害仍是最值得优先跟踪的一类。' },
    { label: '漏水占比', value: `${leakShare}%`, text: '漏水点位次高，意味着很多问题并不是孤立病害，而会继续带动腐朽和表层退化。' }
  ],
  metrics: monitoringMetrics.byType.map((item) => ({
    ...item,
    text: '对应病害类型下的重点高风险点位数量。'
  })),
  sections: [
    {
      title: '样本判断',
      intro: '高风险点位的数量比单纯病害频次更接近管理现实，因为它直接对应需要动作的对象和部位。',
      items: [`高风险点位 ${monitoringMetrics.highRiskPoints} 个，约占全部监测点位的 ${highRiskRatio}% 。`, '开裂与漏水高风险点位最多。', '高风险点位最适合与修缮排序同时阅读。']
    },
    {
      title: '关联判断',
      intro: '预警图更适合解释“先盯哪里”，而不是“什么出现得最多”，因为它天然带有阈值和响应等级。',
      items: ['高风险点位代表真正需要动作的对象。', '覆盖率说明监测体系是否跟得上风险。', '开裂与漏水并列靠前，说明结构和水害是最典型的联合风险。']
    },
    {
      title: '创新表达',
      intro: '可以把这页定义成“行动热区图”，观众会更容易理解为什么监测数据不是背景，而是决定巡检节奏和处置顺序的依据。',
      items: ['先看哪类点位最多。', '再看资金是否足够覆盖。', '最后讲预警为何需要快速响应。']
    }
  ]
}

const diseasePanel = {
  kicker: '病害排序',
  title: '病害严重程度排序',
  subtitle: '按严重程度排序展示病害优先级，让可视化从“识别病害”进一步走到“判断先救谁”。',
  component: DiseaseRankingChart,
  fillClass: 'orange-fill',
  summaryCards: [
    { label: '最高优先病害', value: '开裂', text: '结构与节点风险最高。' },
    { label: '最易连锁扩散', value: '漏水', text: '会带动腐朽和彩画老化。' },
    { label: '长期积累型', value: '风化', text: '石质与表层退化持续积累。' }
  ],
  evidenceCards: [
    { label: '第一优先', value: '开裂 92 分', text: '结构性病害一旦跨过阈值，后续修复成本和安全风险都会快速上升。' },
    { label: '第二优先', value: '漏水 86 分', text: '它最大的危险不只是单一病害，而是会持续放大木构腐朽和彩饰剥落。' },
    { label: '排序逻辑', value: '严重度而非频次', text: '当前排序强调管理急迫性，因此更接近实际修缮决策。' }
  ],
  metrics: diseaseSeverityRanking.map((item) => ({
    label: item.name,
    value: item.score,
    width: `${item.score}%`,
    text: item.note
  })),
  sections: [
    {
      title: '样本判断',
      intro: '词云只能告诉我们常见什么，但不能说明先救什么，所以这里用严重度排序来替代“出现频次”的表达方式。',
      items: ['严重程度排序更适合支持修缮优先级判断。', '开裂与漏水应作为最优先处理对象。']
    },
    {
      title: '关联判断',
      intro: '病害排序最重要的不是谁出现次数多，而是谁更急、更值得优先投入，因此这张图天然适合与预警和资金联动。',
      items: ['开裂偏结构风险。', '漏水偏连锁病害。', '风化偏长期环境退化。']
    },
    {
      title: '创新表达',
      intro: '这一页可以被讲成“病害处置优先级清单”，把专业判断转换成观众能理解的先后顺序。',
      items: ['先排序，再讲资金。', '再回到高风险点位。', '最后解释为什么优先顺序要变成行动顺序。']
    }
  ]
}

const findingsPanel = {
  kicker: '关键发现',
  title: '看板结论汇总',
  subtitle: '把层级、状态、数字化和预警四条线索汇成一页，形成“样本现状 - 风险判断 - 资源响应”的完整叙事。',
  summaryCards: [
    { label: '样本底座', value: `${baseSampleCount} 处`, text: '当前分析叙事建立在国保样本和本地工程支撑数据的联合整理之上。' },
    { label: '危险状态', value: `${dangerShare}%`, text: `约 ${dangerCount} 处对象已经进入需要优先管理的区间。` },
    { label: '关键瓶颈', value: '建模与资金', text: '一个影响数字化闭环，一个决定修缮响应强度。' }
  ],
  evidenceCards: [
    { label: '风险结论', value: '27% 高压区', text: '危险状态已经达到足以改变资源分配策略的规模。' },
    { label: '过程结论', value: `${scanModelGap}pt 数字化落差`, text: '说明当前工作重心应从采集扩展到模型加工与成果沉淀。' },
    { label: '行动结论', value: `${monitoringMetrics.highRiskPoints} 个高风险点`, text: '预警体系已经给出明确的优先巡检和响应对象。' }
  ],
  sections: [
    {
      title: '层级结论',
      intro: '文保等级最适合用来解释资源分配差异。',
      items: ['国保修缮及时率更高。', '县保濒危压力更高。', '层级差异背后本质上是资源密度与管理能力差异。']
    },
    {
      title: '状态结论',
      intro: '一般、破损和濒危对象构成了真正的压力层。',
      items: ['破损与濒危合计 27%。', '濒危对象最需要快速响应。', '破损对象更适合通过提前治理避免继续升级。']
    },
    {
      title: '过程结论',
      intro: '数字化和监测两条线决定了“能不能看见问题”和“能不能及时行动”。',
      items: [`扫描到全景之间仍有 ${fullDigitalGap} 个百分点的成果转化落差。`, `高风险点位 ${monitoringMetrics.highRiskPoints} 个，已经足以支撑差异化巡检策略。`, '数据不再只是展示材料，而是在逐步转化为决策依据。']
    },
    {
      title: '管理建议',
      intro: '整页最重要的创新点，是把分散图表组织成一条能被解释清楚的管理链路。',
      items: ['先用状态图定义问题规模。', '再用病害和预警图定义优先对象。', '最后用资金和数字化图解释治理能力与短板。']
    }
  ]
}

const chartConfigs = {
  radar: {
    key: 'radar',
    title: '风险维度对比',
    subtitle: '木构、石窟、古城等类型在病害风险和环境压力上存在差异。',
    component: RadarChart,
    source: '专题整合指标层',
    sourceText: '基于遗产类型、病害、环境与修缮需求的标准化整理结果。',
    method: '归一化评分 + 雷达比较',
    methodText: '将不同量纲统一到 0-100 分，再进行类型对照。',
    finding: '不同遗产类型的风险画像并不相同',
    findingText: '木构和石窟类对象的高风险项不一样，保护策略不能一刀切。',
    metricCards: [
      { label: '最高风险类型', value: '木构 / 石窟', text: '结构病害与环境压力均落在高值区。' },
      { label: '最高修缮需求', value: '木构古建筑', text: '节点变形、漏水、腐朽会持续累积。' },
      { label: '最高环境压力', value: '石窟寺石刻', text: '风化、渗水与微环境变化更加集中。' }
    ],
    actionCards: [
      { label: '优先监测', value: '木构节点 / 石窟表面', text: '先盯住最容易持续累积风险的位置。' },
      { label: '展示重点', value: '类型差异', text: '核心是说明不同遗产类型的风险画像不同。' },
      { label: '联动图表', value: '病害 / 预警 / 资金', text: '放大图最好和其他板块一起读。' }
    ],
    sections: [
      {
        title: '读图方法',
        intro: '这张图把不同遗产类型的多个风险维度统一到 0-100 分以后再做比较。',
        items: ['越靠外侧说明该维度越突出。', '同一类型的多个维度可以直接看形状差异。', '不同类型之间重点不一样，所以不能只看单个分数。']
      },
      {
        title: '观察结果',
        intro: '图中最明显的是木构与石窟两类对象的高风险维度并不相同。',
        items: ['木构更偏结构与修缮需求。', '石窟更偏环境压力与风化风险。', '古城街区的压力更多来自人为活动与整体治理。']
      },
      {
        title: '展示建议',
        intro: '放大图适合配合右侧细读区做“类型差异”的解释。',
        items: ['点击木构类时重点讲结构监测。', '点击石窟类时重点讲风化与微环境。', '点击古城类时重点讲街区整体预警。']
      }
    ]
  },
  state: {
    key: 'state',
    title: '保存现状比例',
    subtitle: '完好、一般、破损、濒危的比例是整页看板最核心的一组数字。',
    component: PieChart,
    source: '保存现状专题指标',
    sourceText: '状态分类经过统一口径整理后形成比例结构。',
    method: '分组统计 + 比例计算',
    methodText: '按保存状态分类统计对象数量，再计算总体占比。',
    finding: '危险状态合计 27%',
    findingText: '破损与濒危对象的合计占比已经足以支撑风险优先级判断。',
    metricCards: [
      { label: '完好', value: '42%', text: '适合展示预防性保护和长期维护机制。' },
      { label: '一般', value: '31%', text: '构成数量最多的持续维护压力层。' },
      { label: '危险状态', value: '27%', text: '破损与濒危合计，直接支撑优先级判断。' }
    ],
    actionCards: [
      { label: '优先解释', value: '27% 危险状态', text: '先让观众一眼看到风险总量。' },
      { label: '适合联动', value: '资金 / 预警', text: '说明为什么钱和监测要优先投向破损与濒危。' },
      { label: '展示路径', value: '状态切换', text: '点击状态后展示代表建筑与处置逻辑。' }
    ],
    sections: [
      {
        title: '读图方法',
        intro: '这张图先看整体比例，再看每个状态的代表建筑和管理动作。',
        items: ['完好与一般说明维护层。', '破损与濒危说明风险层。', '四个状态合起来构成完整保护结构。']
      },
      {
        title: '观察结果',
        intro: '最重要的不是单独某一项，而是“危险状态合计 27%”这组数字。',
        items: ['说明风险不再是个别案例。', '足以支撑修缮排序和预警加密。', '也解释了资金缺口为什么重要。']
      },
      {
        title: '展示建议',
        intro: '这张放大图适合作为整页看板的总入口。',
        items: ['先看比例，再点状态。', '点状态后看代表建筑。', '最后回到修缮投入与风险预警。']
      }
    ]
  },
  level: {
    key: 'level',
    title: '保护层级结构',
    subtitle: '层级结构适合解释资源投入差异和案例代表性差异。',
    component: BarChart,
    source: '保护层级分组数据',
    sourceText: '国保样本来自本地底座，其余层级用于完整呈现层级结构。',
    method: '频次统计 + 结构比较',
    methodText: '按保护层级分组计数后做柱状对比。',
    finding: '层级差异清晰',
    findingText: '高等级对象更容易获得及时修缮，县保对象的风险压力相对更高。',
    metricCards: [
      { label: '国保样本', value: '439', text: '本地样本是当前看板里最稳定的底座数据。' },
      { label: '风险压力最高', value: '县保', text: '修缮及时率和资源密度均更弱。' },
      { label: '资源响应更快', value: '国保', text: '更容易获得持续监测与阶段性修缮。' }
    ],
    actionCards: [
      { label: '优先比较', value: '国保 vs 县保', text: '最能说明资源响应差异。' },
      { label: '展示重点', value: '层级差异', text: '不是数量本身，而是资源与风险的差异。' },
      { label: '联动图表', value: '状态 / 资金', text: '层级图最适合和保存现状、投入图一起读。' }
    ],
    sections: [
      {
        title: '读图方法',
        intro: '柱高主要用来比较不同保护层级的数量和结构差异。',
        items: ['同一图中按层级展开，阅读路径很直接。', '数量只是基础，还要结合修缮及时率理解。', '不要把层级结构直接等同于风险结构。']
      },
      {
        title: '观察结果',
        intro: '层级越高的对象通常越容易获得更稳定的保护资源。',
        items: ['国保对象的修缮响应更及时。', '县保对象的濒危压力更集中。', '层级差异背后本质上是资源差异。']
      },
      {
        title: '展示建议',
        intro: '这张图适合解释“为什么同样是古建筑，保护效果会不同”。',
        items: ['配合资金图看资源分配。', '配合保存现状图看风险落点。', '配合关键发现讲决策优先级。']
      }
    ]
  },
  trend: {
    key: 'trend',
    title: '数字化覆盖率时间变化',
    subtitle: '趋势图只比较同一单位的覆盖率，避免金额与比例混用。',
    component: LineChart,
    source: '数字化进度专题指标',
    sourceText: '扫描率、建模率和全景采集率按同一覆盖率口径整理。',
    method: '时间序列整理',
    methodText: '把同一单位的覆盖率指标放到统一时间轴中比较。',
    finding: '扫描推进最快',
    findingText: '建模率和全景率仍然低于扫描率，说明数字化链条尚未完全闭合。',
    metricCards: [
      { label: '扫描率', value: '68%', text: '基础采集推进最快，已形成较稳定底图。' },
      { label: '建模率', value: '55%', text: '模型生产仍然是当前链条短板。' },
      { label: '全景率', value: '41%', text: '面向展示的数字成果相对不足。' }
    ],
    actionCards: [
      { label: '先补环节', value: '建模 / 全景', text: '这两段最适合继续投入。' },
      { label: '展示重点', value: '链条闭合', text: '不要只讲一个百分比，要讲完整流程。' },
      { label: '联动图表', value: '状态 / 案例', text: '数字化图适合接到建筑案例说明页。' }
    ],
    sections: [
      {
        title: '读图方法',
        intro: '趋势图只比较同一量纲的覆盖率，避免金额和比例混在一起。',
        items: ['线越陡说明该阶段推进越快。', '三条线之间的距离说明链条是否完整。', '后段如果收敛，代表流程开始闭合。']
      },
      {
        title: '观察结果',
        intro: '目前数字化推进呈现“扫描领先、建模跟进、全景偏慢”的结构。',
        items: ['扫描已形成较高覆盖。', '建模仍需更多加工与整理。', '全景成果更适合后续展示扩展。']
      },
      {
        title: '展示建议',
        intro: '这张图适合解释数字化工作为什么不能只看一个百分比。',
        items: ['扫描说明底图基础。', '建模说明可分析程度。', '全景说明展示传播能力。']
      }
    ]
  }
}

const resolveStatusPanel = (detail) => {
  const title = detail?.title || ''
  if (title.includes('完好')) return statusPanels.完好
  if (title.includes('一般')) return statusPanels.一般
  if (title.includes('破损')) return statusPanels.破损
  if (title.includes('濒危')) return statusPanels.濒危
  return statusPanels.完好
}

const openDetail = (detail) => {
  if (!detail) return
  selectedDetail.value = detail
  detailPulse.value = false
  clearTimeout(pulseTimer)
  requestAnimationFrame(() => {
    detailPulse.value = true
    pulseTimer = setTimeout(() => {
      detailPulse.value = false
    }, 650)
  })
}

const openChartZoom = (chart) => {
  zoomedChart.value = chart
  if (chart.key === 'state') {
    activeStatePanel.value = statusPanels.完好
  }
}

const closeChartZoom = () => {
  zoomedChart.value = null
  activeStatePanel.value = null
}

const openPanelZoom = (panel) => {
  zoomedPanel.value = panel
}

const closePanelZoom = () => {
  zoomedPanel.value = null
}

const handleStatusDetail = (detail) => {
  openDetail(detail)
  const panel = resolveStatusPanel(detail)
  if (zoomedChart.value?.key === 'state') {
    activeStatePanel.value = panel
  } else {
    openPanelZoom(panel)
  }
}
</script>

<style scoped>
.page-shell {
  --mint-1: rgb(214, 235, 232);
  --mint-2: rgb(98, 138, 138);
  --mint-3: rgb(221, 240, 238);
  --ink: rgb(46, 78, 78);
  height: calc(100vh - 28px);
  display: grid;
  grid-template-rows: auto auto minmax(0, 1fr);
  gap: 12px;
  overflow: hidden;
}

.card-shell {
  background: rgba(221, 240, 238, 0.9);
  border: 1px solid rgba(98, 138, 138, 0.18);
  box-shadow: 0 14px 24px rgba(98, 138, 138, 0.1);
  border-radius: 24px;
  backdrop-filter: blur(10px);
}

.hero {
  padding: 14px 18px;
  display: grid;
  gap: 10px;
}

.hero-top {
  display: flex;
  justify-content: space-between;
  gap: 16px;
  align-items: flex-start;
}

.eyebrow,
.mini-label,
.detail-kicker {
  display: inline-flex;
  width: fit-content;
  padding: 4px 10px;
  border-radius: 999px;
  background: rgba(98, 138, 138, 0.14);
  color: var(--mint-2);
  font-size: 0.68rem;
  letter-spacing: 0.08em;
}

.hero-copy h1 {
  margin-top: 8px;
  font-size: clamp(1.45rem, 2vw, 2.15rem);
  line-height: 1.06;
  color: var(--ink);
}

.hero-subtitle,
.metric-text,
.zoom-subtitle,
.zoom-note-card p,
.zoom-section p {
  color: rgba(46, 78, 78, 0.88);
}

.hero-subtitle {
  margin-top: 8px;
  font-size: 0.86rem;
}

.hero-actions {
  margin-top: 10px;
}

.hero-mini-cards {
  display: grid;
  grid-template-columns: repeat(4, minmax(110px, 1fr));
  gap: 8px;
  align-self: start;
  max-width: 620px;
}

.mini-stat {
  min-width: 0;
  padding: 8px 10px;
  border-radius: 18px;
  background: rgba(221, 240, 238, 0.9);
  border: 1px solid rgba(98, 138, 138, 0.16);
  box-shadow: 0 10px 18px rgba(98, 138, 138, 0.08);
}

.mini-stat span,
.funding-metric span,
.state-item strong,
.zoom-note-card span {
  display: block;
  font-size: 0.68rem;
  color: var(--mint-2);
}

.mini-stat strong,
.funding-metric strong,
.zoom-note-card strong,
.zoom-section h3,
.card-head h3 {
  color: var(--ink);
}

.mini-stat strong {
  display: block;
  margin-top: 4px;
  font-size: 1.05rem;
}

.mini-stat p {
  margin-top: 3px;
  font-size: 0.66rem;
  line-height: 1.25;
  color: rgba(46, 78, 78, 0.78);
}

.alert-mini {
  background: linear-gradient(180deg, rgba(255, 242, 240, 0.98) 0%, rgba(255, 230, 225, 0.92) 100%);
  border-color: rgba(201, 81, 93, 0.16);
  cursor: pointer;
}

.alert-mini strong {
  color: #c9515d;
}

.dashboard-grid {
  min-height: 0;
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  grid-template-rows: 0.92fr 1.12fr 0.84fr;
  gap: 12px;
  grid-template-areas:
    'radar digital monitor'
    'level main trend'
    'funding disease findings';
}

.clickable {
  cursor: pointer;
  transition: transform 0.18s ease, box-shadow 0.18s ease, border-color 0.18s ease;
}

.clickable:hover,
.ghost-button:hover {
  transform: translateY(-2px);
}

.clickable:hover {
  border-color: rgba(98, 138, 138, 0.34);
  box-shadow: 0 16px 28px rgba(98, 138, 138, 0.14);
}

.radar-card { grid-area: radar; }
.state-card { grid-area: main; }
.digital-card { grid-area: digital; }
.monitor-card { grid-area: monitor; }
.funding-card { grid-area: funding; }
.level-card { grid-area: level; }
.trend-card { grid-area: trend; }
.disease-card { grid-area: disease; }
.findings-card { grid-area: findings; }

.radar-card,
.state-card,
.digital-card,
.monitor-card,
.funding-card,
.level-card,
.trend-card,
.disease-card,
.findings-card {
  min-height: 0;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  padding: 12px;
}

.digital-card,
.monitor-card,
.funding-card,
.disease-card,
.findings-card {
  padding: 10px;
}

.digital-card {
  padding: 8px;
}

.card-head,
.zoom-head {
  display: flex;
  justify-content: space-between;
  gap: 10px;
  align-items: flex-start;
}

.card-head h3 {
  margin-top: 4px;
  font-size: 0.92rem;
  line-height: 1.18;
}

.ghost-button {
  border: 1px solid rgba(98, 138, 138, 0.22);
  background: rgba(214, 235, 232, 0.8);
  color: var(--mint-2);
  cursor: pointer;
  font-size: 0.76rem;
  border-radius: 999px;
  padding: 6px 10px;
  white-space: nowrap;
}

.radar-card :deep(.chart),
.level-card :deep(.chart),
.trend-card :deep(.chart) {
  height: 100% !important;
  min-height: 138px;
}

.state-layout {
  flex: 1;
  min-height: 0;
  display: grid;
  grid-template-columns: minmax(240px, 0.88fr) minmax(0, 1.12fr);
  gap: 10px;
  margin-top: 6px;
  align-items: start;
}

.state-chart {
  min-height: 0;
  border-radius: 18px;
  background: rgba(214, 235, 232, 0.4);
  border: 1px solid rgba(98, 138, 138, 0.1);
  padding: 4px 6px 2px;
}

.state-chart :deep(.chart) {
  height: 100% !important;
  min-height: 164px;
}

.state-legend {
  display: grid;
  align-content: start;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 6px 8px;
}

.state-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 6px;
  padding: 8px 10px;
  border-radius: 14px;
  background: rgba(214, 235, 232, 0.56);
  border: 1px solid rgba(98, 138, 138, 0.12);
}

.state-label {
  display: flex;
  align-items: center;
  gap: 8px;
}

.state-dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
}

.state-value {
  font-size: 0.92rem;
  font-weight: 700;
  color: var(--ink);
}

.danger-line {
  grid-column: 1 / -1;
  padding: 8px 10px;
  border-radius: 14px;
  background: rgba(255, 235, 231, 0.92);
  border: 1px solid rgba(201, 81, 93, 0.15);
  color: #a44a56;
  font-size: 0.76rem;
  line-height: 1.4;
}

.metric-bars,
.severity-list,
.zoom-bars {
  display: grid;
  gap: 8px;
}

.metric-bars,
.severity-list {
  margin-top: 8px;
}

.chart-panel {
  flex: 1;
  min-height: 0;
  margin-top: 8px;
  padding: 4px 4px 0;
  border-radius: 18px;
  background: rgba(214, 235, 232, 0.38);
  border: 1px solid rgba(98, 138, 138, 0.1);
}

.chart-panel :deep(.chart) {
  height: 100% !important;
  min-height: 148px;
}

.disease-card .chart-panel {
  padding: 2px 2px 0;
}

.disease-card .chart-panel :deep(.chart) {
  min-height: 96px !important;
}

.digital-layout {
  flex: 1;
  min-height: 0;
  margin-top: 6px;
  display: grid;
  grid-template-columns: minmax(150px, 0.82fr) minmax(0, 1.18fr);
  gap: 6px;
  overflow: hidden;
}

.digital-copy {
  display: grid;
  align-content: start;
  gap: 4px;
  min-height: 0;
}

.digital-copy-item {
  padding: 5px 7px;
  border-radius: 12px;
  background: rgba(214, 235, 232, 0.56);
  border: 1px solid rgba(98, 138, 138, 0.12);
  min-height: 0;
}

.digital-copy-top {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  color: var(--ink);
}

.digital-copy-top strong {
  font-size: 0.7rem;
  line-height: 1.2;
}

.digital-copy-top span {
  font-size: 0.78rem;
  font-weight: 700;
}

.digital-copy-item p {
  margin: 3px 0 0;
  font-size: 0.62rem;
  line-height: 1.2;
  color: rgba(46, 78, 78, 0.78);
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.digital-chart {
  min-height: 0;
  padding: 1px 2px 0;
  border-radius: 16px;
  background: rgba(214, 235, 232, 0.38);
  border: 1px solid rgba(98, 138, 138, 0.1);
  overflow: hidden;
}

.digital-chart :deep(.chart) {
  height: 100% !important;
  min-height: 128px;
}

.metric-bar-item {
  display: grid;
  gap: 4px;
}

.metric-top {
  display: flex;
  justify-content: space-between;
  gap: 10px;
  align-items: center;
}

.metric-top span {
  font-size: 0.76rem;
  color: var(--ink);
}

.metric-top strong {
  font-size: 0.8rem;
  color: var(--ink);
}

.metric-track {
  height: 8px;
  border-radius: 999px;
  background: rgba(98, 138, 138, 0.14);
  overflow: hidden;
}

.metric-fill {
  height: 100%;
  border-radius: inherit;
}

.teal-fill {
  background: linear-gradient(90deg, rgba(98, 138, 138, 0.55) 0%, rgba(98, 138, 138, 0.92) 100%);
}

.danger-fill {
  background: linear-gradient(90deg, rgba(201, 81, 93, 0.55) 0%, rgba(201, 81, 93, 0.92) 100%);
}

.orange-fill {
  background: linear-gradient(90deg, rgba(231, 139, 77, 0.55) 0%, rgba(231, 139, 77, 0.92) 100%);
}

.funding-layout {
  flex: 1;
  min-height: 0;
  margin-top: 6px;
  display: grid;
  grid-template-columns: minmax(148px, 0.8fr) minmax(0, 1.2fr);
  gap: 6px;
  overflow: hidden;
}

.funding-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  grid-auto-rows: minmax(0, 1fr);
  gap: 5px;
  align-content: stretch;
  min-height: 0;
}

.funding-metric {
  padding: 6px 7px;
  border-radius: 12px;
  background: rgba(214, 235, 232, 0.56);
  border: 1px solid rgba(98, 138, 138, 0.12);
  min-height: 0;
  overflow: hidden;
}

.funding-metric span {
  display: block;
  font-size: 0.68rem;
  line-height: 1.15;
  color: rgba(46, 78, 78, 0.78);
}

.funding-metric strong {
  display: block;
  margin-top: 3px;
  font-size: 0.8rem;
  line-height: 1.1;
}

.danger-metric {
  background: rgba(255, 235, 231, 0.92);
  border-color: rgba(201, 81, 93, 0.16);
}

.funding-visual {
  min-height: 0;
  padding: 4px 6px 2px;
  border-radius: 18px;
  background: rgba(214, 235, 232, 0.46);
  border: 1px solid rgba(98, 138, 138, 0.12);
  overflow: hidden;
}

.funding-visual :deep(.chart) {
  height: 100% !important;
  min-height: 142px;
}

.findings-list {
  margin-top: 8px;
  padding-left: 18px;
  display: grid;
  gap: 6px;
  font-size: 0.72rem;
  line-height: 1.28;
  color: rgba(46, 78, 78, 0.9);
  overflow: hidden;
}

.findings-list li {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.bottom-conclusion {
  margin-top: auto;
  padding: 8px 10px;
  border-radius: 16px;
  background: rgba(255, 235, 231, 0.92);
  color: #b54855;
  font-size: 0.7rem;
  line-height: 1.24;
  font-weight: 700;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.zoom-mask {
  position: fixed;
  inset: 0;
  background: rgba(46, 78, 78, 0.32);
  backdrop-filter: blur(7px);
  display: grid;
  place-items: center;
  padding: 24px;
  z-index: 20;
}

.zoom-panel {
  width: min(1100px, 100%);
  max-height: 90vh;
  overflow: auto;
  padding: 22px;
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.gallery-panel {
  width: min(1240px, 100%);
}

.detail-kicker {
  background: rgba(98, 138, 138, 0.16);
}

.zoom-head h2 {
  margin-top: 8px;
  font-size: 1.32rem;
  color: var(--ink);
}

.zoom-subtitle {
  margin-top: 6px;
  font-size: 0.84rem;
  line-height: 1.45;
}

.zoom-chart-stage {
  height: 64vh;
  min-height: 400px;
  border-radius: 20px;
  background: linear-gradient(180deg, rgba(221, 240, 238, 0.96) 0%, rgba(214, 235, 232, 0.76) 100%);
  border: 1px solid rgba(98, 138, 138, 0.14);
  padding: 12px;
}

.zoom-chart-stage :deep(.chart) {
  height: 100% !important;
}

.panel-chart-stage {
  min-height: 360px;
  margin-bottom: 10px;
}

.state-zoom-layout {
  display: grid;
  grid-template-columns: 0.88fr 1.12fr;
  gap: 14px;
  align-items: start;
}

.state-chart-stage {
  min-height: 440px;
}

.state-side-panel {
  min-height: 0;
  display: flex;
  flex-direction: column;
  gap: 12px;
  padding: 12px;
  border-radius: 20px;
  background: rgba(214, 235, 232, 0.48);
  border: 1px solid rgba(98, 138, 138, 0.12);
}

.panel-headline h3 {
  margin-top: 8px;
  font-size: 1.08rem;
  color: var(--ink);
}

.panel-headline p {
  margin-top: 6px;
  font-size: 0.82rem;
  line-height: 1.45;
  color: rgba(46, 78, 78, 0.84);
}

.single-note-grid {
  grid-template-columns: 1fr;
}

.building-card-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 10px;
}

.building-card {
  padding: 12px;
  border-radius: 18px;
  background: linear-gradient(180deg, rgba(221, 240, 238, 0.96) 0%, rgba(214, 235, 232, 0.7) 100%);
  border: 1px solid rgba(98, 138, 138, 0.12);
}

.building-card span {
  display: block;
  font-size: 0.72rem;
  color: var(--mint-2);
}

.building-card strong {
  display: block;
  margin-top: 4px;
  font-size: 0.92rem;
  color: var(--ink);
}

.building-card em {
  display: block;
  margin-top: 4px;
  font-style: normal;
  font-size: 0.76rem;
  color: rgba(46, 78, 78, 0.82);
}

.building-card p {
  margin-top: 6px;
  font-size: 0.78rem;
  line-height: 1.42;
  color: rgba(46, 78, 78, 0.88);
}

.zoom-note-grid {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 10px;
}

.zoom-metric-grid {
  margin-top: 10px;
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 10px;
}

.zoom-metric-card {
  padding: 14px;
  border-radius: 18px;
  background: rgba(214, 235, 232, 0.58);
  border: 1px solid rgba(98, 138, 138, 0.12);
}

.state-section-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 10px;
}

.compact-section ul {
  margin-top: 8px;
  padding-left: 18px;
  font-size: 0.8rem;
  line-height: 1.48;
  color: rgba(46, 78, 78, 0.9);
}

.zoom-metric-card span {
  display: block;
  font-size: 0.78rem;
  color: rgba(46, 78, 78, 0.78);
}

.zoom-metric-card strong {
  display: block;
  margin-top: 6px;
  font-size: 1.08rem;
  color: var(--ink);
}

.zoom-metric-card p {
  margin-top: 6px;
  font-size: 0.78rem;
  line-height: 1.45;
  color: rgba(46, 78, 78, 0.88);
}

.action-card {
  background: rgba(255, 235, 231, 0.74);
  border-color: rgba(201, 81, 93, 0.12);
}

.zoom-note-card,
.zoom-section {
  padding: 14px;
  border-radius: 18px;
  background: rgba(214, 235, 232, 0.58);
  border: 1px solid rgba(98, 138, 138, 0.12);
}

.zoom-section ul {
  margin-top: 8px;
  padding-left: 18px;
  font-size: 0.82rem;
  line-height: 1.55;
  color: rgba(46, 78, 78, 0.9);
}

.zoom-sections {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 10px;
}

.detailed-sections {
  margin-top: 10px;
}

.detail-toast {
  position: fixed;
  right: 22px;
  bottom: 20px;
  z-index: 10;
  display: grid;
  gap: 4px;
  padding: 12px 14px;
  border-radius: 18px;
  background: rgba(221, 240, 238, 0.95);
  border: 1px solid rgba(98, 138, 138, 0.18);
  box-shadow: 0 10px 20px rgba(98, 138, 138, 0.14);
}

.detail-toast strong {
  color: var(--ink);
  font-size: 0.88rem;
}

.detail-toast span {
  color: rgba(46, 78, 78, 0.85);
  font-size: 0.76rem;
}

.detail-toast.pulse {
  box-shadow: 0 0 0 2px rgba(98, 138, 138, 0.18), 0 14px 24px rgba(98, 138, 138, 0.18);
}

.zoom-fade-enter-active,
.zoom-fade-leave-active {
  transition: opacity 0.22s ease;
}

.zoom-fade-enter-from,
.zoom-fade-leave-to {
  opacity: 0;
}

@media (max-width: 1480px) {
  .dashboard-grid {
    grid-template-columns: repeat(3, minmax(0, 1fr));
    grid-template-rows: 0.92fr 1.08fr 0.84fr;
  }
}

@media (max-width: 1180px) {
  .page-shell {
    height: auto;
    overflow: visible;
  }

  .dashboard-grid,
  .state-layout,
  .state-zoom-layout,
  .building-card-grid,
  .zoom-note-grid,
  .zoom-sections,
  .zoom-metric-grid {
    grid-template-columns: 1fr;
  }

  .hero-top {
    flex-direction: column;
    align-items: flex-start;
  }

  .hero-mini-cards {
    width: 100%;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    max-width: none;
  }

  .state-legend {
    grid-template-columns: 1fr;
  }

  .dashboard-grid {
    grid-template-areas:
      'radar'
      'digital'
      'monitor'
      'main'
      'level'
      'trend'
      'funding'
      'disease'
      'findings';
    grid-template-rows: none;
  }

  .funding-grid {
    grid-template-columns: 1fr;
  }

  .zoom-chart-stage {
    height: 48vh;
    min-height: 300px;
  }
}
</style>
