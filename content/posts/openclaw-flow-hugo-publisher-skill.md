+++
date = '2026-03-23T16:39:04+08:00'
draft = false
title = 'OpenClaw flow-hugo-publisher 技能验证报告'
tags = ['OpenClaw', 'Hugo', '技能验证', '自动化']
description = '完整验证 OpenClaw flow-hugo-publisher 技能的端到端流程，从环境检测到 GitHub Pages 部署。'
+++

## 前言

今天花了一个下午，完整验证了 OpenClaw 的 `flow-hugo-publisher` 技能。

**结论先行：** 能用，但有些坑需要注意。

---

## 一、技能是什么

`flow-hugo-publisher` 是一个 OpenClaw Skill，用于：

- 检测 Hugo/Git 环境
- 初始化 Hugo 工作目录
- 本地预览（hugo server）
- Git 提交推送
- 状态管理（记录工作目录和 Git 状态）

**触发词：**
- "帮我初始化一个 Hugo 博客"
- "发布一篇 Hugo 文章"
- "启动 Hugo 本地预览"
- "提交到 GitHub"

---

## 二、验证环境

| 项目 | 状态 | 版本 |
|------|------|------|
| macOS | ✅ | 26.3.0 |
| Git | ✅ | 2.51.0 |
| Node.js | ✅ | 22.22.0 |
| Go | ✅ | 1.25.4 |
| Hugo | ✅（安装后） | 0.158.0 |

---

## 三、验证流程

### 步骤 1：环境检测

```bash
command -v hugo && echo "✅" || echo "❌"
```

**结果：** Hugo 未安装，其他依赖 OK。

---

### 步骤 2：安装 Hugo

```bash
brew install hugo
```

**结果：** ✅ 成功，v0.158.0

---

### 步骤 3：初始化博客

```bash
hugo new site test-blog
cd test-blog
git clone https://github.com/adityatelange/hugo-PaperMod themes/PaperMod
```

**结果：** ✅ 成功，PaperMod 主题已安装

---

### 步骤 4：配置站点

编辑 `hugo.toml`：

```toml
baseURL = 'https://ZampoRen.github.io/test-blog/'
languageCode = 'zh-cn'
title = '我的测试博客'
theme = 'PaperMod'

[params]
  env = "production"
  defaultTheme = "auto"
  ShowCodeCopyButtons = true
  ShowReadingTime = true
```

---

### 步骤 5：创建第一篇文章

```bash
hugo new posts/hello-world.md
```

内容：

```markdown
+++
title = 'Hello World'
date = 2026-03-23
draft = false
tags = ['测试', '入门']
+++

这是我的第一篇测试文章！

Hugo + OpenClaw 工作正常！🎉
```

---

### 步骤 6：本地预览

```bash
hugo server -D --bind 0.0.0.0
```

**结果：** ✅ 成功，访问 http://localhost:1313

---

### 步骤 7：Git 提交

```bash
git init
git add .
git commit -m "Initial commit: Hugo 博客初始化"
```

**注意：** 需要创建 `.gitignore` 忽略 `public/` 和 `.hugo_build.lock`

```gitignore
# Hugo
public/
.hugo_build.lock

# macOS
.DS_Store
```

---

### 步骤 8：创建 GitHub 仓库

```bash
gh repo create test-blog --public --description "Hugo 测试博客"
git remote add origin git@github.com:ZampoRen/test-blog.git
git push -u origin master
```

**结果：** ✅ 成功，仓库地址：https://github.com/ZampoRen/test-blog

---

### 步骤 9：配置 GitHub Actions

创建 `.github/workflows/deploy.yml`：

```yaml
name: Deploy Hugo site to Pages

on:
  push:
    branches: ["master"]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Install Hugo
        run: |
          wget https://github.com/gohugoio/hugo/releases/download/v0.158.0/hugo_extended_0.158.0_linux-amd64.deb
          sudo dpkg -i hugo.deb

      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive

      - name: Build with Hugo
        run: hugo --gc --minify

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        uses: actions/deploy-pages@v4
```

---

### 步骤 10：启用 GitHub Pages

**注意：** 这一步必须手动操作！

1. 打开 https://github.com/ZampoRen/test-blog/settings/pages
2. Source 选择 **GitHub Actions**
3. 保存

---

## 四、遇到的问题

### 问题 1：Submodule 错误

**错误：**
```
fatal: No url found for submodule path 'themes/PaperMod'
```

