<template>
  <div :class="{ 'dark-theme': isDarkMode }" class="home-container">
    
    <div class="header-banner">
      <h2>📈 智慧全端看盤系統</h2>
      <button @click="toggleDarkMode" class="theme-toggle-btn">
        {{ isDarkMode ? '☀️ 淺色模式' : '🌙 深色模式' }}
      </button>
    </div>

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
          class="search-btn"
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
          <span class="stock-code">{{ item.code }}</span>
          <span class="stock-name">{{ item.name }}</span>
        </div>
      </div>
    </div>

    <div class="period-bar">
      <button :class="{ active: period === '1mo' }" @click="changePeriod('1mo')">1M</button>
      <button :class="{ active: period === '3mo' }" @click="changePeriod('3mo')">3M</button>
      <button :class="{ active: period === '6mo' }" @click="changePeriod('6mo')">6M</button>
      <button :class="{ active: period === '1y' }" @click="changePeriod('1y')">1Y (日線)</button>
      <button :class="{ active: period === '5y' }" @click="changePeriod('5y')">5Y (週線)</button>
      <button :class="{ active: period === 'max' }" @click="changePeriod('max')">MAX (20年歷史)</button>
    </div>

    <p v-if="errorMsg" class="error">{{ errorMsg }}</p>

    <div v-if="chartData.candlestick.length > 0" class="result-section">

      <div class="result-header">
        <h2>{{ stockInfo.name }} ({{ stockInfo.id }})</h2>
      </div>

      <div class="stock-card">
        <div class="card-item">
          <span class="label">公司全名</span>
          <span class="value">{{ stockInfo.longName || 'N/A' }}</span>
        </div>
        <div class="card-item">
          <span class="label">所屬產業</span>
          <span class="value">{{ stockInfo.industry || 'N/A' }}</span>
        </div>
        <div class="card-item">
          <span class="label">總市值</span>
          <span class="value highlight-red">{{ stockInfo.marketCap || 'N/A' }}</span>
        </div>
        <div class="card-item">
          <span class="label">本益比 (PE)</span>
          <span class="value">{{ stockInfo.trailingPE || 'N/A' }}</span>
        </div>
        <div class="card-item">
          <span class="label">股息殖利率</span>
          <span class="value text-green">{{ stockInfo.dividendYield || '0%' }}</span>
        </div>
        <div class="card-item">
          <span class="label">本筆資料 K 棒總數</span>
          <span class="value text-blue">{{ chartData.candlestick.length }} 根</span>
        </div>
      </div>

      <div class="ai-section">
        <div class="ai-header">
          <h3>🤖 AI 智慧分析大腦</h3>
          <button @click="getAIAnalysis" :disabled="aiLoading" class="ai-btn">
            {{ aiLoading ? '🧠 核心思維運算中...' : '💡 生成今日 AI 股評分析' }}
          </button>
        </div>
        <div v-if="aiAnalysisText" class="ai-body">
          <p class="ai-text">{{ aiAnalysisText }}</p>
        </div>
      </div>

      <StockChart
        :candlestickData="chartData.candlestick"
        :ma5Data="chartData.ma5"
        :ma20Data="chartData.ma20"
        :volumeData="chartData.volume"
        :rsiData="chartData.rsi"
        :isDarkMode="isDarkMode"
      />

    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

// 🎯 核心修復：精準引入改好的 StockChart 元件，這是圖表能不能顯靈、多開成交量與RSI的關鍵！
import StockChart from '../components/StockChart.vue'

// 基礎核心狀態
const searchId = ref('2330')
const period = ref('3mo')
const loading = ref(false)
const errorMsg = ref('')
const suggestions = ref([])

// 智慧化功能狀態
const isDarkMode = ref(true) // 預設看盤採用高質感深色主題
const aiLoading = ref(false)
const aiAnalysisText = ref('')

// 擴充後的股票基本資訊儲存庫
const stockInfo = ref({
  id: '',
  name: '',
  longName: '',
  industry: '',
  marketCap: '',
  trailingPE: '',
  dividendYield: ''
})

// 擴充後的圖表資料結構（加入成交量與 RSI）
const chartData = ref({
  candlestick: [],
  ma5: [],
  ma20: [],
  volume: [],
  rsi: []
})

