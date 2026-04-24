---
title: AGENTS.md 开源项目调研报告
date: 2025-06-20
tags:
  - AI编程
  - 工具研究
categories:
  - 技术调研
---

# AGENTS.md 开源项目调研报告

> 调研时间：2025-06-20
> 信息来源：GitHub、Web Search、Raw Content 抓取

---

## 一、AGENTS.md 是什么

**AGENTS.md** 是一个开放的 Markdown 格式，旨在为 AI 编程代理提供项目级上下文和指令，定位为"**AI 代理的 README**"（README for agents）。

它解决的核心问题：团队没有统一机制告诉 AI 代理"如何构建、测试、部署本项目"，导致 AI 代理每次都需要从代码库中自行推断规则，容易出错且效率低下。

---

## 二、核心项目生态

### 2.1 官方规范与网站

| 项目 | 地址 | 关键数据 |
|------|------|---------|
| 官方规范与官网 | [agentsmd/agents.md](https://github.com/agentsmd/agents.md) | ⭐ 20.4k stars，22 contributors，1.5k forks |
| 官方文档站 | [agents.md](https://agents.md) | 规范说明、工具链整合 |

这是整个生态的核心源头项目，定义了 AGENTS.md 格式规范。

---

### 2.2 生产级示例

| 项目 | 行业 | AGENTS.md 规模 |
|------|------|---------------|
| [apache/airflow](https://github.com/apache/airflow/blob/main/AGENTS.md) | 数据工程 | 18,567 字节 |
| [apache/tvm](https://github.com/apache/tvm) | 深度学习编译 | — |
| Temporal SDK Java | 分布式工作流 | — |

Apache Airflow 的 AGENTS.md 是最知名的生产级参考，包含：
- Breeze 容器环境使用规范
- prek 提交钩子配置
- 多 providers 包测试规范
- Monorepo 下的包选择逻辑

---

### 2.3 生态工具

#### Ruler — 多代理指令分发工具

| 属性 | 值 |
|------|---|
| **地址** | [intellectronica/ruler](https://github.com/intellectronica/ruler) |
| **定位** | 将 AGENTS.md 分发到 30+ 各种 AI 代理的专用格式 |
| **支持代理** | Claude Code、Copilot、Codex CLI、Cursor、Windsurf、Cline、Aider、Gemini CLI、Jules、Junie、OpenCode、Zed 等 30+ |
| **核心原理** | `.ruler/` 目录集中管理 → 分发到各代理的专用配置文件 |
| **安装** | `npm install -g @intellectronica/ruler` |

#### 支持的代理格式对照（部分）

| 代理 | AGENTS.md 路径 | MCP 配置路径 | Skills 路径 |
|------|---------------|-------------|------------|
| Claude Code | `CLAUDE.md` | `.mcp.json` | `.claude/skills/` |
| GitHub Copilot | `AGENTS.md` | `.vscode/mcp.json` | `.claude/skills/` |
| OpenAI Codex CLI | `AGENTS.md` | `.codex/config.toml` | `.codex/skills/` |
| Cursor | `AGENTS.md` | `.cursor/mcp.json` | `.cursor/skills/` |
| Windsurf | `AGENTS.md` | `.windsurf/mcp_config.json` | `.windsurf/skills/` |
| Gemini CLI | `AGENTS.md` | `.gemini/settings.json` | `.gemini/skills/` |
| Aider | `AGENTS.md` + `.aider.conf.yml` | `.mcp.json` | — |
| Cline | `.clinerules` | — | — |
| Zed | `AGENTS.md` | `.zed/settings.json` | — |

---

### 2.4 相关规范项目

#### Agent-Specification

| 属性 | 值 |
|------|---|
| **地址** | [agile-lab-dev/Agent-Specification](https://github.com/agile-lab-dev/Agent-Specification) |
| **定位** | 智能体系统的 OpenAPI 3.0 风格 schema 规范 |
| **规模** | 9,469 字节 |
| **核心概念** | agentic system（具有自主性的 LLM 系统） |

定义了智能体的标准属性：Identity、Ownership、Business Context、Technical Configuration、Dependencies、Security 等。

#### SPEC-AGENTS.md

| 属性 | 值 |
|------|---|
| **来源** | 忍笑实验室 |
| **定位** | 文档优先的 AI 编程规范 |
| **特点** | 将文档作为权威来源，AI 严格遵循 |

---

## 三、格式规范

### 3.1 标准结构

AGENTS.md 本身是 Markdown 文件，无强制 schema，典型结构：

```markdown
# 项目名称

## Dev environment tips
- 环境搭建技巧
- 包管理器命令

## Testing instructions
- 测试运行方式
- 覆盖率要求

## Build instructions
- 构建步骤
- 产物路径

## Conventions
- 代码风格
- 提交流程
```

### 3.2 各平台读取优先级

**OpenAI Codex** 读取顺序：
```
1. ~/.codex/AGENTS.override.md（最高优先级）
2. ~/.codex/AGENTS.md
3. 仓库根目录 AGENTS.md
4. 仓库根目录 TEAM_GUIDE.md
5. 仓库根目录 .agents.md
```

**Ruler** 优先级：
```
1. 仓库根 AGENTS.md（最高）
2. .ruler/AGENTS.md（新默认）
3. .ruler/instructions.md（仅旧版兼容）
4. .ruler/*.md（递归，按字母序）
```

---

## 四、竞争格式

| 格式 | 工具 | 特点 |
|------|------|------|
| `.cursorrules` | Cursor | Cursor 专用，MDC 格式 |
| `.clinerules` | Cline | Cline 专用 |
| `CLAUDE.md` | Claude Code | Claude Code 专用 |
| `WARP.md` | Warp Terminal | 终端 AI 专用 |

这些格式都与 AGENTS.md 目标相同，但工具锁定。AGENTS.md 的优势在于**通用性**——一个格式，多工具兼容（通过 Ruler 等工具分发）。

---

## 五、采用情况

- ⭐ **20.4k+ stars**（agentsmd/agents.md 官方仓库）
- **Apache Airflow**、**Apache TVM** 等顶级开源项目已采用
- **OpenAI Codex**、**GitHub Copilot**、**Google Gemini CLI** 等主流代理均原生支持
- 微软 VS Code 官方文档已收录 AGENTS.md 说明
- 生态工具 Ruler 支持 30+ 代理，覆盖主流开发环境

---

## 六、关键洞察

1. **AGENTS.md 已成事实标准** — 20.4k stars 和 Apache 顶级项目的采用，确立了其行业地位
2. **工具碎片化是最大痛点** — 30+ 代理各有专用格式，Ruler 等工具提供"一个源、分发多端"的桥接层
3. **格式本身极度简单** — 就是 Markdown，无强制 schema，门槛极低；但也因此缺乏约束，不同项目的 AGENTS.md 质量差异大
4. **与 Superpowers 高度互补** — Superpowers 是 AI 代理的**工作流程方法论**（如何执行），AGENTS.md 是**项目上下文定义**（做什么），两者可叠加使用

---

## 七、资源链接

| 资源 | 链接 |
|------|------|
| 官方规范仓库 | https://github.com/agentsmd/agents.md |
| 官方文档站 | https://agents.md |
| Ruler 工具 | https://github.com/intellectronica/ruler |
| Agent-Specification | https://github.com/agile-lab-dev/Agent-Specification |
| Airflow 参考实现 | https://github.com/apache/airflow/blob/main/AGENTS.md |
| OpenAI Codex 指南 | https://developers.openai.com/codex/guides/agents-md |

---

*本报告数据来源：GitHub API/Raw Content、Web Search，star 数据基于 2025-06-20 抓取。*
