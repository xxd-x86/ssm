<template>
  <div class="er-diagram">
    <h2>ER图设计工具</h2>
    
    <div class="tool-container">
      <div class="editor-section">
        <div class="tool-header sticky-header">
          <h3>ER模型定义</h3>
          <div class="tool-actions">
            <button @click="generateDiagram" class="action-btn primary">生成图表</button>
            <button @click="clearEditor" class="action-btn">清空</button>
            <button @click="insertExample" class="action-btn">插入示例</button>
            <button @click="testSimpleDiagram" class="action-btn">测试简单图表</button>
            <button @click="testScreenshotSyntax" class="action-btn">测试截图语法</button>
            <button @click="testDifferentSyntax" class="action-btn">测试不同语法</button>
            <button @click="testBasicSyntax" class="action-btn">测试基本语法</button>
            <button @click="showPreRendered" class="action-btn success">显示预渲染图</button>
          </div>
        </div>
        
        <div class="editor-container">
          <textarea 
            v-model="erDefinition" 
            class="er-editor"
            placeholder="在这里定义您的ER模型..."
            ref="codeEditor"
          ></textarea>
        </div>
        
        <div class="help-section">
          <div class="help-header" @click="toggleHelp">
            <span>ER图语法帮助</span>
            <span class="toggle-icon">{{ helpExpanded ? '▲' : '▼' }}</span>
          </div>
          <div class="help-content" v-if="helpExpanded">
            <h4>语法规则：</h4>
            <pre>
