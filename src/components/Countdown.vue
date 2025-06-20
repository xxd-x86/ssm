<template>
  <div class="countdown-app">
    <h3>纪念日提醒</h3>
    
    <div class="add-reminder-form" v-if="showAddForm">
      <div class="form-group">
        <label>事件名称</label>
        <input 
          type="text" 
          v-model="newReminder.title" 
          placeholder="如: 结婚纪念日" 
          class="form-control"
        />
      </div>
      
      <div class="form-group">
        <label>日期</label>
        <input 
          type="date" 
          v-model="newReminder.date" 
          class="form-control"
        />
      </div>
      
      <div class="form-group">
        <label>类型</label>
        <div class="reminder-type-options">
          <label class="radio-label">
            <input type="radio" v-model="newReminder.type" value="countdown" />
            倒计时
          </label>
          <label class="radio-label">
            <input type="radio" v-model="newReminder.type" value="anniversary" />
            纪念日
          </label>
        </div>
      </div>
      
      <div class="form-group" v-if="newReminder.type === 'anniversary'">
        <label>
          <input type="checkbox" v-model="newReminder.yearly" />
          每年重复
        </label>
      </div>
      
      <div class="form-actions">
        <button @click="addReminder" class="btn-primary" :disabled="!isFormValid">保存</button>
        <button @click="cancelAdd" class="btn-secondary">取消</button>
      </div>
    </div>
    
    <button v-else @click="showAddForm = true" class="add-reminder-btn">
      <span class="plus-icon">+</span> 添加新纪念日
    </button>
    
    <div class="reminders-empty" v-if="reminders.length === 0">
      <p>还没有纪念日，点击"添加新纪念日"开始记录</p>
    </div>
    
    <div class="reminders-list">
      <div 
        v-for="(reminder, index) in sortedReminders" 
        :key="reminder.id" 
        class="reminder-card"
        :class="getReminderClass(reminder)"
      >
        <div class="reminder-header">
          <div class="reminder-title">{{ reminder.title }}</div>
          <div class="reminder-actions">
            <button @click="deleteReminder(index)" class="delete-btn" title="删除">
              ×
            </button>
          </div>
        </div>
        
        <div class="reminder-date">
          {{ formatDate(reminder.date) }}
        </div>
        
        <div class="reminder-time">
          <div class="time-info">
            <div class="time-value">{{ getReminderTimeValue(reminder) }}</div>
            <div class="time-label">{{ getReminderTimeLabel(reminder) }}</div>
          </div>
        </div>
        
        <div class="reminder-type-badge">
          {{ reminder.type === 'countdown' ? '倒计时' : '纪念日' }}
          <span v-if="reminder.type === 'anniversary' && reminder.yearly">
            (每年)
          </span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Countdown',
  data() {
    return {
      reminders: [],
      showAddForm: false,
      newReminder: this.getDefaultReminder(),
      today: new Date()
    };
  },
  computed: {
    isFormValid() {
      return this.newReminder.title.trim() !== '' && this.newReminder.date !== '';
    },
    
    sortedReminders() {
      return [...this.reminders].sort((a, b) => {
        const timeA = this.getDaysInfo(a).days;
        const timeB = this.getDaysInfo(b).days;
        
        if (a.type === 'countdown' && b.type === 'countdown') {
          // 倒计时按剩余天数升序排列
          return timeA - timeB;
        } else if (a.type === 'anniversary' && b.type === 'anniversary') {
          // 纪念日按天数降序排列（最近的排前面）
          if (timeA < 0 && timeB < 0) {
            return timeB - timeA; // 过去的日子，最近的排前面
          } else {
            return timeA - timeB;
          }
        } else {
          // 倒计时优先
          return a.type === 'countdown' ? -1 : 1;
        }
      });
    }
  },
  mounted() {
    this.loadReminders();
    
    // 每天零点更新today值
    this.scheduleNextDayUpdate();
    
    // 同时设置一个24小时的定时器，以防应用长时间运行
    setInterval(() => {
      this.today = new Date();
    }, 24 * 60 * 60 * 1000);
  },
  methods: {
    getDefaultReminder() {
      return {
        title: '',
        date: new Date().toISOString().slice(0, 10), // YYYY-MM-DD
        type: 'countdown',
        yearly: false
      };
    },
    
    scheduleNextDayUpdate() {
      const now = new Date();
      const tomorrow = new Date(now.getFullYear(), now.getMonth(), now.getDate() + 1);
      const timeUntilMidnight = tomorrow - now;
      
      setTimeout(() => {
        this.today = new Date();
        this.scheduleNextDayUpdate();
      }, timeUntilMidnight);
    },
    
    loadReminders() {
      const savedReminders = localStorage.getItem('reminders');
      if (savedReminders) {
        try {
          this.reminders = JSON.parse(savedReminders);
        } catch (e) {
          console.error('Failed to parse reminders:', e);
          this.reminders = [];
        }
      }
    },
    
    saveReminders() {
      localStorage.setItem('reminders', JSON.stringify(this.reminders));
    },
    
    addReminder() {
      if (this.isFormValid) {
        const reminder = {
          ...this.newReminder,
          id: Date.now()
        };
        
        this.reminders.push(reminder);
        this.saveReminders();
        this.showAddForm = false;
        this.newReminder = this.getDefaultReminder();
      }
    },
    
    cancelAdd() {
      this.showAddForm = false;
      this.newReminder = this.getDefaultReminder();
    },
    
    deleteReminder(index) {
      if (confirm('确定要删除这个纪念日吗？')) {
        this.reminders.splice(index, 1);
        this.saveReminders();
      }
    },
    
    formatDate(dateString) {
      const date = new Date(dateString);
      const year = date.getFullYear();
      const month = (date.getMonth() + 1).toString().padStart(2, '0');
      const day = date.getDate().toString().padStart(2, '0');
      
      return `${year}年${month}月${day}日`;
    },
    
    getDaysInfo(reminder) {
      const eventDate = new Date(reminder.date);
      const today = new Date(this.today.getFullYear(), this.today.getMonth(), this.today.getDate());
      
      if (reminder.type === 'anniversary' && reminder.yearly) {
        // 如果是每年重复的纪念日，计算今年的日期
        const thisYearDate = new Date(today.getFullYear(), eventDate.getMonth(), eventDate.getDate());
        
        // 如果今年的日期已经过了，计算到明年的日期
        if (thisYearDate < today) {
          const nextYearDate = new Date(today.getFullYear() + 1, eventDate.getMonth(), eventDate.getDate());
          const daysUntilNext = Math.ceil((nextYearDate - today) / (1000 * 60 * 60 * 24));
          return { days: daysUntilNext, past: false, text: `还有${daysUntilNext}天` };
        } else {
          const daysUntil = Math.ceil((thisYearDate - today) / (1000 * 60 * 60 * 24));
          if (daysUntil === 0) {
            return { days: 0, past: false, text: '就是今天！' };
          }
          return { days: daysUntil, past: false, text: `还有${daysUntil}天` };
        }
      } else if (reminder.type === 'anniversary') {
        // 非重复的纪念日，计算已经过去多少天
        const daysPassed = Math.ceil((today - eventDate) / (1000 * 60 * 60 * 24));
        if (daysPassed <= 0) {
          return { days: -daysPassed, past: false, text: `还有${-daysPassed}天` };
        }
        return { days: -daysPassed, past: true, text: `已过去${daysPassed}天` };
      } else {
        // 倒计时
        const daysLeft = Math.ceil((eventDate - today) / (1000 * 60 * 60 * 24));
        if (daysLeft < 0) {
          return { days: daysLeft, past: true, text: `已经过期${-daysLeft}天` };
        } else if (daysLeft === 0) {
          return { days: 0, past: false, text: '就是今天！' };
        }
        return { days: daysLeft, past: false, text: `还有${daysLeft}天` };
      }
    },
    
    getReminderTimeValue(reminder) {
      const info = this.getDaysInfo(reminder);
      return Math.abs(info.days);
    },
    
    getReminderTimeLabel(reminder) {
      const info = this.getDaysInfo(reminder);
      return info.text;
    },
    
    getReminderClass(reminder) {
      const info = this.getDaysInfo(reminder);
      
      if (info.days === 0) {
        return 'today';
      } else if (reminder.type === 'countdown') {
        if (info.past) {
          return 'expired';
        } else if (info.days <= 7) {
          return 'coming-soon';
        }
      }
      
      return '';
    }
  }
};
</script>

