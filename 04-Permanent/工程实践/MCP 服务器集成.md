---
title: MCP 服务器集成
description: ECC 预置的 MCP（Model Context Protocol）服务器配置和使用指南
date: 2026-05-18
tags: [ECC, MCP, 集成, 工具]
---

# MCP 服务器集成

> MCP（Model Context Protocol）是一种标准协议，允许 AI 编码助手与外部工具和服务交互。ECC 预置了 20+ 个 MCP 服务器配置。

## 什么是 MCP

MCP 让 Claude Code 能够与外部工具和服务交互，扩展其能力边界。通过 MCP，你可以让 Claude Code 操作 Jira、GitHub、数据库、浏览器等。

## 预置 MCP 服务器

### 项目管理

| 服务器 | 功能 | 启动方式 |
|--------|------|---------|
| `jira` | Jira 问题追踪 — 搜索、创建、更新、评论、转换问题 | `uvx mcp-atlassian` |
| `github` | GitHub 操作（PR、Issue、仓库） | `npx @modelcontextprotocol/server-github` |
| `confluence` | Confluence Cloud 集成 — 搜索页面、检索内容、探索空间 | `npx confluence-mcp-server` |

### 数据库与存储

| 服务器 | 功能 |
|--------|------|
| `supabase` | Supabase 数据库操作 |
| `clickhouse` | ClickHouse 分析查询 |
| `memory` | 跨会话持久化记忆 |
| `omega-memory` | 语义搜索 + 知识图谱记忆（比基础 memory 更丰富） |
| `longhand` | 无损会话历史（SQLite + ChromaDB）— 补充 memory/omega-memory |

### 开发与部署

| 服务器 | 功能 |
|--------|------|
| `vercel` | Vercel 部署和项目 |
| `railway` | Railway 部署 |
| `cloudflare-docs` | Cloudflare 文档搜索 |
| `cloudflare-workers-builds` | Cloudflare Workers 构建 |
| `cloudflare-workers-bindings` | Cloudflare Workers 绑定 |
| `cloudflare-observability` | Cloudflare 可观测性/日志 |

### AI 与研究

| 服务器 | 功能 |
|--------|------|
| `context7` | 实时文档查找（配合 `/docs` 命令） |
| `exa-web-search` | Web 搜索和研究 |
| `sequential-thinking` | 链式推理 |
| `fal-ai` | AI 图像/视频/音频生成 |
| `token-optimizer` | 95%+ 上下文压缩 |
| `evalview` | AI Agent 回归测试 |

### 浏览器自动化

| 服务器 | 功能 |
|--------|------|
| `playwright` | Playwright 浏览器自动化和测试 |
| `browserbase` | 云端浏览器会话 |
| `browser-use` | AI 浏览器 Agent |

### 其他

| 服务器 | 功能 |
|--------|------|
| `filesystem` | 文件系统操作 |
| `devfleet` | 多 Agent 编排（并行 Claude Code Agent） |
| `magic` | Magic UI 组件 |
| `laraplugins` | Laravel 插件发现 |

## MCP 配置方式

将需要的 MCP 服务器配置复制到 `~/.claude.json` 的 `mcpServers` 部分：

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "你的Token"
      }
    }
  }
}
```

## 关键配置示例

### GitHub MCP

```json
{
  "github": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-github"],
    "env": {
      "GITHUB_PERSONAL_ACCESS_TOKEN": "YOUR_GITHUB_PAT_HERE"
    }
  }
}
```

### Jira MCP

```json
{
  "jira": {
    "command": "uvx",
    "args": ["mcp-atlassian==0.21.0"],
    "env": {
      "JIRA_URL": "YOUR_JIRA_URL_HERE",
      "JIRA_EMAIL": "YOUR_JIRA_EMAIL_HERE",
      "JIRA_API_TOKEN": "YOUR_JIRA_API_TOKEN_HERE"
    }
  }
}
```

### Supabase MCP

```json
{
  "supabase": {
    "command": "npx",
    "args": ["-y", "@supabase/mcp-server-supabase@latest", "--project-ref=YOUR_PROJECT_REF"]
  }
}
```

### Playwright MCP

```json
{
  "playwright": {
    "command": "npx",
    "args": ["-y", "@playwright/mcp", "--browser", "chrome"]
  }
}
```

### Context7 MCP

```json
{
  "context7": {
    "command": "npx",
    "args": ["-y", "@upstash/context7-mcp@latest"]
  }
}
```

## 重要提示

> ⚠️ **建议同时启用的 MCP 不超过 10 个**，以保留上下文窗口空间。

### 禁用特定 MCP

```bash
# 禁用 bundled ECC MCPs
export ECC_DISABLED_MCPS=github,context7,...

# 或在项目配置中禁用
{
  "disabledMcpServers": ["github", "context7"]
}
```

### 环境变量替换

所有 `YOUR_*_HERE` 占位符需要替换为实际值。

---

## 相关链接

- [[ECC 核心配置（CLAUDE.md）]] — 项目核心指导文件
- [[核心 Agent 系统]] — 专用子 Agent 详解
- [[Everything Claude Code 完全教程]] — 完整教程
