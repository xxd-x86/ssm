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
        <qrcode-vue :value="qrValue" :size="200" level="H" />
        <p class="amount-display">￥{{ amount }}</p>
      </div>
      
      <button 
        class="btn" 
        :class="paymentMethod === 'wechat' ? 'btn-primary' : 'btn-secondary'"
        @click="generateQR"
      >
        生成{{ paymentMethod === 'wechat' ? '微信' : '支付宝' }}收款码
      </button>
    </div>
    
    <div class="card">
      <h2>使用说明</h2>
      <p>1. 输入您想要收款的金额</p>
      <p>2. 可选择填写收款说明</p>
      <p>3. 选择支付方式（微信/支付宝）</p>
      <p>4. 点击生成收款码按钮</p>
      <p>5. 让付款方扫描生成的二维码完成支付</p>
    </div>
  </div>
</template>

<script>
import QrcodeVue from 'qrcode.vue'

export default {
  name: 'App',
  components: {
    QrcodeVue
  },
  data() {
    return {
      amount: '',
      description: '',
      paymentMethod: 'wechat',
      showQR: false,
      qrValue: ''
    }
  },
  methods: {
    generateQR() {
      if (!this.amount || this.amount <= 0) {
        alert('请输入有效金额');
        return;
      }
      
      // 生成支付链接 (这里是模拟的)
      // 实际应用中应该调用微信/支付宝API生成真实的支付链接
      if (this.paymentMethod === 'wechat') {
        // 微信支付码格式
        this.qrValue = `wxp://f2f0${this.amount}/${this.description || '在线收款'}`;
      } else {
        // 支付宝支付码格式
        this.qrValue = `https://qr.alipay.com/fkx${this.amount}/${this.description || '在线收款'}`;
      }
      
      this.showQR = true;
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
</style> 