// 輸入即時推薦
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
    console.error('推薦字抓取失敗:', err)
  }
}

// 時間區間切換
const changePeriod = (newPeriod) => {
  period.value = newPeriod
  if (stockInfo.value.id) {
    handleSearch()
  }
}

// 選擇推薦選項
const selectSuggestion = (item) => {
  searchId.value = item.code
  suggestions.value = []
  handleSearch()
}

// 發動核心數據查詢
const handleSearch = async () => {
  if (!searchId.value.trim()) return

  loading.value = true
  errorMsg.value = ''
  suggestions.value = []
  aiAnalysisText.value = '' // 切換股票時，清空上一次的 AI 分析報告

  try {
    const res = await axios.get(
      `${import.meta.env.VITE_API_URL}/api/stock/${searchId.value}?period=${period.value}`
    )

    if (res.data.status === 'success') {
      // 1. 寫入股票基礎與擴充卡片資料
      stockInfo.value = {
        id: res.data.stock_id,
        name: res.data.stock_name || `股票 ${res.data.stock_id}`,
        longName: res.data.info_card?.longName,
        industry: res.data.info_card?.industry,
        marketCap: res.data.info_card?.marketCap,
        trailingPE: res.data.info_card?.trailingPE,
        dividendYield: res.data.info_card?.dividendYield
      }

      // 2. 寫入所有圖表所需的陣列數據
      chartData.value.candlestick = res.data.candlestick || []
      chartData.value.ma5 = res.data.ma5 || []
      chartData.value.ma20 = res.data.ma20 || []
      chartData.value.volume = res.data.volume || []
      chartData.value.rsi = res.data.rsi || []

    } else {
      errorMsg.value = res.data.message || '讀取失敗'
    }
  } catch (err) {
    errorMsg.value = '無法連線到雲端後端 API，請確認伺服器運作狀態。'
    console.error(err)
  } finally {
    loading.value = false
  }
}

// 請求後端生成 AI 股評導航分析
const getAIAnalysis = async () => {
  if (!stockInfo.value.id) return
  aiLoading.value = true
  try {
    
    const res = await axios.get(
        `${import.meta.env.VITE_API_URL}/api/ai_analysis/${stockInfo.value.id}?period=${period.value}`
    )
    if (res.data.status === 'success') {
      aiAnalysisText.value = res.data.analysis
    } else {
      aiAnalysisText.value = 'AI 大腦暫時開小差了，請稍後再試。'
    }
  } catch (err) {
    console.error(err)
    aiAnalysisText.value = '連線至 AI 運算節點超時。'
  } finally {
    aiLoading.value = false
  }
}

// 切換切換深色/淺色模式
const toggleDarkMode = () => {
  isDarkMode.value = !isDarkMode.value
}

// 網頁初次進來自動查詢預設股票
onMounted(() => {
  handleSearch()
})
</script>

<style scoped>
/* ==========================================================================
   強力蓋台版 CSS 樣式系統 (確保深色模式100%生效)
   ========================================================================== */
.home-container {
  --bg-color: #f3f4f6 !important;
  --panel-bg: #ffffff !important;
  --text-main: #1f2937 !important;
  --text-muted: #6b7280 !important;
  --border-color: #e5e7eb !important;
  --card-item-bg: #f9fafb !important;
  
  background-color: var(--bg-color) !important;
  color: var(--text-main) !important;
  min-height: 100vh;
  padding: 30px 20px;
  transition: all 0.2s ease;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
}

/* 🌙 當啟用深色主題時，強制覆蓋所有背景與文字顏色 */
.home-container.dark-theme {
  --bg-color: #0f1115 !important;
  --panel-bg: #161a22 !important;
  --text-main: #f3f4f6 !important;
  --text-muted: #9ca3af !important;
  --border-color: #262c36 !important;
  --card-item-bg: #1d2430 !important;
}

/* 上方橫幅 */
.header-banner {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 25px;
  padding-bottom: 10px;
  border-bottom: 1px solid var(--border-color);
}
.header-banner h2 { 
  margin: 0; 
  color: var(--text-main) !important; 
}

