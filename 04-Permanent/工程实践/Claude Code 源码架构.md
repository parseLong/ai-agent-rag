---
title: "Claude Code 源码架构"
description: 基于 Claude Code 源码逆向工程的综合架构分析 — 从整体结构到 System Prompt 装配、查询管道、工具系统、子代理协作、安全防御的完整梳理
tags:
  - 概念卡片
  - 永久笔记
  - Claude Code
  - 智能体
  - 架构
  - 工具
  - Query Pipeline
  - System Prompt
  - 提示工程
  - 自修复
  - Tool-Call
  - 多智能体
  - MCP
  - Skill
  - 记忆
  - 压缩
  - 权限
  - 原理
  - LLM
  - 安全
  - 桥接
  - IDE
  - 命令
  - CLI
  - 特征开关
  - 编译优化
  - 状态管理
  - React
  - 架构模式
  - 守护进程
  - AI代理
  - 工程实践
  - 提示注入
  - 分层防御
  - Prompt工程
  - 组合模式
  - clippings
aliases:
  - Claude Code 整体架构
  - Claude Code Query Pipeline与多智能体
  - Claude Code System Prompt装配
  - Claude Code 工具Prompt与自修复
  - Claude Code 扩展子系统与十层防御
  - Claude Code 记忆压缩与权限安全
  - 查询引擎
  - 工具系统
  - 权限系统
  - 子代理系统
  - Bridge 桥接系统
  - 命令系统
  - 功能开关与死代码消除
  - 状态管理（Claude Code）
  - 守护进程模式
  - 分层安全防御
  - 序言组合根模式
created: 2026-05-22
source: "基于 17 篇 Claude Code 源码逆向分析笔记的合并与重组"
---

# Claude Code 源码架构

> 本文件是对 Claude Code 源码逆向工程分析的**综合梳理**，基于约 1,900 文件、512K+ 行 TypeScript 的源码解读，将 17 篇独立分析笔记合并为一份完整的架构参考。Claude Code 是 Anthropic 的官方 CLI 工具，它不是简单的终端聊天机器人，而是**一个完整的生产级 Agent 系统**——拥有查询引擎（决策中枢）、工具系统（执行能力）、权限系统（安全边界）、MCP 协议（外部连接）、Bridge 桥接（IDE 通信）、状态管理（内部协调）。以下按架构层次逐一展开。

---

## 整体架构

Claude Code 的源码约 512K 行 TypeScript，跨 ~1900 个文件，采用 **React + Ink** 渲染终端 UI、**Bun** 运行时、**Zod v4** 做验证、**Commander.js** 解析 CLI 参数。

整个系统围绕"模型请求工具 → 工具执行 → 结果返回 → 模型继续"这个循环运转。理解其架构是理解**生产级 AI Agent**如何构建的最佳窗口——它展示了从推理到行动、从安全到扩展、从单代理到多代理的完整实践。

### 核心架构图

```
main.tsx (CLI 入口)
  │
  ├─ Commander.js 解析参数
  ├─ 并行预取：MDM设置 / Keychain / API预连接
  │
  ▼
AppStateProvider (React Provider)
  │
  ├─ AppStateStore ← 全局状态
  ├─ MailboxProvider ← 异步消息
  ├─ VoiceProvider ← 语音模式
  │
  ▼
App.tsx → BaseTextInput (用户输入)
  │
  ▼
QueryEngine (查询引擎 — 核心循环)
  │
  ├─ 组装消息 (system context + user context + history)
  ├─ 调用 Anthropic API (流式响应)
  ├─ 检测工具调用 → runTools() → 返回结果 → 继续循环
  ├─ 上下文压缩 (snip compaction)
  │
  ├─ Tool System (~40 工具)
  │   ├─ BashTool, FileReadTool, FileWriteTool, FileEditTool
  │   ├─ GlobTool, GrepTool, WebFetchTool, WebSearchTool
  │   ├─ AgentTool (子代理), SkillTool, MCPTool
  │   ├─ TaskCreateTool, EnterPlanModeTool, CronCreateTool ...
  │
  ├─ Permission System (权限检查)
  │   ├─ 规则引擎 (allow/deny/ask + 通配符)
  │   ├─ 分类器 (TRANSCRIPT_CLASSIFIER 特征开关)
  │   ├─ 沙箱执行 (可选)
  │
  ├─ Services Layer
  │   ├─ api/ (Anthropic API 客户端)
  │   ├─ mcp/ (MCP 服务器连接管理)
  │   ├─ oauth/ (OAuth 2.0 认证)
  │   ├─ compact/ (上下文压缩)
  │   ├─ analytics/ (GrowthBook 特征开关 + 事件日志)
  │
  ├─ Bridge System (IDE 通信)
  │   ├─ bridgeMain.ts, bridgeMessaging.ts
  │   ├─ jwtUtils.ts (JWT 认证)
  │
  ▼
Ink/React 渲染终端 UI
```

### 启动优化策略

| 策略 | 实现 | 目的 |
|------|------|------|
| **并行预取** | MDM 设置、Keychain 读取、API 预连接同时发起 | 减少启动延迟 |
| **懒加载** | OpenTelemetry、gRPC、analytics 用动态 `import()` | 避免启动时加载重模块 |
| **死代码消除** | `feature('FLAG_NAME')` 通过 `bun:bundle` 在构建时剥离 | 减小包体积 |

### 数据流：一次查询的生命周期

1. **用户输入** → `main.tsx` 解析 → `QueryEngine.submitMessage()`
2. **QueryEngine** 组装消息（系统上下文 + 用户上下文 + 对话历史）
3. **API 调用** → `services/api/claude.ts` 流式获取 Anthropic API 响应
4. **工具调用检测** → 模型请求工具 → `runTools()` 执行 → 结果追加到对话 → API 调用继续
5. **权限检查** → `useCanUseTool` 检查规则 → 用户确认（如需要）→ 工具执行或拒绝
6. **最终响应** → Ink 组件渲染到终端

架构的核心设计原则：**工具是能力边界，权限是安全边界，状态是协调中枢**。QueryEngine 不是简单的 API 调用封装，它管理整个"思考-行动-观察"循环的生命周期——包括上下文压缩、重试、token 预算追踪。

> 关联知识：[[查询引擎]]、[[工具系统]]、[[权限系统]]、[[Bridge 桥接系统]]、[[MCP 协议实现]]、[[命令系统]]、[[子代理系统]]、[[上下文压缩]]、[[功能开关与死代码消除]]、[[状态管理（Claude Code）]]、[[智能体（Agent）]]、[[ReAct 范式]]

---

## System Prompt 装配与设计哲学

整个 System Prompt 由 `getSystemPrompt()` 函数组装，**返回一个多段字符串数组**，按顺序拼接后送入 LLM。核心装配顺序为：

```
[7个静态段] → [缓存分界线] → [10+个动态段/条件段]
```

