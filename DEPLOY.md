# 部署指南

## 1. 创建GitHub仓库

1. 登录您的GitHub账号
2. 创建一个新的仓库，如"money-collection"
3. 不要初始化仓库（不添加README、.gitignore或许可证）

## 2. 将项目推送到GitHub

```bash
# 初始化Git仓库
git init

# 添加所有文件
git add .

# 提交更改
git commit -m "初始提交"

# 添加远程仓库
git remote add origin https://github.com/您的用户名/money-collection.git

# 推送到main分支
git branch -M main
git push -u origin main
```

## 3. 设置GitHub Pages

1. GitHub会自动运行部署工作流
2. 工作流完成后，转到仓库设置 -> Pages
3. 确认源设置为"gh-pages" /(root)
4. 自定义域名部分输入：xiaxudong.cn
5. 保存设置

## 4. 配置域名DNS

在您的域名管理面板中（如图片所示的xiaxudong.cn管理面板）：

1. 添加以下DNS记录：

   - 类型: A
   - 主机记录: @
   - 值: 185.199.108.153
   - 值: 185.199.109.153
   - 值: 185.199.110.153
   - 值: 185.199.111.153

   或者

   - 类型: CNAME
   - 主机记录: @
   - 值: 您的用户名.github.io

2. 保存DNS设置

## 5. 验证部署

1. 等待DNS生效（可能需要几分钟到24小时）
2. 访问xiaxudong.cn验证网站是否正常工作

## 注意事项

- 如果您使用的是HTTPS，GitHub Pages会自动为您的站点配置SSL证书
- 如果需要对接真实的微信和支付宝支付API，需要进一步开发后端服务 