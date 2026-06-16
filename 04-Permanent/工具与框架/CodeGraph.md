---
title: CodeGraph
description: 基于 MCP Server + 持久化图数据库的代码图谱工具——7个真实仓库benchmark平均：16%更低成本、47%更少Token、58%更少Tool Call
tags: [工具, Token成本, 代码图谱, 智能体, MCP, 工程实践]
aliases:
  - codegraph
date: 2026-06-15
source: "[[一篇搞懂 AI Coding Agent 的 Token 成本控制]]"
github: https://github.com/colbymchenry/codegraph
---

# CodeGraph

> 基于 MCP Server + 持久化图数据库（Neo4j/KuzuDB），7 个真实开源仓库 benchmark：平均 16% 更低成本、47% 更少 Token、58% 更少 Tool Call。

## 🧠 核心洞察

与 [[Graphify]]（Skill 形态、零配置）不同，CodeGraph 是 **MCP Server 形态**，基于持久化图数据库，适合团队和大型仓库。

## 📊 Benchmark 数据

Claude Opus 4.8，2026-06-02 验证，7 个真实开源仓库：

| 代码库 | 语言 | 成本 | Tokens | 工具调用 |
|--------|------|------|--------|----------|
| VS Code | TypeScript | -18% | -64% | -81% |
| Excalidraw | TypeScript | 持平 | -25% | -40% |
| Tokio | Rust | 持平 | -38% | -57% |
| Django | Python | -8% | -60% | -77% |
| Gin | Go | -19% | -23% | -44% |
| OkHttp | Java | -25% | -54% | -50% |
| Alamofire | Swift | -40% | -64% | -58% |

> [!note] 规律
> Rust/TypeScript 大型项目收益最显著，Java/Go 较小项目收益相对有限。

## 🔑 核心工具

| 工具 | 功能 | 使用时机 |
|------|------|----------|
| `codegraph_context` | 找入口点和关键符号 | 第一跳 |
| `codegraph_trace` | 追调用路径（含动态派发） | 理解调用链 |
| `codegraph_impact` | 重构前影响分析 | 变更评估 |

## 🛠️ 安装与配置

```bash
# npm 安装
npm i -g @colbymchenry/codegraph

# 初始化项目
cd your-project
codegraph init -i
```

手动配置 CodeBuddy：

```json
// ~/.codebuddy/.mcp.json
{
  "mcpServers": {
    "codegraph": {
      "type": "stdio",
      "command": "codegraph",
      "args": ["serve", "--mcp"]
    }
  }
}
```

## 🔑 与 Graphify 的区别

| | [[Graphify]] | CodeGraph |
|---|-------------|-----------|
| **形态** | Skill 形态 | MCP Server |
| **配置** | 零配置 | 需要配置 MCP + 图数据库 |
| **持久化** | JSON 文件 | 持久化图数据库（Neo4j/KuzuDB） |
| **适合** | 个人项目、快速上手 | 团队、大型仓库 |

**选哪个**：个人项目、快速上手 → [[Graphify]]；团队、大型仓库、需要持久化查询 → CodeGraph。

## 🔗 关联知识

- [[AI Coding Agent Token 成本控制]] — 五层优化框架中第四层的具体工具
- [[Graphify]] — Skill 形态的代码图谱工具
- [[MCP]] — CodeGraph 以 MCP Server 形态运行
- [[上下文压缩]] — 第三层优化，与第四层互补

## 🏷️ 标签

#工具 #Token成本 #代码图谱 #智能体 #MCP #工程实践
