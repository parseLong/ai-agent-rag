---
title: "Claude Code 源码架构补充 — 知识图谱可视化与模块细节"
description: 从 claude-code 源码分析的 HTML 知识图谱和7个详解页面中提炼的补充内容，查漏补缺，补全已有笔记未覆盖的关键细节
tags:
  - 概念卡片
  - 永久笔记
  - Claude Code
  - 架构
  - 工具
  - 权限
  - 上下文工程
  - 桥接
  - 多智能体
  - 命令
  - 服务
  - Prompt Cache
  - Token预算
  - Compact
  - 记忆注入
  - MCP
  - 安全
  - 工程实践
source: "基于 D:\Code\claude-code 目录下的 architecture.html + 7个 detail-*.html 文件提炼"
date: 2026-06-11
---

# Claude Code 源码架构补充

> 本笔记是对 [[Claude Code 源码架构]] 的**查漏补缺**，提炼自 D:\Code\claude-code 目录下的知识图谱 HTML 和 7 个详解页面中未被已有笔记覆盖的关键细节。

---

## 模块依赖知识图谱

源码分析 HTML 中使用 D3.js 构建了一个交互式知识图谱，包含 **84 个模块节点** 和 **100+ 条依赖连接**，覆盖了 Claude Code 的 14 个模块分类：

| 分类 | 颜色 | 关键模块 |
|------|------|---------|
| 核心入口 | 🔴 | Main, Init, REPL Launcher, Setup, Context, History |
| AI助手 | 🟠 | Assistant, Buddy |
| 桥接服务 | 🟡 | Bridge, Bridge API, Bridge Messaging, Remote Bridge |
| CLI接口 | 🟢 | CLI, CLI Handlers, CCR Client |
| 命令系统 | 🟢 | Commands + 9 个子命令 |
| UI组件 | 🔵 | Ink, UI Components + 5 个专用 UI |
| 上下文 | 🔵 | Context |
| 服务层 | 🟣 | API Service, Analytics, MCP Service, LSP, OAuth 等 13 个 |
| 工具系统 | 🟣 | Tools + 16 个工具 |
| 任务系统 | 🟣 | Tasks, Dream, Local Shell/Agent/Remote Agent, Teammate |
| 插件系统 | 💗 | Plugins, Bundled Plugins |
| 技能系统 | 💗 | Skills, Bundled Skills |
| 工具函数 | ⚪ | Utils + 15 个子模块 |
| 类型定义 | ⚪ | Types, Schemas |

### 跨模块依赖关系（图谱核心连接）

```
Main → Commands, Tools, Tasks, Plugins, Skills, Utils, Bridge, Assistant
Commands → Tools, Tasks
Tools → Utils
Tasks → Utils
Plugins → Tools
Skills → Tools
Bridge → Utils
CLI → Utils
Types → Utils
Context → Utils
MCP Service ↔ MCP Tool ↔ MCP Utils
Plugins Service ↔ Plugins ↔ Plugins Utils
```

> **关键洞察**：Utils 是几乎所有模块的共同依赖层，体现了 Claude Code 的**关注点分离**——核心逻辑在各自的模块中，通用能力下沉到 Utils。MCP 的三层结构（Service → Tool → Utils）形成了完整的从协议到执行到辅助的闭环。

---

## 上下文工程补充细节

[[Claude Code 源码架构]] 已有详细的 System Prompt 装配和上下文注入分析，但以下细节值得补充：

### 系统提示的三层并行组装

```typescript
// src/utils/queryContext.ts
const [defaultSystemPrompt, userContext, systemContext] = await Promise.all([
  getSystemPrompt(tools, mainLoopModel, ...),  // 工具描述 + 命令列表 + 能力声明
  getUserContext(),                             // CLAUDE.md + 当前日期
  getSystemContext(),                           // git status + cache breaker
])
```

三层分工与缓存策略：

| 层次 | 来源 | 内容 | 缓存策略 |
|------|------|------|---------|
| System Prompt | `getSystemPrompt()` | 工具 JSON Schema、命令列表、技能描述、Thinking 配置 | 按工具列表 + 模型 Hash 缓存 |
| User Context | `getUserContext()` | CLAUDE.md 内容、当前日期、记忆文件 | 按文件内容 Hash 缓存 |
| System Context | `getSystemContext()` | git status、分支信息、cache breaker | 会话级 Memoize |

> `getUserContext` 和 `getSystemContext` 使用 **lodash memoize** 缓存整个会话周期，避免重复读取文件和执行 git 命令。

