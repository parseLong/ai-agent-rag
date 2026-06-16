---
title: Graphify
description: 用 Tree-sitter 解析代码构建知识图谱的 Skill 形态工具——AI 查图而不是反复读文件，比直接读文件减少71.5倍Token消耗
tags: [工具, Token成本, 代码图谱, 智能体, 工程实践]
aliases:
  - graphifyy
date: 2026-06-15
source: "[[一篇搞懂 AI Coding Agent 的 Token 成本控制]]"
github: https://github.com/safishamsi/graphify
---

# Graphify

> 让 AI 在动手读文件之前，就知道该读哪里——比直接读文件减少 71.5 倍 Token 消耗。

## 🧠 核心洞察

项目越大，AI 越容易陷入"grep → 读文件 → 发现漏了 → 再 grep"的循环，消耗大量工具调用和上下文。Graphify 用 Tree-sitter 解析代码构建知识图谱，AI 查图而不是反复读文件。

## 🔑 核心机制

- 用 **Tree-sitter** 解析代码构建知识图谱
- AI 读 `graphify-out/graph.json` 而不是反复 grep 源码
- **Skill 形态**——零配置，`/graphify` 一键生成

## 📊 产出物

执行 `/graphify` 后自动生成三份产物：

```
> /graphify

Building knowledge graph...
  Parsed 247 files (TS/Prisma)
  Found 1,832 symbols, 4,217 edges
  Detected 14 communities (clusters)
  Generated:
    - graphify-out/graph.html          ← 交互式可视化
    - graphify-out/GRAPH_REPORT.md     ← 自然语言报告
    - graphify-out/graph.json          ← AI 查询用的数据
```

后续问跨文件问题时，AI 读 `graphify-out/graph.json`：

```
> 我想在 /api/v1/comments 加一个 PATCH 接口。
> 列出会受到影响的所有中间件、Prisma model、和测试文件。

影响分析（基于知识图谱）：

中间件链：
  - middleware/auth.ts → optionalAuth → requireAuth
  - middleware/rate-limit.ts（commenting 路径 30/min 限制）

Prisma model：
  - Comment（直接修改）
  - User（通过 authorId 关联）
  - Mention（通过 commentId 关联，PATCH 后需要重算）

测试：
  - tests/comments.test.ts
  - tests/integration/registration.test.ts（间接依赖）
```

## 🛠️ 安装

```bash
# 安装
uv tool install graphifyy
# 或
pipx install graphifyy

# 注册到 CodeBuddy
graphify install --platform codebuddy
# 或
graphify codebuddy install
```

## 🔑 支持范围

- **30+ 种语言**：Python、TypeScript、Go、Rust、Java、C/C++、Ruby、C#、Kotlin 等
- **多模态**：SQL schema、Markdown 文档、PDF、图片（通过 LLM 提取语义）
- **增量更新**：git hook 触发，每次 commit 自动重建

## 🔑 与 CodeGraph 的区别

| | Graphify | [[CodeGraph]] |
|---|----------|-------------|
| **形态** | Skill 形态 | MCP Server |
| **配置** | 零配置 | 需要配置 MCP + Neo4j/KuzuDB |
| **持久化** | 生成 JSON 文件 | 持久化图数据库 |
| **适合** | 个人项目、快速上手 | 团队、大型仓库、需要持久化查询 |

**选哪个**：个人项目、快速上手 → Graphify；团队、大型仓库 → CodeGraph。

## 🔗 关联知识

- [[AI Coding Agent Token 成本控制]] — 五层优化框架中第四层的具体工具
- [[CodeGraph]] — 另一个代码图谱工具，MCP Server 形态
- [[上下文压缩]] — 第三层优化，与第四层互补

## 🏷️ 标签

#工具 #Token成本 #代码图谱 #智能体 #工程实践
