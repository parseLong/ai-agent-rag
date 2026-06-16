---
tags:
  - 工具
  - 工程实践
  - DeepSeek
aliases:
  - TileLang
date: 2026-06-16
source: https://huggingface.co/deepseek-ai/DeepSeek-V4-Pro/blob/main/DeepSeek_V4.pdf
---

# TileLang

> 面向 tile 的高级抽象语言——让 DeepSeek V4 快速跑得起、跑得快、跑得远的关键支点。

## 核心思想

**数据流逻辑与调度策略解耦**：
- 前者用高层抽象语言简洁表达（开发者只需描述"做什么"）
- 后者通过少量标注交由编译器优化（编译器决定"怎么做最快"）

## 为什么 V4 需要 TileLang

[[DeepSeek V4]] 包含 [[mHC（多流约束残差连接）]]、[[注意力机制]]（CSA/HCA）等多种新机制，如果纯靠 CUDA 编写算子：
- 代码量极大，开发周期长
- 手工优化容易出错
- 难以适配不同硬件后端

TileLang 编写后编译出的 kernel 性能**对齐甚至超过手工 CUTLASS/cuBLAS**，同时代码量大幅减少。

## 可移植性

同一套 TileLang 代码可针对不同后端编译：
- NVIDIA GPU
- 国产硬件

这让 V4 的创新机制能快速移植到更多硬件平台，而不需要为每种硬件重写算子。

## 关联知识

- [[DeepSeek V4]] — TileLang 是 V4 Infra 层的关键工具
- [[mHC（多流约束残差连接）]] — TileLang 用于编写 mHC 的计算内核
- [[注意力机制]] — TileLang 用于编写 CSA/HCA 的算子
- [[DeepSeek V4 Infra 层优化]] — Infra 层全景

## 参考来源

- [[读完这篇，你就搞懂 DeepSeek v4 了]] — 原文索引
