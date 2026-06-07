<template>

<h1 style="color:red">
我是新版 HomeView
</h1>

<div class="home-container">
  <div class="home-container">

    <div class="search-wrapper">

      <div class="search-bar">

        <input
          v-model="searchId"
          placeholder="輸入股票代碼、中文名稱、英文名稱"
          @input="handleSuggest"
          @keyup.enter="handleSearch"
        />

        <button
          @click="handleSearch"
          :disabled="loading"
        >
          {{ loading ? '查詢中...' : '查詢' }}
        </button>

      </div>

      <div
        v-if="suggestions.length"
        class="suggestion-box"
      >
        <div
          v-for="item in suggestions"
          :key="item.code"
          class="suggestion-item"
          @click="selectSuggestion(item)"
        >
          <span class="stock-code">
            {{ item.code }}
          </span>

          <span class="stock-name">
            {{ item.name }}
          </span>
        </div>
      </div>

    </div>

    <!-- 時間區間 -->
    <div class="period-bar">

      <button @click="changePeriod('1mo')">
        1M
      </button>

      <button @click="changePeriod('3mo')">
        3M
      </button>

      <button @click="changePeriod('6mo')">
        6M
      </button>

      <button @click="changePeriod('1y')">
        1Y
      </button>

      <button @click="changePeriod('5y')">
        5Y
      </button>

      <button @click="changePeriod('max')">
        MAX
      </button>

    </div>

    <p
      v-if="errorMsg"
      class="error"
    >
      {{ errorMsg }}
    </p>

    <div
      v-if="chartData.candlestick.length > 0"
      class="result-section"
    >

      <h2>
        {{ stockInfo.name }}
        ({{ stockInfo.id }})
      </h2>

      <!-- 股票資料卡 -->
      <div class="stock-card">

        <div class="card-item">
          <span class="label">目前區間</span>
          <span class="value">
            {{ period }}
          </span>
        </div>

        <div class="card-item">
          <span class="label">K棒數量</span>
          <span class="value">
            {{ chartData.candlestick.length }}
          </span>
        </div>

        <div class="card-item">
          <span class="label">MA5資料</span>
          <span class="value">
            {{ chartData.ma5.length }}
          </span>
        </div>

        <div class="card-item">
          <span class="label">MA20資料</span>
          <span class="value">
            {{ chartData.ma20.length }}
          </span>
        </div>

      </div>

      <StockChart
        :candlestickData="chartData.candlestick"
        :ma5Data="chartData.ma5"
        :ma20Data="chartData.ma20"
      />

    </div>

  </div>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'
import StockChart from '../components/StockChart.vue'

const searchId = ref('2330')

const period = ref('3mo')

const loading = ref(false)

const errorMsg = ref('')

const suggestions = ref([])

const stockInfo = ref({
  id: '',
  name: ''
})

const chartData = ref({
  candlestick: [],
  ma5: [],
  ma20: []
})

const handleSuggest = async () => {

  if (!searchId.value.trim()) {
    suggestions.value = []
    return
  }

  try {

    const res = await axios.get(
      `${import.meta.env.VITE_API_URL}/api/search?q=${searchId.value}`
    )

    suggestions.value = res.data

  } catch (err) {
    console.error(err)
  }
}

const changePeriod = (newPeriod) => {

  period.value = newPeriod

  if (stockInfo.value.id) {
    handleSearch()
  }
}

const selectSuggestion = (item) => {

  searchId.value = item.code

  suggestions.value = []

  handleSearch()
}

const handleSearch = async () => {

  if (!searchId.value.trim()) return

  loading.value = true

  errorMsg.value = ''

  suggestions.value = []

  try {

    const res = await axios.get(
      `${import.meta.env.VITE_API_URL}/api/stock/${searchId.value}?period=${period.value}`
    )

    if (res.data.status === 'success') {

      stockInfo.value = {
        id: res.data.stock_id,
        name:
          res.data.stock_name ||
          `股票 ${res.data.stock_id}`
      }

      chartData.value.candlestick =
        res.data.candlestick

      chartData.value.ma5 =
        res.data.ma5

      chartData.value.ma20 =
        res.data.ma20

    } else {

      errorMsg.value =
        res.data.message || '讀取失敗'
    }

  } catch (err) {

    errorMsg.value =
      '無法連線到後端 API'

    console.error(err)

  } finally {

    loading.value = false
  }
}
</script>

<style scoped>

.home-container {
  max-width: 1000px;
  margin: 0 auto;
  padding: 20px;
}

.search-wrapper {
  position: relative;
}

.search-bar {
  display: flex;
  gap: 10px;
  margin-bottom: 10px;
}

.search-bar input {
  flex: 1;
  padding: 12px;
  font-size: 16px;
  border: 1px solid #ddd;
  border-radius: 6px;
}

.search-bar button {
  padding: 12px 24px;
  background: #41b883;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-weight: bold;
}

.search-bar button:disabled {
  background: #999;
}

.period-bar {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.period-bar button {

  padding: 8px 14px;

  border: none;

  border-radius: 6px;

  background: #f3f4f6;

  cursor: pointer;

  font-weight: bold;
}

.period-bar button:hover {

  background: #41b883;

  color: white;
}

.stock-card {

  display: grid;

  grid-template-columns:
  repeat(4,1fr);

  gap: 15px;

  margin-bottom: 20px;
}

.card-item {

  background: #f8fafc;

  border-radius: 8px;

  padding: 15px;

  text-align: center;
}

.label {

  display: block;

  color: #666;

  margin-bottom: 5px;
}

.value {

  font-size: 18px;

  font-weight: bold;
}

.suggestion-box {
  position: absolute;
  width: 100%;
  background: white;
  border: 1px solid #ddd;
  border-radius: 10px;
  overflow: hidden;
  z-index: 999;
  box-shadow: 0 4px 12px rgba(0,0,0,0.12);
}

.suggestion-item {
  padding: 12px 15px;
  cursor: pointer;
  display: flex;
  gap: 10px;
}

.suggestion-item:hover {
  background: #f3f4f6;
}

.stock-code {
  font-weight: bold;
  color: #41b883;
}

.stock-name {
  color: #333;
}

.error {
  color: #ff4d4f;
  font-weight: bold;
  margin-bottom: 20px;
}

.result-section {
  background: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow:
    0 4px 12px rgba(0,0,0,0.08);
}

h2 {
  margin-top: 0;
  color: #1f2937;
  margin-bottom: 15px;
}
</style>