erDiagram
  "实体名" {
    数据类型 "属性名" PK
    数据类型 "属性名"
  }
  "实体1" o{--|| "实体2" : "关系"
            </pre>
            
            <h4>注意事项：</h4>
            <p>所有中文必须用双引号括起来，包括实体名、属性名和关系描述。</p>
            
            <h4>示例：</h4>
            <pre>
erDiagram
  "学生" {
    varchar "学号" PK
    varchar "姓名"
    int "年龄"
  }
  "学生" o{--|| "班级" : "属于"
            </pre>
          </div>
        </div>
      </div>
      
      <div class="diagram-section">
        <div class="tool-header">
          <h3>ER图预览</h3>
          <div class="tool-actions">
            <button @click="downloadSVG" class="action-btn" :disabled="!diagramRendered">下载SVG</button>
            <button @click="copyMermaidCode" class="action-btn" :disabled="!diagramRendered">复制代码</button>
          </div>
        </div>
        
        <div class="diagram-container">
          <div id="er-diagram-output" class="diagram-output"></div>
          
          <div v-if="loading" class="diagram-overlay loading-overlay">
            <div class="message">生成图表中...</div>
          </div>
          
          <div v-else-if="error" class="diagram-overlay error-overlay">
            <div class="message">{{ error }}</div>
          </div>
          
          <div v-else-if="!diagramRendered" class="diagram-overlay empty-overlay">
            <div class="message">请在左侧定义ER模型，然后点击"生成图表"按钮</div>
          </div>
          
          <!-- 预渲染的ER图示例 -->
          <div v-if="showPreRenderedDiagram" class="pre-rendered-diagram">
            <svg width="100%" height="100%" viewBox="0 0 800 400" xmlns="http://www.w3.org/2000/svg">
              <!-- 学生实体 -->
              <rect x="50" y="50" width="200" height="150" fill="#e8f4ea" stroke="#333" rx="5" />
              <text x="150" y="80" text-anchor="middle" font-weight="bold">学生</text>
              <line x1="50" y1="90" x2="250" y2="90" stroke="#333" />
              <text x="60" y="120">varchar 学号 PK</text>
              <text x="60" y="150">varchar 姓名</text>
              <text x="60" y="180">int 年龄</text>
              
              <!-- 班级实体 -->
              <rect x="350" y="50" width="200" height="150" fill="#e8f4ea" stroke="#333" rx="5" />
              <text x="450" y="80" text-anchor="middle" font-weight="bold">班级</text>
              <line x1="350" y1="90" x2="550" y2="90" stroke="#333" />
              <text x="360" y="120">int 班级编号 PK</text>
              <text x="360" y="150">varchar 班级名称</text>
              <text x="360" y="180">int 教师编号 FK</text>
              
              <!-- 教师实体 -->
              <rect x="650" y="50" width="200" height="150" fill="#e8f4ea" stroke="#333" rx="5" />
              <text x="750" y="80" text-anchor="middle" font-weight="bold">教师</text>
              <line x1="650" y1="90" x2="850" y2="90" stroke="#333" />
              <text x="660" y="120">int 教师编号 PK</text>
              <text x="660" y="150">varchar 姓名</text>
              <text x="660" y="180">varchar 职称</text>
              
              <!-- 学生-班级关系 -->
              <line x1="250" y1="125" x2="350" y2="125" stroke="#333" stroke-width="2" />
              <text x="300" y="115" text-anchor="middle" font-size="14">属于</text>
              <text x="260" y="145" text-anchor="start" font-size="12">多</text>
              <text x="340" y="145" text-anchor="end" font-size="12">一</text>
              
              <!-- 班级-教师关系 -->
              <line x1="550" y1="125" x2="650" y2="125" stroke="#333" stroke-width="2" />
              <text x="600" y="115" text-anchor="middle" font-size="14">由</text>
              <text x="560" y="145" text-anchor="start" font-size="12">多</text>
              <text x="640" y="145" text-anchor="end" font-size="12">一</text>
            </svg>
          </div>
        </div>
        
        <div v-if="error" class="debug-info">
          <p>错误详情: {{ errorDetail }}</p>
          <pre v-if="mermaidCode">{{ mermaidCode }}</pre>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ERDiagram',
  data() {
    return {
      erDefinition: '',
      mermaidCode: '',
      diagramRendered: false,
      loading: false,
      error: null,
      errorDetail: '',
      helpExpanded: false,
      showPreRenderedDiagram: false
    }
  },
  mounted() {
    console.log('挂载组件，检查输出容器:', document.getElementById('er-diagram-output'));
    
    // 确保mermaid库加载
    this.loadMermaidLibrary();
    
    // 添加示例
    this.insertExample();
  },
  methods: {
    loadMermaidLibrary() {
      if (typeof window.mermaid === 'undefined') {
        console.log('开始加载Mermaid库...');
        // 尝试多个CDN源和不同版本
        const cdnSources = [
          'https://cdn.jsdelivr.net/npm/mermaid@9.4.3/dist/mermaid.min.js', // 尝试更稳定的9.4.3版本
          'https://unpkg.com/mermaid@9.4.3/dist/mermaid.min.js',
          'https://cdn.jsdelivr.net/npm/mermaid@8.13.10/dist/mermaid.min.js', // 尝试8.14.0之前的版本
          'https://unpkg.com/mermaid@8.13.10/dist/mermaid.min.js'
        ];
        
        const loadScript = (index) => {
          if (index >= cdnSources.length) {
            console.error('所有CDN源都加载失败');
            this.error = "无法加载图表库，请检查网络连接";
            return;
          }
          
          const mermaidScript = document.createElement('script');
          mermaidScript.src = cdnSources[index];
          mermaidScript.onload = () => {
            console.log('Mermaid库加载成功，来源:', cdnSources[index]);
            this.initializeMermaid();
          };
          mermaidScript.onerror = () => {
            console.error('Mermaid加载失败，尝试下一个源');
            loadScript(index + 1);
          };
          document.head.appendChild(mermaidScript);
        };
        
        loadScript(0);
      } else {
        console.log('Mermaid库已存在');
        this.initializeMermaid();
      }
    },
    
    initializeMermaid() {
      if (window.mermaid) {
        try {
          window.mermaid.initialize({
            startOnLoad: false,
            securityLevel: 'loose',
            theme: 'default',
            er: {
              diagramPadding: 20,
              layoutDirection: 'LR',
              minEntityWidth: 100,
              minEntityHeight: 75,
              entityPadding: 15,
              stroke: 'gray',
              fill: 'honeydew'
            }
          });
          console.log('Mermaid已初始化，版本:', window.mermaid.version());
          
          // 加载成功后自动渲染示例
          setTimeout(() => {
            this.testSimpleDiagram();
          }, 500);
        } catch (error) {
          console.error('Mermaid初始化失败:', error);
        }
      } else {
        console.error('Mermaid库不存在，无法初始化');
      }
    },
    
    async generateDiagram() {
      try {
        if (!this.erDefinition.trim()) {
          this.error = "请先输入ER模型定义";
          return;
        }
        
        if (typeof window.mermaid === 'undefined') {
          console.error("Mermaid库未加载");
          this.error = "Mermaid库尚未加载，请稍候再试";
          this.loadMermaidLibrary();
          return;
        }
        
        this.loading = true;
        this.error = null;
        this.errorDetail = '';
        
        console.log("开始处理ER图定义...");
        
        // 直接使用erDiagram语法
        if (this.erDefinition.trim().startsWith('erDiagram')) {
          this.mermaidCode = this.erDefinition.trim();
          console.log("使用原生Mermaid语法:", this.mermaidCode);
        } else {
          // 解析ER定义并生成mermaid代码
          this.mermaidCode = this.parseERDefinition(this.erDefinition);
          console.log("解析自定义语法得到Mermaid代码:", this.mermaidCode);
        }
        
        // 获取输出容器
        const outputDiv = document.getElementById('er-diagram-output');
        console.log('检索输出容器:', outputDiv);
        
        if (!outputDiv) {
          throw new Error("找不到输出容器元素，这是个严重错误");
        }
        
        // 清空输出容器
        outputDiv.innerHTML = '';
        
        // 使用新的渲染方法
        this.renderMermaidDiagram(this.mermaidCode, outputDiv);
      } catch (e) {
        console.error("处理错误:", e);
        this.error = `处理失败: ${e.message}`;
        this.errorDetail = e.stack;
        this.loading = false;
      }
    },
    
    renderMermaidDiagram(code, container) {
      try {
        console.log('尝试渲染Mermaid图表...');
        
        // 清空容器
        container.innerHTML = '';
        
        // 生成唯一ID
        const id = `mermaid-${Date.now()}`;
        const tempDiv = document.createElement('div');
        tempDiv.id = id;
        tempDiv.className = 'mermaid';
        tempDiv.textContent = code;
        container.appendChild(tempDiv);
        
        console.log('创建Mermaid容器:', id);
        console.log('渲染代码:', code);
        
        // 确保mermaid库已加载
        if (!window.mermaid) {
          this.error = "Mermaid库尚未加载，请刷新页面重试";
          this.loading = false;
          return;
        }
        
        // 使用最新的API渲染
        try {
          if (window.mermaid.version && window.mermaid.version().startsWith('8.')) {
            // 8.x版本API
            console.log('使用Mermaid 8.x API渲染');
            window.mermaid.init(undefined, tempDiv);
            this.diagramRendered = true;
            this.loading = false;
          } else if (window.mermaid.run) {
            // 10.x版本API
            console.log('使用Mermaid 10.x API渲染');
            window.mermaid.run({ nodes: [tempDiv] })
              .then(() => {
                console.log('渲染成功');
                this.diagramRendered = true;
                this.loading = false;
              })
              .catch(error => {
                console.error('渲染失败:', error);
                this.error = `图表渲染失败: ${error.message || '语法错误'}`;
                this.errorDetail = JSON.stringify(error, null, 2);
                this.loading = false;
              });
          } else {
            // 降级到最通用的API
            console.log('使用通用API渲染');
            window.mermaid.init(undefined, document.querySelectorAll('.mermaid'));
            this.diagramRendered = true;
            this.loading = false;
          }
        } catch (renderError) {
          console.error('渲染过程中出错:', renderError);
          this.error = `渲染过程中出错: ${renderError.message || '未知错误'}`;
          this.errorDetail = JSON.stringify(renderError, null, 2);
          this.loading = false;
        }
      } catch (error) {
        console.error('渲染错误:', error);
        this.error = `图表渲染失败: ${error.message || '语法错误'}`;
        this.errorDetail = JSON.stringify(error, null, 2);
        this.loading = false;
      }
    },
    
    testSimpleDiagram() {
      this.erDefinition = `erDiagram
    USER {
        string id PK
        string name
        string email
    }
    PROFILE {
        string id PK
        string user_id FK
        string avatar
    }
    USER ||--o| PROFILE : has`;
      this.generateDiagram();
    },
    
    testScreenshotCode() {
      this.erDefinition = `erDiagram
  "学生" {
    varchar "学号" PK
    varchar "姓名"
    int "年龄"
    int "班级编号" FK
  }
  "班级" {
    int "班级编号" PK
    varchar "班级名称"
    int "教师编号" FK
  }
  "教师" {
    int "教师编号" PK
    varchar "姓名"
    varchar "职称"
  }
  "学生" o{--|| "班级" : "属于"
  "班级" |{--|| "教师" : "由"`;
      this.generateDiagram();
    },
    
    parseERDefinition(definition) {
      try {
        const lines = definition.split('\n');
        let mermaidCode = 'erDiagram\n';
        
        const entities = {};
        const relationships = [];
        
        let currentEntity = null;
        
        for (let i = 0; i < lines.length; i++) {
          const line = lines[i].trim();
          
          if (!line || line.startsWith('//')) continue;
          
          const entityMatch = line.match(/\[(.+?)\]\s*\{/);
          if (entityMatch) {
            currentEntity = entityMatch[1];
            entities[currentEntity] = [];
            continue;
          }
          
          if (line === '}' && currentEntity) {
            currentEntity = null;
            continue;
          }
          
          if (currentEntity && !line.includes('--')) {
            const attributeMatch = line.match(/\s*(.+?)\s+(.+?)(?:\s+(PK|FK))?\s*$/);
            if (attributeMatch) {
              entities[currentEntity].push({
                name: attributeMatch[1],
                type: attributeMatch[2],
                key: attributeMatch[3] || ''
              });
            }
          }
          
          const relationshipMatch = line.match(/\[(.+?)\]\s*(.+?)--(.+?)--(.+?)\s*\[(.+?)\]/);
          if (relationshipMatch) {
            relationships.push({
              entity1: relationshipMatch[1],
              cardinality1: this.translateCardinality(relationshipMatch[2]),
              label: relationshipMatch[3],
              cardinality2: this.translateCardinality(relationshipMatch[4]),
              entity2: relationshipMatch[5]
            });
          }
        }
        
        for (const [entityName, attributes] of Object.entries(entities)) {
          mermaidCode += `  "${entityName}" {\n`;
          
          for (const attr of attributes) {
            let keyType = '';
            if (attr.key === 'PK') keyType = ' PK';
            else if (attr.key === 'FK') keyType = ' FK';
            
            mermaidCode += `    ${attr.type} "${attr.name}"${keyType}\n`;
          }
          
          mermaidCode += '  }\n';
        }
        
        for (const rel of relationships) {
          mermaidCode += `  "${rel.entity1}" ${rel.cardinality1}--${rel.cardinality2} "${rel.entity2}" : "${rel.label}"\n`;
        }
        
        return mermaidCode;
      } catch (error) {
        throw new Error(`解析ER定义时出错: ${error.message}`);
      }
    },
    
    translateCardinality(cardinality) {
      const cardMap = {
        '|o': 'o|',
        '||': '||',
        '}o': 'o{',
        '}|': '|{'
      };
      
      return cardMap[cardinality] || '||';
    },
    
    clearEditor() {
      if (confirm('确定要清空编辑器吗？')) {
        this.erDefinition = '';
        this.diagramRendered = false;
        
        const outputDiv = document.getElementById('er-diagram-output');
        if (outputDiv) outputDiv.innerHTML = '';
        
        this.error = null;
        this.errorDetail = '';
      }
    },
    
    insertExample() {
      this.erDefinition = `erDiagram
  "学生" {
    varchar "学号" PK
    varchar "姓名"
    int "年龄"
    int "班级编号" FK
  }
  "班级" {
    int "班级编号" PK
    varchar "班级名称"
    int "教师编号" FK
  }
  "教师" {
    int "教师编号" PK
    varchar "姓名"
    varchar "职称"
  }
  "学生" o{--|| "班级" : "属于"
  "班级" |{--|| "教师" : "由"`;
    },
    
    downloadSVG() {
      if (!this.diagramRendered) return;
      
      const outputDiv = document.getElementById('er-diagram-output');
      const svgElement = outputDiv ? outputDiv.querySelector('svg') : null;
      if (!svgElement) {
        this.error = "找不到SVG元素，无法下载";
        return;
      }
      
      const svgData = new XMLSerializer().serializeToString(svgElement);
      const blob = new Blob([svgData], { type: 'image/svg+xml' });
      const url = URL.createObjectURL(blob);
      
      const link = document.createElement('a');
      link.href = url;
      link.download = 'er_diagram.svg';
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
      URL.revokeObjectURL(url);
    },
    
    copyMermaidCode() {
      if (!this.mermaidCode) return;
      
      navigator.clipboard.writeText(this.mermaidCode)
        .then(() => {
          alert('Mermaid代码已复制到剪贴板');
        })
        .catch(err => {
          console.error('复制失败:', err);
          this.error = '复制代码失败';
        });
    },
    
    toggleHelp() {
      this.helpExpanded = !this.helpExpanded;
    },
    
    testExactScreenshot() {
      // 使用截图中的确切代码，完全匹配空格和缩进
      const screenshotCode = `erDiagram
  "学生" {
    varchar "学号" PK
    varchar "姓名"
    int "年龄"
    int "班级编号" FK
  }
  "班级" {
    int "班级编号" PK
    varchar "班级名称"
    int "教师编号" FK
  }
  "教师" {
    int "教师编号" PK
    varchar "姓名"
    varchar "职称"
  }
  "学生" o{--|| "班级" : "属于"
  "班级" |{--|| "教师" : "由"`;
      
      this.erDefinition = screenshotCode;
      this.generateDiagram();
    },
    
    testWithExactSyntax() {
      // 完全匹配截图中的语法
      const exactSyntax = `erDiagram
  "学生" {
    varchar "学号" PK
    varchar "姓名"
    int "年龄"
    int "班级编号" FK
  }
  "班级" {
    int "班级编号" PK
    varchar "班级名称"
    int "教师编号" FK
  }
  "教师" {
    int "教师编号" PK
    varchar "姓名"
    varchar "职称"
  }
  "学生" o{--|| "班级" : "属于"
  "班级" |{--|| "教师" : "由"`;
      
      this.erDefinition = exactSyntax;
      this.generateDiagram();
    },
    
    testAlternativeSyntax() {
      // 尝试使用截图中的关系语法
      const alternativeSyntax = `erDiagram
  "学生" {
    varchar "学号" PK
    varchar "姓名"
    int "年龄"
    int "班级编号" FK
  }
  "班级" {
    int "班级编号" PK
    varchar "班级名称"
    int "教师编号" FK
  }
  "教师" {
    int "教师编号" PK
    varchar "姓名"
    varchar "职称"
  }
  "学生" o{--|| "班级" : "属于"
  "班级" |{--|| "教师" : "由"`;
      
      this.erDefinition = alternativeSyntax;
      this.generateDiagram();
    },
    
    testScreenshotSyntax() {
      // 完全使用截图中的语法格式
      const screenshotSyntax = `erDiagram
  "学生" {
    varchar "学号" PK
    varchar "姓名"
    int "年龄"
    int "班级编号" FK
  }
  "班级" {
    int "班级编号" PK
    varchar "班级名称"
    int "教师编号" FK
  }
  "教师" {
    int "教师编号" PK
    varchar "姓名"
    varchar "职称"
  }
  "学生" o{--|| "班级" : "属于"
  "班级" |{--|| "教师" : "由"`;
      
      this.erDefinition = screenshotSyntax;
      
      // 直接尝试使用mermaid渲染
      setTimeout(() => {
        try {
          const outputDiv = document.getElementById('er-diagram-output');
          if (outputDiv) {
            outputDiv.innerHTML = '';
            const tempDiv = document.createElement('div');
            tempDiv.className = 'mermaid';
            tempDiv.textContent = screenshotSyntax;
            outputDiv.appendChild(tempDiv);
            
            if (window.mermaid) {
              window.mermaid.init(undefined, tempDiv);
              this.diagramRendered = true;
              this.loading = false;
              this.error = null;
            }
          }
        } catch (error) {
          console.error('直接渲染失败:', error);
          this.error = `直接渲染失败: ${error.message}`;
        }
      }, 100);
    },
    
    testDifferentSyntax() {
      // 使用更简单的中文语法
      const differentSyntax = `erDiagram
    用户 {
        string 用户ID PK
        string 姓名
        string 邮箱
    }
    用户 ||--o{ 订单 : 拥有
    订单 {
        string 订单ID PK
        string 用户ID FK
        number 金额
    }`;
      
      this.erDefinition = differentSyntax;
      this.generateDiagram();
    },
    
    testBasicSyntax() {
      // 使用最基本的ER图语法
      const basicSyntax = `erDiagram
    A ||--o{ B : has`;
      
      this.erDefinition = basicSyntax;
      this.generateDiagram();
    },
    
    showPreRendered() {
      // 显示预渲染的ER图
      this.showPreRenderedDiagram = true;
      this.diagramRendered = true;
      this.error = null;
      this.loading = false;
      
      // 设置示例代码
      this.erDefinition = `erDiagram
  "学生" {
    varchar "学号" PK
    varchar "姓名"
    int "年龄"
    int "班级编号" FK
  }
  "班级" {
    int "班级编号" PK
    varchar "班级名称"
    int "教师编号" FK
  }
  "教师" {
    int "教师编号" PK
    varchar "姓名"
    varchar "职称"
  }
  "学生" o{--|| "班级" : "属于"
  "班级" |{--|| "教师" : "由"`;
    }
  }
}
</script>

<style scoped>
.er-diagram {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 10px;
}

h2 {
  text-align: center;
  margin-bottom: 24px;
  color: #333;
}

.tool-container {
  display: flex;
  gap: 20px;
  min-height: 500px;
}

.editor-section, .diagram-section {
  flex: 1;
  display: flex;
  flex-direction: column;
  background: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.08);
  overflow: hidden;
  height: calc(100vh - 250px);
  max-height: 800px;
}

.tool-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px 20px;
  background-color: #f5f7fa;
  border-bottom: 1px solid #e0e6ed;
}

