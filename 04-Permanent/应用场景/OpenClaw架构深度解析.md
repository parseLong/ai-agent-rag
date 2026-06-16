---
title: OpenClaw 架构深度解析
tags:
  - 原理
  - Agent框架
  - 开源
  - 微内核
  - 记忆系统
  - 多智能体
  - 上下文工程
  - 工程实践
sources:
  - "[[OpenClaw与Hermes源码架构深度复盘]]"
  - "[[OpenClaw与Hermes：源码里的 AI Agent 架构知识大复盘 1]]"
  - "[[玩转OpenClaw，你需要了解的：核心架构、运作原理、Agent部署步骤]]"
  - 罗福莉（小米）访谈
  - Anthropic Blog: Building Effective Agents (2024.12)
---

> [!quote] 核心范式转变
> OpenClaw 的设计哲学映射了一个行业级范式转变：从 *"Prompt + Tool = Agent"* 到 *"Memory + Context + Orchestration = Agent System"*。OpenClaw 不是在写更好的 Prompt，而是在==设计一套让 Context 自运转的工程系统==。

---

## 定义与核心价值

OpenClaw 是一个划时代的开源 Agent 框架，能够通过精细编排的 Context 弥补模型短板，激发中层模型的能力上限。

| 特点 | 说明 |
|------|------|
| **精细编排的 Context** | 在细节上（如时间感知）做得非常出色，每轮对话前面拼上当前时间 |
| **持久化记忆体系** | 对 memory 有分层和分级设计（Session → 按天记忆 → 精炼记忆） |
| **多模型联合利用** | 能自动选择能力最好的模型完成任务 |
| **多 Agent 天然支持** | Context 瓶颈约束单 Agent，专事专做是共识 |

### 激发中层模型上限

> "因为这些设计，它激发了中层模型的上限。我们如果没有这么一套很复杂的 Agent 框架，中层模型达不到近似 Claude Sonnet 或 Opus 的水平。但你借助了这样一个非常好的 Agent 框架，就能应付绝大部分场景。" — 罗福莉

### 共识推广的力量

> "这也是我第一次感受到，怎么用一群人的智慧去提升一个事情。如果我自己单一去改这个 Agent 框架，但别人感受不到这个框架的智能，好像差点意思。这个框架本身的进步速度非常慢。但如果一群人去改进，进步速度非常快，几小时就迭代一轮。" — 罗福莉

OpenClaw 之前，几乎每人都在自建 Agent 架构。每次交流都要先介绍各自的架构再聊具体落地。OpenClaw 把架构标准化后，开发者之间直接聊保活、RAG 算法替换、多 Agent 部署等实际问题——沟通效率大幅提升。

> [!quote] 关键洞察
> OpenClaw 的技术框架并不复杂（类似「具备初级推荐算法的前后端通信 App」的难度），它的优势在于**共识的推广**。

### 做与 AI 能力正交的事

花时间打造和迭代自己的 Agent，跟培养一个人一样——他可以很聪明，但认知世界和做事的能力需要我们教导，这是千人千面的。当 AI 模型越来越聪明，只需升级底层 LLM 即可，而与 AI 交互积累的长期数据，将成为更好驱动 AI 的私人宝贵数据。

---

## 微内核架构哲学

> [!info] 一句话理解
> "微内核"类比手机操作系统：核心只管消息路由、会话管理、安全网关（如同内核只管进程调度和内存），所有具体能力——IM 通道、LLM 模型、浏览器工具——都是"应用"，通过标准接口注册进系统。好处是换一个"应用"不影响其他，坏处是每个"应用"要自己处理认证/重连/诊断等全生命周期。

5 大设计理念共同支撑"让 Context 自运转"这一目标：

### 理念一：本地优先（Local-First）

OpenClaw 不是云服务，而是运行在用户设备上的 Gateway 进程。所有会话数据、配置、媒体文件存储在 `~/.openclaw/` 目录下。Gateway 是控制平面，Agent 是产品本身。

### 理念二：万物皆插件（Everything is a Plugin）

核心代码只负责编排——消息路由、会话管理、安全网关。所有具体能力（Discord 通道、Anthropic 模型、浏览器工具）以插件形式实现，统一通过 Plugin SDK 注册。

> "Security in OpenClaw is a deliberate tradeoff: strong defaults without killing capability."

### 理念三：安全纵深（Defense in Depth）

不是简单的"开或关"，而是五层递进防御——网络层 TLS → 认证层 Device Identity → 命令执行审批 → 插件安装扫描 → 沙箱隔离。执行策略默认为 `deny`，所有 shell 命令需要白名单或人工审批。

### 理念四：记忆驱动（Memory-Driven）

Agent 不仅有静态工作区文件（SOUL.md / USER.md / MEMORY.md）定义人格与记忆，还有向量记忆引擎实现混合搜索、Dreaming 后台整合和 Active Recall 主动召回。记忆按**Agent 维度隔离**——同一 Agent 下所有用户共享记忆。

### 理念五：配置驱动（Config-Driven）

一个 JSON 文件（`openclaw.json`）定义所有行为——Agent 配置、Channel 凭证、模型选择、安全策略、定时任务。支持运行时热重载，改配置不需要重启。

---

## Agent 配置文件体系

每个 Agent 都有独立的 workspace，核心配置由 **8 个文件** 构成完整人格：

| 文件 | 作用 | 优先级 |
|------|------|--------|
| `AGENTS.md` | 职责声明，决定工具权限 | 最高 ⭐ |
| `SOUL.md` | 个性化提示词，注入 system prompt | |
| `TOOLS.md` | 工具白名单/黑名单，安全边界 | |
| `IDENTITY.md` | 身份标识（name/avatar），通道展示 | |
| `USER.md` | 用户偏好，上下文先验 | |
| `HEARTBEAT.md` | 定时任务配置（可选） | |
| `BOOTSTRAP.md` | 首次 onboarding 引导（一次性消费） | |
| `MEMORY.md` | 用户记忆文档（RAG 源） | 动态检测 |

> [!tip] 重点阅读
> ⭐ **AGENTS.md** 是 OpenClaw 最核心的 Prompt 文件，详细介绍了 Agent 启动和 memory 管理的完整流程。

文件加载顺序即优先级：`AGENTS` 定义能力边界 → `SOUL` 注入灵魂 → `TOOLS` 划定禁区。源码中 `loadWorkspaceBootstrapFiles()` 方法按此顺序逐个读取，同时通过 `openBoundaryFile` 检查 inode/dev/size/mtime，防止路径穿越。

> [!important] 为什么是 Markdown 文件而非代码配置？
> 1. **人类可读可写**：非程序员也能理解和修改 Agent 人格
> 2. **注入友好**：直接作为 system prompt 内容注入 LLM，无需解析/转换
> 3. **版本可控**：每个文件独立演进，可单独追踪变更
> 4. **社区共享**：Markdown 天然适合分享和传播——这是「共识推广」的技术基础

