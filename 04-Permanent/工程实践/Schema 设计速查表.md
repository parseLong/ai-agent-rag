---
title: Schema 设计速查表
description: Tool Schema 设计的5条黄金规则 + 5个常见anti-pattern——写好Schema能省下换大model的成本
tags: [概念卡片, 永久笔记, 方法, 实践, 智能体, 工具调用, Schema设计]
aliases:
  - Tool Schema Design
  - 工具Schema设计
  - Schema黄金规则
date: 2026-06-11
source: "[[Stage 3 — 工具使用与第一个 Agent]]"
---

# Schema 设计速查表

> **写好 Schema 的功夫能省下换大 model 的成本**——小 model 对 Schema 质量比大 model 敏感，相同 bad schema 在 Claude 上可能还能猜对、在 qwen 上几乎必错。Production 想用便宜 model？Schema 必须写到能上线跑的程度。

## 🔑 5 条黄金规则

| # | 规则 | Bad 示例 | Good 示例 | 为什么重要 |
|---|---|---|---|---|
| **1** | **`name` 改具体** | `convert` | `convert_temperature` | 让 LLM 一眼看出这是什么工具 |
| **2** | **`description` 写"何时用"而非"做什么"** | "Convert a value." | "Use when user asks to convert temperatures between Fahrenheit and Celsius." | 写适用情境而非 docstring——LLM 需要知道**什么时候该选这个工具** |
| **3** | **`type` 改精确类型** | `"value": {"type": "string"}` | `"value": {"type": "number", "description": "Temperature value"}` | 参数类型要精确、LLM 才能正确填值 |
| **4** | **加 `required` + `enum`** | `unit: string` | `unit: {"type": "string", "enum": ["celsius", "fahrenheit"]}` | 模糊边界用 enum 强制收敛 |
| **5** | **Error 回传包结构化信息** | `raise Exception("failed")` | `return {"error": "network timeout", "retry_hint": "try again in 1s"}` | 让 LLM 能自主 recover，而非中断 loop |

## 🚫 5 个常见 Anti-Pattern

| # | Anti-Pattern | 问题 | 修复 |
|---|---|---|---|
| **1** | **Description 写成 docstring** | "处理数据"太笼统、3 个 tool 的 description 会撞 | 改写适用情境："Use when user asks to..." |
| **2** | **参数全用 `type: string`** | LLM 不知道该传数字还是字符串、传错类型 | 精确标注 `type: number` / `type: integer` / `type: boolean` |
| **3** | **没分 required / optional** | LLM 可能漏关键参数 | 明标 `required: ["city", "date"]` |
| **4** | **该用 enum 不用** | 模糊值域让 LLM 猜错 | `enum: ["celsius", "fahrenheit"]` 收敛选择 |
| **5** | **Error 用 `raise` 中断** | LLM 没机会 recover | 回传 `{"error": "...", "retry_hint": "..."}` |

## 📊 Bad vs Good Schema 对照

```python
# ❌ BAD — qwen2.5:3b 几乎必错（Claude haiku 还能猜对但几率明显下降）
{"name": "convert",
 "description": "Convert a value.",
 "parameters": {"type": "object", "properties": {
     "value": {"type": "string"},
     "unit": {"type": "string"}}}}

# ✅ GOOD — 小 model 也能稳定挑对
{"name": "convert_temperature",
 "description": "Use when user asks to convert temperatures between Fahrenheit and Celsius.",
 "parameters": {"type": "object", "properties": {
     "value": {"type": "number", "description": "Temperature value"},
     "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]}},
     "required": ["value", "unit"]}}
```

## 🔑 Description 边界原则

**3 个 tool 的 `description` 边界要互斥**：
- `calendar` 写"日历"太笼统、会跟 `web_search` 撞
- 写"Look up events for a specific date"就清楚
- 小 model 对 description 质量比 Claude 更敏感

## 🔑 Error 回传原则

| Bad | Good |
|---|---|
| `raise Exception("failed")` | `return {"error": "network timeout", "retry_hint": "try again in 1s"}` |
| `return "failed"` | `return {"error": "...", "category": "transient", "retry_hint": "..."}` |
| 无限 retry | `max_iter` safety + 业务层 retry quota |

**核心 mental flip**：Production 的 retry 不在 Python 层、而在 LLM 层。tool error 是 data、不是 exception。

## 🔗 关联知识

- [[工具调用（Function Calling）]] — Schema 是 Function Calling 的起点
- [[Harness Engineering 工程]] — Schema 设计属于 Tool Registry 元件
- [[ReAct 范式]] — 在 ReAct loop 中 Schema 质量决定 tool 选择准确度
- [[MCP]] — MCP 把 Schema 标准化为跨平台协议
- [[智能体（Agent）]] — Agent 的工具能力由 Schema 定义
- [[提示工程]] — Description 本质上是给 LLM 的 mini-prompt

## 🏷️ 标签

#概念卡片 #永久笔记 #方法 #实践 #智能体 #工具调用 #Schema设计