**数组而非单字符串**：每个功能块独立为数组元素，便于缓存控制——静态段可跨用户/组织共享缓存，动态段每次会话单独注入。缓存分界线标记 `__SYSTEM_PROMPT_DYNAMIC_BOUNDARY__` 是 Anthropic Prompt Caching 机制的实际分割点。

### 模式选择与动态机制

| 模式 | 身份前缀 | 触发条件 |
|------|----------|----------|
| Vertex AI | 默认前缀 | `isVertexAI()` |
| Agent SDK 预设 | `"...running within the Claude Agent SDK."` | 非交互 + 有 append system prompt |
| Agent SDK 标准 | `"...a Claude agent, built on Anthropic's Claude Agent SDK."` | 非交互 + 无 append |
| 默认（交互） | `"...Anthropic's official CLI for Claude."` | 其他情况 |

Agent SDK 模式下的前缀变化不只是措辞调整，而是一种**行为模式切换**。SDK 模式会跳过某些会话特定的指令段，同时允许宿主程序通过 append system prompt 注入额外行为约束。这种"前缀即模式"的设计是 [[功能开关与死代码消除]] 理念在 Prompt 层的体现。

### CACHED_MICROCOMPACT 精简策略

当 `CACHED_MICROCOMPACT` 功能标志启用时，会话特定指南、工具结果总结指令被缩减，Function Result Clearing 段注入旧结果清除策略提示。这体现了**认知经济学**——上下文紧张时不向模型展示冗余指令，只保留行动所需的最小规则集。

### TOKEN_BUDGET 预算注入

Token Budget 不只是财务工具——它是一个**行为塑形器**。通过告诉模型"你有 X tokens 可以花"，系统鼓励模型在复杂任务上投入更多推理步骤，类似于 [[ReAct 范式]] 中的"思考-行动"循环被赋予了时间资源维度。

### 10 个核心 Prompt 段功能解析

**段 1：身份前缀（Identity）** — 定义 Agent 的自我认知与能力边界认知。

**段 2：归Attribution Header** — `x-anthropic-billing-header` 携带版本指纹、入口点、客户端认证占位符和路由提示。不是面向模型的指令，而是**面向 API 网关的元数据**——用于计费、路由和负载均衡。

**段 3：系统规则（# System）** — 定义 Agent 运行时的基本契约：输出通道、权限模式、标签处理、注入防御、Hook 反馈、上下文透明。这一段本质上是 Agent 的"物理法则"。

**段 4：任务执行指令（# Doing tasks）** — 定义 Agent 解决问题的方法论和工程哲学：先读代码再提议修改，不超出任务范围做优化，三行相似代码优于过早抽象。通过"不做"清单约束 Agent 的过度工程化倾向。

**段 5：安全行动指令（# Executing actions with care）** — 定义风险认知框架：本地可逆操作自由执行，难以逆转的操作确认后再执行。"暂停确认的代价很低，错误行动的代价很高。"这一段在 [[权限系统]] 和 [[分层安全防御]] 之间架起了 Prompt 层的桥梁。

**段 6：工具使用指令（# Using your tools）** — 定义情境化的工具选择策略（读文件用 Read 不用 cat，搜索用 Grep 不用 grep），无依赖的工具调用应并行发出。

**段 7：语气风格（# Tone and style）** — 不使用 emoji，简短回复，工具调用前不加冒号（纯 UI 体验规则）。

**段 8：输出效率** — 外部用户版追求效率（时间敏感），内部版追求可理解性（上下文敏感）——**同一系统的不同用户群需要不同的沟通模式**。

**段 9：会话特定指南** — Agent 合约（非平凡实现完成后必须 spawn 独立验证子代理）、AskUserQuestion、/skill-name 等动态注入内容。

**段 10：环境信息** — 工作目录、git 状态、操作系统、模型名称等，让模型理解"我在哪运行"。

### 动态上下文注入机制

System Prompt 之外，用户消息级别的上下文以 `<system-reminder>` 标签注入首条用户消息：

| 注入项 | 来源 | 说明 |
|--------|------|------|
| `gitStatus` | `getSystemContext()` | 当前分支、文件变更、最近 5 次 commit（截断至 2000 字符） |
| `cacheBreaker` | `getSystemContext()` | Ant-only 调试用缓存破坏标记 |
| `claudeMd` | `getUserContext()` | 项目根目录 `CLAUDE.md` 文件内容 |
| `currentDate` | `getUserContext()` | 当前日期 |

`IMPORTANT: this context may or may not be relevant to your tasks` 这个免责声明告诉模型不要把上下文当硬性约束。

### System Prompt 的优先级与层级设计

```
Override system prompt  →  完全替换 (最高)
  │
Coordinator system prompt  →  Coordinator 模式专用
  │
Agent system prompt  →  子代理专用
  │  ├─ 自主模式 (KAIROS): 追加到默认提示词后
  │  └─ 其他模式: 替换默认提示词
  │
Custom system prompt  →  --system-prompt CLI 参数
  │
Default system prompt  →  标准提示词 (最低)
  │
Append system prompt  →  始终追加 (除非有 Override)
```

这个优先级链很像 [[序言组合根模式]]——不同来源不是随机叠加，而是遵循严格的覆盖规则。Agent SDK 的自主模式不是替换而是追加——保证核心行为指令不被冲掉。

> 关联知识：[[系统提示词工程]]、[[提示工程]]、[[上下文注入管道]]、[[上下文注入]]、[[上下文窗口]]、[[上下文窗口管理]]、[[工具描述工程]]、[[输出解析与路由]]、[[护栏与钩子系统]]、[[序言组合根模式]]、[[状态管理（Claude Code）]]、[[权限系统]]、[[分层安全防御]]、[[子代理系统]]、[[流式渲染控制]]、[[ReAct 范式]]

---

## Query Pipeline 查询管道

Query Pipeline 是 Claude Code 的查询引擎核心，负责从用户输入到 API 聃用再到结果返回的完整生命周期。核心文件存放在 `src/query.ts`（主查询循环，~1,729行）和 `src/QueryEngine.ts`（LLM查询引擎，~46K行）。

### 重试机制 withRetry()

`withRetry()` 是包裹所有 API 聃用的重试包装器，采用**指数退避 + 错误分类 + 条件重试**策略。

| 错误类型 | 触发条件 | 处理策略 |
|---------|---------|---------|
| 401/403 认证错误 | Token 过期或权限不足 | 刷新 OAuth token / 凭证后重试 |
| 429 限流（短延迟） | 请求频率过高 | 用 fast mode 重试 |
| 429 限流（长延迟） | 长时间被限 | 切换到标准速度模型 |
| 529 过载（非前台） | 后台请求遭遇过载 | 立即放弃 |
| 529 过载（<3次） | 前两次连续 529 | 指数退避重试 |
| 529 过载（>=3次） | 连续三次 529 | 触发模型回退（fallbackModel） |
| Max tokens overflow | 输出 token 超限 | 调整 maxTokens 后重试 |
| ECONNRESET / EPIPE | 连接异常断开 | 禁用 keep-alive 后重试 |
| 持久重试模式 | `UNATTENDED_RETRY` 启用 | 无限重试 + 指数退避，6小时总上限 |