### Agent 启动流程

Agent ==不是常驻进程==，而是 **per-session 的瞬态实例**。每个对话都是一次完整的「加载→执行→销毁」循环：

```
1. 加载 bootstrap 上下文（resolveBootstrapContextForRun）
2. 创建 SessionManager（封装 session 持久化）
3. 构建 system prompt（注入 workspace 文件内容）
4. 创建 agent session（createAgentSession）
```

> [!important] System Prompt 是动态生成的
> 每次 run 都会重新读取 workspace 文件，确保配置实时生效。

> [!tip] 瞬态模式的核心优势
> - **状态一致性**：每次从文件加载，保证配置变更实时生效
> - **内存可控**：没有累积膨胀问题
> - **故障隔离**：一次对话崩溃不影响下一次
> - **天然适合多 Agent**：每个 Agent 都是独立的 workspace，互不干扰

代价是每次对话有启动开销，但 OpenClaw 通过 workspace 文件体系的结构化加载来控制这个开销。

---

## Gateway — 系统心脏

Gateway 默认监听 `:18789`，职责远不止消息收发。42 个 RPC handler 模块按功能域归纳为十余类——聊天、会话、配置热加载、模型目录、执行审批、定时任务、远程节点、语音唤醒等几乎所有功能域都通过它暴露和调度。

### Gateway 的 5 大角色

**角色 1：唯一长驻进程（Single Source of Truth）**

一个 Gateway per host，拥有所有 channel session——避免多进程下的"WhatsApp 二次扫码、Telegram session 冲突"等致命问题。

**角色 2：消息总线（一切流量必经之路）**

所有 channel、client、node 流量都走 Gateway 或由 Gateway 分发。不分协议入口——HTTP、SSE、私有 RPC 全部统一到 WS Schema。

**角色 3：多 Agent 路由的物理边界**

通过 Multi-Agent Router 做 Agent 隔离——来自 Telegram@user1 的消息路由到 Agent A，来自 Discord@user2 的路由到 Agent B。**不同 Agent 物理隔离**（独立 workspace / SOUL / MEMORY / sessions）。这是解决单 Agent 三个瓶颈的关键：

- **上下文污染**：同一个 Agent 同时处理编程问题和情感倾诉，LLM 上下文会被无关信息淹没——编程 Agent 里突然出现"我今天心情不好"
- **工具链冲突**：编程 Agent 需要 `browser` 工具，客服 Agent 需要 `telegram_get_chat_members`——同一 Agent 装所有工具既浪费 token 又增加安全风险
- **渠道风格差异**：Discord 用户习惯简短俚语，邮件用户期望正式措辞——同一人格无法同时适配两种风格

> [!warning] 单 Agent 的 Memory 问题
> 单 Agent 讨论多个话题后，Memory 会携带无关上下文。例如：与 RAG tutor Agent 讨论过 C++ 后，它再提供 Flutter 示例时会错误地给出 C++ 相关的 Demo——这就是为什么需要多 Agent ==专事专做==。

**角色 4：认证 + 信任边界（Gateway 是保安，不是大门）**

Gateway 不只是"让人进来"，更关键的是**根据你从哪来，决定信你多少**。类比：公司前台对内部员工刷卡放行、对来访客人登记+审批、对陌生人要求身份证+预约确认。

| 网络环境 | 信任等级 | Gateway 怎么做 | 为什么 |
|----------|----------|----------------|--------|
| **同主机 loopback**（本机 `127.0.0.1`） | 🟢 全信任 | 自动放行，不需要任何认证 | 你已经在机器上了，说明你有这台机器的权限 |
| **内网 / Tailnet**（局域网、VPN 私有网络） | 🟡 半信任 | 要求签名验证 + 配对审批 | 网络里的人不一定都是你——需要证明"我是我之前配对过的那个设备" |
| **公网 ingress**（互联网任意来源） | 🔴 严信任 | 强制共享密钥 + 幂等键 | 任何人都能从公网敲门——必须用密钥证明身份 |

**三个关键设计词**：

- **Pairing v3 协议** = "首次配对"机制。新设备想连 Gateway，必须走一次配对流程，配对后获得身份凭证。一旦凭证变更，必须重新配对——防止"冒名顶替"。
- **Idempotency key（幂等键）** = 每次操作附带一个唯一标识符。类比银行转账的"交易流水号"——网络断了重试，Gateway 发现幂等键相同就知道"这笔已经执行过了"，不会重复扣款/发消息。
- **Token-based device identity** = 每台设备有独一无二的令牌，Gateway 用它识别"这是之前配对过的设备 A"而非"随便一个新连接"。

> [!important] 为什么信任要分级而非一刀切？
> 一刀切的"全部强制认证"会让本地开发体验极差；一刀切的"全部放行"会让公网暴露后门户大开。分级信任是**安全性和便利性的平衡**。

**角色 5：嵌入式 HTTP Host**

同端口 18789 提供 `/__openclaw__/canvas/`（Agent 可编辑的 HTML/CSS/JS）和 `/__openclaw__/a2ui/`（A2UI 主机界面）——Agent 可以**主动构造 UI**让用户在浏览器看。

### "边界 vs 实现"哲学

| 事 | 聁做 | 为什么 |
|----|------|--------|
| 协议定义（WS schema） | Gateway | 协议是"交通规则"，必须由交通局定 |
| 路由 + 认证 | Gateway | 路由是"分诊台"，认证是"门卫"——都是边界职责 |
| WhatsApp 会话生命周期 | Gateway | 会话状态是核心资产，不能散落在各插件里 |
| Agent 推理 | **Embedded Pi Runtime** | 推理是"干活"，不是"管边界" |
| Channel 消息收发 | **Channel Plugins** | 收发是"具体平台操作"，换平台就换插件 |
| Memory 整理 | **memory-core 插件** | 记忆策略可替换 |
| 工具执行 | **Plugins, Skills** | 工具是"能力"，按需插拔 |

---

## Session Key — 消息到 Agent 的精确路由

```
agent:{agentId}:{scope}
```

| 场景 | Session Key 示例 |
|------|-------------------|
| 默认主会话 | `agent:main:main` |
| QQ 私聊 | `agent:main:qqbot:default:direct:207A5B83...` |
| Discord 群组 | `agent:support:discord:acc1:group:123456789` |
| Telegram 线程 | `agent:main:telegram:bot1:direct:user456:thread:msg789` |

### 多 Agent 路由 9 级优先级

> [!info] 为什么需要 9 级？
> 类比邮件路由：最精确的"给张三部门5的工位3"→ 较宽的"给张三部门"→ 最宽的"给公司"。消息路由也是从最具体到最宽泛逐级回退。

