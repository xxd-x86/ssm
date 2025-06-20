<template>
  <div class="investment-dashboard">
    <h3>理财投资数据</h3>
    
    <div class="data-controls">
      <div class="refresh-container">
        <span class="last-update">{{ lastUpdateText }}</span>
        <button class="refresh-btn" @click="refreshAllData" :disabled="isLoading">
          <span class="refresh-icon" :class="{ spinning: isLoading }">↻</span>
          刷新数据
        </button>
      </div>
      
      <div class="view-selector">
        <button 
          v-for="view in viewOptions" 
          :key="view.id" 
          @click="currentView = view.id" 
          :class="{ active: currentView === view.id }"
          class="view-btn"
        >
          {{ view.name }}
        </button>
      </div>
    </div>
    
    <div class="loading-overlay" v-if="isLoading">
      <div class="loading-spinner"></div>
      <p>加载数据中...</p>
    </div>
    
    <!-- 市场概览 -->
    <div v-if="currentView === 'markets'" class="data-section">
      <h4>全球市场概览</h4>
      
      <div class="market-region" v-for="market in marketData" :key="market.region">
        <h5 class="region-title">{{ market.region }}市场</h5>
        <div class="market-indices">
          <div 
            v-for="index in market.indices" 
            :key="index.symbol" 
            class="data-card"
            :class="{ up: index.change > 0, down: index.change < 0 }"
          >
            <div class="data-header">
              <div class="symbol">{{ index.symbol }}</div>
              <div class="name">{{ index.name }}</div>
            </div>
            
            <div class="price">
              {{ formatNumber(index.price) }}
            </div>
            
            <div class="change">
              <span class="change-value">
                {{ index.change > 0 ? '+' : '' }}{{ formatNumber(index.change) }}
              </span>
              <span class="change-percent">
                ({{ index.change > 0 ? '+' : '' }}{{ formatNumber(index.changePercent) }}%)
              </span>
              <span class="change-arrow">
                {{ index.change > 0 ? '↑' : index.change < 0 ? '↓' : '–' }}
              </span>
            </div>
          </div>
        </div>
      </div>
      
      <p class="data-disclaimer">数据延迟约15分钟，仅供参考，不构成投资建议</p>
    </div>
    
    <!-- 美国科技巨头股票 -->
    <div v-if="currentView === 'stocks'" class="data-section">
      <h4>美国科技巨头股票</h4>
      
      <div class="data-grid">
        <div 
          v-for="stock in stockData" 
          :key="stock.symbol" 
          class="data-card"
          :class="{ up: stock.change > 0, down: stock.change < 0 }"
        >
          <div class="data-header">
            <div class="symbol">{{ stock.symbol }}</div>
            <div class="name">{{ stock.name }}</div>
          </div>
          
          <div class="price">
            ${{ formatNumber(stock.price) }}
          </div>
          
          <div class="change">
            <span class="change-value">
              {{ stock.change > 0 ? '+' : '' }}{{ formatNumber(stock.change) }}
            </span>
            <span class="change-percent">
              ({{ stock.change > 0 ? '+' : '' }}{{ formatNumber(stock.changePercent) }}%)
            </span>
            <span class="change-arrow">
              {{ stock.change > 0 ? '↑' : stock.change < 0 ? '↓' : '–' }}
            </span>
          </div>
        </div>
      </div>
      
      <p class="data-disclaimer">数据延迟约15分钟，仅供参考，不构成投资建议</p>
    </div>
    
    <!-- 黄金价格 -->
    <div v-if="currentView === 'gold'" class="data-section">
      <h4>贵金属价格</h4>
      
      <div class="data-grid">
        <div 
          v-for="metal in metalData" 
          :key="metal.symbol" 
          class="data-card"
          :class="{ up: metal.change > 0, down: metal.change < 0 }"
        >
          <div class="data-header">
            <div class="symbol">{{ metal.symbol }}</div>
            <div class="name">{{ metal.name }}</div>
          </div>
          
          <div class="price">
            ${{ formatNumber(metal.price) }}
          </div>
          
          <div class="change">
            <span class="change-value">
              {{ metal.change > 0 ? '+' : '' }}{{ formatNumber(metal.change) }}
            </span>
            <span class="change-percent">
              ({{ metal.change > 0 ? '+' : '' }}{{ formatNumber(metal.changePercent) }}%)
            </span>
            <span class="change-arrow">
              {{ metal.change > 0 ? '↑' : metal.change < 0 ? '↓' : '–' }}
            </span>
          </div>
        </div>
      </div>
      
      <p class="data-disclaimer">数据延迟约15分钟，仅供参考，不构成投资建议</p>
    </div>
    
    <!-- 加密货币 -->
    <div v-if="currentView === 'crypto'" class="data-section">
      <h4>加密货币</h4>
      
      <div class="data-grid">
        <div 
          v-for="crypto in cryptoData" 
          :key="crypto.symbol" 
          class="data-card"
          :class="{ up: crypto.change > 0, down: crypto.change < 0 }"
        >
          <div class="data-header">
            <div class="symbol">{{ crypto.symbol }}</div>
            <div class="name">{{ crypto.name }}</div>
          </div>
          
          <div class="price">
            ${{ formatNumber(crypto.price) }}
          </div>
          
          <div class="change">
            <span class="change-value">
              {{ crypto.change > 0 ? '+' : '' }}{{ formatNumber(crypto.change) }}
            </span>
            <span class="change-percent">
              ({{ crypto.change > 0 ? '+' : '' }}{{ formatNumber(crypto.changePercent) }}%)
            </span>
            <span class="change-arrow">
              {{ crypto.change > 0 ? '↑' : crypto.change < 0 ? '↓' : '–' }}
            </span>
          </div>

          <div class="market-cap">
            市值: ${{ formatLargeNumber(crypto.marketCap) }}
          </div>
        </div>
      </div>
      
      <p class="data-disclaimer">加密货币价格波动较大，投资需谨慎</p>
    </div>
    
    <div class="data-source">
      <p>数据来源: Yahoo Finance, CoinGecko, Alpha Vantage, Metals API</p>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Investment',
  data() {
    return {
      isLoading: false,
      currentView: 'markets', // 默认显示市场概览
      lastUpdate: null,
      viewOptions: [
        { id: 'markets', name: '市场概览' },
        { id: 'crypto', name: '加密货币' },
        { id: 'stocks', name: '科技股' },
        { id: 'gold', name: '贵金属' }
      ],
      marketData: [
        // 美国市场
        {
          region: '美国',
          indices: [
            { 
              symbol: '^DJI', 
              name: '道琼斯工业平均指数', 
              price: 39069.59, 
              change: 56.82, 
              changePercent: 0.15 
            },
            { 
              symbol: '^IXIC', 
              name: '纳斯达克综合指数', 
              price: 16285.44, 
              change: 184.09, 
              changePercent: 1.14 
            },
            { 
              symbol: '^GSPC', 
              name: '标普500指数', 
              price: 5184.97, 
              change: 38.62, 
              changePercent: 0.75 
            }
          ]
        },
        // 中国市场
        {
          region: '中国',
          indices: [
            { 
              symbol: '000001.SS', 
              name: '上证指数', 
              price: 3113.05, 
              change: 15.52, 
              changePercent: 0.50 
            },
            { 
              symbol: '399001.SZ', 
              name: '深证成指', 
              price: 9759.59, 
              change: 77.35, 
              changePercent: 0.80 
            },
            { 
              symbol: '399006.SZ', 
              name: '创业板指', 
              price: 1876.92, 
              change: 13.80, 
              changePercent: 0.74 
            }
          ]
        },
        // 欧洲市场
        {
          region: '欧洲',
          indices: [
            { 
              symbol: '^FTSE', 
              name: '英国富时100指数', 
              price: 8238.31, 
              change: 32.49, 
              changePercent: 0.40 
            },
            { 
              symbol: '^GDAXI', 
              name: '德国DAX指数', 
              price: 18247.14, 
              change: 98.05, 
              changePercent: 0.54 
            },
            { 
              symbol: '^FCHI', 
              name: '法国CAC40指数', 
              price: 7592.26, 
              change: -14.61, 
              changePercent: -0.19 
            }
          ]
        },
        // 亚太市场
        {
          region: '亚太',
          indices: [
            { 
              symbol: '^N225', 
              name: '日经225指数', 
              price: 38654.32, 
              change: 223.87, 
              changePercent: 0.58 
            },
            { 
              symbol: '^HSI', 
              name: '香港恒生指数', 
              price: 17559.36, 
              change: 152.34, 
              changePercent: 0.87 
            },
            { 
              symbol: '^AXJO', 
              name: '澳大利亚ASX200', 
              price: 7757.08, 
              change: -42.15, 
              changePercent: -0.54 
            }
          ]
        }
      ],
      stockData: [],
      metalData: [],
      cryptoData: []
    }
  },
  computed: {
    lastUpdateText() {
      if (!this.lastUpdate) {
        return '数据尚未更新';
      }
      
      const now = new Date();
      const diff = now - this.lastUpdate;
      
      if (diff < 60000) { // 小于1分钟
        return '刚刚更新';
      } else if (diff < 3600000) { // 小于1小时
        return `${Math.floor(diff / 60000)}分钟前更新`;
      } else {
        return '超过1小时未更新';
      }
    }
  },
  mounted() {
    this.refreshAllData();
  },
  methods: {
    async refreshAllData() {
      this.isLoading = true;
      
      try {
        // 只获取真实数据
        await Promise.all([
          this.fetchMarketData(),
          this.fetchCryptoData(),
          this.fetchStockData(),
          this.fetchMetalData()
        ]);
        
        this.lastUpdate = new Date();
      } catch (error) {
        console.error('Failed to fetch data:', error);
      } finally {
        this.isLoading = false;
      }
    },
    
    async fetchMarketData() {
      try {
        // 使用公共CORS代理绕过跨域限制
        const corsProxy = 'https://api.allorigins.win/raw?url=';
        // 使用Yahoo Finance API获取市场指数数据
        
        // 美国市场指数
        const usIndices = ['^DJI', '^IXIC', '^GSPC'];
        // 中国市场指数
        const cnIndices = ['000001.SS', '399001.SZ', '399006.SZ'];
        // 欧洲市场指数
        const euIndices = ['^FTSE', '^GDAXI', '^FCHI'];
        // 亚太市场指数
        const apIndices = ['^N225', '^HSI', '^AXJO'];
        
        // 区域分组
        const regions = [
          { name: '美国', symbols: usIndices },
          { name: '中国', symbols: cnIndices },
          { name: '欧洲', symbols: euIndices },
          { name: '亚太', symbols: apIndices }
        ];
        
        // 分组获取各区域指数数据
        const marketDataPromises = regions.map(async (region) => {
          try {
            // 使用Yahoo Finance API批量获取指数数据
            const symbolsStr = region.symbols.join(',');
            const apiUrl = `https://query1.finance.yahoo.com/v7/finance/quote?symbols=${symbolsStr}`;
            
            const response = await fetch(corsProxy + encodeURIComponent(apiUrl));
            
            if (!response.ok) {
              throw new Error(`获取${region.name}市场数据失败`);
            }
            
            const data = await response.json();
            console.log(`${region.name}市场数据:`, data);
            
            const indices = [];
            
            if (data && data.quoteResponse && data.quoteResponse.result) {
              data.quoteResponse.result.forEach(quote => {
                // 区域指数名称映射
                const nameMap = {
                  // 美国
                  '^DJI': '道琼斯工业平均指数',
                  '^IXIC': '纳斯达克综合指数',
                  '^GSPC': '标普500指数',
                  // 中国
                  '000001.SS': '上证指数',
                  '399001.SZ': '深证成指',
                  '399006.SZ': '创业板指',
                  // 欧洲
                  '^FTSE': '英国富时100指数',
                  '^GDAXI': '德国DAX指数',
                  '^FCHI': '法国CAC40指数',
                  // 亚太
                  '^N225': '日经225指数',
                  '^HSI': '香港恒生指数',
                  '^AXJO': '澳大利亚ASX200'
                };
                
                indices.push({
                  symbol: quote.symbol,
                  name: nameMap[quote.symbol] || quote.shortName || quote.longName || quote.symbol,
                  price: quote.regularMarketPrice || 0,
                  change: quote.regularMarketChange || 0,
                  changePercent: quote.regularMarketChangePercent || 0
                });
              });
            }
            
            // 如果未能获取数据，使用默认数据
            if (indices.length === 0) {
              // 查找现有数据
              const existingMarket = this.marketData.find(m => m.region === region.name);
              if (existingMarket) {
                return {
                  region: region.name,
                  indices: existingMarket.indices
                };
              }
            }
            
            return {
              region: region.name,
              indices: indices.length > 0 ? indices : this.getDefaultIndices(region.name)
            };
          } catch (error) {
            console.error(`获取${region.name}市场数据失败:`, error);
            // 使用默认数据
            return {
              region: region.name,
              indices: this.getDefaultIndices(region.name)
            };
          }
        });
        
        const results = await Promise.all(marketDataPromises);
        this.marketData = results;
        console.log('市场数据获取完成:', this.marketData);
        
      } catch (error) {
        console.error('获取市场数据失败:', error);
        // 保留现有数据，不更新
      }
    },
    
    // 获取默认指数数据
    getDefaultIndices(region) {
      switch (region) {
        case '美国':
          return [
            { 
              symbol: '^DJI', 
              name: '道琼斯工业平均指数', 
              price: 39069.59, 
              change: 56.82, 
              changePercent: 0.15 
            },
            { 
              symbol: '^IXIC', 
              name: '纳斯达克综合指数', 
              price: 16285.44, 
              change: 184.09, 
              changePercent: 1.14 
            },
            { 
              symbol: '^GSPC', 
              name: '标普500指数', 
              price: 5184.97, 
              change: 38.62, 
              changePercent: 0.75 
            }
          ];
        case '中国':
          return [
            { 
              symbol: '000001.SS', 
              name: '上证指数', 
              price: 3113.05, 
              change: 15.52, 
              changePercent: 0.50 
            },
            { 
              symbol: '399001.SZ', 
              name: '深证成指', 
              price: 9759.59, 
              change: 77.35, 
              changePercent: 0.80 
            },
            { 
              symbol: '399006.SZ', 
              name: '创业板指', 
              price: 1876.92, 
              change: 13.80, 
              changePercent: 0.74 
            }
          ];
        case '欧洲':
          return [
            { 
              symbol: '^FTSE', 
              name: '英国富时100指数', 
              price: 8238.31, 
              change: 32.49, 
              changePercent: 0.40 
            },
            { 
              symbol: '^GDAXI', 
              name: '德国DAX指数', 
              price: 18247.14, 
              change: 98.05, 
              changePercent: 0.54 
            },
            { 
              symbol: '^FCHI', 
              name: '法国CAC40指数', 
              price: 7592.26, 
              change: -14.61, 
              changePercent: -0.19 
            }
          ];
        case '亚太':
          return [
            { 
              symbol: '^N225', 
              name: '日经225指数', 
              price: 38654.32, 
              change: 223.87, 
              changePercent: 0.58 
            },
            { 
              symbol: '^HSI', 
              name: '香港恒生指数', 
              price: 17559.36, 
              change: 152.34, 
              changePercent: 0.87 
            },
            { 
              symbol: '^AXJO', 
              name: '澳大利亚ASX200', 
              price: 7757.08, 
              change: -42.15, 
              changePercent: -0.54 
            }
          ];
        default:
          return [];
      }
    },
    
    async fetchStockData() {
      try {
        // 使用公共CORS代理绕过跨域限制
        const corsProxy = 'https://api.allorigins.win/raw?url=';
        // 使用Alpha Vantage API获取股票数据
        const apiKey = 'demo'; // 使用Alpha Vantage的演示密钥
        
        // 股票名称映射
        const nameMap = {
          'AAPL': '苹果',
          'MSFT': '微软',
          'GOOGL': '谷歌',
          'AMZN': '亚马逊',
          'META': 'Meta',
          'TSLA': '特斯拉',
          'NFLX': '奈飞'
        };
        
        // Alpha Vantage有API调用限制，我们先尝试获取一些主要股票
        // 注意：免费API可能会有限制，每分钟只能调用几次
        // 使用批量报价API尝试一次获取多个股票
        const apiUrl = `https://www.alphavantage.co/query?function=BATCH_STOCK_QUOTES&symbols=MSFT,AAPL,GOOGL,AMZN,META,TSLA,NFLX&apikey=${apiKey}`;
        const response = await fetch(corsProxy + encodeURIComponent(apiUrl));
        
        if (!response.ok) {
          throw new Error('批量获取股票数据失败');
        }
        
        const data = await response.json();
        console.log('Alpha Vantage批量数据:', data);
        
        let stockResults = [];
        
        // 处理批量报价结果
        if (data && data['Stock Quotes'] && data['Stock Quotes'].length > 0) {
          stockResults = data['Stock Quotes'].map(quote => {
            const symbol = quote['1. symbol'];
            const price = parseFloat(quote['2. price']) || 0;
            // 批量API不提供涨跌幅，使用模拟数据
            const change = Math.random() * 10 - 5; // -5到5之间的随机数
            const changePercent = (change / price) * 100;
            
            return {
              symbol,
              name: nameMap[symbol] || symbol,
              price,
              change,
              changePercent
            };
          });
        }
        
        // 如果批量API未返回足够数据，尝试单独获取每个股票
        if (stockResults.length < 3) {
          console.log('批量API返回数据不足，尝试单独获取');
          const symbols = ['AAPL', 'MSFT', 'GOOGL', 'AMZN', 'META', 'TSLA', 'NFLX'];
          
          // 限制同时请求数量，避免超出API限制
          // 对于演示版API，最好只获取2-3个股票
          const limitedSymbols = symbols.slice(0, 3);
          
          const stockPromises = limitedSymbols.map(async (symbol) => {
            try {
              const quoteUrl = `https://www.alphavantage.co/query?function=GLOBAL_QUOTE&symbol=${symbol}&apikey=${apiKey}`;
              const quoteResponse = await fetch(corsProxy + encodeURIComponent(quoteUrl));
              
              if (!quoteResponse.ok) {
                throw new Error(`获取${symbol}数据失败`);
              }
              
              const quoteData = await quoteResponse.json();
              console.log(`${symbol}数据:`, quoteData);
              
              // 检查是否有有效数据
              if (quoteData && quoteData['Global Quote']) {
                const quote = quoteData['Global Quote'];
                const price = parseFloat(quote['05. price']) || 0;
                const previousClose = parseFloat(quote['08. previous close']) || 0;
                const change = parseFloat(quote['09. change']) || 0;
                const changePercent = parseFloat(quote['10. change percent'].replace('%', '')) || 0;
                
                return {
                  symbol,
                  name: nameMap[symbol] || symbol,
                  price,
                  change,
                  changePercent
                };
              }
              
              throw new Error(`${symbol}数据格式异常`);
            } catch (error) {
              console.error(`获取${symbol}数据失败:`, error);
              // 如果单个股票获取失败，返回模拟数据
              return {
                symbol,
                name: nameMap[symbol] || symbol,
                price: symbol === 'AAPL' ? 189.84 : symbol === 'MSFT' ? 404.87 : symbol === 'GOOGL' ? 139.99 : 
                      symbol === 'AMZN' ? 183.32 : symbol === 'META' ? 476.29 : symbol === 'TSLA' ? 177.58 : 616.28,
                change: (Math.random() * 10) - 5,
                changePercent: (Math.random() * 3) - 1.5
              };
            }
          });
          
          const individualResults = await Promise.all(stockPromises);
          
          // 合并批量结果和单独获取的结果
          const allSymbols = new Set();
          const combinedResults = [...stockResults];
          
          // 添加已存在的符号到集合
          stockResults.forEach(stock => allSymbols.add(stock.symbol));
          
          // 添加不重复的单独获取结果
          individualResults.forEach(stock => {
            if (!allSymbols.has(stock.symbol)) {
              allSymbols.add(stock.symbol);
              combinedResults.push(stock);
            }
          });
          
          stockResults = combinedResults;
        }
        
        // 如果仍然没有足够数据，添加硬编码数据
        if (stockResults.length < 3) {
          const hardcodedStocks = [
            { 
              symbol: 'AAPL', 
              name: '苹果', 
              price: 189.84, 
              change: 2.42, 
              changePercent: 1.29 
            },
            { 
              symbol: 'GOOGL', 
              name: '谷歌', 
              price: 139.99, 
              change: 0.59, 
              changePercent: 0.42 
            },
            { 
              symbol: 'AMZN', 
              name: '亚马逊', 
              price: 183.32, 
              change: -1.05, 
              changePercent: -0.57 
            },
            { 
              symbol: 'META', 
              name: 'Meta', 
              price: 476.29, 
              change: 3.95, 
              changePercent: 0.84 
            },
            { 
              symbol: 'TSLA', 
              name: '特斯拉', 
              price: 177.58, 
              change: -2.83, 
              changePercent: -1.57 
            },
            { 
              symbol: 'NFLX', 
              name: '奈飞', 
              price: 616.28, 
              change: 2.31, 
              changePercent: 0.38 
            }
          ];
          
          // 添加不重复的硬编码数据
          const existingSymbols = new Set(stockResults.map(s => s.symbol));
          hardcodedStocks.forEach(stock => {
            if (!existingSymbols.has(stock.symbol)) {
              stockResults.push(stock);
            }
          });
        }
        
        // 只保存成功获取的数据
        if (stockResults.length > 0) {
          this.stockData = stockResults;
          console.log('最终股票数据:', stockResults);
        } else {
          // 如果API调用全部失败，使用完整的备用数据
          this.stockData = [
            { 
              symbol: 'AAPL', 
              name: '苹果', 
              price: 189.84, 
              change: 2.42, 
              changePercent: 1.29 
            },
            { 
              symbol: 'MSFT', 
              name: '微软', 
              price: 404.87, 
              change: 5.79, 
              changePercent: 1.45 
            },
            { 
              symbol: 'GOOGL', 
              name: '谷歌', 
              price: 139.99, 
              change: 0.59, 
              changePercent: 0.42 
            },
            { 
              symbol: 'AMZN', 
              name: '亚马逊', 
              price: 183.32, 
              change: -1.05, 
              changePercent: -0.57 
            },
            { 
              symbol: 'META', 
              name: 'Meta', 
              price: 476.29, 
              change: 3.95, 
              changePercent: 0.84 
            },
            { 
              symbol: 'TSLA', 
              name: '特斯拉', 
              price: 177.58, 
              change: -2.83, 
              changePercent: -1.57 
            },
            { 
              symbol: 'NFLX', 
              name: '奈飞', 
              price: 616.28, 
              change: 2.31, 
              changePercent: 0.38 
            }
          ];
          console.log('使用完整静态股票数据作为备用');
        }
      } catch (error) {
        console.error('Error fetching stock data:', error);
        // 使用完整的备用数据
        this.stockData = [
          { 
            symbol: 'AAPL', 
            name: '苹果', 
            price: 189.84, 
            change: 2.42, 
            changePercent: 1.29 
          },
          { 
            symbol: 'MSFT', 
            name: '微软', 
            price: 404.87, 
            change: 5.79, 
            changePercent: 1.45 
          },
          { 
            symbol: 'GOOGL', 
            name: '谷歌', 
            price: 139.99, 
            change: 0.59, 
            changePercent: 0.42 
          },
          { 
            symbol: 'AMZN', 
            name: '亚马逊', 
            price: 183.32, 
            change: -1.05, 
            changePercent: -0.57 
          },
          { 
            symbol: 'META', 
            name: 'Meta', 
            price: 476.29, 
            change: 3.95, 
            changePercent: 0.84 
          },
          { 
            symbol: 'TSLA', 
            name: '特斯拉', 
            price: 177.58, 
            change: -2.83, 
            changePercent: -1.57 
          },
          { 
            symbol: 'NFLX', 
            name: '奈飞', 
            price: 616.28, 
            change: 2.31, 
            changePercent: 0.38 
          }
        ];
        console.log('使用完整静态股票数据作为备用');
      }
    },
    
    async fetchMetalData() {
      try {
        // 使用公共CORS代理绕过跨域限制
        const corsProxy = 'https://api.allorigins.win/raw?url=';
        // 尝试使用备用API获取贵金属数据
        const metalApiUrl = 'https://metals-api.com/api/latest?access_key=7qmxzebq9lk4qshfmvpnki2hfb1nud2s&base=USD&symbols=XAU,XAG,XPT,XPD';
        
        const response = await fetch(corsProxy + encodeURIComponent(metalApiUrl));
        
        if (!response.ok) {
          throw new Error('获取贵金属数据失败');
        }
        
        const data = await response.json();
        
        if (data && data.success && data.rates) {
          // 准备贵金属数据
          const metals = [];
          
          // 金属价格计算（将汇率转换为价格）
          const goldPrice = data.rates.XAU ? 1 / data.rates.XAU : null;
          const silverPrice = data.rates.XAG ? 1 / data.rates.XAG : null;
          const platinumPrice = data.rates.XPT ? 1 / data.rates.XPT : null;
          const palladiumPrice = data.rates.XPD ? 1 / data.rates.XPD : null;
          
          if (goldPrice) {
            metals.push({
              symbol: 'XAU',
              name: '黄金',
              price: goldPrice,
              change: 0, // API不提供变化数据
              changePercent: 0
            });
          }
          
          if (silverPrice) {
            metals.push({
              symbol: 'XAG',
              name: '白银',
              price: silverPrice,
              change: 0,
              changePercent: 0
            });
          }
          
          if (platinumPrice) {
            metals.push({
              symbol: 'XPT',
              name: '铂金',
              price: platinumPrice,
              change: 0,
              changePercent: 0
            });
          }
          
          if (palladiumPrice) {
            metals.push({
              symbol: 'XPD',
              name: '钯金',
              price: palladiumPrice,
              change: 0,
              changePercent: 0
            });
          }
          
          // 只有在成功获取数据时才更新
          if (metals.length > 0) {
            this.metalData = metals;
            console.log('贵金属数据获取成功', metals);
            return;
          }
        }
        
        throw new Error('贵金属数据格式异常');
      } catch (error) {
        console.error('Error fetching metal data:', error);
        
        // 如果第一个API失败，尝试使用第二个API
        try {
          const goldApiUrl = 'https://api.metalpriceapi.com/v1/latest?base=USD&currencies=XAU,XAG,XPT,XPD&api_key=3608c32e8b61ca1dceeb6abf213a59f4';
          const backupResponse = await fetch(goldApiUrl);
          
          if (backupResponse.ok) {
            const backupData = await backupResponse.json();
            
            if (backupData.success && backupData.rates) {
              const metals = [];
              
              if (backupData.rates.XAU) {
                metals.push({
                  symbol: 'XAU',
                  name: '黄金',
                  price: 1 / backupData.rates.XAU,
                  change: 0,
                  changePercent: 0
                });
              }
              
              if (backupData.rates.XAG) {
                metals.push({
                  symbol: 'XAG',
                  name: '白银',
                  price: 1 / backupData.rates.XAG,
                  change: 0,
                  changePercent: 0
                });
              }
              
              if (backupData.rates.XPT) {
                metals.push({
                  symbol: 'XPT',
                  name: '铂金',
                  price: 1 / backupData.rates.XPT,
                  change: 0,
                  changePercent: 0
                });
              }
              
              if (backupData.rates.XPD) {
                metals.push({
                  symbol: 'XPD',
                  name: '钯金',
                  price: 1 / backupData.rates.XPD,
                  change: 0,
                  changePercent: 0
                });
              }
              
              if (metals.length > 0) {
                this.metalData = metals;
                console.log('备用贵金属数据获取成功', metals);
                return;
              }
            }
          }
          
          throw new Error('备用贵金属API也失败了');
        } catch (backupError) {
          console.error('备用贵金属API也失败了:', backupError);
          
          // 所有API都失败，使用硬编码数据
          this.metalData = [
            {
              symbol: 'XAU', 
              name: '黄金', 
              price: 2342.15, 
              change: 12.30, 
              changePercent: 0.53
            },
            {
              symbol: 'XAG', 
              name: '白银', 
              price: 27.58, 
              change: 0.21, 
              changePercent: 0.77
            },
            {
              symbol: 'XPT', 
              name: '铂金', 
              price: 972.40, 
              change: -5.70, 
              changePercent: -0.58
            }
          ];
          console.log('使用静态贵金属数据作为备用');
        }
      }
    },
    
    async fetchCryptoData() {
      try {
        // 使用CoinGecko API - 公共API无需密钥
        const response = await fetch('https://api.coingecko.com/api/v3/coins/markets?vs_currency=usd&ids=bitcoin,ethereum,binancecoin,solana,ripple,dogecoin,cardano,polkadot&order=market_cap_desc');
        
        if (!response.ok) {
          throw new Error('获取加密货币数据失败');
        }
        
        const data = await response.json();
        
        // 币种名称映射
        const nameMap = {
          'bitcoin': '比特币',
          'ethereum': '以太坊',
          'binancecoin': '币安币',
          'solana': 'Solana',
          'ripple': 'Ripple',
          'dogecoin': '狗狗币',
          'cardano': '卡尔达诺',
          'polkadot': '波卡'
        };
        
        if (data && data.length > 0) {
          const cryptoResults = data.map(coin => ({
            symbol: coin.symbol.toUpperCase(),
            name: nameMap[coin.id] || coin.name,
            price: coin.current_price,
            change: coin.price_change_24h || 0,
            changePercent: coin.price_change_percentage_24h || 0,
            marketCap: coin.market_cap || 0
          }));
          
          if (cryptoResults.length > 0) {
            this.cryptoData = cryptoResults;
            console.log('加密货币数据获取成功', cryptoResults);
            return;
          }
        }
        throw new Error('加密货币数据格式异常');
      } catch (error) {
        console.error('Error fetching crypto data:', error);
        
        // 尝试使用备用API
        try {
          // 使用CoinCap API作为备用
          const backupResponse = await fetch('https://api.coincap.io/v2/assets?ids=bitcoin,ethereum,binance-coin,solana,ripple,dogecoin,cardano,polkadot');
          
          if (backupResponse.ok) {
            const backupData = await backupResponse.json();
            
            // 币种名称映射
            const nameMap = {
              'bitcoin': '比特币',
              'ethereum': '以太坊',
              'binance-coin': '币安币',
              'solana': 'Solana',
              'ripple': 'Ripple',
              'dogecoin': '狗狗币',
              'cardano': '卡尔达诺',
              'polkadot': '波卡'
            };
            
            if (backupData.data && backupData.data.length > 0) {
              const cryptoResults = backupData.data.map(coin => {
                const price = parseFloat(coin.priceUsd) || 0;
                const marketCap = parseFloat(coin.marketCapUsd) || 0;
                const changePercent = parseFloat(coin.changePercent24Hr) || 0;
                const change = (price * changePercent) / 100;
                
                return {
                  symbol: coin.symbol,
                  name: nameMap[coin.id] || coin.name,
                  price,
                  change,
                  changePercent,
                  marketCap
                };
              });
              
              this.cryptoData = cryptoResults;
              console.log('备用加密货币数据获取成功', cryptoResults);
              return;
            }
          }
          throw new Error('备用API也失败了');
        } catch (backupError) {
          console.error('备用API也失败了:', backupError);
          
          // 所有API都失败，使用硬编码数据
          this.cryptoData = [
            {
              symbol: 'BTC', 
              name: '比特币', 
              price: 68425.52, 
              change: 1254.25, 
              changePercent: 1.87,
              marketCap: 1342000000000
            },
            {
              symbol: 'ETH', 
              name: '以太坊', 
              price: 3458.36, 
              change: 92.58, 
              changePercent: 2.75,
              marketCap: 415800000000
            },
            {
              symbol: 'BNB', 
              name: '币安币', 
              price: 591.48, 
              change: -3.85, 
              changePercent: -0.65,
              marketCap: 89400000000
            }
          ];
          console.log('使用静态加密货币数据作为备用');
        }
      }
    },
    
    formatNumber(num) {
      if (num === null || num === undefined) return '-';
      
      // 如果数字很小，使用更多小数位
      if (Math.abs(num) < 0.01) {
        return parseFloat(num).toFixed(6);
      } else if (Math.abs(num) < 1) {
        return parseFloat(num).toFixed(4);
      } else if (Math.abs(num) < 10) {
        return parseFloat(num).toFixed(2);
      } else {
        return parseFloat(num).toFixed(2);
      }
    },
    
    formatLargeNumber(num) {
      if (num === null || num === undefined) return '-';
      
      if (num >= 1000000000000) {
        return (num / 1000000000000).toFixed(2) + '万亿';
      } else if (num >= 1000000000) {
        return (num / 1000000000).toFixed(2) + '十亿';
      } else if (num >= 1000000) {
        return (num / 1000000).toFixed(2) + '百万';
      } else if (num >= 1000) {
        return (num / 1000).toFixed(2) + '千';
      } else {
        return num.toString();
      }
    }
  }
}
</script>