退避计算：`delay = BASE_DELAY_MS × 2^(attempt-1)`，抖动 ±25%，最大 32s（标准）/ 5min（持久）。

> 关联知识：[[查询引擎]] | [[重试与恢复机制]] | [[状态管理（Claude Code）]]

### 消息组装管道

```
System Prompt（系统提示词）
    ↓
历史消息（对话上下文，含已缓存的 tool_result）
    ↓
用户输入（当前 Turn 的用户消息）
    ↓
上下文注入（gitStatus + claudeMd + currentDate，以 <system-reminder> 包装）
```

消息准备处理链：

```
原始消息
→ applyToolResultBudget()    — 工具结果大小限制
→ snipCompact()              — 片段压缩（feature-gated）
→ microCompact()             — 微压缩，将旧 tool_result 替换为占位文本
→ contextCollapse()          — 分阶段上下文缩减
→ autoCompact()              — 自动压缩（达到 token 阈值时触发）
```

> 关联知识：[[上下文注入]] | [[上下文压缩]] | [[上下文窗口管理]]

### 流式工具执行

工具执行采用**读并行、写串行**的并发控制模型：

```
读取型工具（Grep, Glob, Read, LSP）→ 并行执行，最多 10 并发
写入型工具（Edit, Write, Bash）     → 串行执行，一次一个
```

StreamingToolExecutor 状态机：`'queued' → 'executing' → 'completed' → 'yielded'`

中断与恢复：用户中断为所有排队/执行中的工具生成合成错误消息；模型回退丢弃旧 executor 创建新实例；兄弟错误 Abort 并行任务中出错的兄弟进程。

> 关联知识：[[工具系统]] | [[流式渲染控制]]

### 查询循环的 7 个 Continue 站点

Continue 站点是查询循环中的**状态机节点**，每个站点代表一个可恢复的位置：

**站点 1: collapse_drain_retry** — 遇到 PTL 错误且启用 `CONTEXT_COLLAPSE`，执行 `contextCollapse()` 排空积累的上下文后重试。

**站点 2: reactive_compact_retry** — 排空不足时，调用 `compact()` 将历史消息总结为压缩摘要后重试。

**站点 3: max_output_tokens_escalate** — 输出 token 超限，将 max_tokens 从 8K 升级到 64K 后重试。

**站点 4: max_output_tokens_recovery** — 注入恢复消息 "Output token limit hit. Resume directly..."，最多尝试 3 次。

**站点 5: stop_hook_blocking** — Stop Hook 拦截后，将 Hook 输出作为用户消息注入新查询。

**站点 6: token_budget_continuation** — Token Budget 模式下接近目标时自动续费继续。

**站点 7: (normal)** — 正常工具执行完成后 `tool_result` 反馈给模型，进入下一轮。

7 个站点共同实现一个**弹性状态机**：站点 1-2 处理上下文空间不足，站点 3-4 处理输出空间不足，站点 5 处理外部干预，站点 6 处理长时间运行任务，站点 7 处理正常反馈循环。每个站点都是可恢复的——模型不会因为单次失败就终止会话。

### 查询引擎核心职责

| 职责 | 说明 |
|------|------|
| **对话生命周期管理** | 管理消息轮次（turns）、文件缓存、对话状态 |
| **流式响应处理** | 从 Anthropic API 实时流式获取模型输出 |
| **工具调用循环** | 模型请求工具 → 工具执行 → 结果返回 → 模型继续 |
| **Thinking mode** | 支持扩展思考（extended thinking），让模型深入推理 |
| **重试逻辑** | API 失败时的自动重试和错误恢复 |
| **Token 计数与预算** | 追踪 token 使用量，管理上下文窗口预算 |
| **上下文压缩** | 当对话过长时触发 snip compaction |

关键设计决策：**流式优先**（用户实时看到输出）、**工具结果是追加消息**（不是覆盖）、**压缩而非截断**（智能压缩保留关键信息）、**Thinking mode 可选**（复杂推理前可深入思考）。

Agent 的循环不是简单的 while loop——生产级的循环需要处理流式、重试、压缩、预算。特别是上下文压缩：LLM 的上下文窗口是有限资源，Agent 长时间运行后必须压缩历史，而压缩的质量直接决定 Agent 是否还能"记住"之前做了什么。

> 关联知识：[[ReAct 范式]]、[[智能体（Agent）]]、[[Claude Code 整体架构]]、[[工具系统]]、[[权限系统]]、[[上下文压缩]]、[[智能体循环]]

---

## 工具系统

工具系统是 Claude Code 的**执行能力层**，位于 `src/Tool.ts`（约 29K 行）+ `src/tools/` 目录。注册了 38+ 工具，每个工具的 Prompt 不仅是给模型的"使用说明"，更是**安全护栏、执行规范、边界约束**的统一载体。关键设计是**每个工具完全自包含**——schema、权限、执行、UI 都在一个模块里，添加新工具只需写一个文件。

### 工具接口定义

```typescript
interface Tool {
  name: string           // 工具名称
  description: string    // 描述（给模型看的）
  inputSchema: ZodSchema // 输入验证 schema
  call(input, context): Promise<ToolResult>  // 执行逻辑
  render?(input, result): InkComponent       // UI 渲染（可选）
}
```

每个工具还携带**权限模型**——定义在什么模式下可以自动执行、什么模式下需要用户确认。

### 核心工具分类

**文件操作（Agent 的"读写"能力）**

| 工具 | 功能 | 权限模式 |
|------|------|---------|
| `FileReadTool` | 读文件（文本、图片、PDF、Jupyter） | 默认允许 |
| `FileWriteTool` | 创建/覆盖文件 | 需确认 |
| `FileEditTool` | 局部修改（字符串替换） | 需确认（auto 模式可自动） |
| `GlobTool` | 文件名模式匹配 | 默认允许 |
| `GrepTool` | ripgrep 内容搜索 | 默认允许 |

**执行能力（Agent 的"动手"能力）**

| 工具 | 功能 |
|------|------|
| `BashTool` | Shell 命令执行（可选沙箱隔离） |
| `WebFetchTool` | 获取网页内容 |
| `WebSearchTool` | 网络搜索 |

**元能力（Agent 的"自组织"能力）**

| 工具 | 功能 | 深层含义 |
|------|------|---------|
| `AgentTool` | 生成子代理 | Agent 可以创建子 Agent → [[子代理系统]] |
| `SkillTool` | 执行 Skill | Agent 可以调用预定义技能 |
| `MCPTool` | MCP 服务器工具调用 | Agent 可以调用外部工具 → [[MCP 协议实现]] |

**流程控制（Agent 的"规划"能力）**

| 工具 | 功能 |
|------|------|
| `EnterPlanModeTool` | 进入规划模式 |
| `ExitPlanModeTool` | 退出规划模式 |
| `TaskCreateTool / TaskUpdateTool / TaskListTool` | 任务管理 |
| `EnterWorktreeTool / ExitWorktreeTool` | Git worktree 隔离 |
| `CronCreateTool` | 定时触发 |