/* 🎯 讓切換按鈕超級明顯的外觀 */
.theme-toggle-btn {
  padding: 10px 20px;
  border-radius: 20px;
  border: 2px solid #10b981;
  background: #10b981;
  color: white !important;
  cursor: pointer;
  font-weight: bold;
  font-size: 14px;
  box-shadow: 0 4px 6px rgba(0,0,0,0.1);
  transition: transform 0.1s;
}
.theme-toggle-btn:active {
  transform: scale(0.95);
}

.search-wrapper {
  position: relative;
  margin-bottom: 20px;
}

.search-bar {
  display: flex;
  gap: 12px;
}

.search-bar input {
  flex: 1;
  padding: 14px;
  font-size: 16px;
  border: 1px solid var(--border-color);
  border-radius: 8px;
  background: var(--panel-bg) !important;
  color: var(--text-main) !important;
}

.search-btn {
  padding: 14px 28px;
  background: #10b981;
  color: white !important;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: bold;
}
.search-btn:disabled { background: #6b7280; }

/* 時間區間選擇列 */
.period-bar {
  display: flex;
  gap: 8px;
  margin-bottom: 25px;
}
.period-bar button {
  padding: 10px 18px;
  border: 1px solid var(--border-color);
  border-radius: 6px;
  background: var(--panel-bg) !important;
  color: var(--text-main) !important;
  cursor: pointer;
  font-weight: bold;
}
.period-bar button:hover, .period-bar button.active {
  background: #10b981 !important;
  color: white !important;
  border-color: #10b981 !important;
}

/* 🪪 股票基本資料卡樣式 */
.stock-card {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
  gap: 12px;
  margin-bottom: 25px;
}

.card-item {
  background: var(--card-item-bg) !important;
  border: 1px solid var(--border-color);
  border-radius: 8px;
  padding: 16px 12px;
  text-align: center;
}

.label {
  display: block;
  color: var(--text-muted) !important;
  font-size: 13px;
  margin-bottom: 6px;
}

.value {
  font-size: 16px;
  font-weight: bold;
  color: var(--text-main) !important;
}
.highlight-red { color: #ef4444 !important; }
.text-green { color: #10b981 !important; }
.text-blue { color: #3b82f6 !important; }

/* 🤖 AI 智慧分析區塊 */
.ai-section {
  border: 1px solid #8b5cf6;
  background: var(--panel-bg) !important;
  border-radius: 10px;
  padding: 20px;
  margin-bottom: 25px;
}
.ai-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 12px;
}
.ai-header h3 { margin: 0; color: var(--text-main) !important; font-size: 18px; }
.ai-btn {
  background: #8b5cf6;
  color: white !important;
  border: none;
  padding: 10px 20px;
  border-radius: 6px;
  cursor: pointer;
  font-weight: bold;
}
.ai-btn:disabled { background: #c084fc; }

.ai-body {
  margin-top: 15px;
  background: rgba(139, 92, 246, 0.06);
  border-left: 4px solid #8b5cf6;
  padding: 15px;
  border-radius: 0 6px 6px 0;
}
.ai-text {
  white-space: pre-line;
  line-height: 1.6;
  margin: 0;
  color: var(--text-main) !important;
}

/* 自動推薦下拉框 */
.suggestion-box {
  position: absolute;
  width: 100%;
  background: var(--panel-bg) !important;
  border: 1px solid var(--border-color);
  border-radius: 8px;
  overflow: hidden;
  z-index: 999;
  box-shadow: 0 4px 15px rgba(0,0,0,0.15);
}

.suggestion-item {
  padding: 12px 15px;
  cursor: pointer;
  display: flex;
  gap: 12px;
  border-bottom: 1px solid var(--border-color);
}
.suggestion-item:hover { background: var(--card-item-bg) !important; }
.stock-code { font-weight: bold; color: #10b981; }
.stock-name { color: var(--text-main) !important; }

.error { color: #ef4444; font-weight: bold; margin-bottom: 20px; }

/* 主看板結果外框 */
.result-section {
  background: var(--panel-bg) !important;
  padding: 25px;
  border-radius: 10px;
  border: 1px solid var(--border-color);
  box-shadow: 0 4px 20px rgba(0,0,0,0.05);
}
</style>
