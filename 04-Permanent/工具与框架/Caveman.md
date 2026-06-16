---
title: Caveman
description: AI 回复输出的废话压缩器——四种模式（lite/full/ultra/wenyan），平均节省65-75%输出Token
tags: [工具, Token成本, 上下文压缩, 智能体, 工程实践]
aliases:
  - caveman
  - 输出压缩
date: 2026-06-15
source: "[[一篇搞懂 AI Coding Agent 的 Token 成本控制]]"
github: https://github.com/JuliusBrussee/caveman
paper: https://arxiv.org/abs/2604.00025
---

# Caveman

> RTK 压缩的是命令输出（输入 Token），Caveman 压缩的是 AI 回复（输出 Token）。两者方向不同，可以叠加。

## 🧠 核心洞察

编码场景最浪费的回答不是"错"，是"废"：先复述问题，再讲背景，礼貌铺垫，最后才给结论。Caveman 直接打掉高频废 Token——不是"简陋"，而是精准。

## 📊 核心数据

来源：arXiv:2604.00025

| 任务类型 | 正常输出 | Caveman | 节省 |
|----------|----------|---------|------|
| 解释 React bug | 118 tokens | 15 tokens | **87%** |
| 配置 PostgreSQL | 234 tokens | 38 tokens | **84%** |
| Docker 多阶段构建 | 104 tokens | 29 tokens | **72%** |
| Code Review PR | 67 tokens | 39 tokens | 41% |
| **平均** | — | — | **65-75%** |

## 🔑 四种模式

| 模式 | 压缩程度 | 说明 | 适合场景 |
|------|----------|------|----------|
| `/caveman lite` | 最轻 | 删填充词，保留冠词，专业风格 | 正式输出 |
| `/caveman`（full） | 平衡 | 删冠词，碎片句 | 日常开发 |
| `/caveman ultra` | 极度 | `A→B→C`箭头因果链 | 快速定位 |
| `/caveman wenyan` | 极度 | 文言文模式（token 效率最高的书面语） | 极限压缩 |

**实际效果示例**（full 模式）：

```
问：为什么 React 组件不断重渲染？

普通输出（69 tokens）：
"The reason your React component is re-rendering is likely 
because you're creating a new object reference on each render 
cycle..."

Caveman 输出（19 tokens）：
"New object ref each render. Inline object prop = new ref = 
re-render. Wrap in useMemo."
```

## 🛠️ 安装

```bash
git clone https://github.com/studyzy/caveman
cd caveman && ./install.sh
# 选中 [2] CodeBuddy 确定即可
```

> [!note] CodeBuddy 支持
> Caveman 官方目前还没有支持 CodeBuddy，使用 [studyzy Fork 版](https://github.com/studyzy/caveman)（已增加 CodeBuddy 支持）。

安装后在 CodeBuddy 里输入 `/caveman` 激活。模式在会话内持续生效，输入 `stop caveman` 关闭。

## 🔑 为什么这类约束值钱

指定输出格式同时省两类：
1. **输出更短** → 输出 Token 减少
2. **结果更可用** → 重试更少 → 避免整包上下文反复付款

## 🔗 关联知识

- [[AI Coding Agent Token 成本控制]] — 五层优化框架中第三层的具体工具
- [[RTK]] — 压缩终端命令输出，与 Caveman 方向不同可叠加
- [[headroom]] — 压缩所有进上下文的内容
- [[context-mode]] — 不干预输出格式，与 Caveman 各管各的，不冲突
- [[上下文压缩]] — 第三层优化的理论基础

## 🏷️ 标签

#工具 #Token成本 #上下文压缩 #智能体 #工程实践
