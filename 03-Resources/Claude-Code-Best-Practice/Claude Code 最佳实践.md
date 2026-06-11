---
title: Claude Code 最佳实践 - 总览
description: 来自 claude-code-best-practice 仓库的结构化知识库
date: 2026-05-19
tags: [Claude Code, AI Agent, Best Practice]
---

# Claude Code 最佳实践 - 总览

> 来源：[shanraisshan/claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice)
> 整理日期：2026-05-19

## 知识库结构

| 序号  | 文档              | 内容                                     |
| :-: | --------------- | -------------------------------------- |
| 01  | [[01-核心概念与子代理]] | Subagents 定义、Frontmatter 字段、官方内置 Agent |
| 02  | [[02-命令大全]]     | 80+ 个内置 Slash Commands                 |
| 03  | [[03-技能系统]]     | Skills 定义、Frontmatter 字段、官方捆绑 Skills   |
| 04  | [[04-设置与配置]]    | settings.json 完整配置参考                   |
| 05  | [[05-编排工作流]]    | Command → Agent → Skill 编排模式           |
| 06  | [[06-MCP服务器]]   | MCP 服务器配置与最佳实践                         |
| 07  | [[07-内存与持久化]]   | CLAUDE.md 加载机制、Memory 系统               |
| 08  | [[08-CLI启动参数]]  | CLI 启动参数、环境变量                          |
| 09  | [[09-开发工作流汇总]]  | 主流开发工作流对比                              |

## 核心概念速查

### 四大组件

| 组件 | 文件位置 | 用途 |
|------|---------|------|
| **Subagents** | `.claude/agents/<name>.md` | 复杂多步任务的专用代理 |
| **Commands** | `.claude/commands/<name>.md` | 用户触发的斜杠命令 |
| **Skills** | `.claude/skills/<name>/SKILL.md` | 可复用的能力模块 |
| **Hooks** | `.claude/hooks/` | 生命周期事件钩子 |

### 两种 Skill 模式

1. **Agent Skill (Preloaded)**: 通过 `skills:` 字段预加载到 Agent 上下文中，作为领域知识注入
2. **Skill (Direct Invocation)**: 通过 `Skill` 工具直接调用，独立执行

### 配置层级（从高到低）

1. **Managed** - 组织强制，不可覆盖
2. **命令行参数** - 单会话临时覆盖
3. **`.claude/settings.local.json`** - 个人项目设置（git-ignored）
4. **`.claude/settings.json`** - 团队共享设置
5. **`~/.claude/settings.json`** - 全局个人默认

## 关键资源

- [Claude Code 官方文档](https://code.claude.com/docs)
- [官方 Skills 仓库](https://github.com/anthropics/skills)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)

## 版本信息

- Claude Code 版本：v2.1.143（截至 2026-05-18）
- 模型家族：Claude 4.X（Opus 4.7 / Sonnet 4.6 / Haiku 4.5）