<style scoped>
.countdown-app {
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

.add-reminder-btn {
  width: 100%;
  background-color: #4a90e2;
  color: white;
  border: none;
  border-radius: 4px;
  padding: 10px;
  margin-bottom: 15px;
  font-size: 15px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 5px;
}

.add-reminder-btn:hover {
  background-color: #3a7bc8;
}

.plus-icon {
  font-size: 18px;
  font-weight: bold;
}

.add-reminder-form {
  background-color: #f9f9f9;
  border-radius: 6px;
  padding: 15px;
  margin-bottom: 15px;
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: 500;
  color: #444;
}

.form-control {
  width: 100%;
  padding: 8px 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 15px;
}

.reminder-type-options {
  display: flex;
  gap: 20px;
}

.radio-label {
  display: flex;
  align-items: center;
  gap: 5px;
}

.form-actions {
  display: flex;
  gap: 10px;
  justify-content: flex-end;
}

.btn-primary {
  background-color: #4a90e2;
  color: white;
  border: none;
  border-radius: 4px;
  padding: 8px 15px;
  cursor: pointer;
}

.btn-primary:hover:not(:disabled) {
  background-color: #3a7bc8;
}

.btn-primary:disabled {
  background-color: #a0c4ef;
  cursor: not-allowed;
}

.btn-secondary {
  background-color: #f0f0f0;
  color: #666;
  border: 1px solid #ddd;
  border-radius: 4px;
  padding: 8px 15px;
  cursor: pointer;
}

.btn-secondary:hover {
  background-color: #e0e0e0;
}

.reminders-empty {
  text-align: center;
  color: #888;
  font-style: italic;
  padding: 20px 0;
}

.reminders-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.reminder-card {
  background-color: #f9f9f9;
  border-left: 4px solid #ccc;
  border-radius: 4px;
  padding: 15px;
  position: relative;
}

.reminder-card.expired {
  border-left-color: #e74c3c;
}

.reminder-card.coming-soon {
  border-left-color: #f1c40f;
}

.reminder-card.today {
  border-left-color: #2ecc71;
  background-color: #eafaf1;
}

.reminder-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px;
}

