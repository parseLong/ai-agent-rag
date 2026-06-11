---
tags:
  - 概念
  - 工具与框架
aliases:
  - LangGraph
---
# LangGraph

> LangChain团队开发的状态图框架，用图结构建模Agent关系，支持复杂的分支、循环、并行。

## 核心要点

- **三大核心概念**：
  - State：全局共享的数据结构，贯穿整个图流程
  - Node：最小执行单元，接收State返回State更新
  - Edge：节点间流转规则，普通边无条件跳转，条件边根据State动态决定下一节点

- **与LangChain Chain的区别**：Chain是链式结构数据线性流动；LangGraph是图结构支持分支循环并行。Chain像流水线，LangGraph像交通网。

- **多Agent协作**：每个Agent作为一个Node，通过State共享信息，通过Edge协调执行。常见模式有监督者模式、流水线模式、多轮协商模式。

> [!example] 循环迭代
> LangGraph通过边的循环引用实现循环（如generate→evaluate→generate），通过State中的计数器或标志位控制终止条件。

## 关联知识

- [[多智能体协作]] — LangGraph是建模Multi-Agent关系的主流框架
- [[ReAct 范式]] — LangGraph可建模ReAct的思考-行动-观察循环

## 参考资料

- [[万字详解面试题库｜框架篇（LangChainLangGraph）]]
- [[万字详解大模型应用发展：RAG、MCP、Agent的爆发之旅]]