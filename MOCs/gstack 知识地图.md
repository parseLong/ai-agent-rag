---
title: gstack 知识地图
description: 从 gstack 代码库提取的核心知识体系，按主题组织 — AI 辅助开发工具栈的完整工程实践案例
tags: [项目, 实践, gstack, AI辅助开发, 工具链, 工程案例]
date: 2026-05-19
---

# gstack 知识地图

> 从 gstack 代码库提取的核心知识体系，按主题组织。

---

## 🏗️ 项目与架构

- [[gstack 项目概览]] — 23 专家角色 + 8 强力工具，将 Claude Code 变成虚拟工程团队
- [[gstack 守护进程架构]] — 持久 Chromium 守护进程：子秒延迟 + 持久状态 + 自动生命周期
- [[gstack Ref 交互系统]] — @e/@c 引用寻址网页元素：ARIA 树 + Playwright Locators

## 🛠️ Skill 与模板

- [[gstack Skill 模板系统]] — .tmpl → gen-skill-docs → SKILL.md 管道 + 序言组合根 + 10 宿主生成

## 🔒 安全

- [[gstack 安全架构]] — 双监听器隧道 + L1-L6 分层提示注入防御 + Unicode 清理管道

## 🧪 测试

- [[gstack 测试体系]] — T1/T2/T3 三层 + diff-based 选择 + gate/periodic 分类

---

## 🧠 通用概念提取

以下概念从 gstack 提取并泛化为可跨项目应用的通用知识：

- [[构建者信条]] — 煮沸湖泊 / 构建前搜索 / 用户主权
- [[守护进程模式]] — AI 代理与外部服务的持久后台进程模式
- [[分层安全防御]] — L1-L6 多层交叉验证对抗提示注入
- [[序言组合根模式]] — 分层 tier gating + 声明式 preamble 组合
- [[代码-文档同步管道]] — 源代码元数据驱动文档生成

---

## 🔗 与现有知识的连接

- [[智能体（Agent）]] — gstack 是 LLM 驱动智能体的实际工程案例
- [[ReAct 范式]] — gstack 的 browse 命令体现观察-行动循环
- [[MCP 服务器集成]] — gstack 选择 HTTP 而非 MCP 的设计决策
- [[Skill 开发指南]] — ECC Skill 开发 vs gstack Skill 模板的对比

---

*最后更新：2026-05-19*
