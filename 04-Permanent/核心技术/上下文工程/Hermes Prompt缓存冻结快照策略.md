---
date: 2026-06-15
tags:
  - 方法
  - Agent框架
  - Hermes
  - Prompt缓存
  - 上下文工程
source: "[[OpenClaw与Hermes源码架构深度复盘]]"
---

# Hermes Prompt 缓存冻结快照策略

> [!quote] 核心定义
> Hermes 用 `system_and_3` 策略主动注入 4 个 Anthropic `cache_control` 断点，首轮构建后冻结系统提示——即使 Agent 在对话中写入了新记忆，当前轮次的系统提示也不会被修改。这保证了 Anthropic prefix cache 在整个会话期间持续命中，代价是**记忆延迟一个会话**。

---

## 4 个 Cache 断点

| 断点 | 目标 | 稳定性 |
|------|------|--------|
| **1** | System prompt | 跨所有轮次稳定（缓存命中率最高） |
| **2–4** | 最后3条非system消息 | 滚动窗口（最近的对话最可能被复用） |

策略名：`system_and_3` — Anthropic 最大允许的 4 个 `cache_control` 断点。

---

## 启用条件

```python
# 仅在 OpenRouter + Claude 或原生 Anthropic API 时启用
self._use_prompt_caching = (is_openrouter and is_claude) or is_native_anthropic
```

---

## 冻结快照的核心思想

系统提示在首轮构建后被缓存到 session DB。即使 Agent 在对话中写入了新的记忆，当前轮次的系统提示也**不会被修改**——新记忆只在下一个会话开始时才注入系统提示。

> [!important] 代价与收益
> - **收益**：Anthropic prefix cache 在整个会话期间持续命中 → **~75% 输入token成本节省**
> - **代价**：记忆**延迟一个会话**——当前轮次看不到本轮新增的记忆

---

## 与 OpenClaw 的对比

| 维度 | OpenClaw | Hermes |
|------|----------|--------|
| **策略** | 依赖 Provider 侧缓存 | 主动注入 4 个 `cache_control` 断点 |
| **系统提示** | 每次 `buildPrompt()` 动态构建 | 首轮构建后**冻结** |
| **记忆注入时机** | 每次 Prompt 构建时 | **仅新会话开始时** |
| **成本节省** | 取决于Provider | ~75% 输入token成本 |
| **动态性** | 高（记忆实时反映） | 低（记忆延迟一个会话） |

> [!tip] "动态性 vs 命中率"的取舍
> 这是没有"正确答案"的工程取舍：成本敏感选 Hermes（冻结快照），记忆驱动场景选 OpenClaw（动态构建）。

---

## 相关笔记

- [[上下文预算与资源分配]] — Token Budget 和 SAFETY_MARGIN=1.2
- [[上下文压缩]] — Compaction vs 缓存的不同上下文管理手段
- [[Hermes Agent 框架]] — Hermes完整架构
- [[OpenClaw vs Hermes 架构对比]] — Prompt缓存对比
- [[上下文重置与交接]] — 冻结快照是隐式的"部分reset"
