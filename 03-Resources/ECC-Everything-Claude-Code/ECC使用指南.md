---
tags:
  - 参考资料
  - 工具与框架
  - Claude-Code
aliases:
  - ECC 使用指南
  - Everything Claude Code 使用指南
---

# ECC 使用指南

> Everything Claude Code (ECC) 的完整使用指南——涵盖 Skills、Hooks、Subagents、MCPs、Plugins 的实战配置与高级模式。

## Skills 与 Commands

Skills 是 Claude Code 的主要工作流界面，它们是带范围的、可复用的知识模块：包含 Prompt、结构、支撑文件和 codemap（代码地图），用于特定执行模式。

| 层级 | 路径 | 说明 |
|------|------|------|
| **Skills** | `~/.claude/skills/` | 标准工作流定义 |
| **Commands** | `~/.claude/commands/` | 遗留 slash 入口兼容层 |

```bash
# Skill 示例结构
~/.claude/skills/
  pmx-guidelines.md      # 项目特定模式
  coding-standards.md    # 语言最佳实践
  tdd-workflow/          # 多文件 Skill（含 SKILL.md）
  security-review/       # 基于检查清单的 Skill
```

**核心要点：** Skills 包含 codemap——让 Claude 快速导航代码库而不消耗上下文探索。Commands 正在向 Skills 迁移，长期方向是 skills-first。

---

## Hooks

Hooks 是基于触发器的自动化，在特定事件发生时执行。与 Skills 不同，Hooks 仅限于工具调用和生命周期事件。

### Hook 类型

| Hook 类型 | 触发时机 | 能做什么 |
|-----------|---------|---------|
| **PreToolUse** | 工具执行前 | 阻止（exit code 2）或警告（stderr） |
| **PostToolUse** | 工具执行后 | 分析输出，不能阻止 |
| **Stop** | Claude 响应完成后 | 每次响应后执行 |
| **SessionStart/End** | 会话生命周期边界 | 开始/结束时执行 |
| **PreCompact** | 上下文压缩前 | 保存状态 |
| **UserPromptSubmit** | 用户发送消息时 | 实时拦截 |

**示例：tmux 提醒（长时间运行命令前）**

```json
{
  "PreToolUse": [
    {
      "matcher": "tool == \"Bash\" && tool_input.command matches \"(npm|pnpm|yarn|cargo|pytest)\"",
      "hooks": [{
        "type": "command",
        "command": "if [ -z \"$TMUX\" ]; then echo '[Hook] Consider tmux for session persistence' >&2; fi"
      }]
    }
  ]
}
```

**实用建议：** 用 `/hookify` 插件对话式创建 Hooks，不用手写 JSON。

---

## Subagents（子智能体）

Subagents 是主 Claude 可以委派任务的进程，具有有限的作用域。它们可以后台或前台运行，为主智能体释放上下文空间。

```bash
# Subagent 示例结构
~/.claude/agents/
  planner.md           # 功能实现规划
  architect.md         # 系统设计决策
  tdd-guide.md         # 测试驱动开发
  code-reviewer.md     # 质量/安全审查
  security-reviewer.md # 漏洞分析
```

**关键原则：** 为每个 Subagent 配置允许的工具、MCPs 和权限，实现正确的范围限定。限制工具 = 专注执行。

---

## Rules 与 Memory

`.rules` 目录存放 `.md` 文件，包含 Claude 应始终遵循的最佳实践。

```bash
~/.claude/rules/
  security.md      # 无硬编码密钥，验证输入
  coding-style.md  # 不可变性，文件组织
  testing.md       # TDD 工作流，80% 覆盖率
  git-workflow.md  # 提交格式，PR 流程
  agents.md        # 何时委派给 Subagent
  performance.md   # 模型选择，上下文管理
```

---

## MCPs（模型上下文协议）

MCPs 连接 Claude 到外部服务。不是替代 API——它是 Prompt 驱动的包装，允许更灵活地导航信息。

**关键警告：上下文窗口管理**

你的 200k 上下文窗口在压缩前，如果加载太多工具可能只剩 70k。性能会显著下降。

**经验法则：** 配置 20-30 个 MCPs，但保持 **10 个以下启用 / 80 个以下工具活跃**。

