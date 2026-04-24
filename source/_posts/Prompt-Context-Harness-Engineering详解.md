---
title: Prompt Engineering、Context Engineering 与 Harness Engineering 详细说明
date: 2025-06-20
tags:
  - AI编程
  - LLM
  - 方法论
categories:
  - 技术调研
---

# Prompt Engineering、Context Engineering 与 Harness Engineering 详细说明

> 调研时间：2025-06-20
> 信息来源：Prompt Engineering Guide、BigData Boutique、Medium、DailyDev、YouTube 视频描述等

---

## 一、Prompt Engineering（提示词工程）

### 1.1 定义

Prompt Engineering 是通过设计清晰、有针对性的**输入文本**，引导大语言模型（LLM）产生准确、符合预期的输出。它是 LLM 应用的第一层门槛，也是其他一切工程层的基础。

> *"Prompt engineering is the practice of writing clear, purposeful inputs that guide AI models to deliver accurate and context-aware outputs."*
> — GitHub

### 1.2 核心原则

| 原则 | 说明 |
|------|------|
| **清晰明确** | 指令结构清晰，避免歧义 |
| **任务分解** | 复杂问题拆解为简单步骤 |
| **格式约束** | 指定输出的结构（如 JSON、表格） |
| **示例引导** | 通过 Few-shot 提供输出模式 |
| **上下文相关** | 提供足够的背景信息 |

### 1.3 核心技术分类

#### A. 基础提示技术

**零样本提示（Zero-shot Prompting）**
- 无需示例，直接给出指令
- 适用于简单任务
- 依赖模型本身的泛化能力

**少样本提示（Few-shot Prompting）**
- 在提示中加入 1~N 个示例
- 模型通过示例学习输出模式
- 适用于有明确输出格式的任务
- 论文：Brown et al., 2020 (GPT-3)

**链式思考提示（Chain-of-Thought, CoT）**
- 在提示中加入中间推理步骤
- 引导模型"一步一步思考"
- 显著提升复杂推理任务准确率
- 论文：Wei et al., 2022 (Google)
- 变体：**自洽性（Self-Consistency）**：多路径推理，取多数票答案

**生成知识提示（Generated Knowledge Prompting）**
- 先让模型生成相关知识或事实
- 再基于生成内容回答问题
- 减少幻觉（hallucination）

#### B. 高级提示框架

**Prompt Chaining（提示链）**
- 将复杂任务分解为多个提示串联
- 每个提示处理一个子任务，输出作为下一级输入
- 适用于长流程任务

**思维树（Tree of Thoughts, ToT）**
- 在 CoT 基础上探索多分支推理树
- 系统性评估不同推理路径
- 适用于需要探索和规划的任务

**检索增强生成（RAG, Retrieval-Augmented Generation）**
- 从外部知识库检索相关文档
- 将检索结果注入上下文
- 解决模型知识过时和幻觉问题
- 核心挑战：检索质量 > 提示词优化

**自动推理并使用工具（ART, Automatic Reasoning and Tool-use）**
- 自动选择合适工具并执行
- 融合推理与行动
- 论文：Paranjape et al., 2023

**ReAct 框架（Reason + Act）**
- 交替进行推理和行动
- 推理步骤触发行动，行动结果反哺推理
- 已成为 Agent 架构的标准范式
- 论文：Yao et al., 2023

**Reflexion**
- 在 ReAct 基础上加入自我反思
- 失败后根据反馈修正策略
- 适用于需要迭代优化的任务

**Active-Prompt**
- 动态选择最有效的示例
- 根据任务特征自适应调整提示

**方向性刺激提示（Directional Stimulus Prompting）**
- 提供方向性线索引导模型输出
- 适用于风格控制、倾向性任务

**Program-Aided Language Models (PAL)**
- 将推理过程用代码表达
- 模型生成代码作为推理中间步骤

**多模态思维链（Multimodal CoT）**
- 在 CoT 中融入图像、音频等多媒体输入
- 适用于多模态推理场景

