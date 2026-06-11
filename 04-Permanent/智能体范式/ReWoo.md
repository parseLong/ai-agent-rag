---
tags:
  - 概念
  - 智能体范式
aliases:
  - ReWoo
  - Reason without Observation
---
# ReWoo

> 无需观察的规划——Planner生成一次性完整工具链，Worker执行，Solver合并结果。减少token消耗和执行时间。

## 核心要点

- **三大组件**：
  - **Planner**：规划器，将任务分解为包含多个相互关联计划的蓝图
  - **Worker**：执行器，根据蓝图使用外部工具获取证据或执行动作
  - **Solver**：合并器，将所有计划和证据结合形成最终解决方案

- **与ReAct的区别**：
  - ReAct：每次行动后都要观察→再思考→再行动，==多次LLM调用==，token消耗大
  - ReWoo：==一次性生成完整工具链==，减少token消耗和执行时间

- **微调优势**：规划数据==不依赖工具输出==，可以在不实际调用工具的情况下微调模型

## 关联知识

- [[ReAct 范式]] — ReWoo是对ReAct"多次观察"模式的优化
- [[Plan-and-Execute]] — ReWoo没有Replan机制，Plan-and-Execute加入了动态调整
- [[LLM Compiler]] — ReWoo的进化版，引入DAG依赖关系实现并行执行

## 参考资料

- [[万字详解大模型应用发展：RAG、MCP、Agent的爆发之旅]]