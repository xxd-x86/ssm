<template>
  <div class="container">
    <div class="header">
      <h1>在线收款</h1>
      <p>安全、快捷的支付方式</p>
    </div>
    
    <div class="card">
      <div class="input-group">
        <label for="amount">金额 (元)</label>
        <input 
          type="number" 
          id="amount" 
          v-model="amount" 
          placeholder="请输入收款金额"
          min="0.01"
          step="0.01"
        />
      </div>
      
      <div class="input-group">
        <label for="description">收款说明 (选填)</label>
        <input 
          type="text" 
          id="description" 
          v-model="description" 
          placeholder="请输入收款说明"
        />
      </div>
      
      <div class="tab-container">
        <div 
          class="tab" 
          :class="{ active: paymentMethod === 'wechat' }"
          @click="paymentMethod = 'wechat'"
        >
          微信支付
        </div>
        <div 
          class="tab" 
          :class="{ active: paymentMethod === 'alipay' }"
          @click="paymentMethod = 'alipay'"
        >
          支付宝
        </div>
      </div>
      
      <div v-if="showQR" class="qr-container">
        <p>请使用{{ paymentMethod === 'wechat' ? '微信' : '支付宝' }}扫描下方二维码完成支付</p>
        <img :src="getQRCodeSrc()" class="qr-image" alt="收款码" />
        <p class="amount-display">￥{{ amount }}</p>
        <p class="description-display">{{ description || '在线收款' }}</p>
        <p class="note">请备注上方金额，否则可能导致订单无法完成</p>
      </div>
      
      <button 
        class="btn" 
        :class="paymentMethod === 'wechat' ? 'btn-primary' : 'btn-secondary'"
        @click="showQRCode"
      >
        显示{{ paymentMethod === 'wechat' ? '微信' : '支付宝' }}收款码
      </button>
    </div>
    
    <div class="card">
      <h2>使用说明</h2>
      <p>1. 输入您想要收款的金额</p>
      <p>2. 可选择填写收款说明</p>
      <p>3. 选择支付方式（微信/支付宝）</p>
      <p>4. 点击显示收款码按钮</p>
      <p>5. 扫描二维码并支付显示的金额</p>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      amount: '',
      description: '',
      paymentMethod: 'wechat',
      showQR: false,
    }
  },
  methods: {
    showQRCode() {
      if (!this.amount || this.amount <= 0) {
        alert('请输入有效金额');
        return;
      }
      
      this.showQR = true;
    },
    getQRCodeSrc() {
      // 使用预设的收款码图片
      return this.paymentMethod === 'wechat' ? '/wechat-pay.png' : '/alipay.png';
    }
  }
}
</script>

<style>
.amount-display {
  font-size: 24px;
  font-weight: bold;
  margin-top: 10px;
}

.description-display {
  margin-top: 5px;
  font-size: 16px;
}

.note {
  margin-top: 10px;
  font-size: 14px;
  color: #ff4d4f;
}

.qr-image {
  max-width: 200px;
  max-height: 200px;
  margin: 10px auto;
  display: block;
}

.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

.header {
  text-align: center;
  margin-bottom: 30px;
}

.card {
  background: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
  padding: 20px;
  margin-bottom: 20px;
}

.input-group {
  margin-bottom: 15px;
}

.input-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

.input-group input {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 16px;
}

.tab-container {
  display: flex;
  margin: 20px 0;
  border-bottom: 1px solid #eee;
}

.tab {
  padding: 10px 20px;
  cursor: pointer;
  border-bottom: 2px solid transparent;
}

.tab.active {
  border-bottom: 2px solid #1890ff;
  color: #1890ff;
}

.qr-container {
  text-align: center;
  margin: 20px 0;
  padding: 20px;
  border: 1px dashed #ddd;
  border-radius: 4px;
}

.btn {
  display: block;
  width: 100%;
  padding: 12px;
  border: none;
  border-radius: 4px;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s;
}

.btn-primary {
  background-color: #1aad19;
  color: white;
}

.btn-primary:hover {
  background-color: #129611;
}

.btn-secondary {
  background-color: #1677ff;
  color: white;
}

.btn-secondary:hover {
  background-color: #0e5ecc;
}
</style> 