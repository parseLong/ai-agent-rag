---
tags:
  - 概念
  - 大语言模型
aliases:
  - Transformer
---
# Transformer 架构

> 2017年提出的革命性架构，通过自注意力机制重新定义NLP，是现代LLM的基础。

## 核心要点

- **核心创新**：自注意力机制让模型关注序列中每个token与所有其他token的关系，解决了RNN/LSTM处理长距离依赖的困难
- **架构组成**：编码器（Encoder）+ 解码器（Decoder），GPT系列只用解码器，BERT只用编码器
- **长上下文挑战**：标准Transformer的注意力计算复杂度是O(L²)，上下文越长计算量和显存消耗指数级增长，必须引入稀疏注意力等优化

## 关联知识

- [[注意力机制]] — Transformer的核心创新
- [[上下文窗口]] — Transformer架构对上下文长度的物理限制

## 参考资料

- [[读完这篇，你就搞懂 DeepSeek v4 了]]
- [[万字详解大模型应用发展：RAG、MCP、Agent的爆发之旅]]