**基于图的提示（Graph Prompting）**
- 将图结构（知识图谱、关系网络）融入提示
- 适用于关联推理任务

**Meta-Prompting**
- 让模型自己生成最优提示
- 递归优化提示本身

### 1.4 典型应用场景

| 场景 | 示例 |
|------|------|
| 代码生成 | 自然语言 → 可执行代码 |
| 调试修复 | 错误信息 + 代码 → 根因分析 |
| 数据分类 | 少量标注示例 → 大规模分类 |
| 摘要生成 | 长文本 → 结构化摘要 |
|问答系统 | 问题 + 上下文 → 精确答案 |

### 1.5 局限性

| 局限性 | 表现 |
|--------|------|
| **单次交互假设** | 假设世界是静态的，写一次调一次 |
| **无法动态注入信息** | 无法在运行时接入实时数据、工具输出 |
| **无法管理历史状态** | 多轮对话中历史信息膨胀失控 |
| **工具调用无力** | 复杂场景需要多次工具调用，单次 prompt 无法支撑 |
| **规模不经济** | 当应用需要工具、记忆、多步工作流时，prompt 优化边际收益急剧下降 |

> 正如 BigData Boutique 所述：*"You can have a perfect prompt and still get terrible results if the surrounding context is wrong."*

---

## 二、Context Engineering（上下文工程）

### 2.1 定义

Context Engineering 是设计和管理 LLM 在整个交互生命周期中接收的**完整信息环境**的学科，涵盖系统指令、检索知识、工具结果、对话历史等所有上下文来源。

> *"Context engineering is the delicate art and science of filling the context window with just the right information for the next step."*
> — Andrej Karpathy

> *"The art of providing all the context for the task to be plausibly solvable by the LLM."*
> — Tobi Lütke (Shopify CEO)

> *"Context engineering covers all four bullets [system instructions, retrieved knowledge, tool results, conversation history] — and the dynamic orchestration between them."*
> — Simon Willison

### 2.2 核心定位

**Prompt Engineering vs Context Engineering 的本质区别**：

| 维度 | Prompt Engineering | Context Engineering |
|------|-------------------|---------------------|
| **范围** | 单次输入文本 | 整个信息管道 |
| **性质** | 静态，写完迭代 | 动态，运行时组装 |
| **方法** | 人工措辞优化 | 系统设计与架构 |
| **交互模型** | 单次或简单多轮 | 多步、Agentic、工具调用 |
| **失败模式** | "模型误解了我" | "模型收到了错误信息" |
| **技能集合** | 写作、领域知识 | 软件工程，信息架构 |

**关系**：Prompt Engineering ⊂ Context Engineering。Prompt Engineering 是 Context Engineering 的子集，所有好的 prompt 仍然需要，但它只是 Context Engineering 的四个组件之一。

### 2.3 LLM 上下文的四大组件

```
┌─────────────────────────────────────────────┐
│           LLM 推理时的完整上下文              │
├─────────────────────────────────────────────┤
│  1. System Instructions（系统指令）           │
│     → Prompt Engineering 覆盖的区域           │
│     → 行为规则、输出格式约束                  │
├─────────────────────────────────────────────┤
│  2. Retrieved Knowledge（检索知识）           │
│     → RAG、搜索、文档注入                     │
│     → 知识图谱、向量数据库                    │
├─────────────────────────────────────────────┤
│  3. Tool Results（工具结果）                 │
│     → 函数调用、API 响应、数据库查询          │
│     → 外部世界状态                            │
├─────────────────────────────────────────────┤
│  4. Conversation History（对话历史）          │
│     → 历史轮次、长期记忆、摘要                │
└─────────────────────────────────────────────┘
```

### 2.4 Context Engineering 四大策略

LangChain 框架将 Context Engineering 分解为四个核心策略：

