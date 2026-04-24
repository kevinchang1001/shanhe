---
title: Superpowers 用户手册
date: 2026-04-24
tags:
  - AI编程
  - 方法论
categories:
  - 技术文档
---

# Superpowers 用户手册

> 面向 AI 编程代理的完整软件开发方法论

## 目录

1. [产品简介](#一产品简介)
2. [安装指南](#二安装指南)
3. [核心工作流程](#三核心工作流程)
4. [技能库详解](#四技能库详解)
5. [方法论哲学](#五方法论哲学)
6. [故障排除](#六故障排除)
7. [附录](#七附录)

---

## 一、产品简介

### 1.1 什么是 Superpowers

Superpowers 是一套完整的**软件开发方法论**，专为 AI 编程代理（coding agents）设计，基于一组可组合的技能（skills）和初始化指令构建。它让 AI 代理从"直接写代码"转变为"先理解需求、再规划、最后执行"的专业开发流程。

### 1.2 核心能力

| 能力维度 | 说明 |
|---------|------|
| **需求澄清** | 在动手前主动追问需求本质，避免方向性错误 |
| **分阶段设计** | 将设计分段展示，确保人类充分审核后再推进 |
| **结构化规划** | 将大任务拆解为 2-5 分钟可完成的小任务 |
| **子代理驱动开发** | AI 可自主工作数小时而不偏离计划 |
| **测试先行** | 强制执行红/绿/重构 TDD 循环 |
| **自动技能触发** | 无需人工干预，代理自动匹配相关技能 |

### 1.3 工作原理

```
你启动编程代理
    ↓
代理感知到你在构建项目
    ↓
【brainstorming】激活 — 追问真实需求，分段展示设计
    ↓
你审核并签署设计
    ↓
【writing-plans】激活 — 制定可执行的小任务计划
    ↓
你说"开始"
    ↓
【subagent-driven-development】激活 — 子代理逐任务执行
    ↓
每个任务经过：RED → GREEN → REFACTOR → 提交
    ↓
【finishing-a-development-branch】激活 — 测试验证、合并决策
```

---

## 二、安装指南

### 2.1 支持平台总览

| 平台 | 安装方式 | 备注 |
|------|---------|------|
| Claude Code（官方市场） | `/plugin install superpowers@claude-plugins-official` | Anthropic 官方插件市场 |
| Claude Code（Superpowers 市场） | 需先注册市场，再安装 | 提供 Superpowers 及相关插件 |
| OpenAI Codex CLI | `/plugins` → 搜索 superpowers | 交互式插件搜索 |
| OpenAI Codex App | 侧边栏 Plugins → Coding → 添加 | 图形界面操作 |
| Cursor | `/add-plugin superpowers` 或市场搜索 | 插件市场集成 |
| OpenCode | 修改 `opencode.json` 配置 | 需重启生效 |
| GitHub Copilot CLI | `copilot plugin` 命令 | 需先添加市场 |
| Gemini CLI | `gemini extensions install` | 自动更新支持 |

### 2.2 各平台详细步骤

#### Claude Code（官方市场）

```bash
/plugin install superpowers@claude-plugins-official
```

#### Claude Code（Superpowers 市场）

```bash
# 1. 注册市场
/plugin marketplace add obra/superpowers-marketplace

# 2. 从该市场安装
/plugin install superpowers@superpowers-marketplace
```

#### OpenAI Codex CLI

```bash
# 打开插件搜索
/plugins

# 搜索并安装
superpowers
# 选择 "Install Plugin"
```

#### OpenAI Codex App

1. 点击侧边栏 **Plugins**
2. 在 Coding 分区找到 **Superpowers**
3. 点击旁边的 **+** 并跟随提示操作

#### Cursor

在 Cursor Agent 聊天中执行：

```
/add-plugin superpowers
```

或直接在插件市场搜索 "superpowers"。

#### OpenCode

在 `opencode.json`（全局或项目级）的 `plugin` 数组中添加：

```json
{
  "plugin": ["superpowers@git+https://github.com/obra/superpowers.git"]
}
```

重启 OpenCode 后生效。验证方法：询问 "Tell me about your superpowers"。

详细文档： [docs/README.opencode.md](docs/README.opencode.md)

#### GitHub Copilot CLI

```bash
# 添加市场
copilot plugin marketplace add obra/superpowers-marketplace

# 安装插件
copilot plugin install superpowers@superpowers-marketplace
```

#### Gemini CLI

```bash
# 安装
gemini extensions install https://github.com/obra/superpowers

# 更新
gemini extensions update superpowers
```

### 2.3 验证安装

安装完成后，向代理发送：

```
Tell me about your superpowers
```

代理应返回 Superpowers 技能系统介绍。

---

## 三、核心工作流程

### 3.1 七阶段开发流程

Superpowers 定义了 7 个严格按顺序执行的阶段，代理在每个阶段自动触发对应技能：

#### 阶段 1：brainstorming（头脑风暴）

**触发时机**：代理感知到你正在构建某个项目，尚未开始写代码

**执行内容**：
- 通过苏格拉底式提问澄清模糊想法
- 探索替代方案
- 分段向用户展示设计（每段足够短，便于阅读和消化）
- 保存设计文档

**产出**：经用户签署同意的设计文档

---

#### 阶段 2：using-git-worktrees（Git 工作树）

**触发时机**：设计获得批准后

**执行内容**：
- 在新分支上创建隔离的工作空间
- 运行项目初始化
- 验证干净的测试基线

**产出**：隔离的开发环境，测试通过

---

#### 阶段 3：writing-plans（编写计划）

**触发时机**：设计获批后，启动实现前

**执行内容**：
- 将工作拆解为 2-5 分钟的小任务
- 每个任务包含：精确文件路径、完整代码、验证步骤

**产出**：可执行的任务清单

---

#### 阶段 4：subagent-driven-development / executing-plans（子代理驱动开发 / 执行计划）

**触发时机**：计划制定完毕，用户说"开始"

**执行内容**：
- 每个任务派发一个全新子代理
- 两阶段审查：规范合规性 → 代码质量
- 或批量执行，设置人工检查点

**特殊能力**：Claude 可自主工作数小时而不偏离计划

---

#### 阶段 5：test-driven-development（测试驱动开发）

**触发时机**：实现过程中

**执行内容**：
严格 RED-GREEN-REFACTOR 循环：
1. **RED** — 写一个会失败的测试
2. **GREEN** — 写最少代码让它通过
3. **REFACTOR** — 重构优化
4. **提交**

> ⚠️ **强制规则**：测试通过前的代码一律删除

---

#### 阶段 6：requesting-code-review（请求代码审查）

**触发时机**：任务之间

**执行内容**：
- 根据计划审查代码
- 按严重级别报告问题
- **严重问题阻塞进度**

---

#### 阶段 7：finishing-a-development-branch（完成开发分支）

**触发时机**：任务全部完成

**执行内容**：
- 验证测试全部通过
- 提供选项：合并 / 创建 PR / 保留 / 丢弃
- 清理工作树

---

### 3.2 流程图

```
启动代理
    ↓
brainstorming ← 你说"帮我做个 XX"
    ↓
你审核设计（分段展示）
    ↓
using-git-worktrees（创建隔离分支）
    ↓
writing-plans（拆解任务清单）
    ↓
你说"go"或"开始"
    ↓
subagent-driven-development
    ↓
每个任务：TDD 循环 → 代码审查
    ↓
finishing-a-development-branch
    ↓
你选择：合并 / PR / 保留 / 丢弃
```

---

## 四、技能库详解

### 4.1 测试类

| 技能 | 用途 |
|------|------|
| **test-driven-development** | RED-GREEN-REFACTOR 全流程，包含测试反模式参考 |

### 4.2 调试类

| 技能 | 用途 |
|------|------|
| **systematic-debugging** | 4 阶段根因分析法（包含根因追踪、纵深防御、条件等待技术） |
| **verification-before-completion** | 确保问题真正被修复，而非表面消除 |

### 4.3 协作类

| 技能 | 用途 |
|------|------|
| **brainstorming** | 苏格拉底式设计精化 |
| **writing-plans** | 详细实现计划编写 |
| **executing-plans** | 批量执行含检查点 |
| **dispatching-parallel-agents** | 并发子代理工作流 |
| **requesting-code-review** | 审查前检查清单 |
| **receiving-code-review** | 响应审查反馈 |
| **using-git-worktrees** | 并行开发分支管理 |
| **finishing-a-development-branch** | 合并/PR 决策工作流 |
| **subagent-driven-development** | 两阶段审查的快速迭代 |

### 4.4 元技能类

| 技能 | 用途 |
|------|------|
| **writing-skills** | 创建符合最佳实践的新技能（含测试方法论） |
| **using-superpowers** | 技能系统入门介绍 |

---

## 五、方法论哲学

### 5.1 四大核心原则

| 原则 | 含义 |
|------|------|
| **测试驱动开发** | 先写测试，永远如此 |
| **系统化优于随机** | 流程优于猜测 |
| **降低复杂度** | 简洁是首要目标 |
| **证据优于声明** | 验证后再宣布成功 |

### 5.2 设计理念

- **YAGNI**（You Aren't Gonna Need It）— 不提前实现不需要的功能
- **DRY**（Don't Repeat Yourself）— 避免重复
- **真 TDD** — 严格遵循红→绿→重构，而非"测试后写"
- **纵深防御** — 调试时追根溯源，不治标不治本

---

## 六、故障排除

### Q1：安装后代理没有响应 Superpowers 技能

**现象**：发送 "Tell me about your superpowers" 无反应

**可能原因**：
- 插件未成功加载
- 平台不支持自动技能触发

**解决**：
1. 重启代理会话
2. 确认安装命令执行无报错
3. 检查平台是否需要手动加载技能

---

### Q2：brainstorming 阶段代理跳过直接写代码

**现象**：代理收到任务后立即开始写代码

**可能原因**：代理未感知到"这是一个需要澄清的项目"

**解决**：明确告诉代理"请先帮我澄清需求"，或主动说"我想做一个 XX，先讨论下设计"

---

### Q3：TDD 循环中测试写在了实现代码之后

**现象**：代码已经写完，测试才补上

**可能原因**：TDD 技能未被正确触发

**解决**：直接指令"请用 TDD 方式，先写测试再看它失败"

---

### Q4：子代理执行时间过长无法跟踪

**现象**：代理自主运行数小时，无法判断当前状态

**解决**：Superpowers 设计为可安全长时间自主运行。如需中断，直接 Ctrl+C，代理会在当前任务完成后停止

---

### Q5：设计文档丢失或未保存

**现象**：brainstorming 后的设计内容没有持久化

**可能原因**：会话上下文窗口耗尽

**解决**：在 brainstorming 开始前说"请将设计保存到 design.md"，代理会自动持久化

---

### Q6：多平台安装，技能行为不一致

**现象**：同一仓库在不同平台表现不同

**可能原因**：各平台对 Superpowers 技能的支持程度不同

**解决**：参考 [Platform Support](https://github.com/obra/superpowers#platform-support)，优先在 Claude Code 和 OpenCode 上使用完整功能

---

## 七、附录

### 7.1 项目信息

| 项目 | 信息 |
|------|------|
| **作者** | Jesse Vincent ([Blog](https://blog.fsck.com)) |
| **公司** | Prime Radiant ([primeradiant.com](https://primeradiant.com)) |
| **许可证** | MIT License |
| **官方发布公告** | [原文发布](https://blog.fsck.com/2025/10/09/superpowers/) |

### 7.2 资源链接

| 资源 | 链接 |
|------|------|
| **GitHub 仓库** | https://github.com/obra/superpowers |
| **问题反馈** | https://github.com/obra/superpowers/issues |
| **Discord 社区** | https://discord.gg/35wsABTejz |
| **版本公告订阅** | https://primeradiant.com/superpowers/ |
| **赞助作者** | https://github.com/sponsors/obra |

### 7.3 CLI 速查表

```bash
# Claude Code - 官方市场安装
/plugin install superpowers@claude-plugins-official

# Claude Code - Superpowers 市场安装
/plugin marketplace add obra/superpowers-marketplace
/plugin install superpowers@superpowers-marketplace

# GitHub Copilot CLI
copilot plugin marketplace add obra/superpowers-marketplace
copilot plugin install superpowers@superpowers-marketplace

# Gemini CLI
gemini extensions install https://github.com/obra/superpowers
gemini extensions update superpowers

# Cursor
/add-plugin superpowers
```

### 7.4 七阶段速查

| 顺序 | 阶段 | 触发时机 |
|------|------|---------|
| 1 | brainstorming | 项目启动，未写代码前 |
| 2 | using-git-worktrees | 设计批准后 |
| 3 | writing-plans | 计划制定，启动实现前 |
| 4 | subagent-driven-development | 用户说"开始"后 |
| 5 | test-driven-development | 实现过程中（持续） |
| 6 | requesting-code-review | 任务之间 |
| 7 | finishing-a-development-branch | 全部任务完成后 |

---

*本文档基于 Superpowers v1.0 README 编制，如有更新请以 [GitHub 仓库](https://github.com/obra/superpowers) 最新内容为准。*
