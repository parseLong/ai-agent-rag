---
date: 2026-06-15
tags:
  - 方法
  - Agent框架
  - 源码分析
  - OpenClaw
  - 上下文工程
source: "[[OpenClaw与Hermes源码架构深度复盘]]"
---

# Bootstrap 截断策略与子 Agent Allowlist

> [!quote] 核心定义
> OpenClaw 启动时加载 8 个工作区 Markdown 文件不是"全量注入"，而是有两道精细过滤层：**截断策略**（head 70% + tail 20%，砍中间）和**子 Agent allowlist**（只保留 5 个文件，剥离状态性数据）。这两道过滤层体现了"保留人格连续性，剥离状态性数据"的设计哲学。

---

## 第一道：截断策略 — Head 70% + Tail 20%（不是前缀截断）

源码 `bootstrap.ts` — 当文件超过 `bootstrapMaxChars=20_000` 字符时：

```
原始文件                    截断后
┌────────┐                ┌────────┐
│ HEAD   │ 70% 保留 ──→  │ HEAD   │
│        │                │ ...    │
│ MIDDLE │ ✗ 砍中间       │ TAIL   │
│        │                └────────┘
│ TAIL   │ 20% 保留
└────────┘
```

### 为什么砍中间而非砍头尾？

| 保留 | 内容 | 为什么不能丢 |
|------|------|------------|
| **Head 70%** | 使命宣言、核心约定、全局原则 | 文档的"宪法"——定义 Agent 的根本行为 |
| **Tail 20%** | 近期更新、最新约定、例外说明 | 最新的修改往往最相关 |
| **Middle 砍** | 中间累积的事例、历史细节 | 损失最小——是"展开说明"而非"定义" |

> [!warning] 常见错误印象
> - ❌ "截断 = 前缀截断" → 砍掉最近的（错）
> - ❌ "截断 = 后缀截断" → 砍掉最远的（也错）
> - ✅ **OpenClaw = 头尾保留，砍中间** — 兼顾"全局约定"和"最新更新"

---

## 第二道：子 Agent Allowlist（5 文件保留）

源码 `src/agents/workspace.ts`：

```
const MINIMAL_BOOTSTRAP_ALLOWLIST = new Set([
  DEFAULT_AGENTS_FILENAME,    // AGENTS.md       ✅
  DEFAULT_TOOLS_FILENAME,     // TOOLS.md        ✅
  DEFAULT_SOUL_FILENAME,      // SOUL.md         ✅
  DEFAULT_IDENTITY_FILENAME,  // IDENTITY.md     ✅
  DEFAULT_USER_FILENAME,      // USER.md         ✅
]);
```

子 Agent 和 Cron session **只注入这 5 个文件**，其他 workspace 文件被剥离：

| 文件 | 注入主 Agent | 注入子Agent/Cron | 为什么 |
|------|-------------|-----------------|--------|
| `AGENTS.md` | ✅ | ✅ | 项目说明必须 |
| `TOOLS.md` | ✅ | ✅ | 工具指南必须 |
| `SOUL.md` | ✅ | ✅ | 人设必须一致 |
| `USER.md` | ✅ | ✅ | 用户画像必须一致 |
| `IDENTITY.md` | ✅ | ✅ | Agent自我认同统一 |
| `HEARTBEAT.md` | ✅ | ❌ | 心跳子Agent不需要 |
| `BOOTSTRAP.md` | ✅ | ❌ | 仅新workspace才注入 |
| `MEMORY.md` | ✅ | ❌ | 避免污染子任务独立上下文 |

---

## 设计哲学：保留人格连续性，剥离状态性数据

子 Agent 必须和主 Agent"同一个人格"（否则用户感受崩——感觉是另一个陌生助手），但跑独立子任务不携带历史包袱（避免主线对话污染子任务判断）：

| 类别 | 文件 | 为什么 |
|------|------|--------|
| **人格连续性** | SOUL / USER / IDENTITY | 子Agent不能变路人 |
| **协作上下文** | AGENTS / TOOLS | 项目和工具说明必须 |
| **状态性数据** | HEARTBEAT / BOOTSTRAP / MEMORY | 子Agent是独立子任务，不要污染 |

---

## Bootstrap 截断告警注入 LLM（自感知机制）

源码 `bootstrap-budget.ts` 的 `appendBootstrapPromptWarning`：

```
Bootstrap文件被截断
    ↓
计算truncation signature（哪些文件被截、各自被截百分比）
    ↓
检查warning mode（off, once, always）
    ↓
once模式 + 之前看过这个signature → 不再警告
once模式 + 新signature → 注入警告到prompt
    ↓
告警内容：
  "[Bootstrap truncation warning]
   Some workspace bootstrap files were truncated before injection.
   - AGENTS.md: 25000 raw → 20000 injected (~20% removed; max/file).
   - SOUL.md:   18000 raw → 18000 injected (...; max/total)."
    ↓
LLM看到警告 → 知道"我看到的context可能不全，必要时主动read_file"
```

> [!important] 告警是给LLM看的，不只是给开发者看
> LLM知道自己被截断后，会主动用工具补读完整文件，而不是基于不完整信息瞎猜。这是**让Agent自知信息不全**的自感知设计。

---

## Bootstrap Budget 三层结构

不是"启动加载哪些文件"写死在代码里，而是做成可配置、可 hook、可缓存的三层：

1. **可配置**：`maxChars` / `totalMaxChars` / `BootstrapContextMode`(full/lightweight) / `BootstrapContextRunKind`(default/heartbeat/cron)
2. **可 Hook**：`bootstrap-hooks.ts` 允许动态修改文件列表（按时间段注入不同上下文）
3. **可缓存**：`bootstrap-cache.ts` 以 sessionKey 为 key 缓存，session切换自动失效

| RunKind | 加载行为 | 适用场景 |
|---------|---------|----------|
| `default` | 加载所有bootstrap文件 | 正常用户对话 |
| `heartbeat` | 只加载HEARTBEAT.md | 定时心跳（不需要完整人格） |
| `cron` | 默认不加载任何bootstrap文件 | 定时任务（只需要特定上下文） |

---

## 相关笔记

- [[OpenClaw架构深度解析]] — Agent启动流程和workspace文件体系
- [[上下文预算与资源分配]] — Bootstrap是稀缺资源预算的一种（char维度）
- [[上下文压缩]] — 截断是上下文管理的手段之一
- [[Lanes分车道并发管控]] — Cron lane的Bootstrap策略与Default不同