| 策略 | 含义 | 典型实现 |
|------|------|---------|
| **Write（写）** | 将信息写到上下文窗口之外，供后续使用 | Scratchpad、长期记忆、外部存储 |
| **Select（选择）** | 按需检索最相关信息注入上下文 | RAG、向量搜索，知识图谱检索、工具选择 |
| **Compress（压缩）** | 在有限 token 预算内保留关键信息 | 摘要、摘要提取、修整旧消息 |
| **Isolate（隔离）** | 跨代理或组件分割上下文 | 多代理独立上下文窗口、隔离执行 |

### 2.5 生产级实践

#### RAG as Context Engineering

RAG 本质上是 Context Engineering 的最纯粹实现——不改变提示词本身，而是改变模型在推理时能访问的知识。工程挑战的核心不在于查询语句，而在于**构建高质量的检索管道**：检索正确文档 → 排序 → 过滤噪声 → 在 token 预算内组装。

#### 工具调用中的上下文工程

当 LLM 调用工具（数据库查询、API、代码解释器）时，工具输出成为下一步推理的上下文。A/B 选择错误或格式错误都会导致"自信的错误答案"。

LangChain 研究发现：动态选择工具描述（基于任务决定向模型展示哪些工具）比将所有工具一股脑注入，上下文准确率提升 **3 倍**。

#### 记忆与上下文窗口管理

长运行 Agent 上下文积累极快：
- Anthropic 多 Agent 研究系统使用的 token 是标准聊天的 **15 倍**
- 无主动管理：要么超出上下文窗口，要么被无关历史淹没

生产策略：
- **Auto-compaction**：Claude Code 在上下文使用率达 95% 时自动压缩，将早期对话压缩为关键要点
- **Context Isolation**：Anthropic 多 Agent 系统中，每个子 Agent 有独立上下文窗口，独立探索后再将结果压缩汇报给主 Agent，这种方式比单 Agent 提升超过 **90%**
- **Selective Memory**：跨会话选择持久化什么、丢弃什么（如 ChatGPT、Cursor 的长期记忆功能）

### 2.6 Context Engineering 失败模式

Drew Breunig 识别出生产系统中四大上下文失败模式：

| 失败模式 | 描述 | 本质问题 |
|---------|------|---------|
| **Poisoning（污染）** | 错误信息被注入上下文 | 信息可信度问题 |
| **Distraction（分心）** | 无关上下文淹没相关信息 | 信号噪声比问题 |
| **Confusion（困惑）** | 上下文中信息相互矛盾 | 一致性问题 |
| **Clash（冲突）** | 不同上下文来源给出冲突指令 | 指令协调问题 |

这些都是 Context Engineering 问题，而非 Prompt Engineering 问题。

### 2.7 行业数据

- **57%** 的组织已在生产环境中部署 AI Agent（LangChain, 2025）
- **32%** 的组织将质量列为首要障碍
- 大多数质量失败**追溯至糟糕的上下文管理**，而非模型能力不足——"模型有了错误的信息，而非错误的指令"

### 2.8 从 Prompt 工程师到上下文架构师

迁移路径：

1. **Map your context（映射上下文）**：追踪 LLM 每一步收到的所有信息片段，绘制完整上下文图谱，你会发现自己从未意识到的上下文和本应存在的上下文缺口
2. **Instrument and observe（仪表化与观察）**：构建可观测性工具，追踪哪些上下文输入与好的/坏的结果相关。问题很少是"模型太笨"，通常是"模型看到了错误的东西"
3. **Design context pipelines（设计上下文管道）**：将上下文组装视为软件工程问题，构建动态检索、排序、过滤、格式化的管道
4. **Budget your tokens（Token 预算管理）**：将上下文窗口视为嵌入式系统的内存——有限资源，设计优先级
5. **Test context, not just prompts（测试上下文而非提示词）**：当出问题时，不要只改 prompt，要检查模型收到的完整上下文

---

## 三、Harness Engineering（驾驭工程）

### 3.1 定义

Harness Engineering 是在 AI Agent 时代让人来"掌舵方向（steer）"、AI 代理来"执行完成（execute）"的软件工程实践。其核心是**构建让 AI 代理能够可靠完成完整软件工程工作的环境、文档和流程**。