工具系统的自包含设计让**添加工具是 O(1) 复杂度**，不需要修改任何已有代码。AgentTool 让 Agent 可以**创建子 Agent**，这意味着 Agent 的能力不仅是工具列表的简单加法，而是**可以递归扩展**。

> 关联知识：[[智能体（Agent）]]、[[ReAct 范式]]、[[Claude Code 整体架构]]、[[查询引擎]]、[[权限系统]]、[[子代理系统]]、[[MCP 协议实现]]

---

## 子代理系统

子代理系统是 Claude Code 的**多代理编排能力**，通过 `AgentTool` 实现。核心创新是**worktree 隔离**——子 Agent 可以在独立 git 分支上工作，不影响主分支。

### 子代理类型与分工

| 类型 | 角色 | 工具权限 | 模型 | 特殊约束 |
|------|------|---------|------|---------|
| **general-purpose**（通用） | 默认类型，无特殊约束 | 全部可用 | inherit | 无特殊指令 |
| **Explore**（探索） | 文件搜索专家，代码库导航 | 只读（禁用 Agent、Edit、Write） | 外部 Haiku / 内部 inherit | `omitClaudeMd: true` |
| **Plan**（架构规划） | 软件架构师，设计方案 | 只读 | inherit | `omitClaudeMd: true`，输出结构化计划 |
| **verification**（验证） | 对抗性验证专家，试图破坏实现 | 只读（可写临时目录） | inherit | 后台运行，Check 格式输出 |

### 各子代理强化 Prompt

**Explore 代理**：严格的禁止修改声明 `=== CRITICAL: READ-ONLY MODE ===`，强调快速响应和专业能力描述。

**Plan 代理**：采用 **Your Process + Required Output + Critical Files** 三段式模板，本质上是 [[Plan-and-Solve]] 范式的工程化落地。

**Verification 代理**：设计哲学是**对抗性验证而非确认性验证**——明确列出典型借口并要求停止，"代码看起来正确"不是验证，"已有测试通过"不可信。强制 Check 格式输出：

```
### Check: [验证项]
**Command run:** [精确命令]
**Output observed:** [实际输出，复制粘贴而非转述]
**Result: PASS** (或 FAIL)

VERDICT: PASS / FAIL / PARTIAL
```

### "Never delegate understanding" 原则

这是 Agent Tool Prompt 中最重要的设计理念：

> 不要写"根据你的发现修复 bug"这类指令。提示词应该证明你已经理解了问题——包含文件路径、行号、具体要改什么。

核心要点：给子代理写提示词像给刚走进房间的聪明同事交代任务——它没看过这个对话。命令式简短提示词产出浅薄、通用的工作。这个原则确保主代理始终保持对任务的**高层理解与控制**，子代理只是执行已明确的任务片段。

### Coordinator 模式

当 `COORDINATOR_MODE` 功能开关启用时，主智能体转变为**调度器角色**：Research → Synthesis → Implementation → Verification。Coordinator 使用 Agent tool 生成异步 workers、SendMessage tool 向已有 workers 发送指令、TaskStop tool 取消运行中的 workers。Worker 的结果以 `<task-notification>` XML 标签形式异步到达 Coordinator。

### Fork 隔离机制

Fork 基于 Git worktree 实现文件系统隔离：
- 复制父消息历史（继承完整上下文）
- 用字节相同的占位文本替换 tool_result（保持 prompt cache 键一致）
- 添加 per-child 指令文本块

核心约束：**Don't peek**（不应 Read Fork 的 output_file，读取中间转录会将工具噪音拉入主上下文）、**Don't race**（永远不伪造或预测 Fork 结果）。优势是极低成本（共享 prompt cache，缓存命中率极高），限制是无法使用不同模型。

### 子代理的上下文窗口管理

多层次隔离与压缩策略：
1. **CLAUDE.md 省略**：Explore 和 Plan 代理设置 `omitClaudeMd: true`，避免上下文膨胀和角色冲突
2. **只读隔离**：Explore、Plan、verification 限制为只读，不产生新的 tool_result 积累
3. **Fork 上下文占位替换**：保持 prompt cache 键一致，防止父上下文被污染
4. **工作目录重置**：所有代理线程 Bash 调用间重置 cwd，强制绝对路径
5. **后台运行验证代理**：结果不阻塞主流程
6. **Fork 的 Don't peek 约束**：协议层面防止子代理工具噪音回流主上下文

多代理协作的关键不是通信协议，而是**隔离机制**——两个 Agent 同时改同一份代码会冲突，worktree 隔离解决了这个问题。子 Agent 不继承主 Agent 的完整对话历史，只获得任务描述——避免上下文膨胀和隐私泄漏。

> 关联知识：[[多智能体]]、[[角色分配]]、[[任务分解]]、[[Plan-and-Solve]]、[[多智能体协作]]、[[上下文窗口管理]]、[[上下文压缩]]、[[上下文窗口]]

---

## Bridge 桥接系统

Bridge 桥接系统是 Claude Code 与 IDE 扩展（VS Code、JetBrains）之间的**双向通信层**，位于 `src/bridge/`。它通过 JWT 认证、消息协议、会话管理实现 IDE 中的 Claude Code 功能。

### 核心组件

| 组件 | 功能 |
|------|------|
| `bridgeMain.ts` (115KB) | 主桥接编排 |
| `replBridge.ts` (100KB) | REPL 包装器 |
| `bridgeMessaging.ts` | 传输层帮助器 |
| `jwtUtils.ts` | JWT 认证 |

### 传输层演进

- **V1**：轮询（Polling）
- **V2**：SSE（Server-Sent Events）
- **HybridTransport**：V2 优先，V1 自动回退

### 控制协议（SDKControlRequest）

- `initialize`：初始化能力协商
- `set_model`：动态模型切换
- `set_permission_mode`：权限模式切换
- `set_max_thinking_tokens`：思考 token 限制
- `interrupt`：中断操作（等效 Ctrl+C）

### 会话管理 API

- `POST /v1/sessions` → 创建会话
- `PATCH /v1/sessions/{id}` → 标题同步
- `POST /v1/sessions/{id}/archive` → 归档

Bridge 的核心洞见：**AI Agent 不应该是一个独立工具，而应该嵌入开发者工作流**。Bridge 不是让 IDE 扩展独立实现 Agent，而是**让 IDE 扩展成为 CLI 的前端**——保持行为一致性。

> 关联知识：[[MCP 协议实现]]、[[智能体（Agent）]]、[[Claude Code 整体架构]]、[[查询引擎]]、[[守护进程模式]]

---

## 命令系统

命令系统是 Claude Code 的**用户交互层**，位于 `src/commands.ts` + `src/commands/`。提供 100+ 个斜杠命令，按执行方式分为三类：

| 类型 | 说明 |
|------|------|
| `prompt` | AI 驱动，展开为提示文本注入对话 |
| `local-jsx` | Ink UI 组件（React），渲染交互界面 |
| `local` | 同步本地操作，直接执行 |

