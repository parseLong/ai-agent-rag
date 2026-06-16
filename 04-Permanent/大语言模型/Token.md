---
title: Token
description: 大语言模型处理文本的最小单元，是文本与数字之间的桥梁
tags: [概念卡片, 永久笔记, LLM, 概念, 基础]
date: 2025-05-19
---

# Token

## 📖 定义

Token 是大语言模型处理文本时的**最小单元**。在将自然语言文本输入模型之前，必须先通过**分词（Tokenization）**将其转换为模型能够处理的数字序列。Token 可以是单词、子词（subword）或字符。

## 🧠 我的理解

如果把 LLM 比作一个“文字接龙”高手，Token 就是它每次接龙时的最小单位。模型不是直接处理原始文本，而是处理文本被切分后的“词元”序列。理解 Token 对于控制成本、管理上下文长度和调试模型行为都至关重要。

## 🎯 为什么重要

- **上下文窗口的计量单位**：模型的上下文窗口（如 8K, 128K）是以 Token 数量计算的，不是字符数
- **API 计费标准**：大多数模型 API 按 Token 数量计费
- **模型表现的关键**：分词方式直接影响模型对文本的理解
- **中英文差异**：中文通常需要更多 Token 来表示相同含义

## 🔑 分词算法详解

### 1. 字节对编码（Byte-Pair Encoding, BPE）

**核心思想**：迭代合并频率最高的相邻字符对。

**训练过程**：
1. **初始化**：将词表初始化为所有在语料库中出现过的基本字符
2. **迭代合并**：在语料库上，统计所有相邻词元对的出现频率，找到频率最高的一对，将它们合并成一个新的词元，并加入词表
3. **重复**：重复第 2 步，直到词表大小达到预设的阈值

```Python
import re, collections

def get_stats(vocab):
    """统计词元对频率"""
    pairs = collections.defaultdict(int)
    for word, freq in vocab.items():
        symbols = word.split()
        for i in range(len(symbols)-1):
            pairs[symbols[i],symbols[i+1]] += freq
    return pairs

def merge_vocab(pair, v_in):
    """合并词元对"""
    v_out = {}
    bigram = re.escape(' '.join(pair))
    p = re.compile(r'(?<!\S)' + bigram + r'(?!\S)')
    for word in v_in:
        w_out = p.sub(''.join(pair), word)
        v_out[w_out] = v_in[word]
    return v_out

# 准备语料库，每个词末尾加上</w>表示结束，并切分好字符
vocab = {'h u g </w>': 1, 'p u g </w>': 1, 'p u n </w>': 1, 'b u n </w>': 1}
num_merges = 4

for i in range(num_merges):
    pairs = get_stats(vocab)
    if not pairs:
        break
    best = max(pairs, key=pairs.get)
    vocab = merge_vocab(best, vocab)
    print(f"第{i+1}次合并: {best} -> {''.join(best)}")
    print(f"新词表（部分）: {list(vocab.keys())}")
    print("-" * 20)
```

**代表模型**：GPT 系列

### 2. WordPiece

**核心思想**：与 BPE 非常相似，但合并词元的标准不是"最高频率"，而是"能最大化提升语料库的语言模型概率"。

**代表模型**：Google BERT

### 3. SentencePiece

**核心思想**：将空格也视作一个普通字符（通常用下划线 `_` 表示）。这使得分词和解码过程完全可逆，且不依赖于特定的语言。

**代表模型**：Llama 系列

## ⚠️ 分词器对开发者的意义

理解分词算法的细节并非目的，但作为智能体的开发者，理解分词器的实际影响十分重要：

- **上下文窗口限制**：模型的上下文窗口（如 8K, 128K）是以 **Token 数量** 计算的，而不是字符数或单词数。同样一段话，在不同语言（如中英文）或不同分词器下，Token 数量可能相差巨大。
- **API 成本**：大多数模型 API 都是按 Token 数量计费的。了解文本会被如何分词，是预估和控制智能体运行成本的关键一步。
- **模型表现的异常**：有时模型的奇怪表现根源在于分词。例如，模型可能很擅长计算 `2 + 2`，但对于 `2+2`（没有空格）就可能出错，因为后者可能被分词器视为一个独立的、不常见的词元。

## 🔗 关联知识

- **相似概念**：[[词]]、[[子词]]、[[字符]]
- **前置知识**：[[分词]]、[[词嵌入]]
- **应用场景**：[[上下文窗口]]、[[API 计费]]、[[提示工程]]、[[AI Coding Agent Token 成本控制]]
- **相关技术**：[[BPE]]、[[WordPiece]]、[[SentencePiece]]
- **成本优化**：[[模型路由]]、[[RTK]]、[[Caveman]]

## 💡 实例/案例

**中英文 Token 数量对比**：

| 文本 | 大致 Token 数 | 说明 |
|------|-------------|------|
| "Hello world" | 2-3 | 英文单词通常 1 个 Token |
| "你好世界" | 4-6 | 中文通常每个字 1-2 个 Token |
| "Datawhale Agent learns" | 4-5 | 常见英文单词各 1 个 Token |

**成本估算**：
- GPT-4：$0.03 / 1K input tokens
- 一篇 3000 字的中文文章 ≈ 4000-6000 tokens
- 单次调用成本 ≈ $0.12 - $0.18

## 📝 个人笔记

在设计智能体时，Token 管理是必须考虑的因素。特别是在处理长文档或需要维持长期对话记忆时，必须精确计算 Token 使用量，避免超出上下文窗口限制。同时，了解分词器的特性有助于设计更鲁棒的提示词。

## 🏷️ 标签

#概念卡片 #永久笔记 #LLM #概念 #基础
