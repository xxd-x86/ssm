<template>
  <div class="financial-tips">
    <h2>财务知识库</h2>
    
    <div class="search-container">
      <input 
        type="text" 
        v-model="searchQuery" 
        placeholder="搜索理财知识..." 
        class="search-input"
        @input="filterTips"
      />
    </div>
    
    <div class="tips-categories">
      <button 
        v-for="(category, index) in categories" 
        :key="index"
        @click="filterByCategory(category)"
        :class="{ active: selectedCategory === category }"
        class="category-btn"
      >
        {{ category }}
      </button>
    </div>
    
    <div class="tips-container">
      <div v-if="filteredTips.length === 0" class="no-tips">
        没有找到相关知识，请尝试其他搜索词或分类
      </div>
      
      <div v-else class="tip-list">
        <div 
          v-for="(tip, index) in filteredTips" 
          :key="index" 
          class="tip-card"
          @click="openTipDetails(tip)"
        >
          <div class="tip-category">{{ tip.category }}</div>
          <h3 class="tip-title">{{ tip.title }}</h3>
          <p class="tip-summary">{{ tip.summary }}</p>
          <div class="tip-footer">
            <span class="read-more">查看详情</span>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 提示详情弹窗 -->
    <div v-if="selectedTip" class="tip-modal">
      <div class="tip-modal-content">
        <div class="tip-modal-header">
          <div class="tip-modal-tag">{{ selectedTip.category }}</div>
          <button @click="closeTipDetails" class="close-btn">&times;</button>
        </div>
        
        <h3 class="tip-modal-title">{{ selectedTip.title }}</h3>
        
        <div class="tip-modal-body" v-html="selectedTip.content"></div>
        
        <div class="tip-modal-footer">
          <button @click="closeTipDetails" class="close-tip-btn">关闭</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'FinancialTips',
  data() {
    return {
      searchQuery: '',
      selectedCategory: '全部',
      selectedTip: null,
      categories: ['全部', '理财基础', '投资知识', '消费观念', '税务规划', '养老准备'],
      allTips: [
        {
          id: 1,
          title: '个人理财的六大原则',
          summary: '建立紧急储备金、控制债务、定期存钱等理财基本原则的详细介绍',
          category: '理财基础',
          content: `
            <p>理财的六大原则是每个人财务健康的基础：</p>
            <ol>
              <li><strong>建立紧急储备金</strong> - 至少准备3-6个月的生活开支，以应对突发情况</li>
              <li><strong>控制债务</strong> - 尽可能避免高息消费贷款，优先偿还高息债务</li>
              <li><strong>定期储蓄</strong> - 坚持"先储蓄后消费"的习惯，至少存储收入的20%</li>
              <li><strong>多元化投资</strong> - 不要把鸡蛋放在同一个篮子里，分散投资风险</li>
              <li><strong>保障先行</strong> - 购买必要的保险，确保意外风险有基本保障</li>
              <li><strong>长期规划</strong> - 为未来做计划，包括退休储蓄和长期财务目标</li>
            </ol>
            <p>这些原则相辅相成，共同构成了个人财务健康的框架。财务自由不是一蹴而就的，而是通过日积月累的良好习惯实现的。</p>
            <p>建议按照这个顺序逐步实施：首先确保有足够的应急资金，然后清理高息债务，接着开始定期储蓄并购买必要的保险，最后再考虑投资增值。</p>
          `
        },
        {
          id: 2,
          title: '如何制定合理的预算计划',
          summary: '50/30/20预算法则解析及个性化预算制定方法',
          category: '理财基础',
          content: `
            <p>预算是理财的第一步，合理的预算计划可以帮助你掌控财务状况。</p>
            <h4>50/30/20预算法则</h4>
            <p>这是一个简单实用的预算分配方法：</p>
            <ul>
              <li><strong>50%</strong> - 用于必要支出（房租/房贷、水电费、食品等基本生活需求）</li>
              <li><strong>30%</strong> - 用于可自由支配的消费（娱乐、购物、旅游等）</li>
              <li><strong>20%</strong> - 用于储蓄和投资（紧急基金、退休金、投资等）</li>
            </ul>
            <h4>制定个性化预算的步骤</h4>
            <ol>
              <li>记录并分析过去3个月的所有支出</li>
              <li>根据实际情况调整各类支出的比例</li>
              <li>设定具体的月度预算目标</li>
              <li>选择合适的工具追踪日常支出</li>
              <li>每月审视预算执行情况并及时调整</li>
            </ol>
            <p>记住，预算不是限制，而是让你的钱花得更有价值的工具。好的预算计划应该既实际又有弹性，能够适应你生活的变化。</p>
          `
        },
        {
          id: 3,
          title: '投资新手入门指南',
          summary: '了解基础投资工具、风险等级及如何开始第一笔投资',
          category: '投资知识',
          content: `
            <p>投资是让财富增长的重要途径，但对新手来说可能有些复杂。以下是开始投资前需要了解的基础知识：</p>
            <h4>投资工具基础</h4>
            <ul>
              <li><strong>储蓄存款</strong> - 风险最低，收益也最低，但灵活性高</li>
              <li><strong>货币基金</strong> - 风险低，流动性好，收益略高于银行存款</li>
              <li><strong>债券/债券基金</strong> - 中低风险，有固定收益，期限较长</li>
              <li><strong>股票/股票基金</strong> - 风险较高，长期收益潜力大，波动性强</li>
              <li><strong>指数基金/ETF</strong> - 追踪市场指数，费用低，适合长期投资</li>
            </ul>
            <h4>新手投资步骤</h4>
            <ol>
              <li><strong>明确投资目标</strong> - 确定投资时间和风险承受能力</li>
              <li><strong>学习基本知识</strong> - 了解不同投资工具的特点</li>
              <li><strong>选择合适平台</strong> - 开立证券账户或基金账户</li>
              <li><strong>从小额开始</strong> - 先投入少量资金进行实践学习</li>
              <li><strong>坚持长期投资</strong> - 避免频繁交易，保持耐心</li>
            </ol>
            <p>记住，投资有风险，入市需谨慎。不要投资你不理解的产品，也不要借钱投资。投资是一个长期学习的过程，重要的是坚持和持续学习。</p>
          `
        },
        {
          id: 4,
          title: '理性消费的十个习惯',
          summary: '避免冲动消费、学会比价和合理利用折扣的实用技巧',
          category: '消费观念',
          content: `
            <p>理性消费是良好财务状况的基础。以下十个习惯可以帮助你优化消费决策：</p>
            <ol>
              <li><strong>24小时冷静期</strong> - 大额购物前等待24小时再决定</li>
              <li><strong>区分需求与欲望</strong> - 问自己"我真的需要它吗？"</li>
              <li><strong>制定购物清单</strong> - 购物前列清单并坚持按单采购</li>
              <li><strong>比较价格</strong> - 利用比价网站/应用找到最优惠价格</li>
              <li><strong>现金支付法则</strong> - 用现金支付更能感受到金钱流出</li>
              <li><strong>设置消费警戒线</strong> - 超过某金额需要特别考虑</li>
              <li><strong>关注性价比</strong> - 不只看价格，更要看质量和使用寿命</li>
              <li><strong>聪明利用折扣</strong> - 只买原本就需要的打折商品</li>
              <li><strong>避免冲动消费场所</strong> - 减少在购物中心闲逛的时间</li>
              <li><strong>定期回顾消费记录</strong> - 找出不必要的支出并调整</li>
            </ol>
            <p>记住，理性消费不是不消费，而是让每一分钱都花得更有价值。最终目标是让消费带来真正的满足感，而不是短暂的愉悦后的后悔。</p>
          `
        },
        {
          id: 5,
          title: '个人所得税优化策略',
          summary: '了解个税专项扣除政策及合法节税方式',
          category: '税务规划',
          content: `
            <p>合理利用税收政策可以有效降低个人税务负担。以下是个人所得税优化的主要方法：</p>
            <h4>专项附加扣除</h4>
            <ul>
              <li><strong>子女教育</strong> - 每个子女可扣除1000元/月</li>
              <li><strong>继续教育</strong> - 学历教育400元/月，职业资格教育3600元/年</li>
              <li><strong>大病医疗</strong> - 年度自付超过15000元的部分可扣除，上限80000元</li>
              <li><strong>住房贷款利息</strong> - 首套房贷可扣除1000元/月，最长20年</li>
              <li><strong>住房租金</strong> - 根据城市等级不同，扣除标准为800-1200元/月</li>
              <li><strong>赡养老人</strong> - 独生子女可扣除2000元/月，非独生子女按比例分配</li>
            </ul>
            <h4>合法税务筹划方法</h4>
            <ol>
              <li><strong>合理安排收入确认时间</strong> - 平衡各年度收入</li>
              <li><strong>充分利用税收优惠政策</strong> - 如个人养老金税收递延</li>
              <li><strong>选择合适的薪酬结构</strong> - 如年终奖单独计税</li>
              <li><strong>及时申报各项专项附加扣除</strong></li>
              <li><strong>合理规划经营所得</strong> - 如合理合法的成本费用列支</li>
            </ol>
            <p>注意：税务筹划必须在法律框架内进行，避免偷税漏税行为。税法政策经常变动，建议定期关注最新税收政策更新。</p>
          `
        },
        {
          id: 6,
          title: '退休规划——提前做好养老准备',
          summary: '如何计算退休需求及多层次养老金体系构建指南',
          category: '养老准备',
          content: `
            <p>退休规划是长期财务规划中最重要的部分之一。提前准备可以让你的晚年生活更有保障。</p>
            <h4>退休资金需求计算</h4>
            <p>一个简单的估算方法：</p>
            <ol>
              <li>估算退休后的月生活费（通常为工作期间的70%-80%）</li>
              <li>乘以12得到年度开支</li>
              <li>乘以预期退休后寿命（一般为20-30年）</li>
              <li>考虑通胀因素（年通胀率约为3%）</li>
            </ol>
            <h4>多层次养老体系构建</h4>
            <ul>
              <li><strong>第一层：社会基础养老金</strong> - 确保社保缴费连续性和完整性</li>
              <li><strong>第二层：职业养老金</strong> - 企业年金、职业年金等</li>
              <li><strong>第三层：个人养老金</strong> - 个人养老金账户、商业养老保险</li>
              <li><strong>第四层：个人投资积累</strong> - 长期投资（指数基金、养老目标基金等）</li>
              <li><strong>第五层：其他资产</strong> - 房产、实物资产等</li>
            </ul>
            <h4>退休规划的关键时间点</h4>
            <ul>
              <li><strong>30岁前</strong> - 开始养老意识培养，确保社保连续缴纳</li>
              <li><strong>30-40岁</strong> - 积极增加个人养老金投入，构建多元投资组合</li>
              <li><strong>40-50岁</strong> - 加大养老准备力度，调整投资组合降低风险</li>
              <li><strong>50岁后</strong> - 细化退休计划，准备退休生活转型</li>
            </ul>
            <p>记住，退休规划不仅是财务问题，也涉及生活方式、健康和心理准备。越早开始规划，所需的月度储蓄额就越小，复利的威力也越大。</p>
          `
        },
        {
          id: 7,
          title: '家庭财务管理的要点',
          summary: '家庭资产配置、风险管理及财务决策的沟通技巧',
          category: '理财基础',
          content: `
            <p>良好的家庭财务管理是家庭幸福的重要基础。以下是几个关键要点：</p>
            <h4>家庭财务目标设定</h4>
            <ul>
              <li><strong>短期目标</strong>（1年内）- 如建立应急基金、清理高息债务</li>
              <li><strong>中期目标</strong>（1-5年）- 如购房首付、教育金规划</li>
              <li><strong>长期目标</strong>（5年以上）- 如子女教育基金、退休规划</li>
            </ul>
            <h4>家庭资产合理配置</h4>
            <p>根据家庭生命周期阶段调整配置比例：</p>
            <ul>
              <li><strong>新婚期</strong> - 高流动性资产60%，增长型资产40%</li>
              <li><strong>育儿期</strong> - 高流动性资产40%，增长型资产40%，保障型资产20%</li>
              <li><strong>成熟期</strong> - 高流动性资产30%，增长型资产50%，保障型资产20%</li>
              <li><strong>退休前</strong> - 高流动性资产40%，增长型资产30%，保障型资产30%</li>
            </ul>
            <h4>家庭财务沟通技巧</h4>
            <ol>
              <li>定期举行家庭财务会议（如每月一次）</li>
              <li>公开透明地分享家庭收支情况</li>
              <li>共同参与重大财务决策</li>
              <li>尊重彼此的消费习惯，设定合理自由支配额</li>
              <li>一起制定并庆祝财务目标的达成</li>
            </ol>
            <h4>家庭风险管理</h4>
            <ul>
              <li><strong>应急基金</strong> - 3-6个月家庭开支</li>
              <li><strong>保险规划</strong> - 医疗、重疾、意外、寿险等</li>
              <li><strong>遗产规划</strong> - 包括遗嘱、财产继承安排等</li>
            </ul>
            <p>良好的家庭财务管理需要所有家庭成员的参与和配合。通过共同规划、沟通和执行，可以实现家庭的财务安全与和谐。</p>
          `
        },
        {
          id: 8,
          title: '指数基金投资策略',
          summary: '长期投资、定投与资产配置的指数基金实操指南',
          category: '投资知识',
          content: `
            <p>指数基金是普通投资者进行长期投资的优秀工具。以下是关于指数基金投资的核心策略：</p>
            <h4>为什么选择指数基金</h4>
            <ul>
              <li><strong>低成本</strong> - 管理费和交易成本远低于主动型基金</li>
              <li><strong>分散风险</strong> - 自动持有众多股票，避免个股风险</li>
              <li><strong>透明度高</strong> - 持仓完全跟踪特定指数，投资者清楚自己买的是什么</li>
              <li><strong>长期业绩佳</strong> - 研究表明大多数主动基金无法长期战胜指数</li>
            </ul>
            <h4>常见指数基金类型</h4>
            <ul>
              <li><strong>宽基指数</strong> - 如沪深300、中证500、标普500等</li>
              <li><strong>行业指数</strong> - 如医药、消费、金融、科技等行业指数</li>
              <li><strong>策略指数</strong> - 如价值、红利、低波动等策略指数</li>
              <li><strong>国际指数</strong> - 如纳斯达克、恒生指数等</li>
            </ul>
            <h4>指数基金投资策略</h4>
            <ol>
              <li><strong>定期定额投资</strong> - 每月固定日期投入固定金额，平滑成本</li>
              <li><strong>价值平均策略</strong> - 根据基金净值高低调整投入金额</li>
              <li><strong>核心-卫星策略</strong> - 以宽基指数为核心，辅以行业/策略指数</li>
              <li><strong>再平衡策略</strong> - 定期调整各类指数基金比例回到目标配置</li>
            </ol>
            <h4>指数估值与择时</h4>
            <p>可参考的估值指标：</p>
            <ul>
              <li><strong>PE</strong> - 市盈率，当前低于历史值为相对低估</li>
              <li><strong>PB</strong> - 市净率，同样关注历史分位</li>
              <li><strong>股息率</strong> - 高于历史平均值时通常代表相对低估</li>
            </ul>
            <p>记住，指数基金投资是一项需要耐心和长期坚持的投资方式。短期波动无法避免，但长期来看，坚持规律投资往往能获得不错的回报。投资成功更多靠的是良好的心态和纪律，而非市场择时。</p>
          `
        }
      ]
    }
  },
  computed: {
    filteredTips() {
      let tips = this.allTips
      
      // 按分类筛选
      if (this.selectedCategory !== '全部') {
        tips = tips.filter(tip => tip.category === this.selectedCategory)
      }
      
      // 按搜索词筛选
      if (this.searchQuery.trim()) {
        const query = this.searchQuery.toLowerCase()
        tips = tips.filter(tip => 
          tip.title.toLowerCase().includes(query) || 
          tip.summary.toLowerCase().includes(query) ||
          tip.content.toLowerCase().includes(query)
        )
      }
      
      return tips
    }
  },
  methods: {
    filterTips() {
      // 搜索触发的筛选在计算属性中实现
    },
    filterByCategory(category) {
      this.selectedCategory = category
    },
    openTipDetails(tip) {
      this.selectedTip = tip
    },
    closeTipDetails() {
      this.selectedTip = null
    }
  }
}
</script>

