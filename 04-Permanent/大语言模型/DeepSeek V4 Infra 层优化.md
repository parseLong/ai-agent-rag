---
tags:
  - 方法
  - 工程实践
  - DeepSeek
aliases:
  - V4 Infra
date: 2026-06-16
source: https://huggingface.co/deepseek-ai/DeepSeek-V4-Pro/blob/main/DeepSeek_V4.pdf
---

# DeepSeek V4 Infra 层优化

> 架构创新需要精细化 Infra 方案来"让硬件吃得饱"——以下每个子主题都有独立的原子笔记，本文仅作索引。

## 🔗 子主题索引

| 主题 | 原子笔记 | 核心内容 |
|------|---------|---------|
| 算子开发 | [[TileLang]] | 数据流逻辑与调度策略解耦，编译性能超手工CUTLASS/cuBLAS，可移植多硬件 |
| 训练可复现 | [[批无关性与计算确定性]] | bit级一致输出 + DeepGEMM + 加法括号结构钉死 |
| 部署效率 | [[FP4量化感知训练]] | 分模块量化：MoE→FP4、CSA索引→FP4、打分→BF16 |
| 训练/推理框架 | [[DeepSeek V4 训练与推理框架优化]] | Muon兼容分布式、mHC流水线调优(额外6.7%)、跨机器压缩、单tensor级重算决策 |
| KV Cache | [[KV 缓存]] | CSA/HCA+SWA异构KV + 分类持久化 + SWA三档取舍 |
| MoE通信 | [[MoE架构]] | 细粒度计算通信重叠，$V_{comp}/V_{comm} > C/B$ 时理论完全消泡 |

## 关联知识

- [[DeepSeek V4]] — Infra 层优化是 V4 系统级闭环的一部分
- [[读完这篇，你就搞懂 DeepSeek v4 了]] — 原文索引
