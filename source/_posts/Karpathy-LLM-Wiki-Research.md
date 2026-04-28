---
title: Karpathy 的 LLM Wiki 调研报告
date: 2026-04-26
tags:
  - LLM
  - 知识管理
  - Karpathy
categories:
  - 研究报告
---

# Karpathy 的 LLM Wiki 调研报告

> 调研时间：2025-06-20
> 核心来源：Karpathy Gist (karpathy/442a6bf555914893e9891c11519de94f)、balukosuri/llm-wiki-karpathy、MindStudio 博客

---

## 一、核心概念

### 1.1 什么是 LLM Wiki

LLM Wiki 是 Andrej Karpathy 提出的一种**个人知识库构建模式**，核心理念是：

> 不用每次查询时从原始文档重新检索（RAG 模式），LLM **增量构建并维护一个持久的 wiki**——一个结构化的、带交叉链接的 markdown 文件集合，介于你和原始来源之间。

**RAG vs LLM Wiki 的本质区别**：

| 维度 | RAG | LLM Wiki |
|------|-----|---------|
| 知识处理 | 每次查询从原始文档重新发现知识 | 知识被编译一次，然后**保持最新** |
| 积累性 | 无积累，每次从零 | 交叉引用已存在，矛盾已标记，综合已反映所有内容 |
| 查询方式 | 检索相关片段，LLM 重新拼装 | 读 wiki 直接回答，引用已就绪 |
| 适用场景 | 一次性问答 | 跨数周/数月的深度研究 |

**Karpathy 的原话**：
> *"Ask a subtle question that requires synthesizing five documents, and the LLM has to find and piece together the relevant fragments every time. Nothing is built up."*

> *"The wiki is a persistent, compounding artifact. The cross-references are already there. The contradictions have already been flagged. The synthesis already reflects everything you've read."*

---

## 二、三层架构

LLM Wiki 包含三个层次：

```
┌─────────────────────────────────────────────────────────┐
│                    THE SCHEMA                           │
│  CLAUDE.md / AGENTS.md                                  │
│  定义 wiki 结构、约定俗成、工作流程                      │
│  (LLM 的操作手册，决定它是有纪律的 wiki 维护者            │
│   而非通用聊天机器人)                                    │
├─────────────────────────────────────────────────────────┤
│                      THE WIKI                           │
│  wiki/ 目录                                             │
│  LLM 生成和维护的 markdown 文件集合                      │
│  - 摘要页、实体页、概念页、对比页、概述页                 │
│  LLM 完全拥有这一层，人类只读不写                         │
├─────────────────────────────────────────────────────────┤
│                   RAW SOURCES                            │
│  sources/ 目录                                          │
│  原始文档的精选集合                                      │
│  不可变 — LLM 只读，从不修改                             │
│  这是你的信任源（source of truth）                       │
└─────────────────────────────────────────────────────────┘
```

### 各层职责

| 层级 | 文件夹 | 拥有者 | 职责 |
|------|--------|--------|------|
| **Schema** | CLAUDE.md | 人类 + LLM 共同演进 | 定义页面类型、工作流、约定俗成 |
| **Wiki** | `wiki/` | LLM 全权维护 | 创建、更新、维护所有页面和交叉引用 |
| **Raw Sources** | `sources/` 或 `raw/` | 人类 | 不可变的源文档，只供 LLM 阅读 |

---

## 三、核心操作（CRUD）

### 3.1 Ingest（摄取）

当你向原始集合放入新来源并告诉 LLM 处理时，典型流程：

1. LLM 读取来源，与你讨论关键收获
2. 在 wiki 中撰写摘要页
3. 更新 index
4. 更新相关的实体页和概念页
5. 记录新数据与旧 claims 矛盾的地方
6. 向 log.md 追加条目

**一个来源可能触及 10-15 个 wiki 页面。**

> Karpathy 个人偏好逐个摄取来源并保持参与——读摘要、检查更新、指导 LLM 强调什么。但也可以批量摄取。

### 3.2 Query（查询）

