---
title: CodeAct
description: Agent写代码当作action的范式——不同于JSON工具调用的另一条路线，agent直接写Python代码执行操作
tags: [概念卡片, 永久笔记, 概念, 智能体, 范式, CodeAct, 代码即行动]
aliases:
  - CodeAct pattern
  - 代码即行动
  - Code-as-Action
date: 2026-06-11
source: "[[Stage 3 — 工具使用与第一个 Agent]], [[Stage 4 — Agent 框架]]"
---

# CodeAct

> CodeAct = Agent **直接写代码**（通常是 Python）当作行动（Action），而非通过 JSON [[工具调用（Function Calling）]]调用预定义函数。是 ReAct 范式中 "Act" 部分的另一条实作路线。

## 🧠 核心洞察

传统 ReAct 范式的 Action 是调用预定义的 JSON tool（Function Calling）；CodeAct 让 agent **自己写 Python 代码来执行操作**。两条路线对比：

| 维度 | JSON Tool Calling | CodeAct |
|---|---|---|
| **Action 形式** | 调用预定义函数 + JSON 参数 | 自己写 Python 代码 |
| **灵活性** | 受限于预定义 schema | 可以组合任意操作 |
| **安全性** | Schema 约束参数范围 | 需要 sandbox 防止危险代码 |
| **适用场景** | API 调用、结构化查询 | 数据处理、复杂计算、需要组合多种操作 |
| **代表框架** | [[LangGraph]] / [[CrewAI]] / OpenAI Agents SDK | Smolagents / Aider / Claude Code |

## 🔑 为什么需要 CodeAct

[[工具调用（Function Calling）]] 有三个痛点：

1. **极高的开发维护成本** — 现实中大量系统没有现成 API 可供调用
2. **API Schema 管理膨胀** — 随着工具数量膨胀，管理变得极其复杂
3. **人为适配模型** — 需要把系统能力封装成标准 API，本质是"人为适配模型"

CodeAct **从"人为适配模型"转向"利用模型原生能力"** — 模型本身就会写代码，让它直接用代码执行，不需要人类预先封装每个 API。

## 🔑 代表框架

| Framework | 特点 | Stars | License |
|---|---|---|---|
| [HuggingFace Smolagents](https://github.com/huggingface/smolagents) | CodeAct pattern 代表，≤1000 LOC | 27k+ | Apache 2.0 |
| [QuantaLogic/quantalogic](https://github.com/quantalogic/quantalogic) | 另一条 CodeAct 路线 | — | Apache 2.0 |
| [Aider](https://github.com/paul-gauthier/aider) | AI pair programming、代码编辑即 action | — | Apache 2.0 |
| [Claude Code](https://github.com/anthropics/claude-code) | 写 Bash 命令当 action | — | — |

## 🔑 两条路线怎么选

| 你的情况 | 建议 |
|---|---|
| 操作固定、API 已有 | JSON Tool Calling（Stage 3 练习1-6） |
| 操作灵活、需要组合 | CodeAct（Smolagents） |
| 安全性要求高 | JSON Tool Calling（Schema 约束更可控） |
| 想省开发成本 | CodeAct（不用为每个操作写 API wrapper） |
| Production 部署 | 两者混用 — 稳定操作用 JSON tool、灵活操作用 CodeAct |

> [!important] Punchline
> CodeAct 和 JSON Tool Calling **不是互斥**，而是互补。Production agent 很可能两者都使用 — 稳定操作走 Function Calling、灵活操作走代码执行。

## 🔑 安全考量

CodeAct 需要更强的 sandbox 机制：

- **隔离执行环境** — agent 写的代码在 sandbox 里跑、不能污染 host
- **限制文件系统访问** — 只允许操作特定目录
- **禁止危险操作** — 网络/系统调用需明确授权
- **结果验证** — 执行后检查输出是否符合预期

> 这就是[[Harness Engineering 工程]] Safety layer 和 Tool registry 在 CodeAct 路线上更重要的原因。

## 🔗 关联知识

- [[ReAct 范式]] — CodeAct 是 ReAct 中 "Action" 的代码实现路线
- [[工具调用（Function Calling）]] — CodeAct 的对照路线（JSON tool calling）
- [[Schema 设计速查表]] — JSON 路线的优势：Schema 约束参数
- [[Harness Engineering 工程]] — CodeAct 需要更强的 Safety layer 和 sandbox
- [[智能体循环]] — 两条路线都走 "LLM → Action → Observation → LLM" 循环
- [[Smolagents]] — CodeAct 的代表框架

## 🏷️ 标签

#概念卡片 #永久笔记 #概念 #智能体 #范式 #CodeAct #代码即行动