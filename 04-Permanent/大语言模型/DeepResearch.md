---
tags:
  - 概念
  - LLM
aliases:
  - DeepResearch
  - OpenAI DeepResearch
  - 深度研究
---
# DeepResearch

> OpenAI于2025年2月推出的端到端研究型语言模型，==从零训练==（非o3套壳），模型内部自主完成搜索任务，无需外部调用。

## 核心要点

- **不是普通LLM**：是==研究型语言模型==（Research Language Model），专为端到端完成搜索类任务设计
- **自主掌握浏览能力**：通过RL自主学会搜索、点击、滚动、理解文件，无需提示词或人工流程
- **端到端实现**：模型内部完成全部搜索→分析→合成流程，All-In-One Agent典范

> [!important] 模型即产品
> DeepResearch是==模型即产品==（The Model is the Product）的标志性案例——不是在推理模型外面套Agent壳，而是让模型本身内生Agent能力。这是从"外挂式Agent"到"==模型化Agent=="的范式转变。

## 关联知识

- [[CoT]] — DeepResearch使用长链思维进行深度推理
- [[GRPO]] — DeepSeek-R1的RL训练方法
- [[智能体（Agent）]] — DeepResearch是模型化Agent的里程碑
- [[RAG]] — DeepResearch内化了RAG的外部检索能力

## 参考资料

- [[万字详解大模型应用发展：RAG、MCP、Agent的爆发之旅]]