> *"Think of an agent as a new grad. It has a very smart brain (the LLM), it has almost all the human knowledge and it knows how to reason. It can use tools like reading files and browsing the internet. It can write scripts and run commands on your computer. But it doesn't know what you want. It can't guess what you want it to do, what you value, or what your goals are. The complete environment you create for the agent: the workflows you define, the goals you set, that's Harness Engineering."*
> — AnyTech

**提出者**：Ryan Lopopolo（OpenAI Technical Staff Member）
**来源**：OpenAI 内部实践——用 AI 代理完全自主开发 9 个月，产出 1M LOC、日均 1B tokens，**0% 人类编写代码或 review**

### 3.2 核心洞察

传统软件工程：人类是**执行者 + 审核者**
Harness Engineering：人类是**方向掌舵者 + 质量守门人**，代理负责完整执行

```
传统模式：
Human → Write Code → Review → Merge

Harness Engineering 模式：
Human → Steer Direction → Agent Executes → Human Quality Gate
         ↑                                    ↓
         ←←←←←←←← Feedback Loop ←←←←←←←←←←←
```

### 3.3 Ralph Wiggum Loop

OpenAI 内部使用的核心迭代循环：

1. 代理对变更进行本地 Review
2. 请求特定的人工/代理 Review（本地 + 云端）
3. 响应反馈
4. 循环迭代，直到**所有代理 Review 都满意**

这一机制被称为 Ralph Wiggum Loop——一个不断追求代理自洽的反馈循环。

### 3.4 核心工程实践

#### 让代码库对 Agent 直接可理解

OpenAI 团队的核心经验：代码库本身必须让代理能**直接从仓库中推理完整业务领域**，而不依赖人类解释。这包括：
- 应用 UI 本身对代理可读
- Logs 对代理可读
- App Metrics 对代理可读

就像团队为新工程师改善代码可读性一样，Human Engineers 的目标是让代理能够直接推理完整的业务领域。

#### Harness 的三层结构（与 Context Engineering 的演进关系）

```
Prompt Engineering（提示词工程）
  ↓ 覆盖单次输入
Context Engineering（上下文工程）
  ↓ 覆盖信息管道
Harness Engineering（驾驭工程）
  ↓ 覆盖完整工程执行环境
```

- Context Engineering 解决"给 AI 正确的信息"
- Harness Engineering 解决"让人能够可靠地驾驭 AI 完成整个工程"

### 3.5 从写代码到脚手架工程

Harness Engineering 带来的核心转变：

| 维度 | 传统软件工程 | Harness Engineering |
|------|-------------|---------------------|
| **主体劳动** | 人类工程师写代码 | 人类搭建环境，代理写代码 |
| **成功定义** | 代码功能正确 | 代理能理解你的目标并执行 |
| **关键技能** | 编程语言、框架 | 定义目标、验证结果、设置 guardrail |
| **质量保障** | Code Review | **目标定义 + 验证 + guardrail** |
| **挑战** | 技术实现 | "代理写得很像，但不是你想要的" |

### 3.6 三者关系全景图

```
┌─────────────────────────────────────────────────────────────┐
│              AI 时代软件工程方法论演进                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Prompt Engineering（提示词工程）                             │
│  ├── 层次：单次输入优化                                     │
│  ├── 工具：CoT、Few-shot、ReAct 等提示技术                  │
│  ├── 定位：LLM 应用入门层                                   │
│  └── 局限：无法支撑生产级 Agent 系统                        │
│                                                             │
│            ↓ 叠加                                           │
│                                                             │
│  Context Engineering（上下文工程）                           │
│  ├── 层次：信息管道设计                                     │
│  ├── 工具：RAG、Knowledge Graph、Memory Management          │
│  ├── 定位：生产级 AI 系统的信息架构                         │
│  ├── 核心：Write/Select/Compress/Isolate 四大策略          │
│  └── 局限：仍聚焦于"AI 看到什么"，未覆盖完整工程执行        │
│                                                             │
│            ↓ 叠加                                           │
│                                                             │
│  Harness Engineering（驾驭工程）                             │
│  ├── 层次：完整工程执行环境                                  │
│  ├── 工具：工作流定义、目标规范、反馈闭环、Guardrail         │
│  ├── 定位：让人"驾驭"AI代理完成全流程软件工程               │
│  ├── 核心：Human Steer + Agent Execute + Ralph Wiggum Loop  │
│  └── 代表：OpenAI 9个月实践（1M LOC，0%人类代码）           │
│                                                             │
└─────────────────────────────────────────────────────────────┘

三者不是替代关系，而是递进叠加：
- Prompt Engineering 是入口（所有方法论的最底层）
- Context Engineering 是生产级 AI 系统的必经之路
- Harness Engineering 是 AI Agent 时代的完整工程范式
```

