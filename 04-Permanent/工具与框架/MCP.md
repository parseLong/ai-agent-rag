---
title: MCP
description: Model Context Protocol，Anthropic 提出的标准化智能体与外部工具/资源通信的协议
tags: [概念, 工具与框架, 概念卡片, 永久笔记, 协议, 智能体, 工具, MCP]
aliases:
  - MCP
  - Model Context Protocol
  - MCP 协议
date: 2025-05-19
---

# MCP

> Anthropic提出的标准化协议，以更简单更可靠的方式将LLM智能体连接到外部数据、工具和服务。MCP 就像是 AI 智能体的"USB 接口"——让任何工具都能连接 Agent。

## 核心要点

- **基本组成**：
  - MCP Host：运行MCP Client的宿主环境（Claude桌面端、IDE等）
  - MCP Client：代表模型一侧发起请求的客户端SDK
  - MCP Server：暴露功能或数据接口的服务提供者
  - 数据源或远程服务：Server后端连接的数据或服务

- **与Function Calling的区别**：
  - Function Calling：LLM根据用户输入识别需要什么工具并格式化调用，缺乏统一标准
  - MCP：提供通用协议框架来发现、定义和调用外部工具，高度标准化

> [!note] CodeAct理念
> MCP不仅可以作为工具直接调用，还可以作为代码API提供——Agent编写代码来与MCP Server交互，按需加载更高效利用上下文。

## MCP 核心概念

| 概念 | 说明 |
|------|------|
| **MCP 服务器** | 提供工具/资源/命令的服务端程序 |
| **MCP 客户端** | 智能体中连接和使用 MCP 服务器的组件 |
| **工具 (Tools)** | 智能体可调用的功能（如搜索、计算） |
| **资源 (Resources)** | 可被读取的数据（如文件、数据库记录） |
| **命令 (Commands)** | 用户可触发的操作 |

## 传输协议

| 协议 | 适用场景 |
|------|---------|
| **stdio** | 本地 CLI 工具 |
| **SSE** | HTTP 长连接服务 |
| **HTTP** | RESTful 服务 |
| **WebSocket** | 双向实时通信 |
| **SDK** | 内嵌式服务 |

## 认证机制

- **API Key**：最简单，直接提供密钥
- **OAuth 2.0**：标准授权流程
- **XAA**：跨应用访问认证
- **Elicitation**：URL-based 认证流程

## MCP 架构与工作流程

MCP 协议采用 **Host、Client、Server 三层架构**：

1. **Host（宿主层）**：如 Claude Desktop，负责接收用户提问并与模型交互
2. **Client（客户端层）**：内置的 MCP Client，负责与 MCP Server 建立连接
3. **Server（服务器层）**：执行实际的工具操作（如文件扫描、数据库查询）

**完整交互流程**：
1. **工具发现阶段**：MCP Client 连接到 Server 后，调用 `list_tools()` 获取可用工具描述
2. **上下文构建**：Client 将工具列表转换为 LLM 能理解的格式
3. **模型推理**：LLM 分析用户问题和可用工具，决定是否需要调用工具
4. **工具执行**：Client 通过 MCP Server 执行所选工具
5. **结果整合**：工具执行结果送回 LLM，生成最终回答

## 💡 代码示例

### 连接到 MCP 服务器

```python
import asyncio
from hello_agents.protocols import MCPClient

async def connect_to_server():
    # 连接到社区提供的文件系统服务器
    client = MCPClient([
        "npx", "-y",
        "@modelcontextprotocol/server-filesystem",
        "."  # 指定根目录
    ])

    async with client:
        tools = await client.list_tools()
        print(f"可用工具: {[t['name'] for t in tools]}")
```

### 在智能体中使用 MCP

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import MCPTool

agent = SimpleAgent(name="文件助手", llm=HelloAgentsLLM())

# 连接到社区文件系统服务器
fs_tool = MCPTool(
    name="filesystem",
    description="访问本地文件系统",
    server_command=["npx", "-y", "@modelcontextprotocol/server-filesystem", "."]
)
agent.add_tool(fs_tool)

