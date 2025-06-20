<template>
  <div class="payment-code">
    <h3>在线收款码</h3>
    
    <div class="input-form">
      <div class="input-group">
        <label for="amount">金额 (元)</label>
        <input 
          type="number" 
          id="amount" 
          v-model="amount" 
          placeholder="多多益善"
          min="0.01"
          step="0.01"
          class="form-control"
        />
      </div>
      
      <div class="input-group">
        <label for="description">收款说明 (选填)</label>
        <input 
          type="text" 
          id="description" 
          v-model="description" 
          placeholder="请莫名其妙给我转钱，谢谢！"
          class="form-control"
        />
      </div>
      
      <div class="payment-tabs">
        <div 
          class="payment-tab" 
          :class="{ active: paymentMethod === 'wechat' }"
          @click="paymentMethod = 'wechat'"
        >
          <img src="/wechat-pay.png" class="payment-icon" alt="微信" />
          <span>微信支付</span>
        </div>
        <div 
          class="payment-tab" 
          :class="{ active: paymentMethod === 'alipay' }"
          @click="paymentMethod = 'alipay'"
        >
          <img src="/alipay.png" class="payment-icon" alt="支付宝" />
          <span>支付宝</span>
        </div>
      </div>
      
      <button 
        class="generate-btn" 
        :class="paymentMethod === 'wechat' ? 'wechat-btn' : 'alipay-btn'"
        @click="showQRCode"
      >
        显示{{ paymentMethod === 'wechat' ? '微信' : '支付宝' }}收款码
      </button>
    </div>
    
    <div v-if="showQR" class="qr-container">
      <p>请使用{{ paymentMethod === 'wechat' ? '微信' : '支付宝' }}扫描下方二维码完成支付</p>
      <div class="qr-wrapper">
        <img :src="getQRCodeSrc()" class="qr-image" alt="收款码" @click="openFullImage" />
        <div class="qr-tip">点击二维码可查看大图</div>
      </div>
      <p class="amount-display">￥{{ amount }}</p>
      <p class="description-display">{{ description || '在线收款' }}</p>
      <p class="note">请备注上方金额，否则可能导致订单无法完成</p>
      <button class="view-btn" @click="openFullImage">查看大图</button>
    </div>
    
    <div class="instructions">
      <h4>使用说明</h4>
      <ol>
        <li>输入您想要收款的金额</li>
        <li>可选择填写收款说明</li>
        <li>选择支付方式（微信/支付宝）</li>
        <li>点击显示收款码按钮</li>
        <li>点击二维码可查看大图</li>
        <li>扫描二维码并支付显示的金额</li>
      </ol>
    </div>
    
    <!-- 全屏二维码模态框 -->
    <div v-if="showFullImage" class="modal" @click="closeFullImage">
      <div class="modal-content" @click.stop>
        <span class="close-btn" @click="closeFullImage">&times;</span>
        <img :src="getQRCodeSrc()" class="full-qr-image" alt="收款码大图" />
        <p class="modal-amount">￥{{ amount }}</p>
        <p class="modal-tip">长按图片可保存或识别</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'PaymentCode',
  data() {
    return {
      amount: '',
      description: '',
      paymentMethod: 'wechat',
      showQR: false,
      showFullImage: false
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
    },
    openFullImage() {
      this.showFullImage = true;
    },
    closeFullImage() {
      this.showFullImage = false;
    }
  }
}
</script>

<style scoped>
.payment-code {
  background-color: #ffffff;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  padding: 20px;
  max-width: 600px;
  margin: 0 auto;
}

h3 {
  color: #333;
  font-size: 1.5rem;
  margin-top: 0;
  margin-bottom: 20px;
  text-align: center;
}

.input-form {
  margin-bottom: 20px;
}

.input-group {
  margin-bottom: 15px;
}

.input-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: 500;
  color: #444;
}

.form-control {
  width: 100%;
  padding: 10px 12px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 16px;
  transition: border-color 0.3s;
}

.form-control:focus {
  outline: none;
  border-color: #4a90e2;
}

.payment-tabs {
  display: flex;
  margin: 20px 0;
  border-bottom: 1px solid #eee;
}

.payment-tab {
  flex: 1;
  padding: 12px 0;
  text-align: center;
  cursor: pointer;
  border-bottom: 3px solid transparent;
  transition: all 0.3s;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
}

.payment-icon {
  width: 24px;
  height: 24px;
  border-radius: 50%;
}

.payment-tab.active {
  border-bottom-color: #4a90e2;
  color: #4a90e2;
  font-weight: 500;
}

.payment-tab:not(.active):hover {
  background-color: #f9f9f9;
}

.generate-btn {
  display: block;
  width: 100%;
  padding: 12px;
  border: none;
  border-radius: 4px;
  font-size: 16px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.3s;
  color: white;
}

.wechat-btn {
  background-color: #1aad19;
}

.wechat-btn:hover {
  background-color: #129611;
}

.alipay-btn {
  background-color: #1677ff;
}

.alipay-btn:hover {
  background-color: #0e5ecc;
}

.qr-container {
  background-color: #f9f9f9;
  border-radius: 6px;
  padding: 20px;
  margin-top: 20px;
  text-align: center;
}

.qr-wrapper {
  position: relative;
  margin: 15px auto;
  width: fit-content;
}

.qr-image {
  max-width: 300px;
  width: 100%;
  height: auto;
  border: 1px solid #eee;
  padding: 10px;
  background-color: white;
  border-radius: 4px;
  cursor: pointer;
  transition: transform 0.3s;
}

.qr-image:hover {
  transform: scale(1.02);
}

.qr-tip {
  margin-top: 8px;
  font-size: 14px;
  color: #888;
}

.amount-display {
  font-size: 24px;
  font-weight: bold;
  margin-top: 15px;
  color: #333;
}

.description-display {
  margin-top: 5px;
  color: #666;
}

.note {
  margin-top: 15px;
  color: #ff4d4f;
  font-size: 14px;
}

.view-btn {
  background: #f5f5f5;
  border: 1px solid #ddd;
  padding: 8px 15px;
  border-radius: 4px;
  margin-top: 15px;
  cursor: pointer;
  transition: all 0.3s;
}

.view-btn:hover {
  background: #e0e0e0;
}

.instructions {
  margin-top: 30px;
  padding-top: 20px;
  border-top: 1px solid #eee;
}

.instructions h4 {
  font-size: 18px;
  margin-bottom: 10px;
  color: #333;
}

.instructions ol {
  padding-left: 20px;
}

.instructions li {
  margin-bottom: 6px;
  color: #666;
}

/* 模态框样式 */
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background-color: white;
  padding: 20px;
  border-radius: 8px;
  max-width: 90%;
  position: relative;
  text-align: center;
}

.close-btn {
  position: absolute;
  top: 10px;
  right: 15px;
  font-size: 24px;
  cursor: pointer;
  color: #999;
}

.close-btn:hover {
  color: #333;
}

.full-qr-image {
  width: 100%;
  max-width: 500px;
  height: auto;
  border: 1px solid #eee;
  padding: 10px;
  background: white;
}

.modal-amount {
  font-size: 28px;
  font-weight: bold;
  margin-top: 15px;
}

.modal-tip {
  margin-top: 10px;
  color: #666;
}

@media (max-width: 576px) {
  .qr-image {
    max-width: 250px;
  }
  
  .modal-content {
    width: 95%;
    padding: 15px;
  }
}
</style> 