<style scoped>
.financial-tips {
  max-width: 900px;
  margin: 0 auto;
}

h2 {
  text-align: center;
  margin-bottom: 24px;
  color: #333;
}

.search-container {
  margin-bottom: 20px;
}

.search-input {
  width: 100%;
  padding: 12px 15px;
  border: 1px solid #dcdfe6;
  border-radius: 4px;
  font-size: 16px;
  transition: border-color 0.3s;
}

.search-input:focus {
  outline: none;
  border-color: #4a90e2;
}

.tips-categories {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-bottom: 25px;
}

.category-btn {
  padding: 8px 16px;
  border-radius: 20px;
  border: 1px solid #e0e6ed;
  background-color: #f5f7fa;
  color: #606266;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.3s;
}

.category-btn:hover {
  background-color: #eef2f7;
}

.category-btn.active {
  background-color: #4a90e2;
  color: white;
  border-color: #4a90e2;
}

.no-tips {
  text-align: center;
  padding: 40px 0;
  color: #909399;
  font-size: 16px;
}

.tip-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 20px;
}

.tip-card {
  background-color: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.08);
  padding: 20px;
  cursor: pointer;
  transition: transform 0.3s ease, box-shadow 0.3s ease;
  height: 100%;
  display: flex;
  flex-direction: column;
}

