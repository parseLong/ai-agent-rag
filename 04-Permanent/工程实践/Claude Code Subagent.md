---
title: Claude Code Subagent
description: Claude Code内建的multi-agent机制——写一个.markdown文件就能创建独立context的子agent，无需framework代码
tags: [概念卡片, 永久笔记, 概念, 智能体, Claude Code, Subagent, 多智能体]
aliases:
  - Subagent
  - Claude Subagent
  - 子Agent
date: 2026-06-11
source: "[[Stage 5 — Claude Code 生态]]"
---

# Claude Code Subagent

> Subagent = 主 Claude session spawn 出有**独立[[上下文窗口]]**的子 agent、跑特定任务、回报结果。**写一个 `.claude/agents/<name>.md` 就能创建**，无需 Python framework 代码。

## 🧠 核心洞察

Multi-agent 不只有 framework 这条路。Claude Code 提供另一种 abstraction：**subagent — 写 markdown 不写 code，天生[[上下文工程]]隔离**。

| 路线 | 启动方式 | Runtime | Context 隔离 |
|---|---|---|---|
| **Framework**（[[LangGraph]]/[[CrewAI]]） | `pip install` + Python code | 自己的 Python process | framework 自己管 |
| **Claude Subagent**（本概念） | `.claude/agents/<name>.md` | Claude Code 内建 Task tool | 天生各 subagent 独立 window |

## 🔑 3 种 Multi-Agent 机制

| 机制 | 何时用 | 派遣方式 |
|---|---|---|
| **Subagent**（稳定版） | delegate 大 context 任务给 isolated worker | 写 `.claude/agents/<name>.md`（frontmatter `name` + `description` + `tools` + 可选 `model`）；Claude 看 description **自动 delegate** |
| **Agent team**（需 opt-in flag） | worker 需要**互相沟通**、debate / peer review | `settings.json` 加 `"CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"`；自然语言派遣 |
| **Background agent**（research preview） | 多个**独立任务**各自背景跑、单一界面监控 | `claude --bg "investigate..."` 或从 session `/bg`；`claude agents` 监控 |

**怎么选**：
- 任务独立、不互动 → **Subagent**（最简单、token 最省）
- Worker 需要互相 debate → **Agent team**（token 3-5x）
- 多个独立任务各自跑 → **Background agent**（适合长时间任务并行）

## 🔑 Subagent vs Skill — 5 个关键差别

| 维度 | Subagent | [[Skills技能]] |
|---|---|---|
| **执行环境** | 新的独立[[上下文窗口]] | 主 session 内、同 context |
| **工具权限** | 自己的 `tools:` 清单 | 主 session 的工具（默认全开） |
| **返回结果** | 一个 final message 摘要回主 session | 没有返回、是行为改变 |
| **适合做** | 长任务 / 并行跑 / 要 context 隔离 | 知识注入 / 规则 / 改 Claude 行为 |
| **范例** | `code-reviewer` / `Explore` / `Plan` | `pdf` / `skill-vetter` |

**判断快速办法**：你**要新[[上下文窗口]]**吗？要 → subagent；不要 → skill。

## 🔑 Description = 路由 Key

主 session 怎么知道该派哪个 subagent？看 `.claude/agents/<name>.md` 的 **`description` 字段**：

| Description 写法 | 触发模式 | 例 |
|---|---|---|
| `...use **PROACTIVELY** when X...` | **主动触发**——X 出现 Claude 自己派 | "use PROACTIVELY when reviewing diffs ≥ 50 lines" |
| `...use when user asks Y...` | **被动触发**——要用户明确要求 | "use when user asks for code review" |
| 空 description | **隐形**——不会被自主选 | 只能代码强制调用 |

> **写 description 像写广告词**——把"我能解决什么问题"写具体。`PROACTIVELY` 是强信号词。

## 🔑 5 条老手才知道的 Gotcha

| # | Gotcha | 为什么重要 |
|---|---|---|
| 1 | **Description 写精准即可** | 过长 description 占[[上下文预算与资源分配]] |
| 2 | **`tools:` 写空 = 继承主 session 全部工具** | 想限制就**明写**工具清单 |
| 3 | **不写 `model:` = 跟主 session 用同 model** | 省 cost 就写 `model: haiku` |
| 4 | **Subagent 没"我之前说过 X"记忆** | 每次派遣都是**全新[[上下文窗口]]**、prompt 必须 self-contained |
| 5 | **Subagent 也吃 hook** | PreToolUse / PostToolUse 在 subagent 内也会 fire |

## 🔑 Subagent 整体优缺点

**5 个优点**：

| 优点 | 怎么帮到你 |
|---|---|
| **[[上下文重置与交接]]隔离** | 主 session window 不被污染 |
| **Tool allowlist** | 限制只能 Read / Grep = 安全 sandbox |
| **Model override** | 简单任务用 Haiku、省成本 |
| **Parallel spawn** | N 个 subagent 平行跑、wall clock ÷ N |
| **专业化 prompt** | 不会被闲聊干扰 |

**5 个缺点**：

| 缺点 | 影响 |
|---|---|
| **Spawn 有 overhead** | 任务 < 5 分钟自己跑更快 |
| **无 cross-call memory** | 每次 spawn 都新窗口 |
| **只回一个 message** | 不适合需要逐步 feedback 的任务 |
| **Token cost N ×** | spawn 4 个 = 4 倍 token |
| **Debug 多一层** | 出错不知道该怪谁 |

> **1 句话判断**：任务 ≥ 5 分钟 + prompt 可以写死（不需要来回对话）+ 结果一次回来够用 → 用 subagent；否则自己跑。

## 📄 Subagent 文件范例

`.claude/agents/code-reviewer.md`：

```markdown
---
name: code-reviewer
description: Review staged git changes for security issues, style violations, and missing tests. Use PROACTIVELY when user asks "review my changes" or runs /review.
tools:
  - Read
  - Grep
  - Bash
model: claude-haiku-4-5
---

You are a senior code reviewer. When invoked:
1. Run `git diff --cached` to get staged changes
2. Check for: hard-coded secrets, SQL injection, missing error handling, missing tests
3. Output: PASS / list of specific issues with file:line references
```

## 🔑 各家 CLI 的 multi-agent 环境现状（2025 后段）

| 平台 | Subagent | Agent team | Background agent |
|---|:---:|:---:|:---:|
| **Claude Code** | ✅ | ✅ | ✅ |
| **OpenAI Codex CLI** | ❌ | ❌ | ❌ |
| **Google Gemini CLI** | ❌ | ❌ | ❌ |
| **Cursor** | ❌ | ❌ | ❌ |

> 目前只有 Claude Code 有完整 native multi-agent stack。

## 🔗 关联知识

- [[Harness Engineering 工程]] — Subagent 属于 Coordination 层（Harness 元件 #5）
- [[Skills技能]] — Subagent vs Skill 的5个关键差别
- [[MCP]] — MCP 是"能力"层，Subagent 是"独立 worker"层
- [[多智能体协作]] — Subagent 是 Claude Code 内建的多智能体机制
- [[上下文窗口]] — Subagent 有独立窗口
- [[上下文重置与交接]] — Subagent 天生 context 隔离
- [[上下文预算与资源分配]] — Description 和 model 选择影响 token 用量
- [[工具调用（Function Calling）]] — Subagent 的 tools 清单基于此机制
- [[LangGraph]] — Framework 路线的多智能体对照
- [[CrewAI]] — Framework 路线的快速雏形对照

## 🏷️ 标签

#概念卡片 #永久笔记 #概念 #智能体 #Claude Code #Subagent #多智能体