| 优先级 | 匹配维度                           | 示例                        |
| --- | ------------------------------ | ------------------------- |
| 1   | `binding.peer` 精确用户            | QQ 用户 A → support Agent   |
| 2   | `binding.peer.parent` 线程父级     | Telegram 线程继承父会话绑定        |
| 3   | `binding.peer.wildcard` 同类型通配  | 所有 QQ 私聊 → 同一 Agent       |
| 4   | `binding.guild+roles` 服务器 + 角色 | Discord 管理员 → dev Agent   |
| 5   | `binding.guild` 服务器            | Discord 服务器 → dev         |
| 6   | `binding.team` 团队              | MS Teams team → ops Agent |
| 7   | `binding.account` Bot 账号       | qqbot:bot2 → bot2 Agent   |
| 8   | `binding.channel` 整个通道         | 所有 Discord → dev          |
| 9   | `default` 兜底                   | 未匹配 → main Agent          |

> [!note] DM 隔离策略
> `session.dmScope` 配置控制私聊会话隔离粒度。默认 `per-channel-peer`；多账号场景可设为 `per-account-channel-peer`。

### 同一 Agent 多用户隔离问题

同一 Agent 下多用户并发使用时，**会话隔离但记忆共享**——所有用户的记忆都写入同一个 `memory/` 目录。

> [!warning] 为什么同一 Agent 多用户不推荐
> OpenClaw 定位为**个人 AI Agent**——设计上假设一个 Agent 服务一个人。**正确做法是为不同用户配置多 Agent 路由绑定**。简单说：**一个 Agent = 一份记忆 = 一个服务对象**。

---

## 多 Agent 通信与部署

### 4 种协作模式

| 模式 | 底层工具 | 做法 | 适用场景 |
|------|----------|------|----------|
| **Supervisor** | `sessions_send` | 主 Agent 调度：编程需求 → @coder，写作需求 → @writer，最后汇总 | 中央统筹 |
| **Router** | `sessions_send` | 主 Agent 只做路由分发，不参与执行 | 分诊台 |
| **Pipeline** | `sessions_send` | A 的输出是 B 的输入，串行传递 | 翻译 → 润色 → 排版 |
| **Parallel** | `sessions_spawn` | 主 Agent spawn 多个子代理并行执行，全部完成后汇总 | 同时翻译 3 篇文章 |

**两套机制的区别**：

- `sessions_send`（Agent 间通信）= 向**已有**的另一个 Agent 发消息，两个 Agent 各自独立存在
- `sessions_spawn`（Subagent 委派）= **创建**一个临时子 session 执行任务，干完即走

框架通过 `maxPingPongTurns`（最大 5 轮）防止 Agent 间 `sessions_send` 无限来回。

### 多 Agent 部署配置

新增 Agent 指令：

```
openclaw agents add <agent-name>
```

按 OpenClaw onboarding 流程配置，填写 IM bot API 和 LLM API 即可。

#### sessions_send 配置

```json
"tools": {
  "agentToAgent": {
    "enabled": true,
    "allow": ["ceo", "iostutor"]   // 所有 Agent 都加进来
  },
  "sessions": {
    "visibility": "all"   // ← 特别容易漏，一定要配
  }
}
```

#### sessions_spawn 配置

```json
{
  "id": "ceo",
  "subagents": {
    "allowAgents": ["iostutor"]   // ← 加上除自己之外的其他 Agent
  }
}
```

#### 决策机制

LLM 自行判断使用哪种方式：
- 「让 iostutor Agent 生成一个文章发给我」→ **spawn**（新任务）
- 「继续跟 iostutor 的那个文章讨论」→ **send**（有明确上下文指向）

> [!bug] Agent 遗忘问题
> 短期记忆遗忘后，Agent 会忘记如何进行 AgentToAgent 通信。解决方案：在 `AGENTS.md` 中明确声明各 Agent 的身份和通信方式：
>
> ```markdown
> ## AgentToAgent 通信
> 当你需要其他角色的意见或具体执行时：
> - **iOS 问题** → `@iostutor` 获取技术评估
> 使用 `callAgent("iostutor")` 来直接通信。
> ```

### 多 Agent 实战经验

| 原则 | 说明 |
|------|------|
| **扁平化** | 不要部署太多 Agent，多层级汇报关系非常不建议 |
| **双向沟通** | Agent 同时配置 sessions_send 和 sessions_spawn |
| **设立核心边界** | 工具型 Agent（如 Markdown 排版）只配 SubAgent，不需记住太多上下文 |

---

## Client vs Channel — 两个正交概念

| | Client | Channel |
|--|--------|---------|
| **定义** | Gateway 外部的连接方（TUI / Control UI / Mobile App） | Gateway 内部的插件模块 |
| **协议** | 通过 WebSocket 连入 Gateway，走 Ed25519 认证 | 跟 Gateway 是函数调用（不需 WS、不需鉴权） |
| **类比** | "谁在操作 Agent" | "Agent 通过哪条线路收发消息" |

两者通过 **SessionKey** 交汇——同一个用户可以在手机 OpenClaw App（Client）上看到 QQ Channel 产生的对话，也能在 TUI（Client）里继续回复。

> [!important] 安全约束
> - **非 loopback 强制 TLS**：只要不是本机访问，就拒绝明文 `ws://` 连接
> - **TLS 证书指纹 Pinning**：不只加密，还验证"加密的对象是不是我信任的那个"
> - **控制平面写操作限流**：改配置、删数据等高风险操作不能无限请求
> - **RBAC Scope 最小权限校验**：每个操作只授予完成该操作所需的最低权限

---

## 插件系统 — 分类、注册与安全

### 插件分类 6 大类

| 分类 | 代表插件 | 说明 |
|------|----------|------|
| **Channel** | Discord / Telegram / QQ Bot / Slack / 飞书 / WhatsApp / Signal / MS Teams | 消息通道接入 |
| **Provider** | Anthropic / OpenAI / Google / DeepSeek / Ollama / Groq / Bedrock | LLM 模型提供商 |
| **Tool** | Browser / Exa / Firecrawl / Tavily / SearXNG / Brave / DuckDuckGo | Agent 外部工具 |
| **Media** | ElevenLabs / Deepgram / MLX Talk / Voice-Call | TTS / STT / 本地推理 |
| **Memory** | Memory-LanceDB / Memory-Wiki / Active Recall / Dreaming | 记忆存储、主动召回、睡眠整理 |
| **基础设施** | Diagnostics-OTEL / Device-Pair / Thread-Ownership / Compaction | 监控、设备配对、压缩等 |

### 插件注册与安全检查

- **注册模式**：每个插件通过 `definePluginEntry()` 或 `api.registerChannel/registerProvider/registerTool()` 注册
- **发现机制**：启动时扫描已安装插件目录，按优先级加载
- **安全检查**：路径遍历防护、文件权限检查、所有权校验、安装时静态代码扫描

### Channel Plugin 完整契约 — 25+ 适配器槽位

> [!info] 一句话理解
> ChannelPlugin 不是"收发消息的管道"，而是一个**IM 平台的完整生命周期协作单元**。