.tip-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
}

.tip-category {
  display: inline-block;
  padding: 4px 8px;
  background-color: #f0f4f8;
  border-radius: 4px;
  font-size: 12px;
  color: #606266;
  margin-bottom: 10px;
}

.tip-title {
  margin: 0 0 10px 0;
  font-size: 18px;
  color: #303133;
  line-height: 1.4;
}

.tip-summary {
  font-size: 14px;
  color: #606266;
  line-height: 1.6;
  flex-grow: 1;
}

.tip-footer {
  margin-top: 15px;
  text-align: right;
}

.read-more {
  color: #4a90e2;
  font-size: 14px;
}

.tip-modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.tip-modal-content {
  background-color: white;
  border-radius: 8px;
  width: 90%;
  max-width: 700px;
  max-height: 90vh;
  overflow-y: auto;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.2);
  padding: 25px;
}

.tip-modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}

.tip-modal-tag {
  display: inline-block;
  padding: 4px 8px;
  background-color: #f0f4f8;
  border-radius: 4px;
  font-size: 12px;
  color: #606266;
}

.close-btn {
  background: none;
  border: none;
  font-size: 24px;
  cursor: pointer;
  line-height: 1;
}

.tip-modal-title {
  font-size: 22px;
  color: #303133;
  margin-bottom: 20px;
}

.tip-modal-body {
  font-size: 16px;
  line-height: 1.8;
  color: #606266;
}

.tip-modal-body h4 {
  font-size: 18px;
  margin: 20px 0 10px;
  color: #303133;
}

.tip-modal-body ul, .tip-modal-body ol {
  padding-left: 20px;
  margin: 15px 0;
}

.tip-modal-body li {
  margin-bottom: 10px;
}

.tip-modal-footer {
  margin-top: 25px;
  text-align: right;
}

.close-tip-btn {
  background-color: #4a90e2;
  color: white;
  border: none;
  border-radius: 4px;
  padding: 10px 16px;
  font-size: 14px;
  cursor: pointer;
}

@media (max-width: 768px) {
  .tip-list {
    grid-template-columns: 1fr;
  }
  
  .tip-modal-content {
    padding: 15px;
  }
  
  .tip-modal-title {
    font-size: 18px;
  }
  
  .tip-modal-body {
    font-size: 14px;
  }
}
</style> 