# 智能体自动使用 MCP 工具
response = agent.run("请读取 my_README.md 文件，并总结其中的主要内容")
```

## 工具调用范式的演进

> 来源：[[Agent技术范式演变（2023-2026）]]

Agent 的工具调用方式正在经历从"人为适配模型"到"利用模型原生能力"的范式转移：

> [!important] Function Call 的核心痛点
> 现实中==大量的系统或数据源并没有现成的 API 可供调用==，团队往往需要投入大量精力去"补全" API。不仅费时费力，而且==随着工具数量的膨胀，API Schema 的管理变得极其复杂==。MCP 在协议层面优化了工具注册与发现机制，实现了"一次注册、自动暴露"，但本质上仍停留在接口标准化的层面，并未从根本上改变工具调用的底层逻辑。

| 维度 | 早期范式（Function Call） | 当前范式（CLI + Script） |
|------|--------------------------|------------------------|
| **开发成本** | 为每个操作封装专用 API，极高 | 利用模型预训练的 CLI 知识，零额外开发 |
| **维护复杂度** | API Schema 管理膨胀 | CLI 自解释性（--help 即时学习） |
| **MCP 的角色** | 协议层面的标准化（一次注册、自动暴露） | 仍停留在接口标准化，未改变底层逻辑 |
| **核心逻辑** | 人为适配模型 | 利用模型原生能力 |

### CLI 原生化的三大优势

1. **零样本学习**：grep、cat、vim 等命令是模型预训练数据的"先天知识"，无需额外定义 API Schema
2. **自解释性**：第三方 CLI 工具遵循标准 Unix 规范（--help），模型可运行时自主理解参数用法
3. **Skill 集成**：新 CLI 工具通过 [[Skills技能]] 包装，SKILL.md 提供安装指南和使用示例

### Script 脚本化

工具逻辑封装为独立脚本文件（Python 等），作为 [[Skills技能]] 的 Resources：
- **本地与远程统一**：脚本可执行本地命令，也可内部封装远程 API 调用
- **协议黑盒化**：API 鉴权、参数拼接等细节隐藏在脚本内部，Agent 只关注"调用哪个脚本"+"传入什么参数"

> [!tip] MCP 的定位：接口标准化的过渡
> MCP 在工具调用演进中扮演的是"协议标准化"的角色，解决了工具注册与发现的问题（一次注册、自动暴露）。但==本质上仍停留在接口标准化的层面，并未从根本上改变工具调用的底层逻辑==。真正的范式转移发生在执行方式上——从 API 封装回归到 CLI/Script 模式。两者可以互补：MCP 提供标准化协议框架，CLI/Script 提供更轻量的执行方式。

> [!important] 交互友好性的倒转
> CLI 对==人类==来说是高门槛的、枯燥的交互方式，但对==机器（LLM）==反而足够友好——这些命令是其预训练数据的"先天知识"。这种"交互友好性倒转"是 Agent 工具范式转移的底层洞察：==对人类不友好的交互方式，可能是对 Agent 最友好的交互方式==。CLI 因此在 Agent 时代再次焕新了生机，成为 Agent 的"天然工具"。

## MCP 的成本维度

> 来源：[[一篇搞懂 AI Coding Agent 的 Token 成本控制]]

MCP 不仅是技术协议，还有隐性成本维度——每个 MCP Server 都带着一份 Tool Definition（描述、参数模式、调用约束），这些信息常驻在每轮请求的上下文中：

| 成本维度 | 说明 |
|----------|------|
| **常驻定义重量** | 每多一个 MCP → 多一段工具定义 → 每轮都带着 |
| **选择空间膨胀** | 工具越多 → 模型决策越慢 → 更多推理 Token |
| **错误调用概率** | 选择空间大 → 更容易调错工具 → 更多重试 |
| **调用回路放大** | 工具返回值塞回上下文 → [[工具调用回路]] |

> [!tip] CLI 优先原则
> 已有成熟命令链路时（`gh pr create`、`kubectl get pods`、`git diff`），不需要额外拉一整套 MCP 工具说明。AI 直接走命令更轻。详见 [[AI Coding Agent Token 成本控制]] 和 [[工具调用回路]]。

## 关联知识

- **相关协议**：[[A2A]]（智能体间通信）、[[ANP]]（智能体网络）
- **实现参考**：[[MCP 协议实现]]、[[MCP 服务器集成]]
- **核心概念**：[[智能体通信协议]]、[[工具系统]]、[[工具调用回路]]
- **成本视角**：[[AI Coding Agent Token 成本控制]]、[[减少重试是成本优化]]
- **范式演变**：[[Agent技术范式演变（2023-2026）]] — Tools 范式演变的完整脉络
- [[万字详解大模型应用发展：RAG、MCP、Agent的爆发之旅]]

## 🏷️ 标签

#概念 #工具与框架 #概念卡片 #永久笔记 #协议 #智能体 #工具 #MCP