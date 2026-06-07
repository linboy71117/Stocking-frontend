<template>
  <div class="charts-wrapper">
    <div class="chart-container">
      <apexchart
        type="candlestick"
        height="350"
        :options="mainChartOptions"
        :series="mainSeries"
      ></apexchart>
    </div>

    <div class="chart-container">
      <apexchart
        type="bar"
        height="150"
        :options="volumeChartOptions"
        :series="volumeSeries"
      ></apexchart>
    </div>

    <div class="chart-container rsi-box">
      <div class="rsi-title">📈 RSI (14) 技術指標強弱型態</div>
      <apexchart
        type="line"
        height="150"
        :options="rsiChartOptions"
        :series="rsiSeries"
      ></apexchart>
    </div>
  </div>
</template>

<script setup>
import { computed, defineProps } from 'vue'

// 接收來自 HomeView.vue 的所有新擴充數據，以及深色模式狀態
const props = defineProps({
  candlestickData: { type: Array, default: () => [] },
  ma5Data: { type: Array, default: () => [] },
  ma20Data: { type: Array, default: () => [] },
  volumeData: { type: Array, default: () => [] },
  rsiData: { type: Array, default: () => [] },
  isDarkMode: { type: Boolean, default: true }
})

// ==========================================
// 1. 圖表數據結構 (Series)
// ==========================================

// 主圖表包含：K線本身、MA5 條、MA20 條
const mainSeries = computed(() => {
  return [
    { name: 'K線', type: 'candlestick', data: props.candlestickData },
    { name: 'MA5', type: 'line', data: props.ma5Data },
    { name: 'MA20', type: 'line', data: props.ma20Data }
  ]
})

// 成交量圖表
const volumeSeries = computed(() => {
  return [{ name: '成交量', data: props.volumeData }]
})

// RSI 圖表
const rsiSeries = computed(() => {
  return [{ name: 'RSI(14)', data: props.rsiData }]
})

// ==========================================
// 2. 圖表外觀與主題設定 (Options)
// ==========================================

// 共通的網格、X軸樣式 (會隨深淺色切換自動變色)
const getCommonOptions = (chartId, groupName) => {
  const isDark = props.isDarkMode
  return {
    chart: {
      id: chartId,
      group: groupName, // 🎯 關鍵：讓同一個 group 的圖表滑鼠十字線與縮放完全連動！
      toolbar: { show: chartId === 'main-kline' }, // 只在主圖顯示工具列
      animations: { enabled: false }, // 關閉動畫，切換20年歷史大數據時才不會卡頓
      background: 'transparent',
      foreColor: isDark ? '#9ca3af' : '#4b5563' // 字體顏色切換
    },
    theme: {
      mode: isDark ? 'dark' : 'light' // 🎯 讓 ApexCharts 內建黑底/白底換裝
    },
    grid: {
      borderColor: isDark ? '#262c36' : '#e5e7eb'
    },
    xaxis: {
      type: 'datetime',
      labels: { datetimeUTC: false }
    },
    tooltip: {
      theme: isDark ? 'dark' : 'light',
      shared: true,
      x: { format: 'yyyy/MM/dd' }
    }
  }
}

// 主 K 線圖設定
const mainChartOptions = computed(() => {
  const options = getCommonOptions('main-kline', 'stock-group')
  return {
    ...options,
    title: { text: 'K 線與移動平均線 (MA)', align: 'left' },
    // 綠買紅賣（符合傳統標準看盤習慣，或可依台股改紅買綠賣）
    plotOptions: {
      candlestick: {
        colors: {
          upward: '#10b981',   // 漲：綠
          downward: '#ef4444'  // 跌：紅
        }
      }
    },
    stroke: {
      width: [1, 2, 2] // K線邊框粗細1，均線粗細2
    },
    colors: ['#10b981', '#3b82f6', '#f59e0b'], // 各條線的顏色 (K線, MA5, MA20)
    yaxis: {
      labels: {
        formatter: (val) => val ? val.toFixed(2) : ''
      },
      tooltip: { enabled: true }
    }
  }
})

// 成交量圖設定
const volumeChartOptions = computed(() => {
  const options = getCommonOptions('volume-chart', 'stock-group')
  return {
    ...options,
    title: { text: '成交量 (Volume)', align: 'left' },
    colors: ['#6b7280'], // 成交量直方圖用中性灰色
    dataLabels: { enabled: false },
    yaxis: {
      labels: {
        formatter: (val) => {
          if (val >= 1000000) return (val / 1000000).toFixed(1) + 'M'
          if (val >= 1000) return (val / 1000).toFixed(0) + 'K'
          return val
        }
      }
    }
  }
})

// RSI 指標圖設定
const rsiChartOptions = computed(() => {
  const options = getCommonOptions('rsi-chart', 'stock-group')
  const isDark = props.isDarkMode
  return {
    ...options,
    colors: ['#8b5cf6'], // RSI 用高貴紫
    stroke: { width: 2 },
    yaxis: {
      min: 0,
      max: 100,
      tickAmount: 2, // 只顯示 0, 50, 100
      labels: { formatter: (val) => val.toFixed(0) }
    },
    // 🎯 技術指標超買超賣警戒線 (RSI > 70 核心警戒，RSI < 30 超跌)
    annotations: {
      yaxis: [
        { y: 70, borderColor: '#ef4444', strokeDashArray: 3, label: { text: '超買區 (70)', style: { color: '#fff', background: '#ef4444' } } },
        { y: 30, borderColor: '#10b981', strokeDashArray: 3, label: { text: '超跌區 (30)', style: { color: '#fff', background: '#10b981' } } }
      ]
    }
  }
})
</script>

<style scoped>
.charts-wrapper {
  display: flex;
  flex-direction: column;
  gap: 20px;
  margin-top: 20px;
}

.chart-container {
  background: transparent;
  width: 100%;
}

.rsi-box {
  border-top: 1px dashed var(--border-color);
  padding-top: 15px;
}

.rsi-title {
  font-size: 14px;
  font-weight: bold;
  margin-bottom: 5px;
  color: var(--text-main);
}
</style>
