---
title: ReAct 范式
description: 推理与行动结合的 Agent 范式
tags: [概念卡片, 永久笔记, 智能体, 方法]
date: 2025-05-15
---

# ReAct 范式

## 📖 定义

ReAct（Reasoning + Acting）是一种将**推理（Reasoning）**与**行动（Acting）**结合的 Agent 范式。它让 LLM 在解决问题时，不仅能生成答案，还能主动调用工具获取信息。

## 🧠 我的理解

ReAct = Re（推理）+ Act（行动）。

传统 LLM 只能"想"，不能"做"。ReAct 让 LLM 既能思考，又能行动，并通过观察结果反馈来修正思考。

## 🎯 为什么重要

ReAct 是理解现代 Agent 框架的基石。它展示了如何让 LLM 从"被动回答"转变为"主动解决问题"。

## 🔗 关联知识

- **相似概念**：[[Plan-and-Solve]]、[[Reflection 范式]]
- **前置知识**：[[智能体（Agent）]]、[[智能体循环]]
- **应用场景**：[[智能旅行助手]]、[[自动化深度研究智能体]]

## 💡 实例/案例

**示例流程**：

```
用户：今天北京天气怎么样？适合去哪里玩？

Agent 思考（Thought）：用户想知道天气和景点推荐。
Agent 行动（Action）：调用 get_weather("北京")

观察（Observation）：北京天气晴，25°C

Agent 思考（Thought）：天气很好，适合户外活动。
Agent 行动（Action）：调用 get_attraction("北京", "晴")

观察（Observation）：推荐颐和园、长城

Agent 思考（Thought）：已获得足够信息，可以回答用户。
Agent 行动（Action）：Finish[今天北京天气晴朗，适合外出。推荐去颐和园或长城。]
```

## 📝 个人笔记

ReAct 的核心洞见是：**推理和行动应该交替进行**，而不是先全部推理完再行动。这样 Agent 可以根据最新的观察动态调整计划。

## 🏷️ 标签

#概念卡片 #永久笔记 #智能体 #方法 #ReAct
