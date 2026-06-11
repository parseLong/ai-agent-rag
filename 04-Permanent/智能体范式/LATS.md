---
tags:
  - 概念
  - 智能体范式
aliases:
  - LATS
  - Language Agent Tree Search
  - 语言Agent树搜索
---
# LATS

> 结合Reflection评估与蒙特卡洛树搜索（MCTS），统一了Reflexion、ToT、Plan-and-Execute等架构的推理、规划、反思组件。

## 核心要点

- **4步搜索循环**：
  1. **选择**：根据累计奖励选最佳下一步行动
  2. **展开+模拟**：并行生成N个行动方案并执行
  3. **反思+评估**：基于反思和外部反馈对决策打分
  4. **反向传播**：根据结果更新根路径分数

- **核心创新**：将RL代理、价值函数、优化器都替换为==对单个LLM的调用==；避免Agent陷入重复循环

## 关联知识

- [[CoT]] — LATS的基础推理能力
- [[ToT]] — LATS继承了ToT的树状搜索
- [[Reflection 范式]] — LATS继承了Reflexion的反思机制
- [[ReAct 范式]] — LATS统一了多种Agent架构

## 参考资料

- [[万字详解大模型应用发展：RAG、MCP、Agent的爆发之旅]]