源码定义了 30+ 个可选槽位，按功能域分为 7 组。**所有槽位都是可选的**——简单 webhook 只需实现 4 个必选项，Telegram/Discord 实现了 30+ 个。

#### ① 必选 4 项

| 槽位 | 作用 |
|------|------|
| `id` | 唯一标识 |
| `meta` | 元数据（图标、名称、类型） |
| `capabilities` | 能力声明 |
| `config` | 配置适配器 |

#### ② Setup 三件套

| 槽位 | 作用 |
|------|------|
| `setupWizard` | 配置向导 |
| `setup` | 安装流程 |
| `configSchema` | 配置校验 |

#### ③ Auth + Security 7 项

| 槽位 | 作用 |
|------|------|
| `auth` | 认证适配 |
| `pairing` | 设备配对 |
| `security` | 安全策略 |
| `approvalCapability` | 审批能力 |
| `elevated` | 提权操作 |
| `secrets` | 密钥管理 |
| `allowlist` | 白名单 |

#### ④ Messaging 7 项

| 槽位 | 作用 | 一句话 |
|------|------|--------|
| `messaging` | 消息路由 | 收到消息后决定转发给谁 |
| `message` | 消息解析 | 翻译成 OpenClaw 内部统一格式 |
| `outbound` | 出站适配 | 翻译回平台格式并发送 |
| `streaming` ⭐ | 流式协议 | 每个 IM 的流式语义不同，这个槽位把差异封装掉 |
| `threading` | 线程管理 | 平台特有的线程机制 |
| `mentions` | @提及 | 解析 @提及事件 |
| `agentPrompt` | 消息转 Prompt | 加工成发给 LLM 的 Prompt |

#### ⑤ 协作能力 7 项

| 槽位 | 作用 |
|------|------|
| `commands` | 斜杠命令 |
| `groups` | 群组管理 |
| `directory` | 目录查询 |
| `resolver` | 实体解析 |
| `bindings` | 路由绑定 |
| `conversationBindings` | 会话级绑定 |
| `actions` | 消息动作 |

#### ⑥ Gateway + 运维 6 项

| 槽位 | 作用 | 一句话 |
|------|------|--------|
| `gateway` ⭐ | 协议绑定 | Channel 与 Gateway 的脐带 |
| `gatewayMethods` | 方法列表 | 暴露给 Gateway 的方法名 |
| `lifecycle` | 生命周期回调 | 启动、连接、断开、重连 |
| `status` | 状态上报 | 让 Gateway 知道 Channel 当前能不能干活 |
| `heartbeat` | 心跳检测 | 定期 ping 确认连接还活着 |
| `doctor` ⭐ | 自诊断 | 问题自查，不等人报 |
| `reload` ⭐ | 精细化热重载 | 配置变更时只重启这个 Channel |

#### ⑦ 反向工具（Channel → LLM）

| 槽位 | 作用 | 一句话 |
|------|------|--------|
| `agentTools` ⭐ | 反向工具工厂 | Channel 给 LLM 提供平台专属工具——Channel 不只是消息通道，还是 LLM 的能力扩展源 |

> [!important] 可选性是微内核和单体框架的分水岭
> Hermes 的 Channel 只有一个统一的 `send/recv` 函数式接口，而 OpenClaw 把 Channel 当 IM 域的**完整协作单元**，包含认证、配对、审批、命令、诊断、热重载等全生命周期能力。

### Channel ↔ Gateway 5 种交互模式

| 模式 | 方向 | 说明 |
|------|------|------|
| 入站消息 | Channel → Gateway → Agent | 归一化 → Gateway 路由 |
| 出站回复 | Agent → Gateway → Channel | Agent 出 turn → Gateway 派单 |
| 客户端控制 | Client → Gateway → Channel | WS method → ChannelGatewayAdapter |
| **反向工具** | Channel → Agent | agentTools 注册到 Agent tool registry |
| 反向通知 | Channel → Gateway → Client | event:presence / event:tick |

### Channel Docking — 跨 Channel 会话迁移

用户在 Telegram 发起会话后想切到 Discord 继续，发 `/dock_discord`，Gateway 验证 `identityLinks` 确认两个账号属于同一用户后，保留 session 上下文不变，只换投递地址。**不重建 session**——相当于"AI 会话的呼叫转移"。

---

## Agent 执行引擎分层架构

### 四层结构

| 层 | 负责模块 | 做什么 | 一句话理解 |
|----|----------|--------|------------|
| **循环层** | `@mariozechner/pi-agent-core` | ReAct 循环、工具调用、流式输出 | "Agent 的手脚" |
| **编排层** | OpenClaw `pi-embedded-runner/` | 预算控制、Auth Profile failover、Compaction、Lane 分车道 | "Agent 的调度员" |
| **拦截层** | OpenClaw hooks | `beforeToolCall` / `afterToolCall` / `transformContext` | "Agent 的安检门" |
| **能力层** | OpenClaw plugins | 记忆检索、消息出站、模型适配 | "Agent 的工具库" |

### 关键设计点

- **AgentMessage ≠ LLM Message**：内部用自定义 `AgentMessage`——支持 `compactionSummary` / `notification` / `steering` 等非 LLM 消息类型。只在调 LLM 边界才通过 `convertToLlm` 转成标准 `Message[]`
- **StreamFn 可替换**：可换成自定义函数——CLI Backend 就是用这个把 Claude Code 的 stdio 流当作"LLM 响应"
- **beforeToolCall / afterToolCall**：执行前后拦截点——审批、安全扫描、截断过长输出、日志记录
- **transformContext**：每次调 LLM 前的上下文变换钩子——核心用途是 Compaction

> [!important] 与 [[Hermes Agent]] 的对比
> Hermes 的 `AIAgent.run_conversation()` 循环和编排耦合在同一个万行类里。OpenClaw 把循环抽成独立包的好处是：升级 ReAct 策略不需要动编排逻辑，反之亦然。

### runEmbeddedPiAgent 主循环 — 七类分支顺序

| 优先级 | 分支 | 不能放后面的原因 |
|--------|------|-----------------|
| 1 | `aborted` | 用户中断必须立即响应 |
| 2 | live model switch | 要在产生任何副作用前重启 |
| 3 | `timed out + 高 token` | **预防性**主动压缩 |
| 4 | context overflow | 三级降级（compact → truncate → 抛错） |
| 5 | assistant error | 分类后选 profile 轮换 |
| 6 | success path | 成功路径 |
| 7 | 兜底 | 迭代上限 → 抛错 |

> [!important] 为什么 timeout compaction 必须在 overflow 之前
> `timed out + 65% context` 是**预防性信号**（LLM 还没报错，但延迟暗示 prefill 慢），先走 overflow 会等到下次明确报错，但那时可能直接被 timeout kill。

### Auth Controller 封装凭证决策

