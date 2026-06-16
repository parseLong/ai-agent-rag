---
title: OpenClaw 框架
tags:
  - 概念
  - Agent框架
  - 开源
aliases:
  - OpenClaw
---

> [!tip] 内容已整合
> 本笔记的所有内容已合并到 → [[OpenClaw架构深度解析]]，包含源码级架构深度 + 实用部署指南 + 横向对比，一篇覆盖全部。

---

## 快速导航

| 你想了解什么 | 去哪里 |
|-------------|--------|
| 核心价值 / 罗福莉引言 / 共识推广 | [[OpenClaw架构深度解析#定义与核心价值]] |
| 微内核5大设计理念 | [[OpenClaw架构深度解析#微内核架构哲学]] |
| 8文件配置体系 / Agent启动流程 | [[OpenClaw架构深度解析#Agent 配置文件体系]] |
| 多Agent部署 / 通信配置 / 实战经验 | [[OpenClaw架构深度解析#多 Agent 通信与部署]] |
| 记忆机制 / Dreaming算法 | [[OpenClaw架构深度解析#记忆系统双层架构]] |
| Skills管控 / Clawhub | [[OpenClaw架构深度解析#Skills 管控]] |
| 部署指南 / 硬件 / IM选择 | [[OpenClaw架构深度解析#部署指南]] |
| 应用案例 | [[OpenClaw架构深度解析#应用案例]] |
| 横向对比（LangGraph/CrewAI等） | [[OpenClaw架构深度解析#横向对比：Agent 框架生态定位]] |
| 安全五层纵深 | [[OpenClaw架构深度解析#安全机制五层纵深]] |
| Gateway 5角色 / Session Key路由 | [[OpenClaw架构深度解析#Gateway — 系统心脏]] |

---

## 相关笔记

- [[OpenClaw架构深度解析]] — 整合后的完整笔记（源码深度 + 实用指南）
- [[OpenClaw vs Hermes 架构对比]] — 两个框架的六维度正面对比
- [[本地Agent（Claw）]] — OpenClaw 的本地优先定位
- [[Coding Plan（AI编程订阅制）]] — OpenClaw按token计费时代的高消耗痛点，推动了Coding Plan的诞生

## 2026年市场回顾

> 来源：[[2026过半：一万字，把这半年AI发生的事讲明白]]

### 春节热潮与消退

2026年春节后，国内AI圈最热的名字是龙虾（OpenClaw）。GitHub Star数达到37万，成为开源Top 1。几乎全民都在养龙虾，Kimi、GLM和MiniMax相继推出Coding Plan，能在OpenClaw里直接挂国产模型。

现在龙虾热潮已经大大降低，留下来的是超级发烧友。

### Token消耗代价

OpenClaw是心思特别细腻的管家，每一轮对话都拖家带口地把系统提示、长期记忆、技能元数据全塞进去。token消耗大概是Claude Code的3到5倍。这不是bug，是它的形态决定的——一个永远在线、跨多通道的Agent，必须随时拎着完整上下文，否则人格、记忆、技能就接不上。

### 历史定位

它把"自动化Agent"从极客玩具拽到了大众能用的水平，这一步意义已经够大。2026下半年的核心战场不在通用ChatBot，而在每个人都可以有自己的专属Agent，OpenClaw是第一个真正能跑通的开源样本。