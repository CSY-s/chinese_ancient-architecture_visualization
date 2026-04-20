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
        <div class="severity-list">
          <article v-for="item in diseaseSeverityRanking" :key="item.name" class="severity-item">
            <div class="metric-top">
              <span>{{ item.name }}</span>
              <strong>{{ item.score }}</strong>
            </div>
            <div class="metric-track">
              <div class="metric-fill orange-fill" :style="{ width: `${item.score}%` }"></div>
            </div>
          </article>
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
          <div v-if="zoomedPanel.summaryCards" class="zoom-note-grid">
            <article v-for="card in zoomedPanel.summaryCards" :key="card.label" class="zoom-note-card">
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
    ]
  }
}

const alertPanel = {
  kicker: '核心风险',
  title: '危险状态占比说明',
  subtitle: '左上角数字用于第一时间告诉观众当前保护压力有多高。',
  fillClass: 'danger-fill',
  summaryCards: [
    { label: '破损占比', value: `${damagedShare}%`, text: '已经出现明显病害累积，需要进入修缮排序。' },
    { label: '濒危占比', value: `${endangeredShare}%`, text: '属于高频监测和快速响应对象。' },
    { label: '危险状态合计', value: `${dangerShare}%`, text: '这一部分对象最能体现现实风险。' }
  ],
  metrics: [
    { label: '破损', value: `${damagedShare}%`, width: `${damagedShare}%`, text: '对应优先修缮与病害诊断。' },
    { label: '濒危', value: `${endangeredShare}%`, width: `${endangeredShare}%`, text: '对应抢险加固与预警加密。' },
    { label: '破损 + 濒危', value: `${dangerShare}%`, width: `${dangerShare}%`, text: '一句话说明当前有多少对象处在危险区间。' }
  ],
  sections: [
    {
      title: '为什么放在左上角',
      intro: '按照 F 型阅读顺序，最先看到的应该是风险总量。',
      items: ['危险状态占比能够最快建立风险感。', '它最适合作为整页的一句话结论。']
    }
  ]
}

const digitalPanel = {
  kicker: '数字化进度',
  title: '数字化推进情况',
  subtitle: '展示三维扫描、建模和全景采集三条数字化工作线。',
  fillClass: 'teal-fill',
  summaryCards: [
    { label: '已三维扫描', value: '68%', text: '基础采集推进最快。' },
    { label: '已建模', value: '55%', text: '建模阶段仍有明显追赶空间。' },
    { label: '已全景采集', value: '41%', text: '展示与巡检档案补全相对较慢。' }
  ],
  metrics: digitalProgressMetrics.map((item) => ({
    ...item,
    value: `${item.value}%`,
    text: `${item.label}对应的当前覆盖率。`
  })),
  sections: [
    {
      title: '分析结论',
      intro: '数字化推进并不均衡，扫描明显快于建模和全景采集。',
      items: ['扫描率高说明基础采集动作已经铺开。', '建模率决定后续分析能力。']
    }
  ]
}

const fundingPanel = {
  kicker: '修缮投入',
  title: '投入规模与资金缺口',
  subtitle: '这一板块用资金图与指标卡同时说明修缮资源是否充足。',
  summaryCards: [
    { label: '总投入', value: fundingMetrics.total, text: '当前专题口径下的累计投入估算。' },
    { label: '资金缺口', value: fundingMetrics.gap, text: '说明实际投入仍不足以覆盖全部需求。' }
  ],
  sections: [
    {
      title: '资金判断',
      intro: '金额类指标更适合作为独立板块呈现，而不是混进覆盖率趋势图。',
      items: ['总投入反映整体资金强度。', '资金缺口直接服务决策判断。', '破损与濒危对象越多，资金压力越明显。']
    }
  ]
}

const monitorPanel = {
  kicker: '风险预警',
  title: '监测覆盖与高风险点位',
  subtitle: '这是页面中最具现实意义的一组指标。',
  fillClass: 'danger-fill',
  summaryCards: [
    { label: '监测点位', value: String(monitoringMetrics.totalPoints), text: '反映整体监测覆盖规模。' },
    { label: '高风险点位', value: String(monitoringMetrics.highRiskPoints), text: '说明最需要预警响应的点位规模。' }
  ],
  metrics: monitoringMetrics.byType.map((item) => ({
    ...item,
    text: '对应病害类型下的重点高风险点位数量。'
  })),
  sections: [
    {
      title: '关键结论',
      intro: '高风险点位的数量比单纯病害频次更接近管理现实。',
      items: ['开裂与漏水高风险点位最多。', '高风险点位最适合与修缮排序同时阅读。']
    }
  ]
}

const diseasePanel = {
  kicker: '病害排序',
  title: '病害严重程度排序',
  subtitle: '按严重程度排序展示病害优先级。',
  fillClass: 'orange-fill',
  metrics: diseaseSeverityRanking.map((item) => ({
    label: item.name,
    value: item.score,
    width: `${item.score}%`,
    text: item.note
  })),
  sections: [
    {
      title: '为什么不用词云做主图',
      intro: '词云只能告诉我们常见什么，但不能说明先救什么。',
      items: ['严重程度排序更适合支持修缮优先级判断。', '开裂与漏水应作为最优先处理对象。']
    }
  ]
}

const findingsPanel = {
  kicker: '关键发现',
  title: '看板结论汇总',
  subtitle: '把最重要的结构判断、状态判断和过程判断汇总到一起。',
  sections: [
    {
      title: '层级结论',
      intro: '文保等级最适合用来解释资源分配差异。',
      items: ['国保修缮及时率更高。', '县保濒危压力更高。']
    },
    {
      title: '状态结论',
      intro: '一般、破损和濒危对象构成了真正的压力层。',
      items: ['破损与濒危合计 27%。', '濒危对象最需要快速响应。']
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
    findingText: '木构和石窟类对象的高风险项不一样，保护策略不能一刀切。'
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
    findingText: '破损与濒危对象的合计占比已经足以支撑风险优先级判断。'
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
    findingText: '高等级对象更容易获得及时修缮，县保对象的风险压力相对更高。'
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
    findingText: '建模率和全景率仍然低于扫描率，说明数字化链条尚未完全闭合。'
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
  min-height: 170px;
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

.funding-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 8px;
  margin-top: 8px;
}

.funding-metric {
  padding: 10px;
  border-radius: 16px;
  background: rgba(214, 235, 232, 0.56);
  border: 1px solid rgba(98, 138, 138, 0.12);
}

.funding-metric strong {
  display: block;
  margin-top: 4px;
  font-size: 0.94rem;
}

.danger-metric {
  background: rgba(255, 235, 231, 0.92);
  border-color: rgba(201, 81, 93, 0.16);
}

.funding-visual {
  flex: 1;
  min-height: 0;
  margin-top: 10px;
  padding: 8px 10px 4px;
  border-radius: 18px;
  background: rgba(214, 235, 232, 0.46);
  border: 1px solid rgba(98, 138, 138, 0.12);
}

.funding-visual :deep(.chart) {
  height: 100% !important;
  min-height: 176px;
}

.findings-list {
  margin-top: 8px;
  padding-left: 18px;
  display: grid;
  gap: 8px;
  font-size: 0.78rem;
  line-height: 1.42;
  color: rgba(46, 78, 78, 0.9);
}

.bottom-conclusion {
  margin-top: auto;
  padding: 8px 10px;
  border-radius: 16px;
  background: rgba(255, 235, 231, 0.92);
  color: #b54855;
  font-size: 0.76rem;
  line-height: 1.35;
  font-weight: 700;
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
  .zoom-sections {
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
