<template>
  <div class="savings-goal">
    <h2>存钱目标跟踪</h2>
    
    <div v-if="!hasGoals" class="no-goals">
      <p>您还没有设置存钱目标</p>
      <button @click="showAddGoalForm = true" class="add-goal-btn">+ 添加目标</button>
    </div>
    
    <div v-else class="goals-list">
      <div v-for="(goal, index) in goals" :key="index" class="goal-item">
        <div class="goal-header">
          <h3>{{ goal.name }}</h3>
          <div class="goal-actions">
            <button @click="updateGoalModal(index)" class="icon-btn">
              <span class="edit-icon">✏️</span>
            </button>
            <button @click="deleteGoal(index)" class="icon-btn">
              <span class="delete-icon">🗑️</span>
            </button>
          </div>
        </div>
        
        <div class="goal-amount">
          目标: <span class="amount">{{ formatCurrency(goal.targetAmount) }}</span>
        </div>
        
        <div class="goal-progress-container">
          <div class="goal-progress">
            <div class="goal-progress-bar" :style="{ width: goalProgressPercent(goal) + '%' }"></div>
          </div>
          <div class="goal-progress-text">
            {{ formatCurrency(goal.currentAmount) }} / {{ formatCurrency(goal.targetAmount) }}
            ({{ goalProgressPercent(goal) }}%)
          </div>
        </div>
        
        <div class="goal-info">
          <div v-if="goal.deadline" class="goal-deadline">
            <span class="info-label">截止日期:</span>
            {{ formatDate(goal.deadline) }}
            <span v-if="getDaysRemaining(goal) >= 0" class="days-remaining">
              (还剩 {{ getDaysRemaining(goal) }} 天)
            </span>
            <span v-else class="days-overdue">
              (已逾期 {{ Math.abs(getDaysRemaining(goal)) }} 天)
            </span>
          </div>
          
          <div class="goal-monthly">
            <span class="info-label">建议月存:</span>
            {{ calculateMonthlySaving(goal) }}
          </div>
        </div>
        
        <div class="update-progress">
          <button @click="openUpdateProgress(index)" class="update-btn">更新进度</button>
        </div>
      </div>
      
      <button @click="showAddGoalForm = true" class="add-goal-btn">+ 添加新目标</button>
    </div>
    
    <!-- 添加/编辑目标表单 -->
    <div v-if="showAddGoalForm" class="modal">
      <div class="modal-content">
        <div class="modal-header">
          <h3>{{ editingIndex === null ? '添加存钱目标' : '编辑存钱目标' }}</h3>
          <button @click="closeAddGoalForm" class="close-btn">&times;</button>
        </div>
        
        <form @submit.prevent="saveGoal" class="goal-form">
          <div class="form-group">
            <label for="goalName">目标名称</label>
            <input 
              type="text" 
              id="goalName" 
              v-model="newGoal.name" 
              placeholder="例如：旅行基金、紧急备用金" 
              required
            >
          </div>
          
          <div class="form-group">
            <label for="goalTarget">目标金额</label>
            <input 
              type="number" 
              id="goalTarget" 
              v-model="newGoal.targetAmount" 
              min="1" 
              step="1" 
              required
            >
          </div>
          
          <div class="form-group">
            <label for="goalCurrent">当前已存</label>
            <input 
              type="number" 
              id="goalCurrent" 
              v-model="newGoal.currentAmount" 
              min="0" 
              step="1"
            >
          </div>
          
          <div class="form-group">
            <label for="goalDeadline">截止日期 (可选)</label>
            <input 
              type="date" 
              id="goalDeadline" 
              v-model="newGoal.deadline"
            >
          </div>
          
          <div class="form-actions">
            <button type="button" @click="closeAddGoalForm" class="cancel-btn">取消</button>
            <button type="submit" class="save-btn">保存</button>
          </div>
        </form>
      </div>
    </div>
    
    <!-- 更新进度表单 -->
    <div v-if="showUpdateProgress" class="modal">
      <div class="modal-content">
        <div class="modal-header">
          <h3>更新进度 - {{ goals[updatingIndex]?.name }}</h3>
          <button @click="showUpdateProgress = false" class="close-btn">&times;</button>
        </div>
        
        <form @submit.prevent="saveProgress" class="update-form">
          <div class="form-group">
            <label for="currentSavings">当前已存金额</label>
            <input 
              type="number" 
              id="currentSavings" 
              v-model="updatedAmount" 
              min="0" 
              step="1" 
              required
            >
          </div>
          
          <div class="form-actions">
            <button type="button" @click="showUpdateProgress = false" class="cancel-btn">取消</button>
            <button type="submit" class="save-btn">保存</button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'SavingsGoal',
  data() {
    return {
      goals: [],
      newGoal: {
        name: '',
        targetAmount: 0,
        currentAmount: 0,
        deadline: ''
      },
      showAddGoalForm: false,
      editingIndex: null,
      showUpdateProgress: false,
      updatingIndex: null,
      updatedAmount: 0
    }
  },
  computed: {
    hasGoals() {
      return this.goals.length > 0
    }
  },
  mounted() {
    this.loadGoals()
  },
  methods: {
    loadGoals() {
      const savedGoals = localStorage.getItem('savingsGoals')
      if (savedGoals) {
        this.goals = JSON.parse(savedGoals)
      }
    },
    saveGoalsToStorage() {
      localStorage.setItem('savingsGoals', JSON.stringify(this.goals))
    },
    goalProgressPercent(goal) {
      const percent = (goal.currentAmount / goal.targetAmount) * 100
      return Math.min(100, Math.round(percent))
    },
    formatCurrency(amount) {
      return new Intl.NumberFormat('zh-CN', { 
        style: 'currency', 
        currency: 'CNY',
        minimumFractionDigits: 0,
        maximumFractionDigits: 0
      }).format(amount)
    },
    formatDate(dateString) {
      const date = new Date(dateString)
      return date.toLocaleDateString('zh-CN', { 
        year: 'numeric', 
        month: 'long', 
        day: 'numeric' 
      })
    },
    getDaysRemaining(goal) {
      if (!goal.deadline) return null
      
      const today = new Date()
      today.setHours(0, 0, 0, 0)
      
      const deadline = new Date(goal.deadline)
      deadline.setHours(0, 0, 0, 0)
      
      const timeDiff = deadline.getTime() - today.getTime()
      return Math.ceil(timeDiff / (1000 * 3600 * 24))
    },
    calculateMonthlySaving(goal) {
      if (!goal.deadline) return '未设置截止日期'
      
      const remaining = goal.targetAmount - goal.currentAmount
      if (remaining <= 0) return this.formatCurrency(0)
      
      const daysRemaining = this.getDaysRemaining(goal)
      if (daysRemaining <= 0) return '已过截止日期'
      
      const monthsRemaining = daysRemaining / 30
      const monthlySaving = remaining / monthsRemaining
      
      return this.formatCurrency(Math.ceil(monthlySaving))
    },
    saveGoal() {
      const goalToSave = {
        name: this.newGoal.name,
        targetAmount: parseFloat(this.newGoal.targetAmount),
        currentAmount: parseFloat(this.newGoal.currentAmount) || 0,
        deadline: this.newGoal.deadline || null
      }
      
      if (this.editingIndex !== null) {
        this.goals.splice(this.editingIndex, 1, goalToSave)
      } else {
        this.goals.push(goalToSave)
      }
      
      this.saveGoalsToStorage()
      this.closeAddGoalForm()
    },
    updateGoalModal(index) {
      this.editingIndex = index
      const goal = this.goals[index]
      this.newGoal = {
        name: goal.name,
        targetAmount: goal.targetAmount,
        currentAmount: goal.currentAmount,
        deadline: goal.deadline || ''
      }
      this.showAddGoalForm = true
    },
    closeAddGoalForm() {
      this.showAddGoalForm = false
      this.editingIndex = null
      this.newGoal = {
        name: '',
        targetAmount: 0,
        currentAmount: 0,
        deadline: ''
      }
    },
    deleteGoal(index) {
      if (confirm('确定要删除这个存钱目标吗？')) {
        this.goals.splice(index, 1)
        this.saveGoalsToStorage()
      }
    },
    openUpdateProgress(index) {
      this.updatingIndex = index
      this.updatedAmount = this.goals[index].currentAmount
      this.showUpdateProgress = true
    },
    saveProgress() {
      if (this.updatingIndex !== null) {
        this.goals[this.updatingIndex].currentAmount = parseFloat(this.updatedAmount)
        this.saveGoalsToStorage()
        this.showUpdateProgress = false
      }
    }
  }
}
</script>