> [!info] 一句话理解
> Auth Controller 是"凭证管家"——主循环只说"给我一个能用的 key"，管家内部自己处理"冷却/刷新/探针"这些复杂度。

对外只暴露 4 个方法：`initializeAuthProfile` / `advanceAuthProfile` / `maybeRefreshRuntimeAuthForAuthError` / `stopRuntimeAuthRefreshTimer`。主循环看不到内部复杂度。

### Live Model Switch 的幂等条件

只有**完全干净的 attempt**（没发消息、没执行工具、没产生 assistant 文本）才允许实时切模型。一旦对外产生过影响，切模型重来会导致重复发送或不可撤销操作。

---

## Auth Profile — 带健康状态的对象

> [!info] 一句话理解
> Auth Profile 不是"一串 API Key"——而是"带病历的健康人"：每个账号有自己的类型、健康状态、冷却原因和到期时间。

```
Profile A: "个人Pro"
  ├─ 类型: OAuth（可自动刷新 token）
  ├─ 状态: ⚠️ 冷却中（billing 错误，5min 后重试）
  └─ 冷却原因: 当日额度用完
```

**三种凭据类型**：`ApiKeyCredential`（最简单） / `TokenCredential`（会过期） / `OAuthCredential`（能自动刷新，最持久）

**选取策略**（`auth-profiles/order.ts`）：
1. 类型偏好：`oauth > token > api_key`
2. 均衡轮转：同类型按 `lastUsed` 升序
3. 冷却探针：冷却中的 profile 排末尾，到期后自动试一次
4. 用户锁定：显式 `preferredProfile` 永远优先

---

## FailoverError — 13 种闭合枚举

| 错误类型 | 含义 | 恢复策略 |
|----------|------|----------|
| `billing` | 账号欠费或额度耗尽 | 冷却该 Profile，到期后探针式重试 |
| `rate_limit` | 请求频率超过限制 | 冷却后自动递增重试间隔 |
| `overloaded` | 服务端过载 | 短暂冷却 + backoff |
| `auth` | 认证临时失败 | 自动刷新 token → 重试 |
| `auth_permanent` | 认证永久失败 | 跳过该 Profile，永不重试 |
| `timeout` | LLM 首 token 超时 | 切换 Profile 或尝试 Compaction 后重试 |
| `format` | 输入格式不匹配 | 不可恢复，需修复输入 |
| `model_not_found` | 模型不存在 | 切换到 fallback 模型 |
| `session_expired` | 会话过期 | 创建新 session 重试 |

分类器是**递归**的——逐级走 HTTP status → 符号码 → errno → `cause` 链 → timeout heuristics，容忍不同 Provider SDK 的错误表达差异。

> [!note] 核心意义
> **可恢复错误的处理是静态可证明的，不靠 LLM 猜**。`runEmbeddedPiAgent` 是"FailoverError 工厂"，`runWithModelFallback` 是"消费者"——两者只通过这一个错误类型交流。

---

## 三级 Compaction + Context Engine

> [!info] 一句话理解
> Compaction = "上下文太长时怎么压缩"。三级递进：L1 是"体检后提前减肥"（主动预防），L2 是"体检发现血压高紧急减肥"（超时触发），L3 是"已经进 ICU 了必须抢救"（溢出降级）。

### 三级 Compaction 策略

| 级别 | 触发条件 | 动作 |
|------|----------|------|
| **L1: Pre-request** | 会话历史 > 阈值 | 主动调 `compaction.ts` 生成摘要 |
| **L2: Timeout-triggered** | LLM 首 token 超时 **且** prompt > 65% context | 紧急压缩后重试 |
| **L3: Context overflow** | API 返回 `context_length_exceeded` | 三级降级：auto-compact → 截断 → 抛错 |

### Compaction 实现的四项精细特性

| 特性 | 作用 |
|------|------|
| **identifier-policy** | 压缩时保留 symbol identifiers 的出现频次 |
| **identifier-preservation** | 压缩后验证关键 identifier 没丢 |
| **tool-result-details** | 专门处理 tool 调用输出的摘要格式 |
| **retry** | 压缩本身的重试——压缩失败时降级而非崩溃 |

### Context Engine 可插拔契约

> [!info] 一句话理解
> ContextEngine 不把"上下文"当成静态的消息列表，而是当成一个**有生命力的引擎**——它自己决定每次给 LLM 喂什么内容、什么时候压缩、怎么管理子 Agent 的上下文边界。

```typescript
interface ContextEngine {
  bootstrap?(ctx)          // 会话初始化
  ingest(msg)              // 吸收一条 message
  ingestBatch?(batch)      // 批量处理
  afterTurn?(ctx)          // 一轮结束后后处理
  assemble(budget)         // 按 tokenBudget 组装 prompt
  compact()                // 压缩
  maintain?()              // 分支重写
  prepareSubagentSpawn?()  // 子 agent 派生前剥离不需要的上下文
  onSubagentEnded?()       // 子 agent 结束后吸收结果
}
```

---

## 记忆系统双层架构

> [!info] 一句话理解
> 类比人脑的两层记忆：海马体（短期记忆）每天自动记录，但不会主动翻出来；皮层（长期记忆）只保存真正重要的认知，每轮对话都会"看见"它们。两层之间靠 Dreaming 来晋升。

### 记忆捕获三条路径

| 路径 | 触发时机 | 实现 |
|------|----------|------|
| **Session Memory Hook** | 用户执行 `/new` 或 `/reset` | 读取最近 15 条消息 → LLM 生成摘要 → 写入 `memory/YYYY-MM-DD-slug.md` |
| **Memory Flush** | Compaction 前 | 上下文即将被压缩时自动保存到记忆文件 |
| **Auto Capture** | 对话中实时检测 | 基于正则规则匹配关键信息（偏好、联系方式、决策等） |

### 混合搜索算法

检索时采用 **BM25 + 向量相似度** 的混合搜索，经三层后处理：

```
查询 → BM25（权重 0.3）+ 向量搜索（权重 0.7）
      → 加权融合
      → 时间衰减（半衰期 30 天）
      → MMR 多样性重排
      → Top-K 结果
```

> [!note] 记忆引擎选择
> 默认 `memory-core`（SQLite + FTS5 + 可选 sqlite-vec，零依赖）；可选 `qmd` 或 `memory-lancedb`。

### 召回层（pull） vs 静态层（push）

| 维度 | 每天 `memory/YYYY-MM-DD.md` | 全局 `MEMORY.md` |
|------|------------------------------|------------------|
| 属于哪层 | 召回层（pull） | 静态层（push） |
| 是否注入 LLM context | ❌ 不自动注入 | ✅ 每轮注入 |
| 写入触发 | 多种来源 | **只有 Dreaming Deep 阶段** |
| 类比 | 海马体的短期记忆 | 皮层的长期记忆 |

