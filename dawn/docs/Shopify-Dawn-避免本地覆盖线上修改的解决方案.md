# 避免本地覆盖线上修改的解决方案

> 本文介绍如何在 Shopify 主题开发中避免本地代码覆盖线上修改，确保代码变更的安全管理。

---

## 问题背景

### 场景描述

你在 Shopify 后台「编辑代码」界面修改了主题并保存，但随后在本地运行 `shopify theme dev` 或 `shopify theme push` 时，本地的旧代码会覆盖掉线上的新代码。

### 根本原因

`shopify theme dev` 默认会**实时同步**本地文件到线上主题，且是「本地覆盖线上」的模式。

---

## 核心原则：Git 作为单一数据源

### 工作流程

```
线上修改 → 拉取到本地 → Git 合并 → 本地开发 → 推送到线上
```

### 关键规则

1. **永远先拉取再开发**：开始工作前，确保本地代码是最新的
2. **Git 管理所有变更**：无论是本地还是线上的修改，都通过 Git 追踪
3. **明确的部署流程**：开发完成后，通过 `shopify theme push` 部署到线上

---

## 方案一：开发主题 + 正式主题分离（推荐）

### 方案概述

创建两个独立的主题：
- **开发主题**：用于本地开发和测试
- **正式主题**：线上实际使用的主题

### 配置步骤

#### 1. 创建开发主题

```bash
# 创建一个新的开发主题
shopify theme init --name "Development Theme"

# 获取开发主题 ID（后台查看）
# 假设开发主题 ID：1234567890123
```

#### 2. 配置多环境

编辑 `shopify.theme.toml`：

```toml
[environments.development]
store = "hito-test-store.myshopify.com"
password = "shptka_xxx"
theme = "1234567890123"  # 开发主题 ID

[environments.production]
store = "hito-test-store.myshopify.com"
password = "shptka_xxx"
theme = "150624174150"  # 正式主题 ID
```

#### 3. 使用流程

```bash
# 在开发主题上开发（不影响线上）
shopify theme dev --environment development

# 测试完成后，部署到正式主题
shopify theme push --environment production
```

---

## 方案二：Git 分支管理

### 方案概述

使用 Git 分支管理不同的功能开发，确保代码的版本控制和协作安全。

### 工作流程

#### 1. 开始开发前，同步线上代码

```bash
# 拉取线上最新代码
shopify theme pull --environment development

# 拉取 GitHub 远程代码
git pull origin main
```

#### 2. 创建功能分支

```bash
git checkout -b feature/new-feature
```

#### 3. 开发和测试

```bash
# 运行开发服务器
shopify theme dev --environment development

# 开发完成后提交
git add .
git commit -m "完成新功能：xxx"
```

#### 4. 合并和部署

```bash
# 切换到主分支
git checkout main

# 拉取最新代码（防止冲突）
git pull origin main

# 合并功能分支
git merge feature/new-feature

# 推送到线上主题
shopify theme push --environment development

# 推送到 GitHub
git push origin main
```

---

## 日常操作指南

### 场景 1：我在后台修改了代码

**操作步骤：**

```bash
# 拉取线上最新代码到本地
shopify theme pull --environment development

# 查看哪些文件被修改了
git status
git diff

# 提交变更到 Git
git add .
git commit -m "同步后台修改：xxx"

# 推送到 GitHub
git push origin main
```

### 场景 2：我想在本地开发新功能

**操作步骤：**

```bash
# 确保本地是最新的
shopify theme pull --environment development
git pull origin main

# 创建分支开发
git checkout -b feature/my-feature

# 开发...
# 运行开发服务器预览
shopify theme dev --environment development

# 完成后提交
git add .
git commit -m "完成新功能：xxx"

# 合并到主分支
git checkout main
git merge feature/my-feature

# 推送到线上和 GitHub
shopify theme push --environment development
git push origin main
```

### 场景 3：本地和线上有冲突

**操作步骤：**

```bash
# 拉取线上代码（会覆盖本地冲突文件）
shopify theme pull --environment development

# 使用 Git 检查哪些文件被修改了
git diff

# 手动合并冲突（需要仔细检查每个冲突）
# ...

# 提交合并结果
git add .
git commit -m "合并线上和本地修改"

# 推送到线上和 GitHub
shopify theme push --environment development
git push origin main
```

### 场景 4：误操作覆盖了线上代码

**恢复步骤：**

```bash
# 查看 Git 提交历史
git log --oneline -10

# 找到覆盖之前的提交
# 假设是 abc1234
git checkout abc1234 .

# 推送到线上恢复
shopify theme push --environment development

# 创建恢复提交
git add .
git commit -m "恢复线上代码"
git push origin main
```

---

## 工具链建议

### 推荐配置

| 工具 | 用途 | 命令 |
|---|---|---|
| **shopify theme pull** | 拉取线上代码 | `shopify theme pull --environment development` |
| **shopify theme push** | 推送代码到线上 | `shopify theme push --environment development` |
| **shopify theme dev** | 开发服务器 | `shopify theme dev --environment development` |
| **git status** | 查看工作区状态 | `git status` |
| **git diff** | 查看文件变更 | `git diff` |
| **git log** | 查看提交历史 | `git log --oneline` |

### 自动化脚本（可选）

创建 `scripts/dev.sh`：

```bash
#!/bin/bash

echo "=== 同步线上代码 ==="
shopify theme pull --environment development
git pull origin main

echo ""
echo "=== 启动开发服务器 ==="
shopify theme dev --environment development
```

---

## 方案对比

| 方案 | 优点 | 缺点 | 适用场景 |
|---|---|---|---|
| **开发主题 + 正式主题** | 安全，不影响线上，可随时回滚 | 需要维护两个主题 | 单人开发、重要项目 |
| **Git 分支管理** | 规范，便于代码审查和协作 | 需要学习 Git 流程 | 团队开发、复杂项目 |
| **直接推送到正式主题** | 简单，不需要额外配置 | 风险高，容易覆盖线上 | 临时修改、紧急修复 |

### 推荐组合

对于大多数项目，建议采用：

```
开发主题（预览测试）→ Git（版本控制）→ 正式主题（部署上线）
```

---

## 命令速查

### 同步代码

```bash
# 拉取线上代码
shopify theme pull --environment development

# 拉取 GitHub 代码
git pull origin main
```

### 开发和测试

```bash
# 启动开发服务器
shopify theme dev --environment development

# 创建功能分支
git checkout -b feature/xxx

# 查看变更
git diff
```

### 提交和部署

```bash
# 提交到 Git
git add .
git commit -m "描述"

# 推送到线上主题
shopify theme push --environment development

# 推送到 GitHub
git push origin main
```

### 恢复操作

```bash
# 查看提交历史
git log --oneline -10

# 恢复到指定提交
git checkout <commit-hash> .

# 强制推送恢复
shopify theme push --environment development
```

---

## 注意事项

1. **定期备份**：重要修改前，使用 `git commit` 创建快照
2. **分支管理**：不要在 main 分支直接开发，使用 feature 分支
3. **冲突处理**：遇到冲突时，仔细检查每个文件的变更
4. **测试环境**：重要变更先在开发主题上测试，再部署到正式主题
5. **文档记录**：复杂的修改建议在 commit message 中详细描述

---

## 参考链接

- [Shopify CLI 主题命令](https://shopify.dev/docs/themes/tools/cli/theme)
- [Git 分支管理最佳实践](https://www.atlassian.com/git/tutorials/using-branches)
- [Shopify Theme Access 文档](https://shopify.dev/docs/themes/tools/theme-access)
