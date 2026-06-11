---
tags:
  - 参考资料
  - 工具与框架
  - Claude-Code
  - Hooks
  - Rules
aliases:
  - Claude Code Hooks 与 Rules
---

# Claude Code Hooks 与 Rules

> Hooks 和 Rules 的完整指南——事件驱动的自动化钩子、始终遵循的规则体系、记忆持久化生命周期。

## Hooks 概述

Hooks 是事件驱动的自动化，在 Claude Code 工具执行前后触发。它们强制代码质量、提早发现错误、自动化重复检查。

### 执行流程

```
用户请求 → Claude 选择工具 → PreToolUse Hook 运行 → 工具执行 → PostToolUse Hook 运行
```

### Hook 类型详解

| Hook 类型 | 触发时机 | 能做什么 | 退出码 |
|-----------|---------|---------|--------|
| **PreToolUse** | 工具执行前 | 阻止（exit 2）或警告（stderr） | 0=继续，2=阻止 |
| **PostToolUse** | 工具执行后 | 分析输出，不能阻止 | 0=成功 |
| **Stop** | Claude 响应完成后 | 后处理 | 0=成功 |
| **SessionStart** | 新会话开始 | 加载上下文 | 0=成功 |
| **SessionEnd** | 会话结束 | 清理 | 0=成功 |
| **PreCompact** | 上下文压缩前 | 保存状态 | 0=成功 |
| **UserPromptSubmit** | 用户发送消息时 | 实时拦截 | 0/2 |

---

## ECC 核心 Hooks

### PreToolUse Hooks

| Hook | 匹配器 | 行为 |
|------|--------|------|
| **Dev server blocker** | `Bash` | 阻止在 tmux 外运行 `npm run dev`（exit 2） |
| **Tmux 提醒** | `Bash` | 长运行命令前建议 tmux（警告） |
| **Git push 提醒** | `Bash` | `git push` 前提醒审查变更 |
| **Pre-commit 质量** | `Bash` | 提交前 lint、验证 commit message、检测 console.log |
| **Doc file warning** | `Write` | 非标准 .md/.txt 文件警告（允许 README/CLAUDE 等） |
| **Strategic compact** | `Edit|Write` | ~50 次工具调用后建议 `/compact` |

### PostToolUse Hooks

| Hook | 匹配器 | 功能 |
|------|--------|------|
| **PR logger** | `Bash` | `gh pr create` 后记录 PR URL |
| **Build analysis** | `Bash` | 构建后后台分析（异步） |
| **Quality gate** | `Edit|Write|MultiEdit` | 编辑后快速质量检查 |
| **Prettier format** | `Edit` | 编辑 JS/TS 文件后自动格式化 |
| **TypeScript check** | `Edit` | `.ts/.tsx` 编辑后运行 `tsc --noEmit` |
| **console.log warning** | `Edit` | 编辑文件中 console.log 警告 |

### 生命周期 Hooks

| Hook | 事件 | 功能 |
|------|------|------|
| **Session start** | SessionStart | 加载前次上下文、检测包管理器 |
| **Pre-compact** | PreCompact | 压缩前保存状态 |
| **Console.log audit** | Stop | 每次响应后检查所有修改文件的 console.log |
| **Session summary** | Stop | 会话状态持久化 |
| **Pattern extraction** | Stop | 会话模式提取（持续学习） |
| **Cost tracker** | Stop | 轻量成本遥测标记 |
| **Session end marker** | SessionEnd | 生命周期标记和清理 |

---

## 记忆持久化生命周期

记忆持久化是 ECC Hooks 的核心子系统，定义了跨会话记忆保存和恢复的完整生命周期。

| 钩子 | 事件 | 行为 |
|------|------|------|
| **SessionStart** | 会话开始 | 加载前次会话的上下文文件 |
| **PreCompact** | 上下文压缩前 | 保存当前会话状态到文件 |
| **PreToolUse** | 工具执行前 | 观察工具使用模式 |
| **PostToolUse** | 工具执行后 | 活动追踪 |
| **Stop** | 会话结束 | 持久化学习、会话总结 |

**设计原则：** 使用 Stop Hook 而非 UserPromptSubmit——后者每次消息都运行（增加延迟），前者仅在会话结束时运行一次（轻量、不拖慢会话）。