**双层架构的本质**：召回层是"原始日记"，静态层是"读书笔记"，Dreaming 是两层之间的"晋升管道"。**用户显式启用 Dreaming 前，每天 memory 永远不会自动晋升**。

---

## Dreaming 三阶段算法详解

> [!info] 一句话理解
> Dreaming 是 Agent 的"睡眠整理"机制——Light Sleep 整理候选，REM Sleep 找出反复出现的主题，Deep Sleep 把真正重要的内容固化到长期记忆里。

⚠️ **Dreaming 默认 opt-in 关闭**（`dreaming.enabled: false`）。

| 阶段              | 做什么                   | 是否写 MEMORY.md |
| --------------- | --------------------- | ------------- |
| **Light Sleep** | 读取近期记忆和脱敏会话，去重后整理候选   | ❌ 仅整理候选       |
| **REM Sleep**   | 提取反复主题 + 选候选"潜在真理"    | ❌ 仅提取信号       |
| **Deep Sleep**  | 加权评分 + 阈值门控 + 回源验证后写入 | ✅ 唯一写入路径      |

### Deep 阶段：6 信号加权评分

```
score = 0.24 × frequency + 0.30 × relevance + 0.15 × diversity
      + 0.15 × recency + 0.10 × consolidation + 0.06 × conceptual
      + Light boost (≤ +0.06) + REM boost (≤ +0.09)
```

### Deep 阶段：3 重门禁（必须全部通过才晋升）

```
DEFAULT_PROMOTION_MIN_SCORE          = 0.75
DEFAULT_PROMOTION_MIN_RECALL_COUNT   = 3
DEFAULT_PROMOTION_MIN_UNIQUE_QUERIES = 2
```

**为什么 3 重门禁缺一不可**：单纯总分高 → 可能一次召回特别准；单纯命中多 → 可能同一 query 反复命中；三个都通过 = 真正"重要" ✅

### REM 阶段深度解析

```
confidence = averageScore × 0.45 + recallStrength × 0.25
           + consolidation × 0.20 + conceptual × 0.10
```

**REM vs Deep 的差异**：REM 找"稳固事实"（不看 diversity/recency），Deep 找"值得晋升到每轮可见的稳固事实"（有 diversity/recency），因为 Deep 是"永久固化"。

### Dream Diary & Active Recall

**Dream Diary**（`DREAMS.md`）：每次 Dreaming 运行后，后台 Subagent 生成诗意"梦境日记"。

**Active Memory Recall** 插件：每次对话前运行阻塞式记忆子 Agent（15 秒超时）：
1. 读取当前对话上下文
2. 搜索记忆库找到相关记忆
3. 生成 ≤220 字符摘要，以隐藏 Prompt Prefix 形式注入

支持 6 种 prompt 风格：`balanced` / `strict` / `contextual` / `recall-heavy` / `precision-heavy` / `preference-only`

---

## CLI Backend 双路径执行

> [!info] 一句话理解
> OpenClaw 不把 Claude Code、Codex CLI 当"竞品"，而是把它们当**可替换的执行 backend**——同一个 Gateway 管理、同一个 Agent 人格、同一套记忆系统，只是底层 LLM 调用换了个 Runner。

```
isCliProvider?
├─ true  → runCliAgent（调用 claude-cli, codex-cli 子进程）
└─ false → runEmbeddedPiAgent（通用 pi-agent 引擎）
```

### CliBackendConfig — 配置驱动的适配层

| 字段组 | 代表字段 | 作用 |
|--------|----------|------|
| 基础执行参数 | `command` / `args` / `input` / `maxPromptArgChars` | 怎么启动 CLI |
| 输出解析 | `output` / `resumeOutput` | 怎么读懂 CLI 的回复 |
| 模型与会话映射 | `modelArg` / `modelAliases` / `sessionArg` / `sessionMode` | OpenClaw → CLI 的翻译层 |
| Prompt 与内容注入 | `systemPromptArg` / `systemPromptMode` / `imageArg` | 怎么把人设和内容传给 CLI |
| 安全与可靠性 | `clearEnv` / `serialize` / `reliability.watchdog` | 进程看门狗、环境隔离 |

### 反向 MCP 注入

`bundleMcp: true` 让 OpenClaw 在启动 CLI 时，通过 MCP 配置注入一个本地 MCP 服务器。CLI 通过 MCP 协议反过来调 OpenClaw 提供的工具。

### 双向连接——OpenClaw 也是别人的 Backend

| 路径 | 协议 | 暴露什么 |
|------|------|----------|
| **MCP Server** | MCP（stdio JSON-RPC） | 9 个具体工具 |
| **ACP Server** | ACP（stdio JSON-RPC） | 整个 Agent 能力 |
| **Gateway API** | HTTP REST/RPC | 所有 Gateway 方法 |

---

## 预算贯穿 Runtime

OpenClaw 把 runtime 里的**每一种稀缺资源都显式量化，并配一条超限后的降级路径**：

| 稀缺资源          | 预算形式                                   | 超预算后的降级               |
| ------------- | -------------------------------------- | --------------------- |
| 上下文窗口         | `contextTokenBudget`                   | 三级 Compaction 降级      |
| 单次工具输出        | 30% 软限 + 16K 硬限                        | head + tail 截断        |
| 启动上下文         | `maxChars` + `totalMaxChars`           | 按优先级裁剪                |
| 循环迭代次数        | 按 profile 数动态算                         | 换模型                   |
| Overflow 压缩尝试 | `MAX_OVERFLOW_COMPACTION_ATTEMPTS = 3` | 截断最大 tool result → 报错 |
| 凭证可用性         | Cooldown 分级递增                          | 轮换 profile + 探针式重试    |
| 子 Agent 递归    | `spawnDepth` + `subagentRole`          | 达到深度后强制 leaf          |
| Lane 并发       | 4 车道独立队列                               | 进队列等待                 |

**核心约束**：reserve 不能挤占 minPromptBudget（`min(8K, 50% context)`）。**SAFETY_MARGIN = 1.2**：所有 token 估算乘 1.2，留 20% 安全边距。

---

## 4 车道 Lane 并发管控

> [!info] 一句话理解
> 类比银行窗口：普通窗口（Default）、VIP 窗口（Nested）、快速窗口（Subagent）、后台窗口（Cron）——窗口之间互不影响。

```
CommandLane: Default | Nested | Subagent | Cron
```

- **Default**：用户交互，优先级最高
- **Nested**：嵌套 Agent，和 Default 分开防递归死锁
- **Subagent**：临时子 Agent，有独立并发预算
- **Cron**：定时任务，堆积不会拖垮用户交互
- Lane 还是**权限边界**——Cron 车道看不到交互审批 UI

---

## Bootstrap 截断策略

当文件超过 `bootstrapMaxChars=20_000` 时：保 head 70%（全局约定）+ 保 tail 20%（最新更新）+ 砍 middle（中间累积细节）。

### 子 Agent allowlist