### Prompt Caching 的精细 Hash 检测

缓存失效检测通过 `promptCacheBreakDetection.ts` 实现，追踪以下状态变化：

```typescript
type PreviousState = {
  systemHash: number           // 系统提示文本 Hash
  toolsHash: number             // 工具 Schema Hash
  cacheControlHash: number      // cache_control 标记 Hash
  toolNames: string[]           // 工具名称列表
  perToolHashes: Record<string, number>  // 每个工具的独立 Hash
  model: string
  betas: string[]               // Beta Header 列表
  globalCacheStrategy: string
}
```

缓存范围策略：

| 范围 | 说明 |
|------|------|
| `ephemeral` | 单次请求有效，不缓存 |
| `default` | 5 分钟 TTL，适合工具描述等稳定内容 |
| `global` / `org` | 更长 TTL，需要特定权限 |

> **关键设计**：只有**前缀稳定**的内容才能被缓存。如果工具列表发生变化（如新增 MCP 工具），整个缓存都会失效，因为 `toolsHash` 变了。

### 消息历史的序列化处理

内部消息在发送给 API 前需要 `normalizeMessagesForAPI()` 转换：

```typescript
export function normalizeMessagesForAPI(messages: Message[]): MessageParam[] {
  return messages
    .filter(msg => msg.type !== 'tombstone')          // 删除墓碑
    .filter(msg => !isCompactBoundaryMessage(msg))     // 跳过压缩边界
    .map(msg => convertToAPIFormat(msg))
}
```

图片剥离策略：在上下文压缩前主动剥离图片内容，避免压缩 API 本身超出 Token 限制——将 image 替换为 `[image]` 文本标记。

### Compact 压缩的精确 Token 预算

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `POST_COMPACT_MAX_FILES_TO_RESTORE` | 5 | 最多恢复 5 个文件 |
| `POST_COMPACT_TOKEN_BUDGET` | 50,000 | 压缩后可用 50K Token |
| `POST_COMPACT_MAX_TOKENS_PER_FILE` | 5,000 | 每个文件最多 5K |
| `POST_COMPACT_MAX_TOKENS_PER_SKILL` | 5,000 | 每个技能最多 5K |
| `POST_COMPACT_SKILLS_TOKEN_BUDGET` | 25,000 | 技能总预算 25K |

> **压缩边界**不是删除消息，而是用一个特殊的系统消息替代。这样 resume 时可以从头恢复完整历史。

### 记忆注入的优先级

1. 项目根目录 `CLAUDE.md`
2. 用户目录 `~/.claude/CLAUDE.md`
3. 动态发现的技能文件

### 上下文工程五条设计哲学

| 原则 | 实现 |
|------|------|
| **缓存最大化** | 通过精细的 Hash 检测和 `cache_control` 标记，最大化 Prompt Caching 命中率 |
| **压缩不可逆** | 压缩边界是单向的，但一旦 resume 可以重建完整历史 |
| **Token 透明** | 所有 Token 计数公开透明，用户可以看到实时使用情况 |
| **成本可控** | 通过 `maxBudgetUsd` 和自动压缩，防止费用失控 |
| **记忆持久** | CLAUDE.md 和技能文件的记忆跨会话持久化 |

---

## 工具系统补充细节

### 工具池组装的 Prompt Cache 稳定性设计

```typescript
export function assembleToolPool(
  permissionContext: ToolPermissionContext,
  mcpTools: Tools,
): Tools {
  const builtInTools = getTools(permissionContext)
  const allowedMcpTools = filterToolsByDenyRules(mcpTools, permissionContext)
  
  // 按名称排序后去重，内置工具优先
  return uniqBy(
    [...builtInTools].sort(byName).concat(allowedMcpTools.sort(byName)),
    'name'
  )
}
```

> **Prompt Cache 稳定性**：工具列表的排序直接影响 Anthropic API 的 Prompt Caching。通过保证内置工具在前、MCP 工具在后，且各自按名称排序，可以在添加/移除 MCP 工具时**最小化缓存失效范围**。

### 权限过滤的三层模型

```
第一层：Deny 规则 → blanket deny 整个工具类别
    ↓
第二层：模式过滤 → REPL 模式隐藏原始工具；Simple 模式仅保留 Bash/Read/Edit
    ↓
第三层：运行时可用性 → 调用每个工具的 isEnabled()
```

### 工具注册的显式枚举策略

