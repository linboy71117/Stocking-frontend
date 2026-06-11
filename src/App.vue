<template>
  <div id="app">
    <header class="app-header">
      <h1>📊 Stock Market 分析系統</h1>
    </header>
    <main>
      <HomeView />
    </main>
  </div>
</template>

<script setup>
import HomeView from './views/HomeView.vue';
</script>

<style>
body {
  margin: 0;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
  background-color: #f0f2f5;
}
.app-header {
  background-color: #1f2937;
  color: white;
  text-align: center;
  padding: 15px 0;
  margin-bottom: 30px;
}
.app-header h1 { margin: 0; font-size: 24px; }
</style>