你针对 wiki 提问。LLM 搜索相关页面，读它们，综合出带引用的答案。

答案可以是不同形式——markdown 页面、对比表格、幻灯片（Marp）、图表（matplotlib）、canvas。**重要洞察**：好的答案可以被归档回 wiki 成为新页面。这样你的探索就像摄取来源一样在知识库中积累。

### 3.3 Lint（健康检查）

定期让 LLM 对 wiki 做健康检查：
- 页面间的矛盾
- 被新来源取代的陈旧 claims
- 没有入站链接的孤立页面
- 被提及但没有自己页面的重要概念
- 缺失的交叉引用
- 可以通过网络搜索填补的数据空白

---

## 四、两个特殊文件

### 4.1 index.md（内容导向）

wiki 的目录——每个页面列出，带链接、一行摘要、可选元数据（如日期或来源数量）。按类别组织（实体、概念、来源等）。

LLM 在每次摄取时更新它。在回答查询时，LLM 先读 index 找到相关页面，再深入阅读。

**在中等规模（~100 个来源，数百页面）下，这种方式出奇地有效，避免了 embedding-based RAG 基础设施的必要性。**

### 4.2 log.md（时间导向）

append-only 的操作记录——摄取、查询、lint 过程。格式：`## [2026-04-02] ingest | Article Title`。

**实用技巧**：每条以一致前缀开头，log 就可以用简单 unix 工具解析——`grep "^## \[" log.md | tail -5` 给你最近 5 条。log 给出 wiki 演化的时间线，并帮助 LLM 理解最近做了什么。

---

## 五、工作界面：Obsidian + AI Agent

Karpathy 的实际工作方式：

> *"In practice, I have the LLM agent open on one side and Obsidian open on the other. The LLM makes edits based on our conversation, and I browse the results in real time — following links, checking the graph view, reading the updated pages. Obsidian is the IDE; the LLM is the programmer; the wiki is the codebase."*

```
┌─────────────────────┬─────────────────────┐
│   AI Agent (Cursor / │   Obsidian           │
│   Claude Code)       │   (浏览 wiki)          │
│                       │                       │
│  • 读取源文档          │  • 实时查看页面更新     │
│  • 撰写/更新 wiki     │  • Graph View 查看    │
│  • 回答问题            │    连接结构           │
│  • 执行 lint           │  • 跟随链接探索        │
└─────────────────────┴─────────────────────┘
```

### Obsidian 的关键插件

| 插件 | 用途 |
|------|------|
| **Obsidian Web Clipper** | 浏览器扩展，将网页文章转为 markdown，快速抓取来源 |
| **Graph View** (`Cmd+G`) | 查看 wiki 的全局结构——哪些是 hub，哪些是孤立页面 |
| **Dataview** | 对页面 frontmatter 运行查询，生成动态表格和列表 |
| **Marp** | 从 markdown 生成幻灯片，输出演示文稿 |

### 图片处理

LLM 不能原生在一个 Pass 中读取带内联图片的 markdown。**解决方案**：
1. 在 Obsidian 设置中将附件文件夹路径设为固定目录（如 `raw/assets/`）
2. 用热键绑定"下载当前文件的所有附件"（`Ctrl+Shift+D`）
3. 让 LLM 先读文本，再单独查看引用的一些图片以获得额外上下文

---

## 六、balukosuri 实现的完整目录结构

这是社区实现 **balukosuri/llm-wiki-karpathy** 的结构：

```
llm-wiki-karpathy/
├── CLAUDE.md              # Schema — AI 的操作手册
├── llm-wiki.md            # Karpathy 原始 idea 文档
├── article.md             # 阐述本项目的文章
│
├── raw/                   # 源文档（AI 读取，从不写入）
│   └── .gitkeep
│
├── wiki/                  # AI 生成的 knowledge base（AI 拥有此层）
│   ├── index.md          # 主目录 — AI 每次查询先读它
│   ├── overview.md       # 大局综合（随每次摄取演化）
│   ├── glossary.md        # 术语、定义和风格约定
│   ├── log.md            # 所有活动的 chronological 记录
│   ├── sources/          # 源文档摘要页
│   ├── features/         # 产品功能页
│   ├── products/         # 产品/工具页
│   ├── personas/         # 用户类型页
│   ├── concepts/         # 领域概念页
│   ├── style/            # 写作约定页
│   └── analyses/         # 综合输出页（对比表、gap 分析等）
│
└── .obsidian/            # 预配置的 Obsidian vault 设置
```

