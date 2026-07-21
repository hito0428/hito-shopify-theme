# Shopify Theme Access Token 创建与使用教程

> 本文介绍如何创建 Theme Access Token，并使用它绕过浏览器授权，直接通过 CLI 进行主题开发和部署。

---

## 什么是 Theme Access Token

Theme Access Token 是一个**永久有效的 API 密钥**，用于：

- ✅ 跳过浏览器 OAuth 授权流程
- ✅ 通过 CLI 直接拉取/推送主题代码
- ✅ 在 CI/CD 流程中自动化部署
- ✅ 避免频繁的登录过期问题

---

## 步骤 1：创建 Theme Access Token

### 1.1 登录 Shopify 后台

打开你的店铺后台：

```
https://你的店铺.myshopify.com/admin
```

### 1.2 进入主题管理

1. 左侧导航 → **在线商店** → **主题**
2. 找到你要开发的主题（通常是 Dawn 主题）
3. 点击右侧 **三个点 (...)** → **管理权限**

### 1.3 创建密码

1. 在「Theme Access」页面，点击 **「创建密码」**
2. 填写表单：
   - **开发人员姓名**：任意填写（如你的名字）
   - **开发人员邮箱**：你的邮箱（用于接收安全链接）
   - **语言**：保持默认即可
3. 点击 **「创建密码」**

### 1.4 复制 Token

创建成功后，系统会显示一个密码（格式：`shptka_xxxxxxxxxxxxxxxxxxxxxxxxxxxx`）

> ⚠️ **重要**：这个密码**只会显示一次**，关闭后无法再次查看！
> 请立即复制并保存到安全的地方。

---

## 步骤 2：配置 shopify.theme.toml

在主题项目根目录创建或更新 `shopify.theme.toml` 文件：

```toml
[environments.development]
store = "你的店铺.myshopify.com"
password = "shptka_xxxxxxxxxxxxxxxxxxxxxxxxxxxx"
theme = "主题ID"
```

### 如何获取主题 ID

1. 在主题管理页面，点击主题的 **「编辑代码」**
2. 浏览器地址栏会显示类似：
   ```
   https://xxx.myshopify.com/admin/themes/1234567890123/editor
   ```
3. 数字部分 `1234567890123` 就是主题 ID

---

## 步骤 3：使用 CLI 命令

### 开发模式（本地预览 + 热更新）

```bash
shopify theme dev --environment development
```

### 拉取线上代码

```bash
shopify theme pull --environment development
```

### 推送代码到线上

```bash
shopify theme push --environment development
```

### 查看命令帮助

```bash
shopify theme --help
```

---

## 步骤 4：验证配置是否正确

运行开发命令后，终端会显示：

```
╭─ info ───────────────────────────────────────────────────────────────────────╮
│                                                                              │
│  Using applicable flags from development environment:                        │
│                                                                              │
│    • password: ********5a49                                                  │
│    • store: your-store.myshopify.com                                         │
│    • theme: 1234567890123                                                    │
│                                                                              │
╰──────────────────────────────────────────────────────────────────────────────╯
```

如果看到以上信息，说明配置成功！

---

## 常见问题

### Q1：登录时出现 "User verification code"

**原因**：CLI 默认使用 OAuth 授权，需要浏览器验证。

**解决**：确保 `shopify.theme.toml` 中配置了 `password` 字段，CLI 会自动使用 Token 登录。

### Q2：Token 过期了怎么办

**原因**：Token 本身不过期，但可能因为以下原因失效：
- 在后台手动删除了该密码
- 店铺被暂停或删除

**解决**：回到步骤 1，重新创建一个新密码。

### Q3：邮件收不到安全链接

**解决**：
1. 检查垃圾邮件文件夹
2. 如果超过 7 天过期，重新创建密码即可

### Q4：CLI 提示 "Environment file not found"

**原因**：找不到 `shopify.theme.toml` 文件。

**解决**：确保文件在主题项目根目录，且文件名正确。

---

## 完整配置示例

```toml
# shopify.theme.toml

# 开发环境
[environments.development]
store = "hito-test-store.myshopify.com"
password = "shptka_6725767b89c2ad7cc54c775c55a05a49"
theme = "150624174150"

# 生产环境（可选）
[environments.production]
store = "hito-test-store.myshopify.com"
password = "shptka_另一个token"
theme = "生产主题ID"
```

### 使用多环境

```bash
# 开发环境
shopify theme dev --environment development

# 生产环境
shopify theme push --environment production
```

---

## 安全注意事项

1. **不要提交 Token 到版本库**：在 `.gitignore` 中添加 `shopify.theme.toml`
2. **定期轮换 Token**：在后台可以删除旧密码，创建新密码
3. **限制访问权限**：Theme Access 只授予主题读写权限，不会泄露店铺其他数据

---

## 参考链接

- [Shopify Theme Access 官方文档](https://shopify.dev/docs/themes/tools/theme-access)
- [Shopify CLI 主题命令](https://shopify.dev/docs/themes/tools/cli/theme)
- [Shopify CLI 配置文件](https://shopify.dev/docs/themes/tools/cli/configuration)
