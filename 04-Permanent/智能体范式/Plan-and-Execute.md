---
tags:
  - 概念
  - 智能体范式
aliases:
  - Plan-and-Execute
  - Plan and Execute
  - 先规划后执行
---
# Plan-and-Execute

> 先把用户问题分解为子任务，执行各子任务，并根据执行情况==动态调整计划==。相比ReWoo最大的不同是加入了Replan机制。

## 核心要点

- **三大组件**：
  - **Planner**：规划器，生成多步计划完成大任务
  - **Executor**：执行器，调用工具完成各步骤
  - **Replanner**：重规划器，根据实际执行反馈==动态调整==计划

- **核心创新**：加入了==Replan机制==，这是与ReWoo最大的区别。执行不是一次性完成，而是根据反馈迭代调整

## 关联知识

- [[ReAct 范式]] — Plan-and-Execute是ReAct"边思考边行动"的替代方案
- [[ReWoo]] — ReWoo无Replan机制，Plan-and-Execute加入动态调整
- [[LLM Compiler]] — 更进一步的并行执行方案

## 参考资料

- [[万字详解大模型应用发展：RAG、MCP、Agent的爆发之旅]]