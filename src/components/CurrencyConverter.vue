<template>
  <div class="currency-converter">
    <h3>汇率转换器</h3>
    <div class="converter-form">
      <div class="input-group">
        <input 
          type="number" 
          v-model="amount" 
          placeholder="输入金额" 
          @input="convertCurrency"
        />
        <select v-model="fromCurrency" @change="convertCurrency">
          <option v-for="currency in currencies" :key="currency.code" :value="currency.code">
            {{ currency.code }} - {{ currency.name }}
          </option>
        </select>
      </div>
      
      <div class="exchange-icon">
        <button @click="swapCurrencies">
          <span class="swap-icon">⇄</span>
        </button>
      </div>
      
      <div class="input-group">
        <input 
          type="number" 
          v-model="result" 
          readonly 
          placeholder="转换结果"
        />
        <select v-model="toCurrency" @change="convertCurrency">
          <option v-for="currency in currencies" :key="currency.code" :value="currency.code">
            {{ currency.code }} - {{ currency.name }}
          </option>
        </select>
      </div>
    </div>
    
    <div class="exchange-rate" v-if="exchangeRate">
      <p>1 {{ fromCurrency }} = {{ exchangeRate }} {{ toCurrency }}</p>
      <p class="update-time">更新时间: {{ lastUpdated }}</p>
    </div>
    
    <div class="error-message" v-if="errorMessage">
      {{ errorMessage }}
    </div>
  </div>
</template>

<script>
export default {
  name: 'CurrencyConverter',
  data() {
    return {
      amount: 1,
      fromCurrency: 'CNY',
      toCurrency: 'USD',
      exchangeRate: null,
      result: null,
      currencies: [
        { code: 'CNY', name: '人民币' },
        { code: 'USD', name: '美元' },
        { code: 'EUR', name: '欧元' },
        { code: 'JPY', name: '日元' },
        { code: 'GBP', name: '英镑' },
        { code: 'AUD', name: '澳元' },
        { code: 'CAD', name: '加元' },
        { code: 'HKD', name: '港币' },
        { code: 'SGD', name: '新加坡元' }
      ],
      errorMessage: '',
      lastUpdated: '',
      rateCache: {},
    };
  },
  mounted() {
    this.loadCachedRates();
    this.convertCurrency();
  },
  methods: {
    async fetchExchangeRate() {
      const cacheKey = `${this.fromCurrency}_${this.toCurrency}`;
      
      // Check if we have a recent cache (less than 6 hours old)
      const cachedData = this.rateCache[cacheKey];
      if (cachedData && (Date.now() - cachedData.timestamp) < 6 * 60 * 60 * 1000) {
        this.exchangeRate = cachedData.rate;
        this.lastUpdated = new Date(cachedData.timestamp).toLocaleString();
        return;
      }
      
      try {
        this.errorMessage = '';
        const response = await fetch(`https://open.er-api.com/v6/latest/${this.fromCurrency}`);
        
        if (!response.ok) {
          throw new Error('无法获取汇率数据');
        }
        
        const data = await response.json();
        
        if (data.result === 'success') {
          this.exchangeRate = data.rates[this.toCurrency];
          
          // Cache the result
          const timestamp = Date.now();
          this.rateCache[cacheKey] = {
            rate: this.exchangeRate,
            timestamp: timestamp
          };
          
          // Save to localStorage
          localStorage.setItem('currencyRateCache', JSON.stringify(this.rateCache));
          
          this.lastUpdated = new Date(timestamp).toLocaleString();
        } else {
          throw new Error('API返回错误');
        }
      } catch (error) {
        this.errorMessage = `获取汇率失败: ${error.message}`;
        console.error('Currency conversion error:', error);
        
        // Use fallback rates if available
        if (this.getFallbackRate()) {
          this.errorMessage += '，使用缓存汇率';
        }
      }
    },
    
    getFallbackRate() {
      // Fallback rates for when API is unavailable
      const fallbackRates = {
        'CNY_USD': 0.1382,
        'USD_CNY': 7.2362,
        'CNY_EUR': 0.1301,
        'EUR_CNY': 7.6864,
        'USD_EUR': 0.9415,
        'EUR_USD': 1.0621
      };
      
      const key = `${this.fromCurrency}_${this.toCurrency}`;
      
      if (fallbackRates[key]) {
        this.exchangeRate = fallbackRates[key];
        this.lastUpdated = '使用离线数据';
        return true;
      }
      
      // Try reverse conversion if direct conversion not available
      const reverseKey = `${this.toCurrency}_${this.fromCurrency}`;
      if (fallbackRates[reverseKey]) {
        this.exchangeRate = 1 / fallbackRates[reverseKey];
        this.lastUpdated = '使用离线数据(反向计算)';
        return true;
      }
      
      return false;
    },
    
    async convertCurrency() {
      if (!this.amount) {
        this.result = '';
        return;
      }
      
      if (this.fromCurrency === this.toCurrency) {
        this.result = this.amount;
        this.exchangeRate = 1;
        this.lastUpdated = new Date().toLocaleString();
        return;
      }
      
      await this.fetchExchangeRate();
      if (this.exchangeRate) {
        this.result = (this.amount * this.exchangeRate).toFixed(2);
      }
    },
    
    swapCurrencies() {
      const temp = this.fromCurrency;
      this.fromCurrency = this.toCurrency;
      this.toCurrency = temp;
      this.convertCurrency();
    },
    
    loadCachedRates() {
      const cached = localStorage.getItem('currencyRateCache');
      if (cached) {
        try {
          this.rateCache = JSON.parse(cached);
        } catch (e) {
          console.error('Failed to parse cached rates:', e);
          this.rateCache = {};
        }
      }
    }
  }
};
</script>

<style scoped>
.currency-converter {
  background-color: #ffffff;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  padding: 20px;
  max-width: 500px;
  margin: 0 auto;
}

h3 {
  color: #333;
  font-size: 1.5rem;
  margin-top: 0;
  margin-bottom: 20px;
  text-align: center;
}

.converter-form {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 20px;
}

.input-group {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 8px;
}

input, select {
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 16px;
}

input:focus, select:focus {
  outline: none;
  border-color: #4a90e2;
}

.exchange-icon {
  display: flex;
  align-items: center;
}

.exchange-icon button {
  background: none;
  border: none;
  cursor: pointer;
  font-size: 24px;
  color: #4a90e2;
  padding: 5px;
  transition: transform 0.3s ease;
}

.exchange-icon button:hover {
  transform: rotate(180deg);
}

.swap-icon {
  display: inline-block;
}

.exchange-rate {
  text-align: center;
  font-size: 14px;
  color: #666;
}

.update-time {
  font-size: 12px;
  color: #888;
}

.error-message {
  color: #e74c3c;
  text-align: center;
  margin-top: 10px;
  font-size: 14px;
}

@media (max-width: 600px) {
  .converter-form {
    flex-direction: column;
  }
  
  .exchange-icon {
    transform: rotate(90deg);
    margin: 10px 0;
  }
}

@media (max-width: 480px) {
  .currency-converter {
    padding: 15px;
  }
  
  .converter-form {
    flex-direction: column;
    gap: 15px;
  }
  
  .exchange-icon {
    transform: rotate(90deg);
    margin: 0;
  }
  
  h3 {
    font-size: 1.3rem;
    margin-bottom: 15px;
  }
  
  input, select {
    font-size: 14px;
    padding: 8px;
  }
}
</style> 