### 核心命令分类

**Git 与版本控制**：`/commit`、`/review`、`/ultrareview`、`/diff`、`/branch`、`/pr-comments`、`/commit-push-pr`、`/security-review` 等。

**对话管理**：`/resume`、`/clear`、`/compact`、`/rewind`、`/copy`、`/rename`、`/export` 等。

**上下文与配置**：`/context`、`/memory`、`/config`、`/plan`、`/permissions`、`/hooks`、`/sandbox` 等。

**模型与推理**：`/model`、`/effort`、`/fast`、`/advisor` 等。

**账户与使用量**：`/login`、`/logout`、`/cost`、`/usage`、`/stats`、`/upgrade` 等。

**工具与扩展**：`/tasks`、`/skills`、`/agents`、`/plugin`、`/mcp`、`/init` 等。

命令系统的两层设计值得注意：PromptCommand 触发 Agent 行为（如 `/commit`），LocalCommand 直接执行（如 `/cost`）。前者是"委托给 Agent"，后者是"直接执行"——不是所有用户意图都应该委托给 Agent。

> 关联知识：[[查询引擎]]、[[功能开关与死代码消除]]、[[智能体（Agent）]]

---

## 状态管理

状态管理是 Claude Code 的**内部协调层**，位于 `src/state/`。使用自定义 store 架构（不是 Redux），基于 React 的 `useSyncExternalStore` 实现选择性订阅。

### 核心架构

```
AppStateStore.ts — 定义完整应用状态
  ├─ settings (用户设置)
  ├─ permissions (权限规则和模式)
  ├─ bridgeStatus (IDE 连接状态)
  ├─ tasks (任务列表)
  ├─ mcpClients (MCP 服务器连接)
  ├─ queryState (查询引擎状态)
  └─ ...

Selectors — 选择性订阅
  useAppState(selector) → 只订阅 selector 返回的状态切片
```

### 与 Redux 的对比

| 方面 | Redux | Claude Code Store |
|------|-------|-------------------|
| 状态变更 | dispatch action → reducer | 直接修改 store（更简单） |
| 订阅 | `useSelector` | `useAppState(selector)` |
| 中间件 | redux-thunk/saga | 无 |
| 复杂度 | 高（三件套） | 低（两件套） |

Claude Code 选择自定义 store 而不是 Redux，核心原因：**Redux 的 action/reducer 模式对 Agent 系统太重了**。Agent 的状态变化频繁且多样，直接修改 store + selector 订阅是更实用的折衷。

> 关联知识：[[查询引擎]]、[[工具系统]]、[[权限系统]]、[[Bridge 桥接系统]]、[[智能体（Agent）]]

---

## 功能开关与死代码消除

功能开关系统基于 GrowthBook 特征开关 + `bun:bundle` 编译时死代码消除。核心创新：**未启用的功能在构建时就被完全删除了**——不是隐藏，而是从包中删除，减小体积并防止未授权访问。

### 关键特征开关

| 开关名 | 功能 |
|--------|------|
| `PROACTIVE` / `KAIROS` | 自主工作模式 |
| `BRIDGE_MODE` | IDE 桥接 |
| `VOICE_MODE` | 语音输入 |
| `AGENT_TRIGGERS` | 定时触发器 |
| `MONITOR_TOOL` | 进程监控工具 |
| `TOKEN_BUDGET` | Token 预算控制 |
| `FORK_SUBAGENT` | Fork 子智能体 |
| `VERIFICATION_AGENT` | 验证智能体 |
| `EXPERIMENTAL_SKILL_SEARCH` | 技能搜索 |
| `COORDINATOR_MODE` | 协调器模式 |
| `CONTEXT_COLLAPSE` | 上下文折叠 |
| `WORKFLOW_SCRIPTS` | 工作流脚本 |

### 与传统运行时检查的区别

| 方式 | 代码存在 | 包体积 | 安全性 |
|------|---------|--------|--------|
| `if (config.enabled)` | 始终存在 | 大 | 低（可绕过） |
| `feature('FLAG')` + `bun:bundle` | 构建时删除 | 小 | 高（代码不存在） |

优化策略组合：死代码消除（构建时）→ 懒加载（运行时）→ 并行预取（启动时）。效果：启动快、包体积小、按需扩展。

死代码消除是最有趣的设计决策——**即使配置被修改，功能也无法启用**，因为执行代码根本不存在。这是安全优先的设计选择。

> 关联知识：[[命令系统]]、[[权限系统]]、[[子代理系统]]、[[智能体（Agent）]]

---

## 记忆压缩系统

### Compact 压缩系统

Claude Code 的上下文压缩将长对话历史**压缩为结构化摘要**。核心代码位于 `src/services/compact/prompt.ts`。

**NO_TOOLS_PREAMBLE**：每次压缩在 Prompt 开头注入此段，确保模型专注于文本摘要，不产生任何工具调用——强调 TEXT ONLY、明确禁止所有工具、警告工具调用会被拒绝且浪费唯一轮次。

**BASE_COMPACT_PROMPT**：两阶段设计（先分析后总结），类似于 [[思维链]] 机制：
- `<analysis>` 分析块：按时间顺序分析每条消息，识别请求意图、方法策略、关键决策、文件名和代码片段、错误及修复方式
- `<summary>` 总结块：九个结构化段落（Primary Request、Key Technical Concepts、Files and Code Sections、Errors and fixes、Problem Solving、All user messages、Pending Tasks、Current Work、Optional Next Step）

**压缩后恢复消息**：关键指令是不确认摘要、不复述背景、不用过渡语，直接继续工作。同时提供完整对话记录路径以便查阅被压缩的细节。

**自动压缩触发条件**：

| 参数 | 值 | 含义 |
|------|-----|------|
| `AUTOCOMPACT_BUFFER_TOKENS` | 13,000 | 低于此值自动触发压缩 |
| `WARNING_THRESHOLD_BUFFER_TOKENS` | 20,000 | 低于此值发出警告 |
| `MAX_CONSECUTIVE_AUTOCOMPACT_FAILURES` | 3 | 连续压缩失败熔断器 |

**MicroCompact**：轻量级压缩，直接将旧工具结果替换为占位文本 `'[Old tool result content cleared]'`，不调用模型重新生成，更快成本更低，但丢失细节。

**压缩策略分层**：

| 层级 | 策略 | 触发条件 | 代价 |
|------|------|----------|------|
| 1 | `applyToolResultBudget` | 每次消息 | 无 |
| 2 | `snipCompact` | 功能开关控制 | 极低 |
| 3 | `microCompact` | 缓存旧 tool_result | 低 |
| 4 | `contextCollapse` | PTL 错误时 | 中 |
| 5 | `autoCompact` | Token < 13,000 缓冲量 | 高 |

这种分层设计体现 [[分层安全防御]] 的思想——不是等到完全耗尽才做最重的压缩，而是逐级应对。

### 记忆系统

