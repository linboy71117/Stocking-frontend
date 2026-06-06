<template>
  <div class="home-container">
    <div class="search-bar">
      <input 
        v-model="searchId" 
        placeholder="輸入股票代碼 (例: 2330 或 AAPL)" 
        @keyup.enter="handleSearch"
      />
      <button @click="handleSearch" :disabled="loading">
        {{ loading ? '查詢中...' : '查詢' }}
      </button>
    </div>

    <p v-if="errorMsg" class="error">{{ errorMsg }}</p>

    <div v-if="chartData.candlestick.length > 0" class="result-section">
      <h2>{{ stockInfo.name }} ({{ stockInfo.id }})</h2>
      
      <StockChart 
        :candlestickData="chartData.candlestick"
        :ma5Data="chartData.ma5"
        :ma20Data="chartData.ma20"
      />
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';
import StockChart from '../components/StockChart.vue';

const searchId = ref('2330');
const loading = ref(false);
const errorMsg = ref('');
const stockInfo = ref({ id: '', name: '' });

// 存放準備餵給圖表的乾淨數據
const chartData = ref({
  candlestick: [],
  ma5: [],
  ma20: []
});

const handleSearch = async () => {
  if (!searchId.value.trim()) return;
  
  loading.value = true;
  errorMsg.value = '';
  
  try {
    // 向 Python 後端發送請求 (確保使用 localhost)
    const res = await axios.get(
  `${import.meta.env.VITE_API_URL}/api/stock/${searchId.value}`
);
    if (res.data.status === 'success') {
      // 關鍵修正：如果後端沒撈到 stock_name (例如台股 info 噴空值)，就用「股票 + 代碼」防呆
      stockInfo.value = { 
        id: res.data.stock_id, 
        name: res.data.stock_name || `股票 ${res.data.stock_id}` 
      };
      
      // 將後端陣列灌進變數，觸發子組件渲染
      chartData.value.candlestick = res.data.candlestick;
      chartData.value.ma5 = res.data.ma5;// 確保與後端欄位對齊
      chartData.value.ma20 = res.data.ma20;
    } else {
      errorMsg.value = res.data.message || '讀取失敗';
    }
  } catch (err) {
    errorMsg.value = '無法連線到後端 API 伺服器，請確認後端 Uvicorn 是否正常運作。';
    console.error(err);
  } finally {
    loading.value = false;
  }
};
</script>

<style scoped>
.home-container {
  max-width: 1000px;
  margin: 0 auto;
  padding: 20px;
}
.search-bar {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
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
.error { 
  color: #ff4d4f; 
  font-weight: bold; 
  margin-bottom: 20px;
}
.result-section {
  background: #ffffff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.08);
}
h2 { 
  margin-top: 0; 
  color: #1f2937; 
  margin-bottom: 15px;
}
</style>
