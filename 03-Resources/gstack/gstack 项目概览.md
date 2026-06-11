---
title: gstack 项目概览
description: Garry Tan 的 AI 辅助开发工具栈 — 23 个专家角色 + 8 个强力工具，将 Claude Code 变成虚拟工程团队
date: 2026-05-19
tags: [gstack, 项目概览, 智能体, 工具链, 参考笔记]
---

# gstack 项目概览

## 📖 定义

gstack 是 Garry Tan（YC 总裁）构建的开源 AI 辅助开发工具栈（MIT 许可证）。它将 Claude Code 变成一个**虚拟工程团队**——CEO 重思产品、架构师锁定设计、设计师捕捉 AI slop、 reviewer 发现生产 bug、QA 打开真实浏览器、安全官运行 OWASP + STRIDE 审计、发布工程师 ship PR。23 个专家角色 + 8 个强力工具，全部是 slash 命令，全部是 Markdown，全部免费。

## 🏗️ 核心架构

gstack 的核心是给 Claude Code 一个**持久浏览器**和一套**有主见的流程技能**。浏览器是难点——其他都是 Markdown。

```
Claude Code → CLI（编译二进制）→ HTTP POST → Server（Bun.serve）→ CDP → Chromium
```

- **持久守护进程**：首次调用启动 ~3s，后续每次 ~100-200ms
- **子秒级延迟**：HTTP POST + Playwright，不需要每次冷启动浏览器
- **持久状态**：登录一次保持登录，tab 保持打开，localStorage 跨命令保留

## 🔑 关键设计决策

| 决策 | 原因 |
|------|------|
| Bun 而非 Node.js | 编译二进制（~58MB 单文件）、原生 SQLite、原生 TypeScript |
| 守护进程而非每次启动 | 持久状态 + 子秒延迟；每次冷启动 3-5s |
| HTTP 而非 MCP | 更轻量、更易调试（curl）、对 token 更友好 |
| localhost only (127.0.0.1) | 安全边界——不暴露到网络 |
| SKILL.md 生成而非手写 | 代码→文档同步，避免 drift |
| 10 宿主配置（Host Config） | 同一套 .tmpl 模板为 Claude/Codex/Cursor/Kiro 等生成不同 SKILL.md |

## 🛠️ 技术栈

- **运行时**：Bun（编译二进制 + 原生 SQLite + TypeScript）
- **浏览器**：Playwright + Chromium（CDP 协议）
- **服务器**：`Bun.serve()`（无 Express/Fastify，~10 个路由）
- **安全**：UUID Bearer Token + 双监听器隧道 + L1-L6 分层防御
- **模板**：`SKILL.md.tmpl` → `gen-skill-docs.ts` → `SKILL.md`（代码驱动文档）

## 🔄 Sprint 流程

```
Think → Plan → Build → Review → Test → Ship → Reflect
```

每个技能知道前一步做了什么，因为设计文档从 `/office-hours` 流到 `/plan-ceo-review` → `/plan-eng-review` → `/qa` → `/review` → `/ship`。

## 🔗 关联知识

- [[gstack 守护进程架构]] — 持久浏览器守护进程的设计
- [[gstack Skill 模板系统]] — 代码驱动文档生成管道
- [[gstack 安全架构]] — 分层防御 + 隧道隔离
- [[gstack Ref 交互系统]] — @e/@c 引用如何解决元素寻址
- [[gstack 测试体系]] — 三层测试 + diff-based 选择
- [[守护进程模式]] — 通用架构模式提取
- [[分层安全防御]] — L1-L6 模式的通用化
- [[构建者信条]] — Boil the Lake / Search Before Building

## 📚 外部资源

- [GitHub](https://github.com/garrytan/gstack) — 开源代码仓库
- [ARCHITECTURE.md](https://github.com/garrytan/gstack/blob/main/ARCHITECTURE.md) — 架构决策文档
- [ETHOS.md](https://github.com/garrytan/gstack/blob/main/ETHOS.md) — 构建者哲学

## 🏷️ 标签

#gstack #项目概览 #智能体 #工具链 #参考笔记