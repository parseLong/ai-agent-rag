---
title: 驾驭工程
description: Agent外围的执行与控制层——把LLM、tools、记忆、state、错误处理、评估与部署串成可执行、可观测、可维护的agent系统
tags:
  - 概念卡片
  - 永久笔记
  - 概念
  - 智能体
  - 驭驭工程
  - Harness
  - 核心工程学科
aliases:
  - Harness Engineering
  - Agent执行系统设计
  - 驭驭层
date: 2026-06-11
source: "[[Stage 7 — 多智能体与生产部署]]"
---
# 驾驭工程

> 驭驭工程（Harness Engineering）= 把 LLM、[[工具调用（Function Calling）]]、[[记忆系统]]、state、workflow control、错误处理、[[智能体评估]]、observability 与 deployment 串成一套**可执行、可观测、可维护**的 agent 系统。

## 🧠 核心洞察

**LLM 像大脑，Harness 像身体**。LLM 负责"想"（推理/决策），Harness 负责"跑"（执行/控制/监控）。没有 Harness，LLM 只是聊天机器人；有了 Harness，它变成能自主完成多步骤任务的 agent。

> [!quote] Simon Willison 2025
> "coding agent = LLM + harness"；harness = 所有**不是 model 本身**的代码。

> [!quote] OpenAI 2026-02
> OpenAI 也使用 "Harness Engineering" 这个说法。

## 🔑 三层工程分工

工程分工分成三层，对应 stack 的不同位置（不是"call 一次 vs 多次"的差别）：

| 层级               | 工程的对象           | 核心问题        | 对应 stage |
| ---------------- | --------------- | ----------- | -------- |
| **1. [[提示工程]]**  | 送进 LLM 的**字符串** | 这一次要怎么问？    | Stage 2  |
| **2. [[Context Engineering]]** | 窗口里装的**信息**     | 这次该给模型哪些信息？ | Stage 6  |
| **3. 驾驭工程**（本概念） | 模型**外围的执行与控制层** | 整个流程怎么跑起来？  | Stage 7  |

> [!important] 三层正交
> 一次 call 的 RAG app 也在做[[Context Engineering]]（重点是组 context）；50 次 call 但没做 retrieval 的 chatbot，仍然只是在做[[提示工程]]。**三层是不同工程位置，不是 call 几次的区别**。

## 🏗 8 个核心元件

所有不属于 model weights、也不只是 prompt string 本身的工程元件都算 Harness 范围：

| # | 元件 | 做什么 | 关键点 |
|---|---|---|---|
| 1 | **[[智能体循环]]** | "LLM → tool → result → LLM" 循环 | `max_iter` safety net 必须设 |
| 2 | **Tool registry** | 动态 tool dispatch、permission gate | Schema 质量决定 LLM 选择准确度 |
| 3 | **[[上下文窗口管理]]** | message history 管理、auto-compact | `/compact` 释放 context |
| 4 | **Safety layer** | permission prompts、sandboxed exec | 破坏性操作必须拦截 |
| 5 | **Retry / recovery** | tool fail 怎么处理 | error 回传**结构化 dict**、不要 `raise` |
| 6 | **Telemetry / Observability** | metrics、logging、trace | 没 observability 的 agent debug = 黑盒 |
| 7 | **[[智能体评估]] harness** | regression test、quality gate | 自有 eval set > 外部 benchmark |
| 8 | **Cost / Latency optimization** | prompt caching、model routing、batching | 2026必修，会用省50-90% |

## 🔑 Framework vs Harness

| 维度 | Framework（Stage 4） | Harness（本概念） |
|---|---|---|
| 规范 | **API** — 你调用的接口长什么样 | **runtime** — 怎么跑、怎么 recovery、怎么观测 |
| 例子 | [[LangGraph]] 的 StateGraph API | Claude Code 的 permission + retry + telemetry |

**Framework 是脚手架，Harness 是运行时**。两者不冲突、但解决的是不同层级的问题。

## 🔑 Cost / Latency Optimization 速查

| 技巧 | 怎么省 | 2026状态 |
|---|---|---|
| Prompt caching | 重复 prefix 一次计费、后续 ~90%折扣 | Anthropic/OpenAI/Gemini 全支持 |
| Model routing / cascade | 简单→小model、难→frontier | [RouteLLM](https://github.com/lm-sys/RouteLLM) / OpenRouter |
| Thinking budget | 控制 reasoning model token 上限 | Claude/Gemini API 参数 |
| Semantic caching | 相似 query 共享回答 | GPTCache / Helicone 内建 |
| Batching | 多 query 并行处理 | vLLM / production inference |

## 🔗 关联知识

- [[提示工程]] — 三层 stack 的第1层
- [[Context Engineering]] — 三层 stack 的第2层
- [[智能体循环]] — Harness 元件 #1
- [[上下文窗口管理]] — Harness 元件 #3
- [[智能体评估]] — Harness 元件 #7
- [[工具调用（Function Calling）]] — Tool registry 的底层机制
- [[Schema 设计速查表]] — Tool schema 质量直接影响 Harness 稳定性
- [[工程设计与安全]] — Safety layer 的工程实践
- [[度量驱动演进闭环]] — Eval + Observability 的循环
- [[5条驭驭工程原则]] — Legibility / SoR / PD / Invariants / Throughput Merge
- [[Agent 工作边界]] — Harness 的自主权范围设定
- [[Claude Code 源码架构]] — Reference harness 实现
- [[运行时环境（Runtime）]] — Harness 的底层依赖

## 🏷️ 标签

#概念卡片 #永久笔记 #概念 #智能体 #驭驭工程 #Harness #核心工程学科