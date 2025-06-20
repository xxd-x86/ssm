<template>
  <div class="daily-quote">
    <h3>{{ isDaily ? '每日一句' : '随机名言' }}</h3>
    <div class="quote-container" @click="getNewQuote">
      <div class="quote-content">
        <div class="quote-text">
          <i class="quote-mark">❝</i>
          {{ quote.text }}
          <i class="quote-mark">❞</i>
        </div>
        <div class="quote-author">—— {{ quote.author }}</div>
      </div>
    </div>
    
    <div class="quote-controls">
      <label class="switch">
        <input type="checkbox" v-model="isDaily">
        <span class="slider"></span>
      </label>
      <span class="mode-text">{{ isDaily ? '每日模式' : '随机模式' }}</span>
      <button @click="getNewQuote" class="refresh-btn" :disabled="isDaily">
        <span class="refresh-icon">↻</span>
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'DailyQuote',
  data() {
    return {
      quote: {
        text: '',
        author: ''
      },
      isDaily: true,
      quotes: [
        {
          text: "生活就像骑自行车，要保持平衡就得不断前进。",
          author: "爱因斯坦"
        },
        {
          text: "人生如同故事。重要的不是它有多长，而是它有多精彩。",
          author: "塞涅卡"
        },
        {
          text: "世上只有一种英雄主义，就是认清生活的真相后依然热爱生活。",
          author: "罗曼·罗兰"
        },
        {
          text: "不要等待机会，而要创造机会。",
          author: "林肯"
        },
        {
          text: "人的一生可能燃烧也可能腐朽，我不能腐朽，我愿意燃烧起来。",
          author: "奥斯特洛夫斯基"
        },
        {
          text: "人生不售来回票，一旦动身，绝不能复返。",
          author: "罗曼·罗兰"
        },
        {
          text: "我们一旦接受了最坏的情况，就没有什么可失去的了。",
          author: "卡耐基"
        },
        {
          text: "人生就是进行一场无法预知结局的旅行。",
          author: "莫泊桑"
        },
        {
          text: "当一个人真正觉醒的时候，他才能够从内心深处发出真正的笑声。",
          author: "梭罗"
        },
        {
          text: "生活中最重要的不是我们拥有什么，而是我们体验了什么。",
          author: "乔纳森"
        },
        {
          text: "生命如同寓言，其价值不在于长短，而在于内容。",
          author: "塞涅卡"
        },
        {
          text: "命运给予我们的不是失望之酒，而是机会之杯。",
          author: "雪莱"
        },
        {
          text: "不要因为已经走了很远就忘记当初为什么出发。",
          author: "纪伯伦"
        },
        {
          text: "没有口水与汗水，就没有成功的泪水。",
          author: "俞敏洪"
        },
        {
          text: "我们必须拿我们所有的，去换我们所没有的。",
          author: "稻盛和夫"
        },
        {
          text: "世界上最快乐的事，莫过于为理想而奋斗。",
          author: "苏格拉底"
        },
        {
          text: "平凡的脚步也可以走完伟大的旅程。",
          author: "高尔基"
        },
        {
          text: "不要见小利而忘大义。",
          author: "《资治通鉴》"
        },
        {
          text: "天行健，君子以自强不息。",
          author: "《周易》"
        },
        {
          text: "人生最大的喜悦是每个人都说你做不到，你却完成它了。",
          author: "布莱恩·特雷西"
        },
        {
          text: "没有人可以回到过去重新开始，但谁都可以从今日开始，书写一个全然不同的结局。",
          author: "玛丽亚·罗宾逊"
        },
        {
          text: "世上没有绝望的处境，只有对处境绝望的人。",
          author: "黑塞"
        },
        {
          text: "能把在面前行走的机会抓住的人，十有八九都会成功。",
          author: "歌德"
        },
        {
          text: "有志者自有千方百计，无志者只感千难万难。",
          author: "苏轼"
        },
        {
          text: "再长的路，一步步也能走完，再短的路，不迈开双脚也无法到达。",
          author: "佚名"
        },
        {
          text: "梦想是一个天真的词，实现梦想是一个残酷的词。",
          author: "马云"
        },
        {
          text: "只有经历过地狱般的折磨，才有征服天堂的力量。",
          author: "尼采"
        },
        {
          text: "人若软弱就是自己最大的敌人。",
          author: "卢梭"
        },
        {
          text: "你若不想做，会找一个借口；你若想做，会找一个方法。",
          author: "阿拉伯谚语"
        },
        {
          text: "宁可慢些，不要太匆忙，因为成功不会偏向任何人，它偏爱有准备的人。",
          author: "拿破仑·希尔"
        }
      ]
    };
  },
  mounted() {
    this.setQuote();
  },
  methods: {
    getDailyQuoteIndex() {
      // 生成每天固定的索引，基于日期
      const now = new Date();
      const day = now.getDate();
      const month = now.getMonth() + 1;
      const yearDay = now.getFullYear() * 1000 + day * month;
      
      return yearDay % this.quotes.length;
    },
    
    getRandomQuoteIndex() {
      return Math.floor(Math.random() * this.quotes.length);
    },
    
    setQuote() {
      const index = this.isDaily ? this.getDailyQuoteIndex() : this.getRandomQuoteIndex();
      this.quote = this.quotes[index];
      
      // 保存上次刷新时间和模式
      const quoteData = {
        lastRefreshed: new Date().toISOString(),
        isDaily: this.isDaily
      };
      localStorage.setItem('quotePreference', JSON.stringify(quoteData));
    },
    
    getNewQuote() {
      if (!this.isDaily) {
        // 仅在随机模式下才能刷新
        this.setQuote();
      }
    },
    
    loadPreferences() {
      const savedPref = localStorage.getItem('quotePreference');
      if (savedPref) {
        try {
          const prefs = JSON.parse(savedPref);
          this.isDaily = prefs.isDaily;
        } catch (e) {
          console.error('Failed to parse quote preferences:', e);
        }
      }
    }
  },
  watch: {
    isDaily() {
      this.setQuote();
    }
  }
};
</script>

