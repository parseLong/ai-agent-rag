---
title: RTK
description: 终端命令输出的 Token 压缩器——四层过滤策略（智能过滤→分组聚合→智能截断→去重合并），实测30分钟会话节省80% Token
tags: [工具, Token成本, 上下文压缩, 智能体, 工程实践]
aliases:
  - Rust Token Killer
  - rtk
date: 2026-06-15
source: "[[一篇搞懂 AI Coding Agent 的 Token 成本控制]]"
github: https://github.com/rtk-ai/rtk
---

# RTK

> 终端命令输出才是真正的 Token 大杀手——30 分钟开发会话，RTK 节省 80% Token。

## 🧠 核心洞察

很多人把 Token 浪费归因于"聊太多轮"，但忽略了一个隐性大户：**终端命令的冗余输出**。`cargo test` 成千上万行日志、警告、进度条、ANSI 颜色码全部涌进上下文。

RTK 压缩的是**终端命令输出**（输入 Token），方向不同于 [[Caveman]]（输出 Token）和 [[headroom]]（所有上下文）。三者可以叠加。

## 🔑 四层过滤策略

| 层级 | 策略 | 说明 |
|------|------|------|
| 1 | **智能过滤** | 剔除注释、空行、ANSI 颜色码、进度条、无关警告 |
| 2 | **分组聚合** | 同类输出合并展示（搜索结果按文件分组、错误按类型归类） |
| 3 | **智能截断** | 按信息密度取样，保留关键片段，砍掉重复长尾 |
| 4 | **去重合并** | 「连接超时」重复 10 次 → 「连接超时（×10）」 |

## 📊 实测数据

**30 分钟开发会话对比**（中等规模 TypeScript/Rust 项目）：

| 场景 | Token 消耗 |
|------|-----------|
| 不用 RTK | ~118,000 |
| 用 RTK | ~23,900 |
| **节省** | **80%** |

按命令类别的典型压缩率：

| 命令类别 | 原始 tokens | RTK 后 | 节省 |
|----------|-------------|--------|------|
| cargo test / npm test | 25,000 | 2,500 | **-90%** |
| pytest | 8,000 | 800 | **-90%** |
| vitest run | 102,199 字符 | 377 字符 | **-99.6%** |
| ls / tree | 2,000 | 400 | -80% |
| grep / rg | 16,000 | 3,200 | -80% |
| git status | 3,000 | 600 | -80% |
| git diff | 10,000 | 2,500 | -75% |

> [!note] 统计口径说明
> 前几行是 token 统计，`vitest run` 是字符长度统计，用于说明压缩幅度，不能和 token 行直接横向比较。

## 🛠️ 安装与使用

```bash
# macOS（推荐）
brew install rtk

# Linux / WSL
curl -fsSL https://raw.githubusercontent.com/rtk-ai/rtk/master/install.sh | sh

# 全局启用 CodeBuddy（推荐）
rtk init -g --agent codebuddy

# 仅启用 Hook，不修改 CODEBUDDY.md
rtk init -g --hook-only
```

重启 CodeBuddy 后自动生效，所有命令透明经过 RTK 过滤。

```bash
# 常用命令
rtk gain                 # 查看节省统计
rtk gain --history       # 带命令历史
rtk discover             # 扫描哪些命令还没用 RTK

# Git（压缩率 60-80%）
rtk git status
rtk git diff

# 测试（压缩率 90-99.6%）
rtk test cargo test
rtk vitest run
```

> [!warning] 注意
> 安装后用 `rtk gain` 验证。如果命令不存在，可能装了同名的 Rust Type Kit，需卸载重装。

## 🔗 关联知识

- [[AI Coding Agent Token 成本控制]] — 五层优化框架中第三层的具体工具
- [[Caveman]] — 压缩输出端废 Token，与 RTK 方向不同可叠加
- [[headroom]] — 压缩所有进上下文的内容，集成 RTK 可叠加
- [[context-mode]] — 压缩 MCP 工具返回值 + 会话连续性
- [[上下文压缩]] — 第三层优化的理论基础

## 🏷️ 标签

#工具 #Token成本 #上下文压缩 #智能体 #工程实践
