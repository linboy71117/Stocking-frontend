<template>
  <div class="chart-box">
    <apexchart 
      type="line" 
      height="450" 
      :options="chartOptions" 
      :series="series"
    ></apexchart>
  </div>
</template>

<script setup>
import { computed } from 'vue';
import VueApexCharts from 'vue3-apexcharts';

const apexchart = VueApexCharts;

const props = defineProps({
  candlestickData: { type: Array, default: () => [] },
  ma5Data: { type: Array, default: () => [] },
  ma20Data: { type: Array, default: () => [] }
});

// 🎯 關鍵修正：直接在 series 裡面指定均線的折線顏色 (color)
const series = computed(() => [
  { 
    name: 'K線', 
    type: 'candlestick', 
    data: props.candlestickData 
  },
  { 
    name: '5日均線(MA5)', 
    type: 'line', 
    color: '#ff4560', // 強制指定 MA5 為紅色折線
    data: props.ma5Data 
  },
  { 
    name: '20日均線(MA20)', 
    type: 'line', 
    color: '#feb019', // 強制指定 MA20 為橘色折線
    data: props.ma20Data 
  }
]);

const chartOptions = {
  chart: {
    type: 'line',
    height: 450,
    toolbar: { show: true, autoSelected: 'zoom' }
  },
  xaxis: { 
    type: 'datetime' 
  },
  yaxis: { 
    tooltip: { enabled: true } 
  },
  stroke: { 
    width: [1, 2, 2] // K線外框細，MA5、MA20 線條稍粗
  },
  // 🎯 關鍵修正：移除原本全域的 colors 屬性，避免覆蓋 K 線的紅綠色
  plotOptions: {
    candlestick: {
      colors: {
        upward: '#ef4444',  // 台灣習慣：紅漲 (使用更鮮明的 Tailwind 紅)
        downward: '#22c55e' // 台灣習慣：綠跌 (使用更鮮明的 Tailwind 綠)
      }
    }
  }
};
</script>

<style scoped>
.chart-box {
  background: #ffffff;
  padding: 10px;
  border-radius: 8px;
}
</style>