Claude Code 维护**两种记忆存储**：持久记忆（extractMemories）和会话记忆（SessionMemory），分别对应 [[长时记忆]] 和 [[短时记忆]]。

**记忆提取机制**：由子代理完成，只有有限工具（只读工具 + 仅限记忆目录的 Edit/Write），限时预算（两轮完成），严禁进一步调查。

**What NOT to Save 排除规则**：
1. 代码模式、约定、架构、文件路径 — 可从代码推导
2. Git 历史、最近变更 — `git log`/`blame` 是权威来源
3. 调试解决方案或修复配方 — 修复已体现在代码中
4. 已在 CLAUDE.md 文件中有文档记录的内容 — 避免冗余
5. 临时任务细节 — 会话结束后不再有价值

**四种记忆类型**：user（用户偏好）、feedback（用户反馈）、project（项目知识）、reference（参考资料）。

**会话记忆 10 字段摘要模板**：

| # | 字段 | 描述 |
|---|------|------|
| 1 | **Session Title** | 5-10 词简短标题 |
| 2 | **Current State** | 当前正在处理什么？ |
| 3 | **Task specification** | 用户要求构建什么？ |
| 4 | **Files and Functions** | 重要文件及关联原因 |
| 5 | **Workflow** | 常用 Bash 命令及顺序 |
| 6 | **Errors & Corrections** | 错误及修复方式 |
| 7 | **Codebase and System Documentation** | 重要系统组件 |
| 8 | **Learnings** | 什么效果好，什么不好？ |
| 9 | **Key results** | 用户要求的精确输出 |
| 10 | **Worklog** | 逐步记录 |

容量限制：每段 ≤ 2000 tokens，总会话记忆 ≤ 12,000 tokens。

> 关联知识：[[记忆系统]]、[[短时记忆]]、[[长时记忆]]、[[上下文压缩]]、[[上下文窗口管理]]、[[上下文窗口]]、[[Token]]、[[功能开关与死代码消除]]、[[重试与恢复机制]]

---

## 权限与安全

### 权限决策管道

Claude Code 的 [[权限系统]] 采用**四步决策管道**：

```
工具调用请求
    ↓
Step 1: 规则检查 (hasPermissionsToUseToolInner)
  ├── 整个工具被拒绝？ → deny
  ├── 工具特定的 checkPermissions？ → deny/ask
  ├── 安全检查（.git, .claude, .vscode, shell configs）？ → 必须提示
  ├── bypassPermissions 模式？ → auto-allow
  └── always-allowed 规则匹配？ → auto-allow
    ↓
Step 2: 模式转换
  ├── dontAsk → deny
  ├── auto → 运行分类器
  └── plan + autoModeActive → 运行分类器
    ↓
Step 3: 分类器（两阶段 XML）
  ├── 安全允许列表匹配？ → 跳过分类器，直接允许
  │   (Read, Grep, Glob, LSP, TaskCreate, TaskList, AskUserQuestion,
  │    EnterPlanMode, ExitPlanMode, Sleep, SendMessage, TeamCreate/Delete)
  ├── Stage 1 (fast): max_tokens=64, instant yes/no
  ├── Stage 2 (thinking): max_tokens=4096, chain-of-thought
  └── 拒绝限制追踪（连续拒绝 → 回退到用户提示）
    ↓
Step 4: 交互处理（behavior === 'ask'）
  ├── 交互式: 竞速 4 个源（hooks / 分类器 / bridge / 用户 UI）
  ├── Coordinator: 顺序 hooks → 分类器 → 对话框
  └── Swarm Worker: 分类器 → 转发给 leader → 等待响应
```

关键设计模式：**允许列表 + 分类器 + 回退机制**的 [[分层安全防御]] 架构。

### 两阶段分类器

- **Stage 1 快速判断**：`max_tokens=64`，处理明显安全/危险的请求
- **Stage 2 深度思考**：`max_tokens=4096`，使用 chain-of-thought 推理处理模糊场景
- **拒绝限制追踪**：连续拒绝达阈值后回退到用户提示

分类器输入经过精心设计：剥离助手文本输出防止模型"说服"分类器，只保留客观的工具调用信息。

### 权限模式

| 模式 | 行为 |
|------|------|
| **手动模式** | 每个工具调用需用户确认 |
| **自动模式** | 分类器自动判断，仅模糊场景回退到用户 |
| **dontAsk 模式** | 拒绝所有未在允许列表中的工具调用 |

权限系统核心设计哲学：**最小权限原则 + 信任递增**。ML 分类器让 Agent 在 auto 模式下不是"全开"或"全关"，而是**每个操作独立判断**。

### 沙箱隔离机制

沙箱通过 Bash Tool 的 `dangerouslyDisableSandbox` 参数控制：
- **文件系统限制**：read 维度 denyOnly（默认允许，仅禁止特定路径），write 维度 allowOnly（默认禁止，仅允许特定路径）
- **网络限制**：仅允许 `allowedHosts` 列表

默认启用，只有用户明确要求或命令因沙箱限制而失败时才能禁用——**默认拒绝、最小权限**的安全设计。

### Hook 系统

Hook 系统是权限管道中的**可编程拦截点**：

**四种实现类型**：Command（Shell 命令）、Prompt（LLM 评估）、HTTP（HTTP POST）、Agent（智能体验证）。

**九种触发事件**：PreToolUse、PostToolUse、PostToolUseFailure、PermissionRequest、PermissionDenied、PreCompact、PostCompact、SessionStart / SessionEnd、Stop、Notification。

Hook 系统与 [[护栏与钩子系统]] 设计一脉相承——通过可编程拦截点实现 Agent 行为的**精细控制**。

权限系统是**生产级 Agent 与 demo Agent 的根本区别**——Demo Agent 不需要权限，生产 Agent 必须在真实环境中运行，而真实环境有危险操作。

> 关联知识：[[分层安全防御]]、[[护栏与钩子系统]]、[[智能体（Agent）]]、[[工具系统]]、[[查询引擎]]

---

## 守护进程模式

守护进程模式（Daemon Pattern）是 AI 代理与外部服务交互时使用**长寿命后台进程**而非每次重新启动的架构模式。核心权衡：首次启动成本 vs 每次调用延迟 vs 状态持久性。

### 何时使用

| 场景 | 推荐模式 | 原因 |
|------|---------|------|
| 单次操作 | 每次启动 | 状态不重要，启动成本可接受 |
| 多步交互 | **守护进程** | 状态持久 + 延迟子秒 |
| 高频调用 | **守护进程** | 启动成本无法承受 |
| 长时间空闲 | 自动关闭 + 惰性重启 | 资源回收 |

### 设计要素

1. **状态文件**：守护进程写入端口/PID/token 到文件，客户端读取以发现服务
2. **原子写入**：tmp + rename，防止读到半写状态
3. **健康检查**：HTTP health check 比 PID 更可靠
4. **版本自动重启**：版本不匹配 → 自动杀死重启
5. **崩溃恢复**：崩溃后客户端在下次调用时重启
6. **空闲超时**：30min 无请求 → 自动关闭
7. **随机端口**：避免多实例端口冲突