子 Agent 和 Cron session 只注入 5 个文件（AGENTS / TOOLS / SOUL / USER / IDENTITY），剥离状态性数据（HEARTBEAT / BOOTSTRAP / MEMORY）。保留**人格连续性**，剥离**状态性数据**。

---

## Skills 管控

### Skills 加载流程

Skills ==不是每个文档的完整注入==，只是注入列表：

```
1. 收集 Skills 来源（3 级优先级）
   workspace-xxx/skills/  ← 最高（per-agent）
   ~/.openclaw/skills/    ← 中等（shared）
   bundled skills (npm包) ← 最低（内置）
2. 过滤：检查 requires.bins/env/config，跳过不满足条件的
3. 生成 Skills 注入列表（名称、描述、路径）
4. 注入 system prompt → Agent 需要时用 read 工具读取完整 SKILL.md
```

> [!important] 按需加载
> System prompt 只注入 skill 列表，Agent 需要时才读取完整内容。这是节省 Context 的关键设计。

### 精细化 Skills 注入

每个 Agent 的 Skills 配置应为：**基础通用能力 Skills + 专属 Skills**

- `brave_search` → 基础通用 Skill（所有 Agent 都需要联网检索）
- `weather` → 专属 Skill（只让「家庭 Agent」定时天气汇报）

> [!warning] Skills 膨胀风险
> Skills 太多会造成 Context 负担，错误的 Skills 会导致 Agent 错误调用工具。定期使用高阶模型扫描：清理低质量、冲突、不必要的 Skill。

### Skills 延迟加载问题

修改 Skills 架构后，Agent 可能仍使用旧配置。原因：OpenClaw 在 session 启动时创建 Skills 快照，整个 session 期间复用。

> [!tip] 解决方案
> 删除对应 session 后重新咨询 Agent，新加的 Skills 即可生效。

### Clawhub — Skills 市场

```bash
clawhub search "calendar management"  # 语义搜索
clawhub install <skill-slug>           # 安装
clawhub list                           # 列出已安装
clawhub update --all                   # 更新所有（谨慎）
clawhub sync                           # 同步并备份
```

推荐 Skills：`find-skills`、`summarize`、`self-improving-agent`、`brave-search`、`frontend-design`

---

## Shell 命令执行与审批

### 两种执行 Host

| Host | 何时使用 | 怎么执行 |
|------|----------|----------|
| **exec-host-node** | 本地 CLI 模式 | Node.js `spawn`，进程内直接执行 |
| **exec-host-gateway** | 通过 Gateway 运行 | 命令经 RPC 送到 Gateway，审批通过后才执行 |

Gateway host 可跨机器代理执行——Agent 跑在 macOS，shell 命令在远端 Linux 节点执行。

### Exec Approval

| 决策 | 含义 |
|------|------|
| `allow-once` | 仅允许本次执行 |
| `allow-always` | 将命令模式加入白名单 |
| `deny` | 拒绝执行 |

混淆检测规则：`curl-pipe-shell` / `base64-pipe-exec` / `eval-decode` / `python-exec-encoded`

---

## 出站消息管道

Agent 生成回复后要经过标准化、分块、适配三个阶段：

1. **标准化（ReplyPayload）**：统一为 text / mediaUrl / interactive / audioAsVoice
2. **智能分块**：按各平台字符限制自动切分
3. **Outbound 适配**：每个 Channel 实现自己的 `sendText` / `sendMedia`
4. **message:sent Hook**：消息成功发出后触发审计/镜像等后处理

---

## 安全机制五层纵深

| 层 | OpenClaw | 一句话解释 |
|----|----------|------------|
| 网络层 | TLS 强制 + 证书 Pinning + SSRF 防护 | 非本地连接必须加密 |
| 认证层 | Device Identity + Ed25519 签名 + RBAC | "进来后能干什么"按角色权限控制 |
| 命令执行层 | Allowlist + 交互式审批 | shell 命令必须白名单或用户批准 |
| 插件安装层 | 静态代码扫描 + 路径遍历防护 + 文件权限检查 | 安装新插件时扫描恶意行为 |
| 沙箱隔离层 | Docker / SSH | 最危险的操作放进容器或远端执行 |

---

## 两个"反直觉"设计选择

1. **Agent 是单进程串行的**——同一个 Agent 同一时刻只跑一个 turn。原因：workspace / memory / sessions 是状态化共享资源，并发会互相污染。多用户并发通过多 Agent 路由实现。

2. **核心功能通过插件 API 注入**——Memory Search 是插件注册的 Tool + Capability，Compaction 通过 hook 暴露扩展点。Runtime 本质是**编排壳**，调度、容错、预算是它的，具体能力由插件填充。

---

## Cache Trace — 全链路可观测性

`src/agents/cache-trace.ts` 把每次 LLM 调用的上下文变化分 **7 个阶段**落盘到 `~/.openclaw/state/cache-trace/`：

```
session loaded → sanitized → limited → prompt before → images → stream context → session after
```

可以精确定位 prompt cache miss 的原因。这是 production agent 才会关心的运维能力。

---

## 配置系统 — 单文件掌控全局

所有行为由 `~/.openclaw/openclaw.json` 驱动，支持运行时热重载。配置变更时，Gateway 只重启关心该 config prefix 的 Channel——**精细化热重载**。

---

## 部署指南

### 部署方式

| 方式 | 说明 | 适合人群 |
|------|------|----------|
| **云机** | 腾讯云已支持一键部署 | 能接受云机操作习惯的用户 |
| **自部署** | 本机运行，数据完全掌控 | 需要本地模型、数据隐私的用户 |

### 硬件选择

| 场景 | 芯片 | 内存 | 磁盘 | 参考价格 |
|------|------|------|------|----------|
| 只跑 OpenClaw（不跑本地模型） | M1（性价比最高） | 基础即可 | 基础即可 | Mac Mini M1 1TB ≈ 3K |
| 需要跑文生图/文生视频模型 | M4 | 24G | 256G+ 起步 | Mac Mini M4 512G ≈ 7K |

> [!note] Mac vs Windows
> OpenClaw 及诸多工具对 Mac 天然友好。Windows 也可部署但更折腾。

### IM 工具选择三原则

| 原则 | 说明 |
|------|------|
| **安全性** | 千万不要把 OpenClaw 当「文件传输助手」！按最坏情况预估风险 |
| **可用性** | 多 Agent 时 IM 额度飞速消耗（网关每 60 秒 ping IM），需选额度充足或无限制的 IM |
| **易用性** | 国内 IM 软件易用性高于国外推荐软件 |

### 配置复杂度

| 场景 | 时间 |
|------|------|
| 单 Agent 跑起来 + 配 IM 机器人 | ≈ 30 分钟 |
| 多 Agent + Skills 调试 + 身份定制 + 定时任务 | ≈ 2-3 天 |

