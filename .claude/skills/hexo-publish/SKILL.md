---
name: hexo-publish
description: Hexo 博客文章发布技能。当用户提到发布文章、hexo 文章、博客文章、md 文件发布、将 md 转为博客文章、deploy hexo、博客更新时自动触发。专门处理将本地 Markdown 文件添加 front-matter 后发布到 Hexo 博客，并同步到 GitHub Pages 的完整流程。
---

# Hexo 博客文章发布技能

将本地 md 文件发布到 Hexo 博客并同步到 GitHub Pages。

## 适用场景

- 用户提供 md 文件路径，要求发布到博客
- 用户说"发布文章"、"更新博客"、"同步到 github.io"
- 用户上传了 md 文件需要发布

## 工作流程

### 步骤 1：确定 md 文件位置

用户可能提供：
- 绝对路径：如 `/Users/nexlume/Downloads/article.md`
- 相对路径：如 `~/Downloads/article.md`
- 文件名：如 `article.md`

如果是相对路径或文件名，尝试在以下位置查找：
- 用户主目录的 Downloads 文件夹
- 当前工作目录

### 步骤 2：读取 md 文件内容

使用 Read 工具读取文件内容。

### 步骤 3：检查并添加 Front-matter

Front-matter 是文章顶部的 YAML 元数据区块，格式如下：

```markdown
---
title: 文章标题
date: 2026-04-24
tags:
  - 标签1
  - 标签2
categories:
  - 分类
---

正文内容...
```

**判断规则：**
- 如果文件已以 `---` 开头并包含有效的 front-matter，跳过
- 如果文件没有 front-matter，添加
- 如果文件有 front-matter 但不完整，补充缺失字段

**提取标题：**
- 优先使用 front-matter 中的 `title`
- 如果没有，提取文件第一行 `# 标题` 中的标题
- 如果都没有，使用文件名作为标题

**日期处理：**
- 使用文件修改时间或当前日期，格式 `YYYY-MM-DD`

### 步骤 4：复制到博客目录

目标目录：`{博客根目录}/source/_posts/`

文件名处理：
- 提取标题中的特殊字符
- 用中划线连接
- 添加 `.md` 后缀
- 示例：`我的第一篇文章` → `我的第一篇文章.md`

### 步骤 5：Git 操作

```bash
cd {博客根目录}
git add -A
git commit -m "发布文章：{标题}"
git push
```

### 步骤 6：验证

GitHub Actions 会自动构建部署。告诉用户：
- 博客地址
- 大约等待 30 秒后可在 https://kevinchang1001.github.io/shanhe/ 查看

## 博客配置信息

| 配置项 | 值 |
|--------|-----|
| 博客根目录 | `/Users/nexlume/AI-Workspace/blog` |
| 文章目录 | `source/_posts/` |
| GitHub 仓库 | `https://github.com/kevinchang1001/shanhe` |
| 博客地址 | `https://kevinchang1001.github.io/shanhe/` |
| 主题 | Next |
| 语言 | zh-CN |

## 删除文章

如果用户要删除文章：
1. 从 `source/_posts/` 删除对应 md 文件
2. 执行 `git add -A && git commit && git push`

## 注意事项

- 始终使用 `git add -A` 确保所有更改被追踪
- commit message 使用中文，格式：`发布文章：标题` 或 `删除文章：标题`
- GitHub Actions 自动处理构建和部署，无需手动触发