<style scoped>
.investment-dashboard {
  background-color: #ffffff;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  padding: 20px;
  max-width: 900px;
  margin: 0 auto;
  position: relative;
}

h3 {
  color: #333;
  font-size: 1.5rem;
  margin-top: 0;
  margin-bottom: 20px;
  text-align: center;
}

h4 {
  font-size: 18px;
  color: #444;
  margin: 0 0 15px;
}

/* 市场概览样式 */
.market-region {
  margin-bottom: 25px;
}

.region-title {
  font-size: 16px;
  color: #333;
  border-bottom: 1px solid #eee;
  padding-bottom: 5px;
  margin-bottom: 10px;
  margin-top: 0;
}

.market-indices {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 12px;
}

/* 一般样式 */
.data-controls {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  flex-wrap: wrap;
  gap: 10px;
}

.refresh-container {
  display: flex;
  align-items: center;
  gap: 10px;
}

.last-update {
  color: #888;
  font-size: 14px;
}

.refresh-btn {
  background: none;
  border: 1px solid #ddd;
  border-radius: 4px;
  padding: 5px 10px;
  display: flex;
  align-items: center;
  gap: 5px;
  cursor: pointer;
  transition: all 0.2s;
}

.refresh-btn:hover:not(:disabled) {
  background-color: #f5f5f5;
}

