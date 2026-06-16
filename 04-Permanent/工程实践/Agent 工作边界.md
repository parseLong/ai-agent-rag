---
title: Agent 工作边界
description: Types→Config→Repo→Service四层工作边界——agent的自主权范围到底到哪里为止？越界会造成什么事故？
tags: [概念卡片, 永久笔记, 概念, 智能体, 驭驭工程, 工作边界, 自主权]
aliases:
  - Work Boundary
  - Scope Discipline
  - 四层工作边界
  - Agentic Stack
date: 2026-06-11
source: "[[Stage 7.5 — 进阶 Agentic 概念]]"
---

# Agent 工作边界

> Agent 操作的范围 = 它的自主权范围。把 agent 系统拆成 4 层（Types → Config → Repo → Service），每一层上下都是一个**工作边界**。agent 能动到 stack 多深，决定了它能做什么、以及越界时会造成什么事故。

## 🧠 核心洞察

很多进阶 agentic 设计问题最后都回到同一个问题：**agent 的自主权到底到哪里为止？**

- **Agent at Types layer** = 只能符合既有 contract，不能改 schema
- **Agent at Config layer** = 可以调 budget / policy，但不能改 memory
- **Agent at Repo layer** = 可以读写 memory / [[向量存储]]，但不能 redesign workflow
- **Agent at Service layer** = 可以重组整个 workflow，拥有最高自主权

> [!warning] 这套 4 层跟 [[上下文工程三层Stack]] 不一样
> - **Prompt → Context → Harness**（三层 Stack）：**stack 位置**——你正在工程"字符串 / 信息 / runtime"的哪一个对象？
> - **Types → Config → Repo → Service**（本概念）：**自主权范围**——agent 能动到 stack 多深？
>
> 两者**正交**，解决不同问题。

## 🔑 3 个真实越界案例

### 案例 1：越界没收手 — Cognition Flappy Bird

用[[多智能体协作]]拆解任务，一个 subagent 负责画绿色管道、另一个负责画云朵，合起来双方风格完全对不上——因为每个 subagent 只看到自己那块、没有对方的[[上下文窗口]]。

→ [[上下文腐蚀与注意力衰减]] + 缺乏 shared context

### 案例 2：加料 — Anthropic Multi-Agent Research

subagent 被指派"研究某主题"、它在报告里擅自加上"我推测 X 也可能成立"这类没人要的推论。Anthropic 在论文里专门讲为什么这种"主动补完"要通过[[Harness Engineering 工程]]消除。

→ **Evaluator-Optimizer loop** 过滤 speculative output

### 案例 3：operator 给太多权限 — Replit Agent

用户把 production database access 直接交给 agent、没设"破坏性操作要先 confirm"的 gate，结果 agent 在"修 bug"过程跑了破坏性 SQL、清掉 production 数据。

→ **Autonomy gradient**：suggest / propose / execute 三段授权

## 🔑 3 个案例的核心教训

1. **Agent 不会"刚好停在你交代的那个点"** → brief 要明确写"只能动 X、绝对不要动 Y"
2. **Agent 会主动"补完"没被要求的东西** → structured output schema + evaluator-optimizer loop 过滤
3. **规则"装好了 ≠ 会被遵守"** → 必须有 **mechanical gate**（permission check / cost cap / destructive op confirm）

## 🔑 Failure-mode Lifecycle

每个产业级 agent failure mode 都走过 **发现 → 文档化 → encode 成 pattern → 自动消除** 的循环：

| # | Incident | Codify 成什么 pattern |
|---|---|---|
| 1 | Multi-agent context desync | **Single-thread principle**: 别堆 multi-agent、用 linear orchestration |
| 2 | Subagent speculative leap | **Evaluator-optimizer loop**: 加 critique step |
| 3 | Production permission drift | **Autonomy gradient**: suggest / propose / execute 三段授权 |
| 4 | Agent looping without self-criticism | **PAR loop**: 加 self-critique + revise |
| 5 | Skill library corruption | **Pre-verify before commit**: skill 入库前必跑 test |

> 遇到自己 agent 搞砸时、查上表"最像哪一行"、然后读对应 pattern 的 deep dive。

## 🔑 12 个进阶概念按工作边界分群

| # | 概念 | 动到哪一层 | 一句话 |
|---|---|---|---|
| 1 | **Work Boundary** | 跨所有层 | agent 只动 brief 指定的对象、不越界 |
| 2 | **Contract-driven Hand-offs** | Types + Service | 上游承诺的 artifacts、下游必须验证 |
| 3 | **Speculative / Parallel Exploration** | Service | 跑 N 条 alternative 路径、取最佳 |
| 4 | **Agent-as-Judge** | Service | 用一个 agent 评另一个的输出 |
| 5 | **PAR Loop** | Service | plan → execute → critique → revise → re-execute |
| 6 | **Hierarchical Task Decomposition** | Service | supervisor → worker → sub-worker |
| 7 | **Autonomy Gradients** | Config | 不同任务给不同自主权 |
| 8 | **Cost-aware Budget Gates** | Config | 超预算自动停或升级审核 |
| 9 | **Failure Injection / Chaos Eval** | Service | 故意给 broken input 看怎么处理 |
| 10 | **Self-organizing Teams** | Service | agents 根据任务动态分工 |
| 11 | **Spec-driven Development** | Types | agent task 由 formal spec 定义 |
| 12 | **Graceful Degradation** | Config | frontier model 挂掉回退到便宜 model |

> **Work Boundary（#1）是贯穿全部 12 个概念的 root discipline**。

## 🔗 关联知识

- [[Harness Engineering 工程]] — 工作边界是 Harness 的自主权设定
- [[5条驭驭工程原则]] — Legibility / SoR / PD / Invariants / Throughput Merge + 工作边界 = 完整约束
- [[多智能体协作]] — 案例1和2都是 multi-agent 越界事故
- [[上下文腐蚀与注意力衰减]] — 案例1的底层原因
- [[智能体评估]] — Failure Injection / Chaos Eval 是 eval 的进阶形态
- [[上下文工程三层Stack]] — 4层工作边界和3层stack正交
- [[工程设计与安全]] — Autonomy gradient + mechanical gate
- [[任务分解]] — Hierarchical Task Decomposition 的核心能力
- [[智能体（Agent）]] — 工作边界定义了 Agent 的权限范围
- [[AI Coding Agent Token 成本控制]] — Orchestrator-Worker 模式中的 Worker 工作边界裁剪

## 🏷️ 标签

#概念卡片 #永久笔记 #概念 #智能体 #驭驭工程 #工作边界 #自主权