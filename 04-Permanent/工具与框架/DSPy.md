---
title: DSPy
description: 不写Prompt，用程序自动优化——定义签名（输入/输出类型），DSPy编译出最佳prompt和retriever设置
tags: [概念卡片, 永久笔记, 概念, 智能体, 框架, DSPy, 自动优化, Path3范式]
aliases:
  - DSPy框架
  - Declarative Self-improving Python
date: 2026-06-11
source: "[[Stage 2 — 提示工程]], [[Stage 6 — 记忆与 RAG]]"
---

# DSPy

> DSPy 消除了手动编写 prompt — 你只需定义"签名"（输入/输出类型），编写程序；DSPy 会用 LLM 编译出最佳的 prompt 和 retriever 设置。这是 **Path 3 范式**：程序自动搜索最佳 prompt + retriever 组合。

## 🧠 核心洞察

传统 LLM 应用有两条路径：
- **Path 1**：手动编写 prompt（[[提示工程]]）
- **Path 2**：将 reflection 训练进模型权重（reasoning model）

DSPy 开辟了 **Path 3**：**用程序自动搜索最佳 prompt 和配置**。不需要手动调 prompt，也不需要训练模型。

## 🔑 核心概念

| 概念 | 是什么 | 作用 |
|---|---|---|
| **Signature** | 输入/输出类型定义 | `"question -> answer"` 或 `"document, question -> summary, confidence"` |
| **Module** | 可组合的 DSPy 组件 | 类似 PyTorch module，可嵌套组合 |
| **Compiler / Optimizer** | 自动搜索最佳配置 | 用 LLM 试不同 prompt + retriever 设置，选出最佳组合 |
| **Teleprompter** | 具体优化策略 | `BootstrapFewShot` / `MIPROv2` / `BootstrapFewShotWithRandomSearch` |

## 🔑 三条路径对比

| 维度 | Path 1（手动 Prompt） | Path 2（训练进权重） | **Path 3（DSPy）** |
|---|---|---|---|
| **优化对象** | 手动写的字符串 | 模型权重 | Prompt + Retriever 配置 |
| **谁来优化** | 人类 | 训练数据 | DSPy compiler |
| **维护难度** | 6 个月后 prompt 积累难维护 | 训练成本高 | 程序化管理、可自动重新编译 |
| **跨 LLM 适配** | 换 provider 要重写 | 换模型要重训 | **切换 provider → DSPy 自动重新编译** |
| **适合谁** | LLM 新手、简单任务 | 有训练资源的大团队 | **RAG/Agent prompt 已积累半年、想自动化优化** |

## 🔑 何时使用 DSPy

✅ **适合**：
- RAG prompt 已积累 6 个月，维护困难，想自动优化
- 同一程序需要切换不同 LLM provider
- Agent 系统有多个步骤，想跟踪 metrics 和 traces

❌ **不适合**：
- 你只有一个 prompt，不需要优化
- 你是 LLM 新手，还没摸过 prompting

> [!important] DSPy 是 Framework 不是 Tutorial
> DSPy 门槛较高。**搭配 [dspy.ai 官方 tutorial](https://dspy.ai/learn/) 读**，先跑 dair-ai guide（理论） → 再学 DSPy（自动化）。

## 🔑 与 RAG 的关系

DSPy 对 RAG 特别有用——传统 RAG 需要手动调整 chunk size / top-k / retriever model / prompt，DSPy **自动搜索这些配置的最佳组合**：

```python
# 定义签名
class RAGSignature(dspy.Signature):
    """Answer questions given context."""
    context: str = dspy.InputField()
    question: str = dspy.InputField()
    answer: str = dspy.OutputField()

# 定义模块
class RAGModule(dspy.Module):
    def __init__(self):
        self.retriever = dspy.Retriever()
        self.generator = dspy.ChainOfThought(RAGSignature)
    
    def forward(self, question):
        context = self.retriever(question)
        return self.generator(context=context, question=question)

# 编译（自动优化）
optimizer = dspy.MIPROv2(metric=my_metric, num_threads=4)
optimized_rag = optimizer.compile(RAGModule(), trainset=my_trainset)
```

## 🔑 Stars & License

[stanfordnlp/dspy](https://github.com/stanfordnlp/dspy) ★ 34k+、MIT

## 🔗 关联知识

- [[提示工程]] — Path 1（手动 Prompt），DSPy 是 Path 3 的自动化替代
- [[RAG]] — DSPy 自动优化 RAG 的 prompt + retriever 配置
- [[CoT]] — DSPy 的 `ChainOfThought` module 自动优化 CoT prompt
- [[智能体评估]] — DSPy compiler 用 metric 做评估驱动优化
- [[ReAct 范式]] — DSPy 可以优化 ReAct 的 prompt 配置
- [[Harness Engineering 工程]] — DSPy compiler 属于 Eval + Optimization 层面
- [[大语言模型]] — DSPy 编译出的 prompt 适配特定 LLM
- [[上下文工程]] — DSPy 自动决定 context 中放什么信息

## 🏷️ 标签

#概念卡片 #永久笔记 #概念 #智能体 #框架 #DSPy #自动优化 #Path3范式