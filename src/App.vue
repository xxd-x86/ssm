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
        <div class="qr-wrapper">
          <img :src="getQRCodeSrc()" class="qr-image" alt="收款码" @click="openFullImage" />
          <div class="qr-tip">点击二维码可查看大图</div>
        </div>
        <p class="amount-display">￥{{ amount }}</p>
        <p class="description-display">{{ description || '在线收款' }}</p>
        <p class="note">请备注上方金额，否则可能导致订单无法完成</p>
        <button class="save-btn" @click="openFullImage">查看大图</button>
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
      <p>5. 点击二维码可查看大图</p>
      <p>6. 扫描二维码并支付显示的金额</p>
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
  name: 'App',
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

.qr-wrapper {
  position: relative;
  margin: 0 auto;
  width: fit-content;
}

.qr-tip {
  margin-top: 5px;
  font-size: 14px;
  color: #666;
}

.qr-image {
  max-width: 300px; /* 增大尺寸 */
  width: 80%; /* 确保在移动设备上占据足够宽度 */
  height: auto;
  margin: 15px auto;
  display: block;
  border: 1px solid #eee; /* 添加边框使图片更清晰 */
  padding: 10px; /* 添加内边距 */
  background-color: white; /* 确保背景为白色以提高对比度 */
  cursor: pointer;
}

.save-btn {
  background: #f5f5f5;
  border: 1px solid #ddd;
  padding: 8px 15px;
  border-radius: 4px;
  margin-top: 10px;
  cursor: pointer;
  transition: background 0.3s;
}

.save-btn:hover {
  background: #e0e0e0;
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
  color: #666;
}

.full-qr-image {
  width: 100%;
  max-width: 500px;
  height: auto;
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

.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

@media (max-width: 480px) {
  .container {
    padding: 10px;
  }
  
  .qr-image {
    max-width: 100%;
    width: 90%;
  }
  
  .amount-display {
    font-size: 28px;
  }
  
  .modal-content {
    width: 90%;
    padding: 15px;
  }
  
  .full-qr-image {
    width: 100%;
  }
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