```bash
# 查看启用的 MCPs
/mcp
# 在 ~/.claude/settings.json 或 .mcp.json 中禁用不需要的
```

---

## Plugins（插件）

Plugins 将工具打包以便安装，无需繁琐的手动配置。一个插件可以是 Skill + MCP 的组合，或 Hooks + Tools 的捆绑。

**LSP 插件** 特别有用——提供实时类型检查、go-to-definition 和智能补全，无需 IDE。

---

## 上下文与记忆管理

### 跨会话记忆

最有效的方法：Skill 或 Command 在会话结束时总结进度并保存到 `.tmp` 文件。下一个会话可以使用该文件作为上下文继续工作。这些文件应包含：
- 哪些方法有效（附验证证据）
- 哪些方法尝试过但失败
- 哪些方法尚未尝试及剩余工作

### 动态系统 Prompt 注入

不要只依赖 CLAUDE.md 或 `.claude/rules/`——用 CLI flags 动态注入上下文：

```bash
# 每日开发
alias claude-dev='claude --system-prompt "$(cat ~/.claude/contexts/dev.md)"'

# PR 审查模式
alias claude-review='claude --system-prompt "$(cat ~/.claude/contexts/review.md)"'

# 研究探索模式
alias claude-research='claude --system-prompt "$(cat ~/.claude/contexts/research.md)"'
```

### 记忆持久化 Hooks

- **PreCompact Hook**：上下文压缩前保存重要状态
- **Stop Hook（会话结束）**：会话结束时持久化学习到文件
- **SessionStart Hook**：新会话时自动加载之前的上下文

---

## Token 优化

### 模型选择策略

| 任务类型 | 推荐模型 | 原因 |
|----------|---------|------|
| 探索/搜索 | Haiku | 快速、便宜、足以找文件 |
| 简单编辑 | Haiku | 单文件修改、指令清晰 |
| 多文件实现 | Sonnet | 编码的最佳平衡 |
| 复杂架构 | Opus | 需要深度推理 |
| PR 审查 | Sonnet | 理解上下文、捕捉细微问题 |
| 安全分析 | Opus | 不能漏掉漏洞 |
| 写文档 | Haiku | 结构简单 |
| 调试复杂 Bug | Opus | 需要把握整个系统 |

**默认：** 90% 的编码任务用 Sonnet。升级到 Opus 的条件：首次尝试失败、任务跨 5+ 文件、架构决策、安全关键代码。

### MCP 替代策略

很多 MCP（如 GitHub、Supabase、Vercel）只是 CLI 的包装。可以用 Skills + Commands 替代，节省上下文。例如：用 `/gh-pr` Command 包装 `gh pr create`，替代 GitHub MCP。

---

## 并行化策略

### Git Worktrees

```bash
# 为并行工作创建 worktrees
git worktree add ../project-feature-a feature-a
git worktree add ../project-feature-b feature-b

# 每个 worktree 运行独立的 Claude 实例
cd ../project-feature-a && claude
```

### 双实例启动模式

- **实例 1：脚手架智能体** — 创建项目结构、配置文件
- **实例 2：深度研究智能体** — 连接所有服务、创建 PRD、架构图

---

## 关键要点

1. **不要过度复杂化** — 配置是微调，不是架构
2. **上下文窗口是珍贵的** — 禁用未使用的 MCPs 和 Plugins
3. **并行执行** — Fork 对话、使用 Git Worktrees
4. **自动化重复任务** — Hooks 用于格式化、Lint、提醒
5. **限定 Subagent 范围** — 有限工具 = 专注执行
6. **投资可复用模式** — Skills、Commands、Planning 模式有复利效应

## 关联知识

- [[MCP]] — MCP 协议概念
- [[智能体通信协议]] — MCP/A2A/Function Calling 对比
- [[上下文窗口管理]] — 长对话处理策略
- [[上下文压缩]] — 上下文压缩方法
- [[记忆系统]] — 短期与长期记忆
- [[Claude Code 知识地图]] — Claude Code 架构深度解析

## 参考资料

- [ECC GitHub](https://github.com/affoon-m/everything-claude-code)
- [Plugins Reference](https://code.claude.com/docs/en/plugins-reference)
- [Hooks Documentation](https://code.claude.com/docs/en/hooks)
- [Memory System](https://code.claude.com/docs/en/memory)