### Wiki 页面类型详解

| 类型 | 内容 |
|------|------|
| **Source** | 原始文档摘要——关键事实、引文、元数据 |
| **Feature** | 产品功能——做什么、如何工作 |
| **Product** | 产品或工具——概述、版本、相关功能 |
| **Persona** | 用户类型——目标、痛点、专业水平 |
| **Concept** | 领域思想——定义、相关术语、常见误解 |
| **Style Rule** | 写作约定——何时适用、示例、例外 |
| **Analysis** | 综合输出——对比表、gap 分析、大纲 |

---

## 七、典型工作流示例

### 场景：产品经理构建竞争分析知识库

**步骤 1：初始化**
```
在 Cursor 中打开项目文件夹
在 Obsidian 中打开同一文件夹作为 vault
两个窗口并排放置
```

**步骤 2：摄取来源**
```
将某竞品的 PRD PDF 放入 raw/ 文件夹
在 Cursor 中输入：ingest raw/competitor-prd.pdf
LLM 读取文档，讨论关键收获，创建摘要页，更新相关功能/概念页
在 Obsidian 中实时观察页面出现
```

**步骤 3：提问**
```
在 Cursor 中输入：What are the main risks identified across all my sources?
LLM 读 wiki，综合出带引用的答案，并问："Should I save this as a wiki page?"
你说"是"，答案成为永久的分析页面
```

**步骤 4：定期 lint**
```
每 10 次摄取左右，在 Cursor 中输入：lint the wiki
LLM 检查矛盾、陈旧 claims、孤立页面、缺失链接
报告发现，问你哪些修复要应用
```

---

## 八、适用场景

| 场景 | 说明 |
|------|------|
| **个人成长** | 追踪目标、健康、心理、自我提升——归档日记条目、文章、播客笔记 |
| **学术研究** | 数周或数月深度研究一个主题——读论文、文章、报告，逐步构建综合 wiki |
| **读书** | 边读边归档每章，为角色、主题、情节线建立页面，最终构建丰富的配套 wiki |
| **商业/团队** | 由 LLM 维护的内部 wiki，来源包括 Slack 帖子、会议记录、项目文档、客户电话 |
| **竞争分析/尽职调查/旅行规划/课程笔记/爱好深潜** | 任何需要跨时间积累知识的场景 |

---

## 九、为什么维护负担接近零

Karpathy 的核心洞察：

> *"The tedious part of maintaining a knowledge base is not the reading or the thinking — it's the bookkeeping. Updating cross-references, keeping summaries current, noting when new data contradicts old claims, maintaining consistency across dozens of pages. Humans abandon wikis because the maintenance burden grows faster than the value. LLMs don't get bored, don't forget to update a cross-reference, and can touch 15 files in one pass."*

人类放弃 wiki 是因为维护负担增长快于价值。LLM 不会厌倦、不会忘记更新交叉引用、可以一次触及 15 个文件。**维护成本接近零，wiki 就能保持更新。**

### 人类 vs LLM 的分工

| 人类负责 | LLM 负责 |
|---------|---------|
| 策划来源 | 所有其他工作 |
| 指导分析 | 总结、交叉引用 |
| 问好问题 | 归档、记录、保存 |
| 思考意义 | 维护、整理、检查 |

---

## 十、与 Memex 的思想联系

Karpathy 指出这个想法在精神上与 **Vannevar Bush's Memex (1945)** 相关——一种个人策划的知识存储，带有文档间的关联轨迹。

