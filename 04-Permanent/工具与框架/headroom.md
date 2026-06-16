---
title: headroom
description: 通用上下文压缩代理层——CacheAligner稳定前缀+ContentRouter按类型路由+CCR可逆压缩，实测47-92%节省
tags: [工具, Token成本, 上下文压缩, 智能体, 工程实践]
aliases:
  - headroom-ai
date: 2026-06-15
source: "[[一篇搞懂 AI Coding Agent 的 Token 成本控制]]"
github: https://github.com/chopratejas/headroom
---

# headroom

> 压缩的不是命令输出，也不是 AI 回复，而是第三个方向：**所有读进上下文的内容**——文件内容、工具调用返回值、会话历史。

## 🧠 核心洞察

![[笔记同步助手/images/064859508ed799b0f5c00397de07b84c_MD5.png]]

headroom 首先是**代理层**，不只是压缩器：
1. Agent 不直接连 LLM，先经过本地 headroom 代理入口
2. 代理层内部是一条处理流水线：CacheAligner → ContentRouter → 按类型压缩器
3. CCR 负责"可逆压缩"——原始数据全部保留在本地，模型只收到压缩版；需要细节时通过 `headroom_retrieve` 按需取回

> **不是删掉信息，而是延后加载细节。**

## 🔑 内部流水线

| 组件 | 功能 |
|------|------|
| **CacheAligner** | 稳定前缀，帮助 [[KV 缓存]] 命中 |
| **ContentRouter** | 判断内容类型（JSON/代码/文本） |
| **SmartCrusher** | 处理 JSON 数据 |
| **CodeCompressor** | 处理代码 |
| **Kompress-base** | 处理文本 |
| **CCR** | 可逆压缩 + 按需还原 |

## 📊 实测数据

| 场景 | 压缩前 | 压缩后 | 节省 |
|------|--------|--------|------|
| 代码搜索（100 条结果） | 17,765 | 1,408 | **92%** |
| SRE 故障排查 | 65,694 | 5,118 | **92%** |
| GitHub Issue 分类 | 54,174 | 14,761 | **73%** |
| 代码库探索 | 78,502 | 41,254 | **47%** |

压缩后的准确率在 GSM8K、TruthfulQA 等标准基准上基本持平甚至略优。

## 🛠️ 安装与使用

```bash
pip install "headroom-ai[all]"
```

> [!note] CodeBuddy 支持
> 使用 [studyzy Fork 版](https://github.com/studyzy/headroom)（已增加 CodeBuddy 支持）。

### 接入 CodeBuddy

```bash
# 基础接入
headroom wrap codebuddy

# 推荐：同时开启跨 session 记忆 + 代码图谱集成
headroom wrap codebuddy --memory --code-graph
```

之后正常用 CodeBuddy，headroom 在后台透明处理。

### MCP Server 模式

```bash
headroom mcp install
```

安装后自动注册三个工具：`headroom_compress`、`headroom_retrieve`、`headroom_stats`。

## 🔑 与 RTK 的区别

| | [[RTK]] | headroom |
|---|---------|----------|
| **压缩对象** | 终端命令的输出 | 所有进上下文的内容 |
| **工作方式** | 过滤 + 截断 | 可逆压缩 + 按类型路由 + 按需还原 |
| **安装方式** | brew / curl | pip + wrap 命令 |
| **典型节省** | 89% | 47-92% |

两者方向不同，headroom 甚至集成了 RTK，可以叠加使用。只装一个→选 RTK（解决最大单点问题）；追求更全面覆盖→加上 headroom。

## 🔗 关联知识

- [[AI Coding Agent Token 成本控制]] — 五层优化框架中第三层的具体工具
- [[RTK]] — 压缩终端命令输出，headroom 集成了 RTK 可叠加
- [[Caveman]] — 压缩输出端，方向不同可叠加
- [[context-mode]] — 压缩 MCP 工具返回值 + 会话连续性
- [[上下文压缩]] — 第三层优化的理论基础
- [[可逆压缩]] — headroom 的 CCR 就是可逆压缩的实现
- [[KV 缓存]] — CacheAligner 的底层机制

## 🏷️ 标签

#工具 #Token成本 #上下文压缩 #智能体 #工程实践