<style scoped>
.savings-goal {
  max-width: 800px;
  margin: 0 auto;
}

h2 {
  text-align: center;
  margin-bottom: 24px;
  color: #333;
}

.no-goals {
  text-align: center;
  margin: 40px 0;
}

.add-goal-btn {
  background-color: #4a90e2;
  color: white;
  border: none;
  border-radius: 4px;
  padding: 10px 16px;
  font-size: 15px;
  cursor: pointer;
  transition: background-color 0.3s;
  display: block;
  margin: 20px auto;
}

.add-goal-btn:hover {
  background-color: #3a7bc8;
}

.goals-list {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.goal-item {
  background-color: #fff;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.08);
}

.goal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}

.goal-header h3 {
  margin: 0;
  font-size: 18px;
  color: #333;
}

.goal-actions {
  display: flex;
  gap: 10px;
}

.icon-btn {
  background: none;
  border: none;
  cursor: pointer;
  font-size: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 5px;
}

.goal-amount {
  margin-bottom: 15px;
  font-size: 16px;
}

.amount {
  font-weight: bold;
  color: #4a90e2;
}

.goal-progress-container {
  margin-bottom: 15px;
}

.goal-progress {
  height: 10px;
  background-color: #e0e6ed;
  border-radius: 5px;
  overflow: hidden;
  margin-bottom: 5px;
}