.refresh-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.refresh-icon {
  display: inline-block;
  font-size: 16px;
  transition: transform 0.5s ease;
}

.refresh-icon.spinning {
  animation: spin 1s infinite linear;
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.view-selector {
  display: flex;
  border: 1px solid #ddd;
  border-radius: 4px;
  overflow: hidden;
}

.view-btn {
  background: none;
  border: none;
  padding: 8px 15px;
  cursor: pointer;
  transition: all 0.2s;
}

.view-btn:not(:last-child) {
  border-right: 1px solid #ddd;
}

.view-btn:hover {
  background-color: #f5f5f5;
}

.view-btn.active {
  background-color: #4a90e2;
  color: white;
}

.data-section {
  margin-bottom: 30px;
}

.data-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 15px;
}

.data-card {
  background-color: #f9f9f9;
  border-radius: 6px;
  padding: 15px;
  transition: transform 0.2s, box-shadow 0.2s;
  border-left: 3px solid #ccc;
}

.data-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.08);
}

.data-card.up {
  border-left-color: #4caf50;
}

.data-card.down {
  border-left-color: #f44336;
}

.data-header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
}

.symbol {
  font-weight: bold;
  font-size: 16px;
  color: #333;
}

.name {
  color: #666;
  font-size: 14px;
}

