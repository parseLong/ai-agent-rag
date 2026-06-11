---
title: Claude Code 知识地图
description: Claude Code 源码核心功能的 MOC — 从整体架构到驾驭工程深度解析
tags: [MOC, Claude-Code, 架构, 智能体, 驾驶工程]
date: 2026-05-20
---

# Claude Code 知识地图

> Claude Code 是 Anthropic 的官方 CLI 工具，让用户在终端中与 Claude 交互完成软件工程任务。
> 本 MOC 基于其源码（约 512K 行 TypeScript），梳理核心功能的架构和原理。

---

## 🗺️ 核心架构

- [[Claude Code 整体架构]] — 入口流程、架构全景、数据流、启动优化策略

## 🧠 决策与执行

- [[查询引擎]] — Agent 的"大脑运行时"，管理对话生命周期、流式响应、工具调用循环
- [[工具系统]] — Agent 的"手和脚"，~40 种工具的自包含架构
- [[子代理系统]] — 多代理编排，worktree 隔离、前台/后台模式

## 🔒 安全与边界

- [[权限系统]] — Agent 的"安全带"，多层权限模型、规则引擎、ML 分类器
- [[功能开关与死代码消除]] — bun:bundle 编译时剥离未启用代码

## 🔌 外部连接

- [[MCP 协议实现]] — Agent 的"外部连接器"，传输层、认证、工具发现
- [[Bridge 桥接系统]] — IDE-CLI 双向通信层
- [[MCP 服务器集成]] — 配置层（与 MCP 协议实现互补）

## 📦 内部协调

- [[状态管理（Claude Code）]] — Agent 的"神经系统"，自定义 store + 选择性订阅
- [[上下文压缩]] — Agent 的"记忆管理"，snip compaction 和 reactive compact
- [[命令系统]] — Agent 的"快捷方式"，~50 个斜杠命令的懒加载架构

---

## 📐 源码架构与设计

- [[Claude Code 源码架构]] — 完整架构梳理（~512K 行 TypeScript 的综合分析）
- [[Claude Code 源码架构补充 — 知识图谱与模块细节]] — HTML 知识图谱提炼的补充细节
- [[Claude Code 源码设计亮点深度分析]] — 12 个值得学习的优秀设计决策
- [[Claude Code 架构解析]] — 从单代理到自主团队的演进路径

---

## 🛷 驾驶工程深度解析

> 驾驶工程（Harness Engineering）是 2025-2026 年新兴的 AI 工程实践领域，专注于为智能体设计和实施控制层——将原始 LLM 转化为可控、安全、高效的智能体。
> Claude Code 的整个架构就是一个驾驶工程的**完整实现**。

- [[驾驶工程]] — 控制层/驾驶层概念全景，LLM 是引擎、驾驶层是方向盘和安全带
  - [[系统提示词工程]] — 方向盘：静态段+动态段+缓存边界+行为塑形
  - [[工具描述工程]] — 操作手册：schema+权限标记+描述文本的三位一体
  - [[上下文注入管道]] — 导航仪：CLAUDE.md 优先级合并+git status+记忆加载
  - [[护栏与钩子系统]] — 安全带：19 种事件钩子+权限分类器+Abort 机制
  - [[输出解析与路由]] — 信号处理器：流式检测+并行/串行调度+Sibling Abort
  - [[重试与恢复机制]] — 自动修复：API 10次重试+5层压缩策略+Fast Mode 降级
  - [[流式渲染控制]] — 仪表盘：工具自渲染+进度折叠+Spinner 状态+分级透明

---

## 🗺️ 与 AI Agent 教程的对照

| 教程概念 | Claude Code 实现 | 笔记 |
|---------|-----------------|------|
| ReAct 范式 | QueryEngine 的工具调用循环 | [[查询引擎]] ↔ [[ReAct 范式]] |
| Agent 工具调用 | Tool System (~40 工具) | [[工具系统]] |
| 多智能体协作 | AgentTool + worktree 隔离 | [[子代理系统]] |
| 记忆系统 | 上下文压缩 | [[上下文压缩]] ↔ [[记忆系统]] |
| 上下文工程 | System/User context + snip | [[上下文注入管道]] ↔ [[第九章 上下文工程]] |
| Agent 通信协议 | MCP + Bridge | [[MCP 协议实现]] ↔ [[第十章 智能体通信协议]] |
| Agent 安全 | 权限系统 + 护栏 + 钩子 | [[权限系统]] ↔ [[分层安全防御]] ↔ [[护栏与钩子系统]] |
| 驾驶工程 | 整个 Claude Code 架构 | [[驾驶工程]] — 新兴概念 |

---

## 🔗 外部资源

- [Claude Code 官方文档](https://docs.anthropic.com/en/docs/claude-code)
- [Claude Code GitHub](https://github.com/anthropics/claude-code)

---

*最后更新：2026-05-20*