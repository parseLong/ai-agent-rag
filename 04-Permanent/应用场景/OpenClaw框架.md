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