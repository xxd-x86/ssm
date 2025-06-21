<template>
  <main>
    <div class="app-container">
      <header class="app-header">
        <img src="/logo.png" class="logo" alt="Logo">
        <h1>生活助手</h1>
      </header>
      
      <div class="tabs-container">
        <div class="tab-buttons">
          <button 
            v-for="(tab, index) in tabs" 
            :key="index" 
            @click="activeTabIndex = index"
            :class="{ active: activeTabIndex === index }"
            class="tab-button"
          >
            {{ tab.name }}
          </button>
        </div>
        
        <div class="tab-content">
          <transition name="fade" mode="out-in">
            <component :is="tabs[activeTabIndex].component"></component>
          </transition>
        </div>
      </div>
      
      <footer class="app-footer">
        <p>© {{ new Date().getFullYear() }} 生活助手 - 让生活更简单</p>
      </footer>
    </div>
  </main>
</template>

<script>
import Weather from './components/Weather.vue'
import CurrencyConverter from './components/CurrencyConverter.vue'
import DailyQuote from './components/DailyQuote.vue'
import Notes from './components/Notes.vue'
import Countdown from './components/Countdown.vue'
import PaymentCode from './components/PaymentCode.vue'
import Investment from './components/Investment.vue'
import FinanceNews from './components/FinanceNews.vue'
import SavingsGoal from './components/SavingsGoal.vue'
import FinancialTips from './components/FinancialTips.vue'
import ERDiagram from './components/ERDiagram.vue'

export default {
  name: 'App',
  components: {
    Weather,
    CurrencyConverter,
    DailyQuote,
    Notes,
    Countdown,
    PaymentCode,
    Investment,
    FinanceNews,
    SavingsGoal,
    FinancialTips,
    ERDiagram
  },
  data() {
    return {
      activeTabIndex: 0,
      tabs: [
        { name: 'ER图工具', component: 'ERDiagram' },
        { name: '天气查询', component: 'Weather' },
        { name: '汇率转换', component: 'CurrencyConverter' },
        { name: '每日名言', component: 'DailyQuote' },
        { name: '便签记事', component: 'Notes' },
        { name: '纪念日', component: 'Countdown' },
        { name: '投资理财', component: 'Investment' },
        { name: '财经新闻', component: 'FinanceNews' },
        { name: '存钱目标', component: 'SavingsGoal' },
        { name: '财务知识', component: 'FinancialTips' },
        { name: '收款码', component: 'PaymentCode' }
      ]
    }
  }
}
</script>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: 'Helvetica Neue', Helvetica, 'PingFang SC', 'Hiragino Sans GB',
    'Microsoft YaHei', '微软雅黑', Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  background-color: #f5f7fa;
  color: #2c3e50;
}

.app-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.app-header {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 30px;
  padding: 20px 0;
}

.logo {
  height: 40px;
  margin-right: 15px;
}

h1 {
  font-size: 2rem;
  font-weight: 600;
  color: #333;
}

.tabs-container {
  flex: 1;
  background-color: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  display: flex;
  flex-direction: column;
  margin-bottom: 30px;
}

.tab-buttons {
  display: flex;
  background-color: #f0f4f8;
  border-bottom: 1px solid #e0e6ed;
  overflow-x: auto;
  scrollbar-width: thin;
}

.tab-buttons::-webkit-scrollbar {
  height: 4px;
}

.tab-buttons::-webkit-scrollbar-thumb {
  background-color: rgba(0, 0, 0, 0.2);
  border-radius: 2px;
}

.tab-button {
  padding: 15px 20px;
  background: none;
  border: none;
  cursor: pointer;
  font-size: 16px;
  font-weight: 500;
  color: #606266;
  white-space: nowrap;
  border-bottom: 3px solid transparent;
  transition: all 0.3s ease;
}

.tab-button:hover {
  background-color: rgba(74, 144, 226, 0.05);
  color: #4a90e2;
}

.tab-button.active {
  color: #4a90e2;
  border-bottom-color: #4a90e2;
  background-color: #fff;
}

.tab-content {
  flex: 1;
  padding: 30px;
  overflow-y: auto;
}

.app-footer {
  text-align: center;
  padding: 20px 0;
  color: #909399;
  font-size: 14px;
}

/* 组件过渡效果 */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

@media (max-width: 768px) {
  .tabs-container {
    flex-direction: column;
  }
  
  .tab-buttons {
    flex-wrap: nowrap;
    justify-content: flex-start;
  }
  
  .tab-content {
    padding: 20px;
  }
  
  .app-header {
    margin-bottom: 20px;
  }
  
  h1 {
    font-size: 1.5rem;
  }
}

@media (max-width: 480px) {
  .app-container {
    padding: 10px;
  }
  
  .tab-button {
    padding: 12px 15px;
    font-size: 14px;
  }
  
  .tab-content {
    padding: 15px 10px;
  }
  
  .app-header {
    padding: 15px 0;
  }
  
  .logo {
    height: 30px;
  }
  
  h1 {
    font-size: 1.3rem;
  }
}
</style> 