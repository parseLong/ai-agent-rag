---
title: 上下文工程三层Stack
description: Prompt→Context→Harness三层工程分工——不是call一次vs多次的区别，而是每一层工程的对象不同（字符串→信息→外围runtime）
tags:
  - 概念卡片
  - 永久笔记
  - 概念
  - 智能体
  - 上下文工程
  - 提示工程
  - 驭驭工程
  - 三层Stack
aliases:
  - 三层工程Stack
  - Prompt-Context-Harness Stack
  - 三层工程分工
date: 2026-06-11
source:
---

# 上下文工程三层Stack

> LLM-powered system 的工程实践可以分成三层 stack——每一层工程的对象**不一样**：Prompt = 字符串、Context = 信息、Harness = runtime。

## 🧠 核心洞察

这三层不是"1 次 call vs N 次 call"的区别，而是**每一层工程的对象不同**：

| Discipline | 工程"什么" | 核心问题 | 在哪学 |
|---|---|---|---|
| **1. [[提示工程]]** | 送进 LLM 的**字符串本身**（system prompt / few-shot / format） | 这一次要怎么问？ | Stage 2 |
| **2. [[Context Engineering]]** | [[上下文窗口]]里装**什么信息**（RAG / memory / tool defs / history） | 这次该给模型哪些信息？ | Stage 6 |
| **3. [[Harness Engineering 工程]]** | 模型**外围的执行与控制层**（agent loop / retry / sandbox / observability） | 整个流程怎么跑起来？ | Stage 7 |

> [!important] 三层正交
> 一次 call 的 RAG app 也在做**上下文工程**（重点是组 context，不是 call 几次）；50 次 call 但没做 retrieval 的 chatbot，仍然只是在做**提示工程**。

## 🔑 Karpathy 的定义

> [!quote] Karpathy 2025-06
> [[Context Engineering]]是把**刚好对下一步有用的信息**填进[[上下文窗口]]的精细艺术。

## 🔑 Simon Willison / Addy Osmani

> "coding agent = LLM + harness" — harness 是"模型外围的控制系统"、retry / loop / 监测 / 沙盒 / 部署这些不是 LLM 本身的代码。[OpenAI 也在 2026-02 使用了 "Harness Engineering" 这个说法](https://openai.com/index/harness-engineering)。

## 🔑 Claude Code 7-Layer Architecture Map 中的三层 overlay

Claude Code 的 7-Layer Architecture Map 中，3 个 discipline overlay 跨层分布：

| Discipline | 负责哪些 Layer | 1 句话 |
|---|---|---|
| **Prompt Engineering** | L1（Foundation）+ L6（Workflow） | "送进 LLM 的字符串怎么设计" |
| **Context Engineering** | L4（Memory/Context）+ L2.5（Tool Provider） | "context window 装什么信息" |
| **Harness Engineering** | L3（Control Plane）+ L5（Coordination）+ L7（Interface） | "LLM 外面的 runtime scaffolding" |

> [[MCP]] 的特殊位置：MCP 严格说是[[Context Engineering]]（feed context source）+ Tool design 跨层东西，标为 Layer 2.5。

## 🔑 怎么分辨自己在做哪一层？

1. 我改的是**字符串本身**吗？→ [[提示工程]]
2. 我改的是**塞进窗口的信息**吗？→ [[Context Engineering]]
3. 我改的是**调用模型的外围程序**吗？→ [[Harness Engineering 工程]]

**三层独立但互补**——学会其中一个，不会自动会另一个。

## 🔑 实例对照

| 场景 | 做的是哪一层？ |
|---|---|
| 写 system prompt 让 LLM 回 JSON | **提示工程** |
| 把 RAG retrieve 的3个 chunk 拼进 prompt | **上下文工程** |
| 在 agent loop 里加 max_iter + retry | **驭驭工程** |
| 给 subagent 指定 `model: haiku` 省 cost | **驭驭工程**（Cost / Latency 层） |
| `/compact` 压缩对话历史 | **上下文工程**（[[上下文压缩]]） |

## 🔗 关联知识

- [[提示工程]] — 第1层：字符串设计
- [[Context Engineering]] — 第2层：信息组装
- [[Harness Engineering 工程]] — 第3层：runtime 控制
- [[上下文窗口]] — 三层都受此约束
- [[上下文窗口管理]] — 第2层和第3层的交汇
- [[RAG]] — 第2层的核心实现
- [[记忆系统]] — 第2层的信息来源
- [[智能体循环]] — 第3层的核心元件
- [[MCP]] — 跨第2层和第3层的工具协议
- [[Skills技能]] — 跨第1层和第3层的行为包
- [[Claude Code Subagent]] — 第3层 Coordination 的实现
- [[AI Coding Agent Token 成本控制]] — 五层成本优化框架，是三层Stack在Token成本维度的完整实践案例
- [[模型路由]] — Skill/Agent/Command绑定模型属于 Harness Engineering（第3层）
- [[Schema 设计速查表]] — 第1层（description）+ 第3层（Tool Registry）

## 🏷️ 标签

#概念卡片 #永久笔记 #概念 #智能体 #上下文工程 #提示工程 #驭驭工程 #三层Stack