---
title: GSSC流水线
description: 上下文构建的通用流水线模式——Gather（汇集）→ Select（选择）→ Structure（结构化）→ Compress（压缩），从 Hello-Agents 教材到 Claude Code 和 OpenClaw 的工程实践
tags: [方法, 上下文工程, 概念卡片, 永久笔记, 智能体]
aliases:
  - GSSC Pipeline
  - 上下文构建流水线
  - Gather-Select-Structure-Compress
date: 2026-06-04
---

# GSSC 流水线

> 上下文构建不是一次性拼接，而是四阶段流水线——Gather（汇集）→ Select（选择）→ Structure（结构化）→ Compress（压缩）。从 Hello-Agents 教材到 Claude Code 的 Token 转化流水线再到 OpenClaw 的 Context Engine 契约，GSSC 是上下文工程的通用模式。

## 四阶段详解

### ① Gather：多源信息汇集

将所有可能的候选信息收集起来——系统指令、对话历史、记忆检索、RAG 结果、工具输出、自定义信息包。关键设计考虑：

- **容错机制**：每个外部数据源的调用都应有 try-except 包裹，确保单个源失败不影响整体
- **优先级标记**：系统指令标记为最高优先级（relevance_score=1.0），确保始终被保留
- **历史限制**：对话历史只保留最近几条，避免窗口被历史占据

### ② Select：智能信息选择

根据相关性和新近性对候选信息评分，在 [[上下文预算与资源分配|Token 预算]] 内选择最有价值的信息：

```
综合分数 = relevance_weight × 相关性 + recency_weight × 新近性

过滤规则：
  ├─ min_relevance: 低于阈值的信息直接丢弃
  ├─ system_instructions: 系统指令不参与评分，始终保留
  └─ 贪心填充: 按分数从高到低，直到 Token 预算用完
```

> [!important] 系统指令的优先保障
> Select 阶段首先分离系统指令和其他信息，计算系统指令占用的 Token，确保剩余预算足够容纳其他信息。如果系统指令已占满所有预算，只返回系统指令——**永远不能让检索结果挤占系统指令的空间**。

### ③ Structure：结构化输出

将选中信息组织成清晰的分区模板：

```
[Role & Policies]  ── 系统指令和角色定义
[Task]             ── 当前用户查询
[Evidence]         ── RAG 检索结果和知识证据
[Context]          ── 对话历史和记忆
[Output]           ── 期望的输出格式
```

分区设计的三重优势：
- **可读性**：人类和模型都更容易理解上下文结构
- **可调试性**：快速识别哪个区域的信息有问题
- **可扩展性**：添加新信息源只需创建新分区

### ④ Compress：兜底压缩

当结构化输出仍超出 Token 预算时，执行压缩。关键原则：**保持结构完整性**——即使在预算紧张的情况下，也尽量保留每个分区的关键信息，而非简单地从头截断。

## Claude Code 的 Token 转化流水线

Claude Code 的上下文管理本质上也是 GSSC，但以更工程化的形式实现：

| GSSC 阶段 | Claude Code 实现 |
|----------|----------------|
| **Gather** | 三管道注入（System Context + User Context + CLAUDE.md Memory） |
| **Select** | CLAUDE.md 优先级合并（距离 cwd 更近的文件优先级更高）+ 工具结果 Budget 限制 |
| **Structure** | System Prompt 多段数组（7 静态段 + 10+ 动态段）+ `<system-reminder>` 标签包装 |
| **Compress** | 五层递进压缩管道（详见 [[上下文压缩]]） |

```
Claude Code 的消息组装管道：

原始消息
→ applyToolResultBudget()    — 工具结果大小限制（Gather 阶段的 Budget 限制）
→ snipCompact()              — 片段压缩（Compress 阶段的第一层）
→ microCompact()             — 微压缩（Compress 阶段的第二层）
→ contextCollapse()          — 上下文折叠（Compress 阶段的第三层）
→ autoCompact()              — 自动压缩（Compress 阶段的第四层）
→ normalizeMessagesForAPI()  — 最终组装（Structure 阶段）
```

## OpenClaw 的 Context Engine 契约

OpenClaw 把 GSSC 模式抽象为可插拔的 Context Engine 契约接口：

```typescript
interface ContextEngine {
  bootstrap?(ctx)          // 会话初始化（Gather 预准备）
  ingest(msg)              // 吸收一条 message（Gather 消息流入）
  ingestBatch?(batch)      // 吸收一轮 turn（Gather 批量流入）
  afterTurn?(ctx)          // 一轮结束后做后处理（Select 评估时机）
  assemble(budget)         // 按 tokenBudget 组装 prompt（Select + Structure）
  compact()                // 压缩（Compress）
  maintain?()              // 分支重写（Compress 后的维护）
  prepareSubagentSpawn?()  // 子 agent 派生前（为子代理做精简版 GSSC）
  onSubagentEnded?()       // 子 agent 结束后（回收子代理产出）
}
```

> [!important] 可插拔的本质
> OpenClaw 把"上下文怎么管"从 runtime 剥离——核心只管编排（调度、容错、预算），具体策略由 Context Engine 插件填充。这意味着换一个 Context Engine 就换一套 GSSC 实现，不需要改 runtime 代码。

## GSSC 在不同框架中的对比

| 维度 | Hello-Agents（教材） | Claude Code（生产） | OpenClaw（可插拔） |
|------|---------------------|-------------------|------------------|
| **Gather** | 多源显式调用 | 三管道自动注入 | Context Engine ingest |
| **Select** | 相关性+新近性评分 | 优先级合并+Budget限制 | assemble(budget) |
| **Structure** | 5 区模板 | System Prompt 多段数组 | assemble() 内部实现 |
| **Compress** | 分区截断 | 五层递进管道 | compact() + 三级触发 |
| **可扩展性** | 代码级 | 需改源码 | 契约接口可插拔 |

## GSSC 的设计目标

一个优秀的 GSSC 实现应解决以下问题：

1. **统一入口**：将四阶段抽象为可复用流水线，减少重复模板代码
2. **稳定形态**：输出固定骨架的上下文模板，便于调试和 A/B 测试
3. **预算守护**：在 Token 预算内尽量保留高价值信息，超限提供兜底压缩策略
4. **最小规则**：不引入来源/优先级等分类维度，避免复杂度增长

> [!tip] 最小规则的实践验证
> 实践表明，基于相关性和新近性的简单评分机制，在大多数场景下已经足够有效。不需要引入复杂的分类维度——越简单的规则越容易调试和维护。

## 关联知识

- [[Context Engineering]] — GSSC 是上下文工程的核心执行模式
- [[上下文注入]] — Gather 和 Structure 阶段涉及注入设计
- [[上下文压缩]] — Compress 阶段的详细实现
- [[上下文窗口管理]] — Select 阶段的核心约束是窗口限制
- [[上下文预算与资源分配]] — Budget 是 Select 阶段的关键输入
- [[上下文腐蚀与注意力衰减]] — GSSC 的目标是对抗腐蚀
- [[Claude Code 源码架构]] — Claude Code 的 GSSC 实现
- [[OpenClaw架构深度解析]] — OpenClaw 的 Context Engine 契约

## 参考资料

- [[第九章 上下文工程]] — §9.3 ContextBuilder 完整实现
- [[万字干货：理解 Harness Engineering，看这一篇就够了]] — Token 转化流水线
- [[Claude Code 源码架构]] — 消息组装管道与 System Prompt 装配
- [[OpenClaw与Hermes：源码里的 AI Agent 架构知识大复盘 1]] — §6.6 Context Engine 契约