```typescript
export function getAllBaseTools(): Tools {
  return [
    AgentTool, BashTool, FileReadTool, ...
    // 特性开关控制的工具
    ...(isWorktreeModeEnabled() ? [EnterWorktreeTool, ExitWorktreeTool] : []),
    ...(isAgentSwarmsEnabled() ? [getTeamCreateTool(), getTeamDeleteTool()] : []),
  ]
}
```

> **为什么不用自动扫描？** 自动扫描虽然方便，但会破坏 Bun 的死代码消除。手动枚举确保每个工具都有明确的注册路径，也便于代码审查时追踪工具的依赖关系。

### 工具系统的五大设计模式

| 模式 | 实现 | 收益 |
|------|------|------|
| 策略模式 | 每个工具是 Tool 接口的不同实现 | 统一处理、易于扩展 |
| 模板方法 | 工具调用遵循固定生命周期 | 可插入权限、日志、重试 |
| 装饰器模式 | 权限过滤包装原始工具 | 安全与业务解耦 |
| 工厂模式 | `getTools()` 根据环境组装工具 | 运行时动态配置 |
| 适配器模式 | MCP 工具转换为 Tool 接口 | 无缝集成第三方工具 |

---

## 命令系统补充细节

### 三种命令类型的深层区别

| 类型 | 用途 | 执行方式 | 示例 |
|------|------|---------|------|
| `prompt` | 展开为 Prompt 发送给 LLM | 生成提示词 → 发送给 Claude API | `/commit`, `/review` |
| `local` | 本地执行，不经过 LLM | 执行 JS 函数，返回字符串 | `/cost`, `/theme` |
| `local-jsx` | 渲染 Ink UI 组件 | 调用 Ink 组件渲染 React 节点 | `/doctor`, `/resume` |

> **关键区别**：prompt 命令本质是**提示词模板**，展开后变成用户消息的一部分发送给 LLM；local 命令是纯本地逻辑，不涉及 API 调用。

### 命令池的 7 个动态组装来源

```
1. Bundled Skills → 内置技能
2. Built-in Plugin Skills → 内置插件技能
3. Skill Dir Commands → ~/.claude/skills/ 自定义技能
4. Workflow Commands → 工作流脚本
5. Plugin Commands → 第三方插件
6. Plugin Skills → 插件技能命令
7. Built-in Commands → 核心内置命令（50+ 个）
```

`loadAllCommands` 使用 **lodash memoize** 缓存结果，避免重复的磁盘 I/O。

### 内部命令隔离

`INTERNAL_ONLY_COMMANDS` 集合（如 `backfillSessions`, `breakCache`, `bughunter`）只在 `USER_TYPE === 'ant'` 时注册，确保外部用户无法访问。

### 远程安全命令白名单

| 集合 | 用途 | 示例 |
|------|------|------|
| `REMOTE_SAFE_COMMANDS` | 远程模式下可安全执行 | session, exit, clear, help, theme, cost |
| `BRIDGE_SAFE_COMMANDS` | 桥接时可执行 | compact, clear, cost, summary, files |

> 远程模式下 `/commit` 等会修改本地文件系统的命令会被自动过滤，防止远程控制意外破坏代码库。

---

## 服务层补充细节

### API 请求生命周期

```
1. 消息序列化 → normalizeMessagesForAPI
2. Beta Header 注入 → prompt-caching, thinking, effort 等
3. 流式请求 → SSE 流式
4. 事件解析 → message_start → content_block_delta → message_stop
5. Token 追踪 → 累加 usage，更新 cost-tracker
```

### MCP 传输协议的五种类型

| 传输 | 用途 | 配置示例 |
|------|------|---------|
| `stdio` | 本地子进程 | `command: "npx", args: [...]` |
| `sse` | Server-Sent Events | `url: "https://mcp.example.com/sse"` |
| `ws` | WebSocket | `url: "wss://mcp.example.com/ws"` |
| `http` | HTTP 长轮询 | `url: "https://mcp.example.com"` |
| `sdk` | 内置 SDK 模式 | `name: "claude-code-sdk"` |

> **MCP 工具与内置工具完全对等**：LLM 无法区分一个工具是内置的还是 MCP 提供的。MCP 工具通过 `tool.name` 和 `tool.inputJSONSchema` 描述自己，权限系统也统一处理。

### 特性开关的两种模式

| 模式 | API | 用途 |
|------|-----|------|
| Cached | `checkGate_CACHED_MAY_BE_STALE` | 快速读取，可能返回过期值 |
| Blocking | `checkGate_CACHED_OR_BLOCKING` | 保证新鲜，可能阻塞 |