.reminder-title {
  font-weight: 500;
  font-size: 17px;
  color: #333;
}

.reminder-date {
  color: #777;
  font-size: 14px;
  margin-bottom: 10px;
}

.reminder-time {
  display: flex;
  align-items: center;
  margin-bottom: 5px;
}

.time-info {
  display: flex;
  align-items: baseline;
  gap: 8px;
}

.time-value {
  font-size: 24px;
  font-weight: bold;
  color: #333;
}

.time-label {
  font-size: 15px;
  color: #555;
}

.reminder-type-badge {
  position: absolute;
  top: 15px;
  right: 30px;
  font-size: 12px;
  background-color: rgba(0, 0, 0, 0.05);
  color: #777;
  padding: 2px 8px;
  border-radius: 10px;
}

.delete-btn {
  background: none;
  border: none;
  color: #ccc;
  font-size: 18px;
  font-weight: bold;
  cursor: pointer;
  padding: 0 5px;
  line-height: 1;
}

.delete-btn:hover {
  color: #e74c3c;
}

@media (max-width: 480px) {
  .reminder-header {
    flex-direction: column;
    align-items: flex-start;
  }
  
  .reminder-actions {
    position: absolute;
    top: 10px;
    right: 10px;
  }
  
  .reminder-type-badge {
    position: static;
    display: inline-block;
    margin-top: 5px;
  }
}
</style> 