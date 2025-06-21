<template>
  <div class="finance-news">
    <h2>财经新闻</h2>
    <div v-if="loading" class="loading">加载中...</div>
    <div v-else-if="error" class="error">{{ error }}</div>
    <div v-else class="news-container">
      <div v-for="(news, index) in newsList" :key="index" class="news-item">
        <h3 class="news-title">
          <a :href="news.url" target="_blank" rel="noopener noreferrer">{{ news.title }}</a>
        </h3>
        <p class="news-source">来源: {{ news.source }} | {{ formatDate(news.publishedAt) }}</p>
        <p class="news-description">{{ news.description }}</p>
      </div>
      <div class="attribution">
        <p>新闻数据由 <a href="https://newsapi.org" target="_blank" rel="noopener noreferrer">NewsAPI.org</a> 提供</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'FinanceNews',
  data() {
    return {
      newsList: [],
      loading: true,
      error: null
    }
  },
  mounted() {
    this.fetchNews()
  },
  methods: {
    async fetchNews() {
      this.loading = true
      this.error = null
      try {
        // 注意：在实际环境中，你需要:
        // 1. 注册 NewsAPI.org 账号并获取 API 密钥
        // 2. 创建后端代理来隐藏 API 密钥（生产环境中不要直接在前端使用 API 密钥）
        
        // 这里使用模拟数据，实际使用时替换为真实 API 调用
        // const response = await fetch(`https://newsapi.org/v2/top-headlines?country=cn&category=business&apiKey=YOUR_API_KEY`)
        // const data = await response.json()
        // this.newsList = data.articles
        
        // 模拟数据
        setTimeout(() => {
          this.newsList = [
            {
              title: '中国央行出台新政策促进经济发展',
              description: '央行宣布一系列新措施，旨在提振经济并支持小微企业发展。',
              url: 'https://example.com/news/1',
              source: '财经时报',
              publishedAt: '2023-05-10T08:30:00Z'
            },
            {
              title: '大型科技公司财报超出市场预期',
              description: '多家科技巨头公布第一季度财报，业绩普遍超过分析师预期，带动股市上涨。',
              url: 'https://example.com/news/2',
              source: '证券日报',
              publishedAt: '2023-05-09T14:15:00Z'
            },
            {
              title: '新能源产业投资持续增长',
              description: '报告显示，清洁能源领域投资在过去一年增长了30%，创历史新高。',
              url: 'https://example.com/news/3',
              source: '经济观察',
              publishedAt: '2023-05-08T09:45:00Z'
            },
            {
              title: '全球供应链逐步恢复正常',
              description: '经过多年调整，全球供应链问题得到缓解，物流成本显著下降。',
              url: 'https://example.com/news/4',
              source: '国际商报',
              publishedAt: '2023-05-07T11:20:00Z'
            },
            {
              title: '数字人民币试点范围进一步扩大',
              description: '央行宣布数字人民币试点将扩展到更多城市，应用场景持续丰富。',
              url: 'https://example.com/news/5',
              source: '金融时报',
              publishedAt: '2023-05-06T16:40:00Z'
            }
          ]
          this.loading = false
        }, 800)
      } catch (err) {
        this.error = '获取新闻失败，请稍后再试'
        console.error('Failed to fetch news:', err)
      } finally {
        this.loading = false
      }
    },
    formatDate(dateString) {
      const options = { year: 'numeric', month: 'long', day: 'numeric' }
      return new Date(dateString).toLocaleDateString('zh-CN', options)
    }
  }
}
</script>

<style scoped>
.finance-news {
  max-width: 900px;
  margin: 0 auto;
}

h2 {
  text-align: center;
  margin-bottom: 24px;
  color: #333;
}

.loading, .error {
  text-align: center;
  margin: 40px 0;
  font-size: 16px;
  color: #909399;
}

.error {
  color: #f56c6c;
}

.news-container {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.news-item {
  padding: 20px;
  border-radius: 8px;
  background-color: #fff;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.08);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.news-item:hover {
  transform: translateY(-3px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.12);
}

.news-title {
  margin-bottom: 10px;
  font-size: 18px;
  line-height: 1.5;
}

.news-title a {
  color: #303133;
  text-decoration: none;
  transition: color 0.3s;
}

.news-title a:hover {
  color: #4a90e2;
}

.news-source {
  font-size: 14px;
  color: #909399;
  margin-bottom: 10px;
}

.news-description {
  font-size: 15px;
  line-height: 1.6;
  color: #606266;
}

.attribution {
  font-size: 13px;
  color: #909399;
  text-align: center;
  margin-top: 20px;
}

.attribution a {
  color: #606266;
  text-decoration: none;
}

.attribution a:hover {
  text-decoration: underline;
}

@media (max-width: 768px) {
  .news-item {
    padding: 15px;
  }
  
  .news-title {
    font-size: 16px;
  }
  
  .news-description {
    font-size: 14px;
  }
}
</style> 