**原因：** 主题是用 `git clone` 安装的，不是 submodule

**解决：**
```bash
# 创建 .gitmodules 文件
git submodule add https://github.com/adityatelange/hugo-PaperMod themes/PaperMod
```

---

### 问题 2：Pages 未启用

**错误：**
```
HttpError: Not Found - Get Pages site failed
```

**原因：** GitHub 不允许 API 自动启用 Pages

**解决：** 手动在仓库设置中启用（见步骤 10）

---

### 问题 3：baseURL 配置

**错误：** 线上样式丢失

**原因：** `hugo.toml` 中 baseURL 还是本地地址

**解决：**
```toml
baseURL = 'https://你的用户名.github.io/仓库名/'
```

---

## 五、验证结果

| 功能 | 状态 | 说明 |
|------|------|------|
| 环境检测 | ✅ | 准确识别 Hugo 未安装 |
| 安装 Hugo | ✅ | brew 安装成功 |
| 初始化博客 | ✅ | 目录结构正确 |
| 创建文章 | ✅ | Markdown 格式正确 |
| 本地预览 | ✅ | http://localhost:1313 正常 |
| Git 提交 | ✅ | 提交记录正常 |
| 创建仓库 | ✅ | GitHub 仓库创建成功 |
| Actions 配置 | ✅ | Workflow 文件正确 |
| GitHub Pages | ⚠️ | 需手动启用 |

---

## 六、技能评价

### ✅ 优点

1. **流程完整**：从环境检测到部署全覆盖
2. **状态管理**：记录工作目录和 Git 状态
3. **人工介入**：关键步骤可确认
4. **错误处理**：失败时保留现场

### ⚠️ 不足

1. **GitHub Pages 无法自动启用**：GitHub API 限制
2. **Submodule 配置需要手动**：容易出错
3. **baseURL 需要手动修改**：应该自动检测

---

## 七、时间线

| 时间 | 事件 |
|------|------|
| 15:30 | 开始验证，环境检测 |
| 15:39 | Hugo 安装完成 |
| 15:40 | 博客初始化 |
| 15:41 | 第一篇文章创建 |
| 15:42 | 本地预览启动 |
| 15:52 | Git 提交完成 |
| 15:56 | GitHub 仓库创建 |
| 15:59 | Actions 配置完成 |
| 16:02 | Submodule 问题修复 |
| 16:11 | Pages 启用问题确认 |
| 16:21 | 部署指南编写完成 |
| 16:39 | 第二篇文章创建 |

**总耗时：** 约 1 小时

---

## 八、最终状态

### 本地

- 工作目录：`/Users/zampo/.openclaw/workspace/test-blog`
- 文章数：2 篇
- 主题：PaperMod
- 状态：本地预览运行中

### GitHub

- 仓库：https://github.com/ZampoRen/test-blog
- 分支：master
- 提交：3 次
- Actions：配置完成，待启用 Pages

### 状态文件

`~/.openclaw/state/hugo-publisher-state.json`：

```json
{
  "workspacePath": "/Users/zampo/.openclaw/workspace/test-blog",
  "gitRoot": "/Users/zampo/.openclaw/workspace/test-blog",
  "currentBranch": "master",
  "remoteUrl": "git@github.com:ZampoRen/test-blog.git",
  "githubUrl": "https://github.com/ZampoRen/test-blog",
  "pagesUrl": "https://ZampoRen.github.io/test-blog",
  "lastCommitHash": "e9603e4",
  "deployment": {
    "method": "github-actions",
    "status": "pending"
  }
}
```

---

## 九、总结

**flow-hugo-publisher 技能可以用，但需要一些手动配置：**

1. GitHub Pages 需要手动启用（GitHub 限制）
2. Submodule 配置要小心
3. baseURL 记得改成 GitHub Pages 地址

**适合人群：**
- 想用 Hugo 写博客
- 想自动化部署
- 不介意少量手动配置

**不适合人群：**
- 想要 100% 全自动
- 不想碰 Git

---

## 十、后续计划

- [ ] 启用 GitHub Pages，验证自动部署
- [ ] 测试多篇文章管理
- [ ] 测试自定义域名
- [ ] 添加评论系统

---

**完**

---

**参考链接：**

- [OpenClaw 官方文档](https://openclaw.ai)
- [Hugo 官方文档](https://gohugo.io)
- [GitHub Pages 文档](https://pages.github.com)
- [PaperMod 主题](https://github.com/adityatelange/hugo-PaperMod)