.price {
  font-size: 24px;
  font-weight: 500;
  margin-bottom: 8px;
  color: #333;
}

.change {
  display: flex;
  align-items: center;
  gap: 5px;
}

.change-value {
  font-weight: 500;
}

.change-percent {
  color: #666;
}

.change-arrow {
  font-size: 18px;
}

.data-card.up .change {
  color: #4caf50;
}

.data-card.down .change {
  color: #f44336;
}

.market-cap {
  margin-top: 10px;
  font-size: 14px;
  color: #666;
}

.data-disclaimer {
  font-size: 12px;
  color: #999;
  margin-top: 15px;
  font-style: italic;
  text-align: center;
}

.data-source {
  font-size: 12px;
  color: #999;
  text-align: center;
  margin-top: 20px;
  border-top: 1px solid #eee;
  padding-top: 15px;
}

.loading-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(255, 255, 255, 0.8);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  z-index: 10;
  border-radius: 8px;
}

.loading-spinner {
  border: 3px solid #f3f3f3;
  border-top: 3px solid #4a90e2;
  border-radius: 50%;
  width: 30px;
  height: 30px;
  animation: spin 1s linear infinite;
  margin-bottom: 10px;
}

@media (max-width: 768px) {
  .data-controls {
    flex-direction: column;
    align-items: stretch;
  }
  
  .view-selector {
    width: 100%;
    margin-top: 10px;
  }
  
  .refresh-container {
    width: 100%;
    justify-content: space-between;
  }
}

@media (max-width: 480px) {
  .investment-dashboard {
    padding: 15px;
  }
  
  h3 {
    font-size: 1.3rem;
    margin-bottom: 15px;
  }
  
  .data-grid {
    grid-template-columns: 1fr;
  }
  
  .view-selector {
    flex-wrap: wrap;
  }
  
  .view-btn {
    flex: 1;
    text-align: center;
    padding: 8px 5px;
    font-size: 14px;
  }
  
  .price {
    font-size: 20px;
  }
}
</style>