Bush 的愿景比 Web 更接近此：私人的、积极策划的，文档间的连接与文档本身一样有价值。**他无法解决的是谁来维护。LLM 解决了这个问题。**

---

## 十一、行业实现

### 社区实现

| 实现 | 链接 | 特点 |
|------|------|------|
| **balukosuri/llm-wiki-karpathy** | GitHub | Cursor + Obsidian 工作流，完整 CLAUDE.md schema |
| **lucasastorian/llmwiki** | llmwiki.app | 在线版本，支持 PDF/文章/Office 文档上传 |
| **Astro-Han/karpathy-llm-wiki** | GitHub | 工作流和提示结构可复用 |
| **balukosuri/llm-wiki-karpathy** | GitHub | 基于 Cursor 和 Obsidian |

### levelup.gitconnected 详细架构

文章提供了完整的 Python 实现架构：

```
LLM_wiki/
├── main.py                 # 入口: python main.py <command>
├── pyproject.toml
├── requirements.txt
├── .env                    # API keys
├── llm_wiki/
│   ├── cli.py             # Click CLI: init/ingest/query/lint/status/prompts
│   ├── config.py          # WikiSchema, PageSpec dataclasses
│   ├── prompts.py         # 所有 Claude prompts + 24 QUERY_TEMPLATES
│   ├── embeddings.py      # OpenAI text-embedding-3-small wrapper
│   ├── index.py           # EmbeddingIndex: cosine similarity search
│   ├── wiki.py            # WikiPage CRUD (markdown + YAML front matter)
│   ├── ingest.py          # Route → Synthesize → Embed → Index pipeline
│   ├── query.py           # Embed → Search → Assemble → Stream pipeline
│   └── lint.py            # Health checks: orphans/broken xrefs/stale embeddings
├── sources/               # Raw input documents (immutable)
├── wiki/                  # LLM-maintained knowledge base
│   ├── index.md           # Auto-updated page catalog
│   ├── log.md             # Append-only operation history
│   ├── neural_networks.md
│   ├── backpropagation.md
│   ├── attention_mechanism.md
│   ├── transformers.md
│   └── applications.md
└── .meta/
    ├── schema.json        # Page universe definition (PageSpec list)
    └── embeddings.json    # Vector index (OpenAI 1536-dim per page)
```

---

## 十二、与 RAG 的对比总结

| 维度 | RAG | LLM Wiki |
|------|-----|---------|
| **知识处理时机** | 查询时（on-the-fly） | 写入时（at write time） |
| **是否积累** | 否，每次重新发现 | 是，交叉引用已固化 |
| **重复计算** | 每次查询重新综合同样信息 | 一次综合，多次使用 |
| **基础设施** | 向量数据库 + embedding 模型 | 纯 markdown 文件 |
| **维护** | 无需维护索引 | LLM 主动维护交叉引用 |
| **扩展性** | 更好（向量检索天然支持海量文档） | 中等（index.md 在 ~100 来源内足够） |
| **延迟** | 查询时额外检索延迟 | 查询直接读 wiki，无额外延迟 |
| **知识一致性** | 依赖每次查询的综合质量 | LLM 在写入时已解决矛盾 |
| **适用规模** | 大规模（百万文档级） | 小中型（个人/小团队知识库） |

---

## 十三、引用来源

| 来源 | 链接 |
|------|------|
| Karpathy 原始 Gist | https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f |
| balukosuri/llm-wiki-karpathy | https://github.com/balukosuri/llm-wiki-karpathy |
| MindStudio 博客 | https://www.mindstudio.ai/blog/andrej-karpathy-llm-wiki-knowledge-base-claude-code/ |
| levelup.gitconnected 架构详解 | https://levelup.gitconnected.com/beyond-rag-how-andrej-karpathys-llm-wiki-pattern-builds-knowledge-that-actually-compounds-31a08528665e |

---

*本报告数据来源：Karpathy Gist (llm-wiki.md)、balukosuri/llm-wiki-karpathy (GitHub README)、MindStudio 博客 (2026-04-06)。*