守护进程模式在 AI 代理场景特别有价值：代理的每次工具调用需要快速响应，经常需要跨步骤保持状态，启动外部服务的代价远高于保持运行。这不只是"性能优化"——它是**功能需求**：没有持久状态，代理根本无法完成多步交互任务。

> 关联知识：[[连接池模式]]、[[对象池模式]]、[[单例模式]]、[[Bridge 桥接系统]]

---

## 序言组合根模式

序言组合根（Preamble Composition Root）是为 AI 技能系统设计的**分层序言组合架构**。每个技能在执行前先运行一段"序言"（preamble），根据技能的**层级（tier）**声明式组合不同模块。

### 分层组合

| 层 | 包含模块 | 适用场景 |
|----|---------|---------|
| T1 | 核心 + 升级检查 + 遥测 + 精简语音 | 简单工具命令 |
| T2 | T1 + 完整语音 + AskUserQuestion格式 + 完整性 + 困惑协议 | 评审/规划技能 |
| T3 | T2 + 仓库模式 + 构建前搜索 | 高级分析技能 |
| T4 | 同 T3 | 最高层技能 |

### 声明式组合

核心原则：组合根**零内联逻辑**。所有功能模块位于独立文件（`./preamble/*.ts`），组合根仅做声明式拼接。修改某个层只需编辑对应的 preamble 文件，**不触碰核心组合逻辑**。

### 运行时条件激活

由于生成的是静态 Markdown，序言中的条件行为通过**自然语言指令**实现——代理在运行时根据环境变量或用户偏好遵守或跳过这些指令。

这个模式的精髓是**将"选择哪些功能"和"每个功能做什么"分离**。组合根只负责前者（声明式 tier gating），preamble 文件只负责后者（具体指令内容），使得全局变更变成单文件修改而非全系统扫描。

> 关联知识：[[组合模式]]、[[中间件管道]]、[[洋葱架构]]、[[Prompt工程]]、[[技能系统]]

---

## 分层安全防御

分层安全防御（Layered Security Defense）是为 AI 代理对抗提示注入设计的多层级防护架构。核心原则：**单层防御不可靠，需要多层独立机制交叉验证**。BLOCK 需要**两个独立分类器同意**，而非单层高分直接阻断。

AI 代理**读取敌意网页**——这是与其他 AI 应用不同的安全威胁。单一 ML 分类器对此不够——误报率太高（合法的指令式文本如 Stack Overflow 常被标记）。

### 分层架构

| 层 | 机制 | 信任来源 | 误报容忍度 |
|----|------|---------|-----------|
| **L1-L3 内容清洗** | 数据标记 + 隐藏元素剥离 + ARIA + URL 黑名单 + 信封 | 页面结构 | 高（粗过滤） |
| **L4 ML 分类器** | ONNX 模型（TestSavantAI / DeBERTa） | 统计模式 | 中（单层高分→WARN） |
| **L4b 转录分类器** | LLM（Claude Haiku）看完整对话形状 | 语义理解 | 中 |
| **L5 Canary Token** | 随机 token 注入→检测泄漏 | 确定性 | **零**（泄漏→always BLOCK） |
| **L6 集成判定** | 组合所有层结果 | 多源交叉 | 低（需要两层同意） |

### 集成规则（关键设计）

```
BLOCK 规则:
  canary 泄漏 → always BLOCK（确定性）
  ML内容分类器 AND 转录分类器 >= WARN → BLOCK
  单层高分 → 降级为 WARN（不直接 BLOCK）

原因: Stack Overflow 式指令文本是合法内容，
      单分类器无法区分"这是注入"和"这看起来像钓鱼"
```

**两层交叉同意**是最关键的创新：它在误报率和漏报率之间找到了实用平衡点。单系统误报很常见，但两个独立系统同时误报的概率极低。

> 关联知识：[[多层防火墙]]、[[零信任架构]]、[[纵深防御]]、[[提示注入]]、[[ML分类器]]、[[Canary Token]]、[[权限系统]]

---

## 扩展子系统与十层防御

### MCP/LSP/Plugin/Skill 子系统

**MCP 集成**位于 `src/services/mcp/`，支持 stdio、SSE、HTTP、WebSocket、SDK 传输类型。服务器作用域层级：local（项目目录）→ user（~/.claude）→ project（.claude）→ dynamic（运行时）→ enterprise → claudeai → managed（策略强制）。连接流程包含 OAuth 认证、权限确认（25 字母表生成 5 字符 ID）和指数退避断线重连。

**LSP 集成**位于 `src/services/lsp/`：LSPServerManager 按文件扩展名路由、LSPServerInstance 管理生命周期、LSPClient 基于 vscode-jsonrpc、LSPDiagnosticRegistry 异步诊断收集。诊断自动附加到对话上下文，每文件最多 10 条，总计最多 30 条，LRU 去重缓存最大 500 文件。

**Plugin 系统**位于 `src/plugins/`：内置插件注册在 builtinPlugins.ts，用户通过 /plugin 切换启用/禁用，插件可提供 skills、hooks、MCP servers。Plugin 是包含多个 Skill、Hook、MCP 服务器的集合单元，且支持 UI 切换。

