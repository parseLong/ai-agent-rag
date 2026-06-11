---
title: Claude Code MCP 服务器
description: MCP (Model Context Protocol) 服务器配置与最佳实践
date: 2026-05-19
tags: [Claude Code, MCP, Model Context Protocol]
---

# Claude Code MCP 服务器

## 什么是 MCP？

MCP (Model Context Protocol) 服务器扩展 Claude Code 的能力，连接到外部工具、数据库和 API。

## 推荐的 MCP 服务器

| MCP 服务器 | 用途 | 资源 |
|-----------|------|------|
| **Context7** | 获取最新库文档到上下文 | [npm](https://www.npmjs.com/package/@upstash/context7-mcp) |
| **Playwright** | 浏览器自动化 — 实现、测试和验证 UI | [Docs](https://playwright.dev/) |
| **Claude in Chrome** | 连接真实 Chrome 浏览器 | [GitHub](https://github.com/nicobailon/claude-code-in-chrome-mcp) |
| **DeepWiki** | 获取 GitHub 仓库的结构化文档 | [GitHub](https://github.com/devanshusemwal/deepwiki-mcp) |
| **Excalidraw** | 生成架构图和流程图 | [GitHub](https://github.com/antonpk1/excalidraw-mcp-app) |

## 配置

### 服务器类型

| 类型 | 传输 | 示例 |
|------|------|------|
| **stdio** | 生成本地进程 | `npx`, `python`, binary |
| **http** | 连接远程 URL | HTTP/SSE 端点 |

### 示例 `.mcp.json`

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"]
    },
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp"]
    },
    "deepwiki": {
      "command": "npx",
      "args": ["-y", "deepwiki-mcp"]
    },
    "remote-api": {
      "type": "http",
      "url": "https://mcp.example.com/mcp"
    }
  }
}
```

### 环境变量扩展

```json
{
  "mcpServers": {
    "remote-api": {
      "type": "http",
      "url": "https://mcp.example.com/mcp?token=${MCP_API_TOKEN}"
    }
  }
}
```

## MCP 作用域

| 作用域 | 位置 | 用途 |
|------|------|------|
| **Project** | `.mcp.json` (仓库根目录) | 团队共享服务器 |
| **User** | `~/.claude.json` (`mcpServers` 键) | 个人跨项目服务器 |
| **Subagent** | Agent frontmatter (`mcpServers` 字段) | 子代理专用服务器 |

优先级: **Subagent > Project > User**

## 设置中的 MCP 控制

| 键 | 类型 | 说明 |
|---|------|------|
| `enableAllProjectMcpServers` | boolean | 自动批准所有 `.mcp.json` 服务器 |
| `enabledMcpjsonServers` | array | 允许特定服务器名 |
| `disabledMcpjsonServers` | array | 阻止特定服务器名 |
| `allowedMcpServers` | array | 仅允许列表（仅 Managed） |
| `deniedMcpServers` | array | 阻止列表（仅 Managed） |

## 工具权限语法

```json
{
  "permissions": {
    "allow": [
      "mcp__*",
      "mcp__context7__*",
      "mcp__playwright__browser_snapshot"
    ],
    "deny": [
      "mcp__dangerous-server__*"
    ]
  }
}
```

## `alwaysLoad` 配置 (v2.1.121+)

默认情况下，MCP 工具定义是延迟加载的。设置 `alwaysLoad: true` 让服务器在会话启动时加载所有工具：

```json
{
  "mcpServers": {
    "always-on-server": {
      "type": "http",
      "url": "https://mcp.example.com",
      "alwaysLoad": true
    }
  }
}
```

## 参考来源

- [MCP Servers — Claude Code Docs](https://code.claude.com/docs/en/mcp)
- [Model Context Protocol Specification](https://modelcontextprotocol.io/)
