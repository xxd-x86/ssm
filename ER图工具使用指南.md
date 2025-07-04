# ER图设计工具使用指南

## 功能简介

ER图设计工具是一个用于创建和可视化实体关系(Entity Relationship)图的工具。它可以帮助您设计数据库模型，直观地展示实体之间的关系，支持标准的ER图表示法。

## 技术实现

此工具基于以下技术构建：

- **Vue.js** - 提供响应式UI界面
- **Mermaid.js** - 用于渲染ER图表
- **自定义解析器** - 将简化的ER语法转换为Mermaid兼容的格式

## 使用方法

### 基本操作

1. 在左侧文本编辑器中输入ER模型定义
2. 点击"生成图表"按钮显示ER图
3. 可以使用"清空"按钮重置编辑器
4. "插入示例"按钮提供一个样例ER模型作为参考

### 语法规则

#### 两种语法选择

工具支持两种输入语法：

1. **自定义语法** - 使用 `[实体名称]` 格式（见下文）
2. **原生Mermaid语法** - 以 `erDiagram` 开头（推荐）

> 注意：如果您的文本以 `erDiagram` 开头，系统将直接将其当作Mermaid语法处理

#### 实体定义（自定义语法）

```
[实体名称] {
  属性名 数据类型 [PK|FK]
}
```

其中：
- `实体名称` - 实体的名称，如"学生"、"课程"等
- `属性名` - 实体的属性名称，如"学号"、"姓名"等
- `数据类型` - 属性的数据类型，如"varchar"、"int"等
- `PK` - 表示该属性是主键(Primary Key)
- `FK` - 表示该属性是外键(Foreign Key)

#### 关系定义（自定义语法）

```
[实体1] 基数性 -- 关系名称 -- 基数性 [实体2]
```

其中：
- `实体1`、`实体2` - 参与关系的两个实体名称
- `关系名称` - 描述两个实体间的关系，如"属于"、"拥有"等
- `基数性` - 实体间的基数关系，支持以下符号：
  - `|o` - 表示"零或一"(0..1)
  - `||` - 表示"恰好一个"(1)
  - `}o` - 表示"零或多个"(0..*)
  - `}|` - 表示"一或多个"(1..*)

#### 原生Mermaid语法（推荐）

```
erDiagram
  "实体1" {
    数据类型 "属性名1" PK
    数据类型 "属性名2"
  }
  "实体1" 基数性1--基数性2 "实体2" : "关系描述"
```

其中：
- 实体名称和属性名都**必须用双引号**括起来，特别是包含中文或特殊字符时
- 基数性使用：o|（零或一）、||（恰好一个）、o{（零或多个）、|{（一或多个）

#### 注释

使用`//`开头的行会被视为注释，不会影响ER图的生成。

### 中文支持说明

为了确保中文实体名和属性名能正确显示，请遵循以下原则：

1. **使用原生Mermaid语法** - 以 `erDiagram` 开头（强烈推荐）
2. **实体和属性名必须加引号** - 所有中文必须使用双引号 `"` 括起来
3. **关系描述必须加引号** - 如 `"属于"`、`"拥有"`

正确示例：
```
erDiagram
  "学生" {
    varchar "学号" PK
    varchar "姓名"
  }
  "学生" o{--|| "班级" : "属于"
```

### 示例

以下是一个学生-班级-教师关系模型的例子（使用原生Mermaid语法）：

```
erDiagram
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
  "班级" |{--|| "教师" : "由"
```

这个例子定义了：
1. 三个实体：学生、班级和教师
2. 两种关系：
   - 学生属于班级（多对一关系）
   - 班级由教师负责（多对一关系）

### 基数性说明

ER图中使用的基数性符号对应的含义：

| 符号 | 描述 | 含义 |
|------|------|------|
| `o|` | 零或一 | 关系的这一端可以没有关联实体，或者最多关联一个 |
| `||` | 恰好一个 | 关系的这一端必须有且只有一个关联实体 |
| `o{` | 零或多个 | 关系的这一端可以没有关联实体，或者有多个 |
| `|{` | 一或多个 | 关系的这一端至少要有一个关联实体，可以有多个 |

## 高级功能

### 导出图表

生成ER图后，您可以：
1. 点击"下载SVG"按钮将图表保存为SVG矢量图
2. 点击"复制代码"按钮复制生成的Mermaid代码，可用于其他支持Mermaid的平台

### 进阶使用技巧

1. **复杂关系建模** - 对于多对多关系，通常引入中间实体来表示
2. **关系属性** - 在关系有自己的属性时，将关系提升为实体
3. **继承关系** - 可以使用一对一关系和注释来模拟继承关系

## 常见问题

1. **图表无法生成** - 检查语法是否正确，特别是确保所有中文实体和属性名称都用双引号括起来
2. **中文显示问题** - 必须使用原生Mermaid语法且所有中文必须用双引号括起
3. **实体间关系显示不正确** - 确认基数性符号使用正确
4. **大型模型布局问题** - 尝试将相关实体分组，减少交叉关系

## 后续开发计划

1. 支持更多ER图元素，如弱实体集
2. 导出到SQL DDL语句功能
3. 从现有数据库导入ER模型
4. 协作编辑功能 