---

## 桥接系统补充细节

### 三种桥接模式

| 模式 | 触发方式 | 用途 |
|------|---------|------|
| IDE Bridge | `--ide` 参数 | 连接 VS Code / JetBrains 扩展 |
| Remote Control | `--remote` 参数 | 手机/网页远程控制 |
| REPL Bridge | 自动检测 | REPL 会话中建立桥接 |

### 权限桥接的请求-响应模式

```typescript
function bridgePermissionRequest(request: PermissionRequest): Promise<PermissionResponse> {
  return new Promise((resolve) => {
    sendToBridge({ type: 'permission_request', payload: request })
    onBridgeMessage((msg) => {
      if (msg.type === 'permission_response') {
        resolve(msg.payload)
      }
    })
  })
}
```

> **关键设计**：权限桥接采用**请求-响应**模式，而非事件广播。每个权限请求都有唯一的 requestId，确保在多并发请求时不会混淆。

### 安全模型五要素

1. **JWT 认证**：每个连接都需要有效的 JWT 令牌
2. **Token 刷新**：JWT 过期前自动刷新
3. **权限中继**：权限请求必须通过桥接转发，不能绕过
4. **命令白名单**：远程模式下只有白名单命令可用
5. **会话隔离**：每个会话独立，不共享状态

---

## 权限系统补充细节

### 权限决策的六步流程

```
1. 模式检查 → bypassPermissions 直接允许
    ↓
2. Deny 规则匹配 → alwaysDenyRules 匹配则拒绝
    ↓
3. Allow 规则匹配 → alwaysAllowRules 匹配则允许
    ↓
4. Hook 检查 → 权限 Hook 可能修改决策
    ↓
5. Classifier 检查 → Bash 命令通过 Classifier 自动分类
    ↓
6. 用户确认 → 弹出权限请求对话框
```

### Bash Classifier 的三层风险分类

| 风险等级 | 命令类型 | 处理策略 |
|---------|---------|---------|
| Low Risk | `ls`, `cat`, `pwd`, `git status` | 自动批准 |
| Medium Risk | `git commit`, `git push` | 询问用户 |
| High Risk | `rm -rf`, `curl | sh` | 必须用户确认 |

> **Classifier 不是 LLM**：基于规则 + 简单启发式的分类器，不需要调用 API，几毫秒内完成判断，不阻塞用户。

### 权限持久化的三层作用域

| 作用域 | 存储位置 | 生命周期 |
|--------|---------|---------|
| Session | 内存 | 当前会话 |
| Local | `~/.claude-code/settings.json` | 所有会话 |
| Global | `~/.claude-code/settings.json` | 所有项目 |

### 权限系统的四大设计模式

- **责任链模式**：权限检查按规则 → hook → classifier → 用户的顺序链式执行
- **策略模式**：不同权限模式是不同的策略实现
- **观察者模式**：权限请求通过队列异步处理，UI 可以订阅队列变化
- **备忘录模式**：权限规则持久化存储，支持恢复

---

## 多智能体与技能补充细节

### AgentTool 的子智能体隔离性

- **独立 QueryEngine**：有自己的消息历史、文件缓存和 Token 计数
- **工具过滤**：子智能体只能使用 `filterToolsForAgent` 允许的子集
- **上下文隔离**：子智能体看不到父智能体的消息历史
- **工作目录隔离**：可以指定不同的 cwd

### Skill 加载的四种来源

| 来源 | 路径 | 优先级 |
|------|------|--------|
| Bundled | 内置在源码中 | 最低 |
| Skill Dir | `~/.claude/skills/` | 中 |
| Plugin | 插件提供 | 高 |
| Dynamic | 运行时发现 | 最高 |

### 插件系统的四项能力

1. **注册命令**：添加新的 slash 命令
2. **注册工具**：添加新的 Agent 工具
3. **注册 MCP 服务器**：自动配置 MCP 连接
4. **修改 UI**：通过 Ink 组件扩展界面

---

## 相关知识

- [[Claude Code 源码架构]] — 已有的完整架构梳理
- [[Claude Code 架构解析]] — 从单代理到自主团队的演进路径
- [[Claude Code 知识地图]] — MOC 导航入口
- [[上下文工程]] — 上下文管理的完整方法论
- [[Prompt Cache]] — 缓存机制的核心设计
- [[权限系统]] — 安全边界的详细实现