---
date: 2026-06-15
tags:
  - 方法
  - Agent框架
  - 源码分析
  - OpenClaw
  - 协议适配
source: "[[OpenClaw与Hermes源码架构深度复盘]]"
---

# CLI Backend 双路径执行

> [!quote] 核心定义
> OpenClaw 不把 Claude Code、Codex CLI 当竞品，而是把它们当**可替换的执行 backend**。同一个 Gateway 管理、同一个 Agent 人格、同一套记忆系统、同一个会话转录格式，只是底层 LLM 调用换了个 Runner。这是微内核架构的真正红利。

---

## 两种执行路径

```
isCliProvider?
├─ true  → runCliAgent
│          • 调用 claude-cli / codex-cli 子进程
│          • 通过 cli-session.ts 管理生命周期
│          • 共享同一套 workspace, memory, session
└─ false → runEmbeddedPiAgent
           • 通用 pi-agent 引擎（基于 @mariozechner/pi-agent-core）
           • 直接调用 Provider SDK
```

---

## 各家 CLI 输出协议互不相同

| Backend | 输出协议 | 说明 |
|---------|----------|------|
| **claude-cli** | stream-json（一行一个JSON对象） | Claude Code 自定义的私有格式 |
| **codex-cli** | JSONL事件流 | Codex 的私有格式 |
| **gemini-cli** | 单一JSON对象 | Gemini 私有格式 |

> **这三种格式互不相同**——不是ACP、不是MCP、不是OpenAI Chat Completions。各家CLI为自己的IDE集成设计的私有协议，早于ACP标准出现。

---

## CliBackendConfig — 配置驱动的适配层

由于没有标准协议，OpenClaw 用一个**配置对象**抽象差异：

```
type CliBackendConfig = {
  command, args, output, input,
  maxPromptArgChars,     // arg超长转stdin
  modelArg, modelAliases,  // OpenClaw model id → CLI model id
  sessionArg, sessionMode,
  systemPromptArg, systemPromptMode,
  imageArg, imageMode,
  bundleMcp,             // ⭐ 关键：反向MCP注入
  reliability: { watchdog: {...} }
};
```

---

## 反向 MCP 注入 — 最巧妙的一环

```
OpenClaw runtime
          ↓ spawn
┌──────────────────────┐
│  Claude CLI 子进程    │
│  → Anthropic API      │ ← LLM调用是CLI自己做的（用CLI已登录的OAuth）
│  → MCP Client         │ ← 调OpenClaw工具
└─────────┬─────────────┘
          │ stdio MCP
          ▼
┌────────────────────┐
│ OpenClaw MCP Server │ ← 提供 send_message / subagents 等扩展工具
└────────────────────┘
```

**CLI ↔ OpenClaw 之间实际是混合协议**：LLM输出走各家CLI的私有stdout协议，工具调用走MCP。

---

## 三种 Provider 类型对比

| 类型 | 协议 | 适用场景 |
|------|------|----------|
| **embedded** | 各厂商 HTTP API | 走自己的API Key |
| **CLI provider** | 私有stdout流 + 反向MCP | 复用CLI登录态 |
| **ACP provider** | ACP JSON-RPC | 把别的ACP agent当LLM backend |

---

## 为什么私有协议能当backend用？

1. **私有协议是"事实开放"的**——CLI厂商为了支持IDE集成必须让协议对外可解析且稳定
2. **协议适配做成可配置层**——`CliBackendConfig` 是外部可注册的，任何人都能加新CLI backend
3. **OpenClaw不需要懂LLM协议本身**——CLI backend把"LLM协议适配"委托给CLI厂商，OpenClaw只解决"如何驱动CLI"这个更窄的问题

> **复杂度从"N套HTTP协议"降到"N套stdout格式"**——后者天然更简单。

---

## 双向连接——OpenClaw 同时是主和辅

| 路径 | 暴露面 | 说明 |
|------|--------|------|
| MCP Server | 工具粒度 | Claude Code 等MCP客户端可调OpenClaw的IM通讯工具 |
| ACP Server | Agent粒度 | Zed等IDE可把OpenClaw当完整Agent |
| Gateway API | 系统粒度 | 任何HTTP客户端可调所有Gateway方法 |

> **OpenClaw 不试图取代任何现有工具，而是和它们互联**——在主/辅/中间层任意位置运转。

---

## 典型配置场景

- Agent A 用 `claude-cli` backend → 复用Claude Code的内置工具链，适合重度编程
- Agent B 用 `embedded` backend + DeepSeek → 自有API Key直连，token成本低
- Agent C 用 `codex-cli` backend → 走ChatGPT Plus订阅额度

---

## 相关笔记

- [[OpenClaw架构深度解析]] — Gateway微内核的完整设计
- [[MCP]] — 工具暴露协议（CLI Backend的反向MCP使用此协议）
- [[Auth Profile与FailoverError]] — embedded路径的凭证和错误处理
- [[OpenClaw vs Hermes 架构对比]] — Hermes仅支持Copilot ACP作为chat backend
