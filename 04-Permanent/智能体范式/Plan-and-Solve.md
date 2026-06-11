---
tags:
  - 概念
  - 智能体范式
aliases:
  - Plan-and-Execute
  - Plan-and-Solve
  - Plan-Execute-Replan
---
# Plan-and-Solve

> 先制定多步计划，再逐步执行，根据执行情况动态调整计划。适合复杂且任务关系明确的长期任务。

## 核心要点

- **三阶段**：Plan（规划，LLM拆解任务为子任务）→Execute（按序执行每个子任务）→Replan（执行中发现原计划不合适就动态调整）
- **与ReAct的核心区别**：ReAct边想边做每步单独决策，Plan-Execute-Replan先全局规划再执行

| 维度 | ReAct | Plan-Execute-Replan |
|------|-------|---------------------|
| 规划方式 | 边想边做 | 先全局规划再执行 |
| 灵活性 | 高，实时响应 | 中，Replan有滞后 |
| 适用场景 | 动态交互、对话 | 复杂多步、数据分析 |

- **组合使用**：复杂任务先用Plan-Execute-Replan做粗粒度规划，每个子任务再用ReAct做细粒度执行

## 关联知识

- [[ReAct 范式]] — 边想边做的对比范式
- [[任务分解]] — Plan阶段的任务拆解策略
- [[智能体循环]] — 执行阶段的底层机制

## 参考资料

- [[万字详解面试题库｜Agent篇]]
- [[详尽地带你从零开始设计实现一个AI Agent框架]]