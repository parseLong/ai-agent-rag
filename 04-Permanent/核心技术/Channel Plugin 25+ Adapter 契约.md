---
date: 2026-06-15
tags:
  - 概念
  - Agent框架
  - 源码分析
  - OpenClaw
  - 插件系统
source: "[[OpenClaw与Hermes源码架构深度复盘]]"
---

# Channel Plugin 25+ Adapter 契约

> [!quote] 核心定义
> OpenClaw 的 `ChannelPlugin` 不是简单的消息适配器，而是同时承担**协议适配、身份配对、安全审批、命令路由、配置生命周期、Gateway协议绑定**等角色的完整 IM 域协作单元。所有 25+ 槽位都是可选的——Telegram/Discord 实现了 30+ 个，简单 webhook 只需 4 个必选 + 5 个可选。

---

## Channel 完整契约接口

```
type ChannelPlugin = {
  // ━━━ 必选 4 项 ━━━
  id, meta, capabilities, config

  // ━━━ Setup 三件套 ━━━
  setupWizard?, setup?, configSchema?

  // ━━━ Auth + Security 7 项 ━━━
  auth?, pairing?, security?, approvalCapability?,
  elevated?, secrets?, allowlist?

  // ━━━ Messaging 7 项 ━━━
  messaging?, message?, outbound?, streaming?,     // ⭐ per-channel 流式协议
  threading?, mentions?, agentPrompt?

  // ━━━ 协作能力 7 项 ━━━
  commands?, groups?, directory?, resolver?,
  bindings?, conversationBindings?, actions?

  // ━━━ Gateway + 运维 6 项 ━━━
  gateway?,                                    // ⭐ Gateway 协议绑定（核心）
  gatewayMethods?, lifecycle?, status?,
  heartbeat?, doctor?, reload?                 // ⭐ 精细化热重载

  // ━━━ 反向工具 ━━━
  agentTools?                                  // ⭐ Channel 给 LLM 提供工具
};
```

---

## 5 种交互模式

| 模式 | 方向 | 说明 |
|------|------|------|
| 入站消息 | Channel → Gateway → Agent | Channel inbound 归一化 → Gateway 路由到 Agent |
| 出站回复 | Agent → Gateway → Channel | Agent 出 turn → Gateway 派单 → Channel outbound |
| 客户端控制 | Client → Gateway → Channel | WS method 调 ChannelGatewayAdapter |
| **反向工具** | Channel → Agent | agentTools 注册到 Agent tool registry（Telegram提供查群成员、Discord提供加reaction） |
| 反向通知 | Channel → Gateway → Client | event:presence / event:tick |

> [!important] Channel 不只是消息通道，还是 LLM 的能力扩展源

---

## Per-channel Streaming Adapter — 核心价值

LLM 流式输出的"语义"在每个 IM 协议里完全不同：
- **Telegram**：用 `editMessageText` 反复编辑同一条消息
- **Discord**：用 `interaction.followUp`
- **iMessage**：不支持流式，退化为分段发送

Channel 把这些差异封装掉。

---

## Channel Docking — 跨 Channel 会话迁移（独门能力）

用户 Alice 在 Telegram 发起会话 → 想切到 Discord继续 → 发 `/dock_discord` → Gateway 验证 `identityLinks`（两个账号属于同一用户） → 保留 session 上下文不变，只换投递地址。

**不重建 session**——相当于"AI会话的呼叫转移"。这是 Hermes, Claude Code 等单 channel 框架做不到的。

---

## 精细化热重载

每个 Channel 声明自己关心哪些 config prefix，Gateway 只在对应配置变更时重启该 Channel，不重启整个进程。

---

## 与 Hermes Channel 对比

| 维度 | Hermes Channel | OpenClaw Channel |
|------|----------------|-------------------|
| 抽象层级 | 函数式 send/recv | **25+ 可选槽位完整契约** |
| Setup 流程 | 改源码/手填配置 | SetupWizard + Schema + UI引导 |
| 认证 | API Key 写文件 | **Auth + Pairing + Security + Approval 5层** |
| Streaming | 单一实现 | **Per-channel Streaming Adapter** |
| Docking | ❌ | ✅ 跨channel会话转发 |
| Doctor | ❌ | ✅ 自诊断 |
| 热重载 | 重启 | ✅ 精细化 reload prefix |
| 反向工具 | ❌ | ✅ Channel给LLM提供工具 |

> [!tip] 设计取舍
> Hermes 把 Channel 当**消息收发管道**——轻量、容易加新平台；OpenClaw 把 Channel 当**需要长期维护的平台集成点**——重、但加上之后不用再操心认证/重载/诊断。

---

## Client vs Channel — 容易混淆

- **Client** = "谁在操作 Agent"（TUI, Control UI, Mobile App, 外部程序），通过 WebSocket 连入 Gateway，走 Ed25519 认证
- **Channel** = "Agent 通过哪条线路收发消息"（QQ Bot, Discord, Telegram），是 Gateway 内部插件模块
- 两者通过 **SessionKey** 交汇：同一个用户可以在不同 Client 上看到同一 Channel 的对话

---

## 相关笔记

- [[OpenClaw架构深度解析]] — OpenClaw 完整架构
- [[MCP]] — 工具暴露协议（Channel 反向工具使用 MCP 模式）
- [[Smart Approval三态审批]] — Hermes 的审批方式对比
- [[OpenClaw vs Hermes 架构对比]] — 两套框架正面对比