### 3.7 前沿数据

- OpenAI 用 AI 代理完全自主开发达到 **1M LOC** 规模
- 日均 **1B tokens** 处理量
- **0% 人类编写代码或 Review**
- 代理可自主工作**数小时**不偏离计划

---

## 四、关键术语对照表

| 英文术语 | 中文翻译 | 所属领域 |
|---------|---------|---------|
| Prompt Engineering | 提示词工程 | LLM 应用基础 |
| Context Engineering | 上下文工程 | LLM 应用生产化 |
| Harness Engineering | 驾驭工程 | AI Agent 软件工程 |
| Chain-of-Thought (CoT) | 链式思考 | Prompt Engineering |
| Self-Consistency | 自洽性 | Prompt Engineering |
| Retrieval-Augmented Generation (RAG) | 检索增强生成 | Context Engineering |
| Knowledge Graph | 知识图谱 | Context Engineering |
| GraphRAG | 图检索增强生成 | Context Engineering |
| Human Steer, Agent Execute | 人掌舵，代理执行 | Harness Engineering |
| Ralph Wiggum Loop | Ralph Wiggum 循环 | Harness Engineering |
| Write/Select/Compress/Isolate | 写/选/压/隔 | Context Engineering 四大策略 |

---

## 五、核心引用来源

| 来源 | 引用内容 |
|------|---------|
| Andrej Karpathy | "context engineering is the delicate art and science of filling the context window with just the right information for the next step" |
| Simon Willison | Context engineering covers all four context components and the dynamic orchestration between them |
| Tobi Lütke (Shopify CEO) | "the art of providing all the context for the task to be plausibly solvable by the LLM" |
| BigData Boutique (2026) | 57% of organizations have AI agents in production; 32% cite quality as top barrier |
| LangChain Research | Dynamic tool selection via RAG improved accuracy 3-fold vs dumping all tools |
| Anthropic Research | Multi-agent context isolation outperformed single-agent by over 90% |
| Drew Breunig | Four context failure modes: poisoning, distraction, confusion, clash |
| Ryan Lopopolo (OpenAI) | 9 months, 1M LOC, 1B tokens/day, 0% human code or review |
| AnyTech (2026) | Agent as new grad analogy: smart brain but doesn't know what you want |

---

## 六、对电力行业 AI 应用的启示

结合用户当前研究的**电网企业语义层建设**背景，三个工程方法论的应用价值：

| 方法论 | 在电网语义层中的潜在应用 |
|--------|------------------------|
| **Prompt Engineering** | 设备台账查询，自然语言报表生成、调度指令解释等基础 AI 交互 |
| **Context Engineering** | 构建电网知识图谱作为上下文源、RAG 管道设计、调度 Agent 多轮对话记忆管理 |
| **Harness Engineering** | 建立 AI 代理的完整开发规范（目标定义、验证机制、Guardrail），实现人驾驭 AI 完成电力业务开发 |

---

*本报告数据来源：Prompt Engineering Guide、BigData Boutique（March 2026）、Medium（AnyTech, March 2026）、DailyDev、YouTube (Ryan Lopopolo @ OpenAI)。*
