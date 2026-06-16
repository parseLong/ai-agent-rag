---
date: 2026-06-15
tags:
  - 方法
  - Agent框架
  - 记忆系统
  - Hermes
  - 搜索
source: "[[OpenClaw与Hermes源码架构深度复盘]]"
---

# Session Search (Hermes)

> [!quote] 核心定义
> Hermes 独有的"翻日记本式回忆"——Agent 能搜索**过去所有对话的完整历史**（不只是经过整理的记忆摘要）。核心是 SQLite FTS5 全文索引 + 辅助 LLM 做摘要，让 Agent 在历史 session 中找到相关上下文而不会让 context 爆炸。

---

## 核心流程

```
每条消息实时写入 SQLite (WAL模式)
    ↓
FTS5 索引由触发器自动维护（标准分词 + trigram 双索引）
    ↓
Agent 调 session_search 搜索
    ↓
取 top 3 唯一 session，以匹配位置为中心截断
    ↓
并发调辅助 LLM 做摘要（max 10K tokens）
    ↓
摘要返回给主 Agent
```

---

## 关键设计决策

| 设计 | 做法 | 为什么 |
|------|------|--------|
| **摘要而非原文** | 命中session不直接返回，调辅助LLM做摘要 | 一个历史session可能几万token，直接塞context会爆 |
| **截断策略** | 25%前文 + 75%后文 | 前因已知不需要多追溯，往后展开看"后来怎么解决的" |
| **子session溯源** | 搜到delegate子session时沿`parent_session_id`链向上回溯到根session | 展示完整对话语境而非子Agent片段 |
| **排除当前session** | `current_session_id` 过滤 | Agent已有当前对话上下文 |
| **双FTS5索引** | 标准分词 + trigram 分词 | 标准分词处理英文，trigram处理中文/日文 |
| **WAL模式** | `PRAGMA journal_mode=WAL` | gateway同时服务多平台并发读写不阻塞 |

---

## DB膨胀治理

社区报告 384MB+ / 68K+ 消息时 FTS5 变慢，有 vacuum / 分库讨论。这是"全量保存"策略的已知代价。

---

## 与 OpenClaw 记忆系统的互补

| 层级 | OpenClaw做得重 | Hermes做得重 |
|------|----------------|--------------|
| **Memory层** | Dreaming三阶段加权晋升（自动沉淀） | ❌ 无等价机制 |
| **Session层** | ❌ JSONL append-only，无跨会话搜索 | **Session Search**（FTS5+LLM摘要） |

> 理想形态是两层都做好——Session层保证"找得到原始出处"，Memory层保证"不用每次都重读原始"。

---

## 相关笔记

- [[Dreaming三阶段记忆晋升]] — OpenClaw的自动记忆沉淀
- [[记忆系统]] — 通用记忆理论框架
- [[Hermes Agent 框架]] — Hermes完整架构
- [[OpenClaw vs Hermes 架构对比]] — 记忆系统对比