---

## Hook 运行时控制

通过环境变量控制 Hook 行为，无需编辑 `hooks.json`：

```bash
# Hook 严格级别：minimal | standard | strict（默认：standard）
export ECC_HOOK_PROFILE=standard

# 禁用特定 Hook（逗号分隔）
export ECC_DISABLED_HOOKS="pre:bash:tmux-reminder,post:edit:typecheck"

# 限制 SessionStart 加载的上下文量（默认：8000 字符）
export ECC_SESSION_START_MAX_CHARS=4000

# 完全禁用 SessionStart 附加上下文
export ECC_SESSION_START_CONTEXT=off
```

---

## 编写自定义 Hook

Hooks 是 Shell 命令，在 stdin 接收工具输入 JSON，在 stdout 输出 JSON。

### Hook 输入 Schema

```typescript
interface HookInput {
  tool_name: string;          // "Bash", "Edit", "Write", "Read" 等
  tool_input: {
    command?: string;         // Bash: 运行的命令
    file_path?: string;       // Edit/Write/Read: 目标文件
    old_string?: string;      // Edit: 替换的文本
    new_string?: string;      // Edit: 替换文本
    content?: string;         // Write: 文件内容
  };
  tool_output?: {             // PostToolUse only
    output?: string;
  };
}
```

### 退出码

| 退出码 | 含义 |
|--------|------|
| `0` | 成功（继续执行） |
| `2` | 阻止工具调用（仅 PreToolUse） |
| 其他非零 | 错误（记录但不阻止） |

### 异步 Hooks

```json
{
  "type": "command",
  "command": "node my-slow-hook.js",
  "async": true,
  "timeout": 30
}
```

异步 Hooks 在后台运行，不能阻止工具执行。

---

## 常用 Hook 配方

### 阻止大文件创建

```json
{
  "matcher": "Write",
  "hooks": [{
    "type": "command",
    "command": "node -e \"let d='';...if(lines>800){process.exit(2)}...\""
  }],
  "description": "阻止创建超过 800 行的文件"
}
```

### Python 文件用 ruff 自动格式化

```json
{
  "matcher": "Edit",
  "hooks": [{
    "type": "command",
    "command": "node -e \"...execFileSync('ruff',['format',p])...\""
  }],
  "description": "编辑后用 ruff 自动格式化 Python 文件"
}
```

---

## Rules 规则体系

Rules 是 `.md` 文件，包含 Claude 应始终遵循的指南。两种方式：

1. **单一 CLAUDE.md** — 所有内容在一个文件
2. **Rules 目录** — 模块化 `.md` 文件按关注点分组

### Rule 分类

| 分类 | 示例内容 |
|------|---------|
| 安全 | 无硬编码密钥、验证输入、OWASP 检查 |
| 编码风格 | 不可变性、文件组织、命名约定 |
| 测试 | TDD 工作流、80% 覆盖率要求 |
| Git 工作流 | Conventional commits、PR 流程 |
| 智能体 | 何时委派给 Subagents |
| 性能 | 模型选择策略、上下文管理 |

### Rule 格式规范

Agent 规则：Markdown + YAML frontmatter（name、description、tools、model）
Skill 规则：清晰段落（When to Activate、How It Works、示例）
Hook 规则：JSON（matcher + hooks 数组）

---

## Command-Agent 映射

Slash Command 与 Agent 的对应关系：

| Command | 主要 Agent/Skill |
|---------|-----------------|
| `/plan` | planner |
| `/architect` | architect |
| `/tdd` | tdd-guide |
| `/code-review` | code-reviewer |
| `/security-scan` | security-reviewer |
| `/build-fix` | build-error-resolver |
| `/refactor-clean` | refactor-cleaner |
| `/e2e` | e2e-runner |
| `/learn` | continuous-learning |
| `/skill-create` | skill creator |

---

## 关联知识

- [[智能体循环]] — Agent Loop 机制
- [[上下文窗口管理]] — 上下文管理策略
- [[记忆系统]] — 短期与长期记忆
- [[上下文压缩]] — 上下文压缩方法
- [[上下文注入]] — 上下文注入策略
- [[ECC使用指南]] — Skills 和 Hooks 的配置方法