.goal-progress-bar {
  height: 100%;
  background-color: #4a90e2;
  border-radius: 5px;
  transition: width 0.5s ease;
}

.goal-progress-text {
  font-size: 14px;
  color: #606266;
  text-align: right;
}

.goal-info {
  margin-bottom: 15px;
  font-size: 14px;
  color: #606266;
}

.info-label {
  font-weight: bold;
  margin-right: 4px;
}

.days-remaining {
  color: #67c23a;
}

.days-overdue {
  color: #f56c6c;
}

.goal-monthly {
  margin-top: 5px;
}

.update-progress {
  text-align: center;
}

.update-btn {
  background-color: #67c23a;
  color: white;
  border: none;
  border-radius: 4px;
  padding: 8px 14px;
  font-size: 14px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.update-btn:hover {
  background-color: #5daf33;
}

.modal {
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

.modal-content {
  background-color: white;
  border-radius: 8px;
  width: 90%;
  max-width: 500px;
  max-height: 90vh;
  overflow-y: auto;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.2);
}

.modal-header {
  padding: 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid #e0e6ed;
}

.modal-header h3 {
  margin: 0;
  font-size: 18px;
}

.close-btn {
  background: none;
  border: none;
  font-size: 24px;
  cursor: pointer;
  line-height: 1;
}

.goal-form, .update-form {
  padding: 20px;
}

.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
  color: #606266;
}

.form-group input {
  width: 100%;
  padding: 10px;
  border: 1px solid #dcdfe6;
  border-radius: 4px;
  font-size: 16px;
}

.form-actions {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  margin-top: 20px;
}

.cancel-btn {
  background-color: #f0f4f8;
  color: #606266;
  border: none;
  border-radius: 4px;
  padding: 10px 16px;
  font-size: 14px;
  cursor: pointer;
}

.save-btn {
  background-color: #4a90e2;
  color: white;
  border: none;
  border-radius: 4px;
  padding: 10px 16px;
  font-size: 14px;
  cursor: pointer;
}

@media (max-width: 768px) {
  .goal-item {
    padding: 15px;
  }
  
  .form-actions {
    flex-direction: column;
    gap: 10px;
  }
  
  .cancel-btn, .save-btn {
    width: 100%;
  }
}
</style> 