> [!danger] 不要用 Claude Code 配置 OpenClaw
> CC 目前对 OpenClaw 项目配置的了解有偏差，极易出现「it works, but it also broke」的情况。建议：下载源码 → 让 CC 先学习源码 → 再让 CC 辅助 Debug。

---

## 应用案例

| 案例 | 说明 |
|------|------|
| **Daily Paper** | Agent 每日抓取 HuggingFace 论文，提炼要点（建议搭配 jina.ai） |
| **Summary** | 获取 Subscribe 博主内容，定期分析评分 |
| **DeepResearch** | 深入研判消息，交付质量受限于 LLM 能力 |
| **RAG Tutor** | 在 Memory 目录配置学习资料，让 Agent 成为垂直领域专属 tutor |
| **ComfyUI 本地生成** | 本地部署文生图/文生视频，无需付费 API |
| **TTS 语音合成** | 本机部署 qwen3-tts，定时朗读英语/新闻 |
| **家庭助理** | 配置家庭专属 Agent + Memory 隔离，定时提醒 |

---

## Model 选择建议

> [!tip] Use SOTA model first
> 在预算 OK 的情况下建议使用 SOTA 模型。落后 LM 会影响心智判断——交付不及格时容易得出「AI 能力还不行」的结论，实际大概率是因为 LLM 不够 SOTA。

- 不建议按「年」订阅单一模型，模型迭代速度很快
- LLM 越聪明，OpenClaw 发挥越好、越稳定

---

## 精细化管控

### 版本控制

| 管控点 | 建议 |
|--------|------|
| **memory 切割** | Session、memory/xxx.md、MEMORY.md 不建议上云，在 `.gitignore` 中排除 |
| **密钥动态注入** | `openclaw.json` 同时存密钥和核心 config，密钥部分通过注入管理，避免泄露 |

---

## 横向对比：Agent 框架生态定位

| 维度 | OpenClaw | [[LangGraph]] | [[CrewAI]] | [[Claude Code]] | Letta (MemGPT) |
|------|----------|--------------|------------|-----------------|----------------|
| **核心哲学** | Context 编排弥补模型短板 | 图即程序，状态流转 | 团队角色协作 | 软件工程 Agent | OS-like Memory 管理 |
| **编排模式** | 文件配置 + LLM 动态决策 | DAG 图结构 | Sequential/Hierarchical | 黑盒内置 | Self-managed paging |
| **Memory** | 3 层（Session/按天/MEMORY.md） | State + Checkpoint | 共享 Memory Store | Session 内压缩 | L0-L3 全层级虚拟分页 |
| **Skills/工具** | Markdown 文件 + 按需加载 + Clawhub 市场 | Tool 节点 | Tool 定义 + Delegation | 内置工具链 | 外部工具调用 |
| **多 Agent** | sessions_send + sessions_spawn | 子图嵌套 | Crew + Process | SubAgent 委托 | 单 Agent 自管理 |
| **开源程度** | 完全开源 | 完全开源 | 完全开源 | 闭源 | 完全开源 |
| **适用场景** | 个人/家庭日常 Agent | 复杂工作流编排 | 团队角色模拟协作 | 软件工程 | 长期记忆密集型 Agent |

> [!important] OpenClaw 的三大差异化
> 1. **File-as-Configuration** — 用 Markdown 文件定义 Agent 人格，而非代码/JSON
> 2. **本地优先** — 作为 [[本地Agent（Claw）]] 运行，闭合「决策→执行→验证」回路
> 3. **共识推广驱动** — 技术难度不高，但通过标准化释放群体协作红利

OpenClaw 与 [[LangGraph]] 的核心差异：LangGraph 是「开发者编排图结构来控制 Agent 流程」，OpenClaw 是「Agent 自己在瞬态加载的上下文中动态决策」——前者是 **Architecture-driven（人控）**，后者是 **Context-driven（Agent 自控）**。

---

## 可提炼的架构模式

### 模式一：人格文件化（Persona-as-Files）

用结构化的自然语言文件而非代码配置定义 Agent 的全部属性。核心原则：文件加载顺序即优先级，每个文件独立可改可共享，内容直接注入 system prompt 零转换开销。

### 模式二：瞬态实例（Transient Instance）

Agent 不是常驻进程，而是 per-session 的「加载→执行→销毁」循环。代价是启动开销，收益是：状态从文件恢复、无内存膨胀、故障天然隔离。

### 模式三：按需加载（Lazy Skills Loading）

System prompt 只注入 Skill 列表，Agent 需要时才 Read 完整内容。全量注入 ≈ 10K+ tokens，按需加载 ≈ 4K tokens。

### 模式四：三层 Session 优化（Compaction + Pruning + History Limit）

| 优化层 | 对应记忆层 | 机制 | 关键特性 |
|--------|-----------|------|----------|
| Compaction | 短→中长期压缩 | 旧消息总结为 summary | **持久化** |
| Pruning | 短期精简 | 临时裁剪旧 tool 结果 | **临时性** |
| History Limit | 窗口硬约束 | 限制发送消息条数 | **安全阀** |

### 模式五：双通道多 Agent 通信（Send + Spawn）

| 机制 | 对应协作模式 | 类比 |
|------|-------------|------|
| `sessions_send` | **对等讨论 / 接力传递** | 给同事发消息 |
| `sessions_spawn` | **主从委托（Orchestrator-Worker）** | 雇佣临时工 |

两种通信方式共存而非互斥——LLM 根据指令上下文自行决策。

---

## 前瞻展望

### 在范式演变中的定位

对照 [[Agent技术范式演变（2023-2026）]] 的四阶段模型，OpenClaw 处于 **阶段三→阶段四** 的过渡地带：

- **阶段三特征**：Agent 拥有持久 Memory、定时任务、多 Agent 协作——OpenClaw 已具备
- **阶段四方向**：Agent 自进化、Skills 自迭代、群体共创加速——OpenClaw 正在演进

### 潜在演进路径

| 方向 | 当前状态 | 可能演进 |
|------|----------|----------|
| **Memory** | 文件系统 + LLM 精炼 | 向 [[向量存储]] 检索演进 |
| **Skills** | Clawhub 市场 + 手动管理 | 自动质量评估、Skills 自进化 |
| **多 Agent** | 手动配置通信关系 | 动态拓扑自适应编排 |
| **安全** | 依赖用户自觉 | Context 访问控制、隐私分层隔离 |
| **共识推广** | 社区手动迭代 | 标准化协议（类似 [[MCP]]、[[A2A]]） |

---

## 与 Claude Code 的区别

| 维度 | Claude Code | OpenClaw |
|------|-------------|----------|
| 性质 | 闭源 | 开源 |
| 可修改性 | 黑盒，无法修改 | 可改可优化 |
| 设计 | 针对软件工程 | 针对日常任务 |
| 记忆 | Session 内压缩 | 持久化分层 |
| 消息通道 | 单一 | 多种 |
