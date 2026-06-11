---
tags:
  - 概念
  - 智能体范式
aliases:
  - LLM Compiler
  - LLM编译器
---
# LLM Compiler

> ReWoo的进化版，Planner输出有向无环图（DAG），明确依赖关系后==并行执行==任务，实现类似处理器"乱序执行"的效果。

## 核心要点

- **三大组件**：
  - **Planner**：输出流式DAG，每个任务包含工具、参数和==依赖项列表==（相比ReWoo的最大不同）
  - **Task Fetching Unit**：调度并执行任务，一旦满足依赖性就安排执行，==并行性显著提速==
  - **Joiner**：LLM根据完整历史记录决定是否输出最终答案或将进度重新传递回Planner

- **核心创新**：在ReWoo的变量分配基础上，进一步训练LLM生成==DAG==类的规划，明确各步骤依赖关系→并行执行→类似处理器乱序执行

## 关联知识

- [[ReWoo]] — LLM Compiler是ReWoo的进化版，增加了依赖项列表和并行调度
- [[Plan-and-Execute]] — Plan-and-Execute有Replan机制但无并行执行
- [[ReAct 范式]] — LLM Compiler从根本上减少了ReAct模式的多轮LLM调用开销

## 参考资料

- [[万字详解大模型应用发展：RAG、MCP、Agent的爆发之旅]]