**Skill 系统**位于 `src/skills/`：技能来源包括内置（src/skills/bundled/）、用户（~/.claude/skills/*.md）、项目（.claude/skills/*.md）、MCP 技能。技能前端格式为 YAML frontmatter + Markdown body。Skill 是轻量级提示词 + 工具声明组合，Plugin 是包含多个 Skill、Hook、MCP 的集合单元。

> 关联知识：[[MCP 协议]]、[[MCP 协议实现]]、[[MCP 服务器集成]]、[[Skills技能]]、[[Skill 开发指南]]、[[命令系统]]

### IDE Bridge 与远程会话

**Bridge 系统**传输层演进：V1（轮询）→ V2（SSE）→ HybridTransport（V2 优先 + V1 自动回退）。

**远程会话**位于 `src/remote/`：RemoteSessionManager 使用 WebSocket 订阅 + HTTP POST 消息双通道，重连 5 次尝试。SessionsWebSocket 有 Ping/Pong keepalive（30秒间隔），永久关闭码 4003（unauthorized），暂态恢复 4001 有限重试。

> 关联知识：[[Bridge 桥接系统]]、[[守护进程模式]]

### 其他关键模块

**Vim 模式**（`src/vim/`）：完整的 Vim 状态机，INSERT/NORMAL 两种模式，操作符 d/c/y，动作 hjkl/wbWBE/0^$，文本对象、查找、持久状态。

**快捷键系统**（`src/keybindings/`）：默认绑定 + 用户自定义 `~/.claude/keybindings.json`，冲突检测 + Chord 绑定支持。

**输出格式**（`src/outputStyles/`）：YAML frontmatter + 自定义输出指令，项目级覆盖用户级。

**自主工作模式**：由 KAIROS / PROACTIVE feature flag 控制，Pacing 调度（Sleep 工具控制等待）、首次唤醒简短问候、Bias toward action（读取文件搜索代码无需征得同意）、终端焦点感知。

**Voice 系统**（`src/voice/`）：GrowthBook feature flag 作为 kill-switch，OAuth token 验证。

**Cost Tracking**（`src/cost-tracker.ts`）：跟踪 inputTokens、outputTokens、cacheRead/Creation、webSearchRequests、costUSD、contextWindow 等，会话持久化按 sessionId 恢复。

**知识截止日期管理**：硬编码于常量文件，不同模型对应不同截止日期。

**配置迁移机制**（`src/migrations/`）：从旧版本到新版本的平滑配置迁移。

### 十层自防御架构

Claude Code 的"智能修复"能力是多层机制协同运作的结果：

> [!important] 十层自防御架构
>
> **1. 主循环闭环控制（反馈循环）** — 工具执行结果作为 `tool_result` 返回给模型，模型在下一轮看到完整错误信息并调整策略。是整个系统最核心的自修复机制。
>
> **2. 分层递进的 System Prompt** — "诊断原因再行动"而非"盲目重试"的指导原则贯穿整个系统提示词。
>
> **3. 四层自修复** — PTL 恢复 → 输出超限恢复 → 模型过载回退 → 工具错误自然恢复，从 token 级到模型级的完整故障恢复链路。
>
> **4. 多智能体协作** — Explore → Plan → Implementation → Verification 的分工让每个环节更专注。
>
> **5. 会话状态保存（错误记忆保持）** — Compact 压缩中保留 "Errors and fixes" 段，Session Memory 中有 "Errors & Corrections" 段。
>
> **6. 自反馈验证（对抗性验证）** — [[子代理系统|Verification Agent]] 专门设计为"试图破坏实现"而非"确认它工作"。
>
> **7. 多模型分工** — Haiku 处理轻量任务，Sonnet/Opus 处理核心推理，Advisor 模型提供第二意见。
>
> **8. 权限安全层** — [[权限系统|分类器 + 沙箱 + Hook 系统]] 形成多层防护。
>
> **9. Hook 扩展机制** — 9 种 Hook 事件允许在工具执行前后、会话边界插入自定义逻辑。
>
> **10. Prompt Cache 优化** — 静态/动态分界线 + Fork 共享缓存 + 工具描述稳定化。

> 关联知识：[[Claude Code 整体架构]]、[[命令系统]]、[[MCP 协议]]、[[Bridge 桥接系统]]、[[守护进程模式]]、[[功能开关与死代码消除]]、[[分层安全防御]]、[[权限系统]]、[[护栏与钩子系统]]、[[流式渲染控制]]、[[子代理系统]]、[[状态管理（Claude Code）]]、[[工具系统]]、[[上下文窗口管理]]、[[Token]]、[[构建者信条]]

---

## 工具 Prompt 与自修复机制

### 核心工具 Prompt 原语

**Bash Tool** — 最高权限的工具，安全护栏最密集：
- 沙箱安全指令：默认沙箱执行，仅用户明确要求或有证据显示沙箱限制导致失败时才能禁用
- 专用工具优先原则：find/ls → Glob、grep/rg → Grep、cat/head/tail → Read、sed/awk → Edit、echo > → Write
- Git 操作规范：永不修改 git config、永不跳过 hooks、永不 force push 到 main/master
- 后台执行与命令编排：`run_in_background` 支持长时间命令，独立命令并行、依赖命令 `&&` 串联

**Edit Tool** — 精确字符串替换而非行号编辑：
- 先读后改约束：编辑前必须先用 Read Tool 读取文件
- old_string 唯一性要求：不唯一则编辑失败，解决方式是提供更大上下文或使用 `replace_all`
- 缩进保护：保留 Read Tool 输出中的 exact indentation

**Read / Write Tool** — Read 支持多模态（图片、PDF、Jupyter），Write 覆盖写入前必须已用 Read 读过该文件。严禁创建 `*.md` 文档或 README 文件。

**Glob / Grep Tool** — 分工明确：Glob 用于文件名模式匹配，Grep 用于内容搜索。对于需要多轮搜索的开放场景，建议使用 Agent Tool（Explore 类型）而非循环调用。

**Agent Tool** — 子代理调度核心入口：
- "Never delegate understanding" 原则
- Fork 模式：中间工具输出不值得保留在主上下文时使用
- Don't peek / Don't race 两条核心戒律

**Skill Tool** — 阻塞要求：当用户请求匹配某技能时，**必须先**调用 Skill Tool，禁止"提及但不调用"。

**任务规划类工具** — TaskCreate 用于 3+ 步骤复杂任务，EnterPlanMode 有明确触发条件，EnterWorktree 仅在用户明确说 "worktree" 时使用。

**辅助工具** — LSP Tool 提供代码智能操作，Sleep Tool 替代 Bash sleep 不占用 shell 进程，CronCreate Tool 避免整点和半点的雷群效应。

### Tool-Call Loop 与四层自修复机制

**工具调用闭环**：

```
Claude 生成 tool_use
    ↓
工具执行（成功或失败）
    ↓
tool_result 返回给 Claude（含 is_error 标志）
    ↓
Claude 在下一轮看到错误信息 → 分析原因 → 尝试新策略
    ↓
再次调用工具 → 循环继续
```

**第 1 层：PTL 恢复** — 上下文超模型限制时触发，三层递进：排空（contextCollapse drain）→ 压缩（reactive compact）→ 向用户报告错误。

**第 2 层：输出 Token 超限恢复** — 升级 max_tokens（8K → 64K）→ 注入恢复消息让模型从断点继续 → 最多 3 次后放弃。

**第 3 层：模型过载恢复** — 连续 529 ≥ 3 次 → 切换 fallbackModel → 丢弃失败结果 → 重试整个回合。在 `withRetry()` 中同时处理 429、401/403、ECONNRESET 等。

**第 4 层：工具执行错误恢复** — 错误消息作为 `tool_result` 返回（`is_error: true`）→ Claude 分析原因 → 调整策略 → 再次尝试。超过 10K 字符的错误消息保留头尾各 5K。

**上下文消息裁剪管道**：

```
原始消息 → applyToolResultBudget() → snipCompact() → microCompact()
         → contextCollapse() → autoCompact() → normalizeMessagesForAPI()
```

自修复机制的协同总结：反馈循环（最核心）→ 系统 Prompt 指导 → 四层恢复策略 → 防错设计 → 错误记忆保持 → 对抗性验证 → 上下文管理——多层机制协同运作的结果。

> 关联知识：[[工具系统]]、[[Claude Code 整体架构]]、[[重试与恢复机制]]、[[护栏与钩子系统]]、[[输出解析与路由]]、[[权限系统]]、[[分层安全防御]]、[[子代理系统]]、[[上下文窗口管理]]、[[Token]]、[[命令系统]]、[[守护进程模式]]、[[工具描述工程]]、[[Skills技能]]