<style scoped>
.daily-quote {
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

.quote-container {
  background-color: #f9f9f9;
  border-radius: 6px;
  padding: 20px;
  margin-bottom: 15px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.quote-container:hover {
  background-color: #f0f0f0;
  transform: translateY(-3px);
}

.quote-content {
  position: relative;
}

.quote-text {
  font-size: 18px;
  line-height: 1.5;
  color: #444;
  text-align: center;
  margin-bottom: 15px;
}

.quote-mark {
  font-style: normal;
  color: #aaa;
  font-size: 24px;
  margin: 0 4px;
}

.quote-author {
  font-size: 16px;
  color: #888;
  text-align: right;
  font-style: italic;
}

.quote-controls {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 12px;
}

.mode-text {
  font-size: 14px;
  color: #666;
}

.refresh-btn {
  background: none;
  border: none;
  color: #4a90e2;
  font-size: 20px;
  cursor: pointer;
  padding: 5px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  width: 30px;
  height: 30px;
  transition: all 0.3s ease;
}

.refresh-btn:hover:not(:disabled) {
  background-color: #eaf2fd;
  transform: rotate(180deg);
}

.refresh-btn:disabled {
  color: #ccc;
  cursor: not-allowed;
}

.refresh-icon {
  display: inline-block;
}

/* 切换开关样式 */
.switch {
  position: relative;
  display: inline-block;
  width: 50px;
  height: 24px;
}

.switch input {
  opacity: 0;
  width: 0;
  height: 0;
}

.slider {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #ccc;
  transition: .4s;
  border-radius: 24px;
}

.slider:before {
  position: absolute;
  content: "";
  height: 18px;
  width: 18px;
  left: 3px;
  bottom: 3px;
  background-color: white;
  transition: .4s;
  border-radius: 50%;
}

input:checked + .slider {
  background-color: #4a90e2;
}

input:focus + .slider {
  box-shadow: 0 0 1px #4a90e2;
}

input:checked + .slider:before {
  transform: translateX(26px);
}

@media (max-width: 480px) {
  .daily-quote {
    padding: 15px;
  }
  
  h3 {
    font-size: 1.3rem;
    margin-bottom: 15px;
  }
  
  .quote-container {
    padding: 15px;
  }
  
  .quote-text {
    font-size: 16px;
    line-height: 1.4;
  }
  
  .quote-author {
    font-size: 14px;
  }
  
  .quote-mark {
    font-size: 18px;
  }
}
</style> 