.sticky-header {
  position: sticky;
  top: 0;
  z-index: 10;
}

.tool-header h3 {
  margin: 0;
  font-size: 16px;
  color: #333;
}

.tool-actions {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

.action-btn {
  padding: 6px 12px;
  border-radius: 4px;
  font-size: 14px;
  cursor: pointer;
  border: 1px solid #dcdfe6;
  background-color: white;
  color: #606266;
  transition: all 0.3s;
}

.action-btn:hover {
  border-color: #c6e2ff;
  background-color: #ecf5ff;
  color: #409eff;
}

.action-btn.primary {
  background-color: #409eff;
  border-color: #409eff;
  color: white;
}

.action-btn.primary:hover {
  background-color: #66b1ff;
  border-color: #66b1ff;
}

.action-btn.success {
  background-color: #67c23a;
  border-color: #67c23a;
  color: white;
}

.action-btn.success:hover {
  background-color: #85ce61;
  border-color: #85ce61;
}

.action-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.editor-container {
  flex: 1;
  padding: 15px;
  position: relative;
  overflow-y: auto;
}

.er-editor {
  width: 100%;
  height: 100%;
  min-height: 300px;
  border: 1px solid #dcdfe6;
  border-radius: 4px;
  padding: 10px;
  font-family: 'Courier New', monospace;
  font-size: 14px;
  line-height: 1.5;
  resize: none;
  overflow-y: auto;
}

.er-editor:focus {
  outline: none;
  border-color: #409eff;
}

.diagram-container {
  flex: 1;
  padding: 15px;
  overflow: auto;
  position: relative;
  background-color: #f9f9f9;
}

.diagram-output {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.diagram-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 5;
}

.loading-overlay {
  background-color: rgba(255, 255, 255, 0.8);
}

.error-overlay {
  background-color: rgba(255, 240, 240, 0.9);
}

.empty-overlay {
  background-color: rgba(249, 249, 249, 0.8);
}

.diagram-overlay .message {
  padding: 20px;
  border-radius: 4px;
  text-align: center;
  color: #606266;
}

.error-overlay .message {
  color: #f56c6c;
}

.debug-info {
  padding: 10px 15px;
  background-color: #ffeeee;
  border-top: 1px solid #e0e6ed;
  font-size: 12px;
  color: #f56c6c;
  max-height: 150px;
  overflow-y: auto;
}

.debug-info pre {
  background-color: #f8f8f8;
  padding: 5px;
  border-radius: 4px;
  overflow-x: auto;
  margin-top: 5px;
  font-size: 11px;
  line-height: 1.4;
}

.help-section {
  border-top: 1px solid #e0e6ed;
  background-color: #f5f7fa;
}

.help-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 15px;
  cursor: pointer;
  font-size: 14px;
  color: #606266;
}

.help-content {
  padding: 0 15px 15px;
  font-size: 13px;
  overflow-y: auto;
  max-height: 200px;
  border-top: 1px solid #e0e6ed;
}

.help-content h4 {
  margin: 15px 0 8px;
  font-size: 14px;
  color: #333;
}

.help-content pre {
  background-color: #f0f0f0;
  padding: 8px;
  border-radius: 4px;
  overflow-x: auto;
  margin: 5px 0;
  font-family: 'Courier New', monospace;
  font-size: 12px;
  line-height: 1.5;
}

.pre-rendered-diagram {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 10;
}

.pre-rendered-diagram svg {
  max-width: 100%;
  max-height: 100%;
}

@media (max-width: 1024px) {
  .tool-container {
    flex-direction: column;
  }
  
  .editor-section, .diagram-section {
    width: 100%;
    height: auto;
    max-height: none;
  }
  
  .editor-section {
    height: 500px;
  }
  
  .diagram-section {
    height: 500px;
  }
}

@media (max-width: 768px) {
  .tool-header {
    flex-direction: column;
    gap: 10px;
    align-items: flex-start;
  }
  
  .tool-actions {
    width: 100%;
    justify-content: space-between;
  }
}
</style> 