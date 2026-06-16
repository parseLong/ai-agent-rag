---
title: 运行时环境（Runtime）
description: Agent 的专属 Workspace——从无状态调用到有状态的隔离运行时系统，是 Agent 从"问答机器"到"数字员工"的关键基础设施
tags: [概念卡片, 永久笔记, 概念, 智能体, 运行时, Runtime, Environment, 安全]
aliases:
  - Runtime
  - Agent Runtime
  - Environment
  - 运行环境
  - Workspace
date: 2026-05-22
source: "[[Agent核心技术概念与范式发生了哪些演变以及背后的思考]]"
author: "[[飞樰]]"
---

# 运行时环境（Runtime）

> 早期 Agent 对工具调用是无状态的，但随着 Agent 能力增强——引入文件系统操作、代码执行等——它不再仅仅是"问答机器"，而是需要持久化存储、文件读写和状态管理的"数字员工"。这就意味着 Agent 必须拥有一个专属的 Workspace。

---

## 📖 定义

运行时环境（Runtime / Environment）是 Agent 执行任务的专属工作空间，提供持久化存储、文件读写、状态管理和安全隔离。它是 Agent 从无状态问答到有状态执行的关键基础设施转变。

**核心公式**：Agent System = Agent Loop + Harness + **Runtime**

---

## 🔄 演进脉络

| 阶段 | Runtime 特征 | 说明 |
|------|-------------|------|
| **早期 Agent** | 无状态 | 每次调用独立，无需持久化 |
| **自主 Agent** | 有状态 Workspace | Agent 需要读写文件、保存中间产物、管理配置 |
| **自进化 Agent** | 长期 Runtime | Agent 在 Runtime 中沉淀 Skill、记忆、知识库 |

---

## 🔧 两种形态

### 本地个人电脑（Local Desktop）

| 特性 | 说明 |
|------|------|
| **优势** | 极高便利性和灵活性，直接操作用户本地文件/应用/网络 |
| **典型任务** | 整理桌面文件、自动化办公流程 |
| **代表** | [[本地Agent（Claw）]] 最早基于个人电脑操作而火爆 |
| **风险** | 缺乏严格隔离机制，操作失误可能导致重要数据丢失或系统配置混乱 |
| **应对** | 引入更严格的用户确认机制或权限控制 |

### 沙箱环境（Sandbox / Cloud Server）

| 特性 | 说明 |
|------|------|
| **定位** | 企业级生产环境的主流选择 |
| **技术** | Docker、[[Kubernetes]] 等容器化技术构建隔离沙箱 |
| **隔离机制** | Agent 所有操作被限制在特定虚拟文件系统内 |
| **安全性** | 即使执行破坏性命令，也不影响宿主机或其他服务 |
| **价值** | 提供必要的安全边界和资源管控 |

---

## 🧠 Runtime 的核心功能

```
Runtime Workspace
├── 配置层 — Agent读取配置文件（CLAUDE.md / AGENTS.md / SOUL.md）
├── Skill层 — 管理和加载Agent技能库（SKILL.md + scripts/）
├── Memory层 — 写入记忆文件（MEMORY.md / 日志）
├── 产物层 — 生成中间文件、输出产物
├── 日志层 — 记录执行日志、状态变化
└── 安全层 — 权限控制、操作审计、用户确认
```

---

## 🔗 与其他模块的关系

| 关联模块 | Runtime 的角色 |
|---------|---------------|
| [[系统提示词工程\|Prompt]] | 渐进式加载的文件系统依托于 Runtime |
| [[Skills技能\|Tools]] | CLI 和 Script 的执行环境就是 Runtime |
| [[记忆系统\|Memory]] | 文件系统化记忆的存储载体就是 Runtime |
| [[Harness Engineering\|Harness]] | Runtime 是 Harness 中安全隔离和权限控制的物理实现 |

---

## 📝 个人笔记

Runtime 的出现标志着 Agent 从"纯推理系统"到"完整工作系统"的质变。一个没有 Runtime 的 Agent 只能回答问题；有了 Runtime 的 Agent 能真正执行任务、积累经验、自我进化。Runtime 和 [[Harness Engineering]] 的关系值得深思：Harness 是"逻辑层面的驾驭"，Runtime 是"物理层面的驾驭"——两者共同构成了 Agent 的安全边界。

---

## 🔗 关联知识

- **前置概念**：[[智能体（Agent）]]、[[Harness Engineering]]
- **技术实现**：[[Kubernetes]]、Docker
- **相关模块**：[[系统提示词工程]]、[[Skills技能]]、[[记忆系统]]
- **演进背景**：[[Agent技术范式演变（2023-2026）]]

## 🏷️ 标签

#概念卡片 #永久笔记 #概念 #智能体 #运行时 #Runtime #Environment #安全