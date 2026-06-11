---
title: Claude Code CLI 启动参数
description: CLI 启动标志、子命令、环境变量完整参考
date: 2026-05-19
tags: [Claude Code, CLI, Configuration]
---

# Claude Code CLI 启动参数

## 会话管理

| 标志 | 简写 | 说明 |
|------|------|------|
| `--continue` | `-c` | 继续当前目录最近的会话 |
| `--resume` | `-r` | 按 ID 或名称恢复会话 |
| `--from-pr <NUMBER\|URL>` | | 从特定 PR 恢复会话 |
| `--fork-session` | | 恢复时创建新会话 ID |
| `--session-id <UUID>` | | 使用特定会话 ID |
| `--no-session-persistence` | | 禁用会话持久化 |
| `--remote` | | 在 claude.ai 创建新的网页会话 |
| `--teleport` | | 在本地终端恢复网页会话 |

## 模型与配置

| 标志 | 简写 | 说明 |
|------|------|------|
| `--model <NAME>` | | 设置模型（别名或完整 ID） |
| `--fallback-model <NAME>` | | 默认模型超载时的回退模型 |
| `--betas <LIST>` | | Beta 请求头 |

## 权限与安全

| 标志 | 简写 | 说明 |
|------|------|------|
| `--dangerously-skip-permissions` | | 跳过所有权限提示（谨慎使用） |
| `--allow-dangerously-skip-permissions` | | 启用权限绕过选项 |
| `--permission-mode <MODE>` | | 初始权限模式 |
| `--allowedTools <TOOLS>` | | 无需提示即可执行的工具 |
| `--disallowedTools <TOOLS>` | | 从上下文移除的工具 |
| `--tools <TOOLS>` | | 限制可用工具 |

## 输出与格式

| 标志 | 简写 | 说明 |
|------|------|------|
| `--print` | `-p` | 无交互模式输出 |
| `--output-format <FORMAT>` | | 输出格式：`text`、`json`、`stream-json` |
| `--input-format <FORMAT>` | | 输入格式 |
| `--json-schema <SCHEMA>` | | 获取验证 JSON |
| `--verbose` | | 启用详细日志 |

## 代理与子代理

| 标志 | 简写 | 说明 |
|------|------|------|
| `--agent <NAME>` | | 指定当前会话的代理 |
| `--agents <JSON>` | | 通过 JSON 动态定义子代理 |
| `--teammate-mode <MODE>` | | 设置代理团队显示方式 |

## MCP 与插件

| 标志 | 简写 | 说明 |
|------|------|------|
| `--mcp-config <PATH\|JSON>` | | 从文件或字符串加载 MCP 服务器 |
| `--strict-mcp-config` | | 仅使用 `--mcp-config` 的服务器 |
| `--plugin-dir <PATH>` | | 从目录加载插件 |

## 目录与工作区

| 标志 | 简写 | 说明 |
|------|------|------|
| `--add-dir <PATH>` | | 添加额外工作目录 |
| `--worktree` | `-w` | 在隔离的 git 工作树中启动 |

## 预算与限制

| 标志 | 简写 | 说明 |
|------|------|------|
| `--max-budget-usd <AMOUNT>` | | API 调用最大金额 |
| `--max-turns <NUMBER>` | | 限制代理轮数 |

## 系统提示

| 标志 | 简写 | 说明 |
|------|------|------|
| `--system-prompt <TEXT>` | | 替换整个系统提示 |
| `--system-prompt-file <PATH>` | | 从文件加载系统提示 |
| `--append-system-prompt <TEXT>` | | 追加自定义文本到系统提示 |

## 调试与诊断

| 标志 | 简写 | 说明 |
|------|------|------|
| `--debug <CATEGORIES>` | | 启用调试模式 |
| `--doctor` | | 运行诊断 |

## 版本与帮助

| 标志 | 简写 | 说明 |
|------|------|------|
| `--version` | `-v` | 输出版本号 |
| `--help` | `-h` | 显示帮助 |

## 子命令

| 子命令 | 说明 |
|--------|------|
| `claude` | 启动交互式 REPL |
| `claude "query"` | 带初始提示启动 REPL |
| `claude agents` | 列出配置的代理 |
| `claude auth` | 管理认证 |
| `claude doctor` | 运行诊断 |
| `claude install` | 安装/切换原生构建 |
| `claude mcp` | 配置 MCP 服务器 |
| `claude plugin` | 管理插件 |
| `claude remote-control` | 管理远程控制会话 |
| `claude update` / `claude upgrade` | 更新到最新版本 |

## 启动环境变量

| 变量 | 说明 |
|------|------|
| `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` | 启用实验性代理团队 |
| `CLAUDE_CODE_TMPDIR` | 覆盖临时目录 |
| `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1` | 启用额外目录 CLAUDE.md 加载 |
| `DISABLE_AUTOUPDATER=1` | 禁用自动更新 |
| `CLAUDE_CODE_EFFORT_LEVEL` | 控制思考深度 |
| `USE_BUILTIN_RIPGREP=0` | 使用系统 ripgrep |
| `CLAUDE_CODE_SIMPLE` | 启用简化模式 |
| `CLAUDE_BASH_NO_LOGIN=1` | 跳过 Bash 登录 shell |
| `CCR_FORCE_BUNDLE=1` | 强制打包本地仓库 |

## 参考来源

- [Claude Code CLI Reference](https://code.claude.com/docs/en/cli-reference)
- [Claude Code Headless Mode](https://code.claude.com/docs/en/headless)
