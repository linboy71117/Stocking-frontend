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

// 🎯 核心修復：從 vue3-apexcharts 套件中精準引入 apexchart 元件，解決解不開組件的噴錯！
import apexchart from 'vue3-apexcharts'

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
const mainSeries = computed(() => {
  return [
    { name: 'K線', type: 'candlestick', data: props.candlestickData },
    { name: 'MA5', type: 'line', data: props.ma5Data },
    { name: 'MA20', type: 'line', data: props.ma20Data }
  ]
})

const volumeSeries = computed(() => {
  return [{ name: '成交量', data: props.volumeData }]
})

const rsiSeries = computed(() => {
  return [{ name: 'RSI(14)', data: props.rsiData }]
})

// ==========================================
// 2. 圖表外觀與主題設定 (Options)
// ==========================================
const getCommonOptions = (chartId, groupName) => {
  const isDark = props.isDarkMode
  return {
    chart: {
      id: chartId,
      group: groupName, // 讓三張圖的十字游標完全同步對齊
      toolbar: { show: chartId === 'main-kline' },
      animations: { enabled: true }, // 關閉動畫，切換大數據時才不卡頓
      background: 'transparent',
      foreColor: isDark ? '#9ca3af' : '#4b5563'
    },
    theme: {
      mode: isDark ? 'dark' : 'light'
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

const mainChartOptions = computed(() => {
  const options = getCommonOptions('main-kline', 'stock-group')
  return {
    ...options,
    title: { text: 'K 線與移動平均線 (MA)', align: 'left' },
    plotOptions: {
      candlestick: {
        colors: {
          upward: '#10b981',   // 漲：綠
          downward: '#ef4444'  // 跌：紅
        }
      }
    },
    stroke: {
      width: [1, 2, 2]
    },
    colors: ['#10b981', '#3b82f6', '#f59e0b'],
    yaxis: {
      labels: {
        formatter: (val) => val ? val.toFixed(2) : ''
      },
      tooltip: { enabled: true }
    }
  }
})

const volumeChartOptions = computed(() => {
  const options = getCommonOptions('volume-chart', 'stock-group')
  return {
    ...options,
    title: { text: '成交量 (Volume)', align: 'left' },
    colors: ['#6b7280'],
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

const rsiChartOptions = computed(() => {
  const options = getCommonOptions('rsi-chart', 'stock-group')
  return {
    ...options,
    colors: ['#8b5cf6'],
    stroke: { width: 2 },
    yaxis: {
      min: 0,
      max: 100,
      tickAmount: 2,
      labels: { formatter: (val) => val.toFixed(0) }
    },
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