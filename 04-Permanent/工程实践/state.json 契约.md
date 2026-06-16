---
title: state.json 契约
date: 2026-06-16
tags:
  - 概念卡片
  - 永久笔记
  - 方法
  - 驭驭工程
  - 智能体
  - 多智能体
source: [[AI不缺智商缺纪律-我的Harness工程化实践]]
---

# state.json 契约

> state.json 是 Dispatcher 状态机的==单一真相来源==（Single Source of Truth）——所有 agent 通过它读写流程状态，所有门禁通过它判断流程进展，所有 hook 通过它保证写一致性。它是 [[Dispatcher 状态机与文件交接]] 的骨架。

---

## 核心字段

state.json 作为 agent 间交接的契约文件，承载以下信息：

| 字段 | 类型 | 含义 | 写入者 |
|------|------|------|-------|
| `current_phase` | string | 当前流程阶段（如 DEVELOPING, REVIEWING, DEPLOYING） | dispatcher |
| `intent` | string | 意图分类（QUERY / BUG_FIX / FEATURE / REFACTOR） | intent-classifier |
| `risk_level` | string | 风险等级（LOW / MEDIUM / HIGH） | intent-classifier |
| `required_gates` | string[] | intent×risk 裁剪后的必需门禁列表 | dispatcher |
| `passed_gates` | string[] | 已通过的门禁列表 | verifier |
| `next_agent` | string | 下一步该调的 agent 名称 | dispatcher |
| `artifacts` | object[] | 各阶段产物文件路径和状态 | 各 agent |

---

## 三条铁律

### 1. 单写者原则

**state-keeper 单写者**——只有编排层 agent（dispatcher）有权写入 state.json。其他 agent 绕过直接写 → hook 拦截 reject。

> 这是 [[Harness Engineering#声明式策略]] 原则的运行时实现——状态修改权限声明式定义，hook 在运行时强制执行。

### 2. ajv schema 校验前置

state.json 的结构由 ajv JSON Schema 定义——写入前先校验，从架构层面消除多 agent 写冲突。

> 这与 [[Auth Profile与FailoverError]] 中 OpenClaw 的闭合枚举错误分类思路一致——==用类型系统防止不可能的状态==。

### 3. 文件交接而非消息传递

agent A 写 `phases/05-design.md`，agent B 读它。state.json 记录每步产物的路径和状态。

---

## 核心优势

| 优势 | 说明 | 对应遗忘根因 |
|------|------|------------|
| **天然持久化** | 进程崩了文件还在，跨天需求 Read state.json 即续 | 检索失败 → 状态不丢 |
| **可审计** | 每步产物都是人可读 markdown，git diff 一眼看清谁在哪步写了什么 | 压缩丢失 → 产物外置 |
| **强一致性** | 单写者 + schema 校验，从架构层面消除写冲突 | 指令遵循失败 → 确定性代码保证 |

> [!note] 与 [[Agent 遗忘三重根因]] 的精确对应
> state.json 的三个优势恰好对应三重遗忘根因的三个对策：持久化→检索失败对策、可审计→压缩丢失对策、强一致性→指令遵循失败对策。

---

## 与其他状态管理方案的对照

| 方案 | 状态载体 | 持久化 | 可审计 | 强一致性 |
|------|---------|--------|--------|---------|
| **state.json** | 文件系统 | ✅ 进程崩溃不丢 | ✅ markdown 人可读 | ✅ 单写者 + schema 校验 |
| **Agent Team TaskList** | 内存 | ❌ 中断后靠推断 | ❌ 内部数据结构 | ❌ 多写者无协调 |
| **Workflow resumeFromRunId** | session 内 | ⚠️ 跨 session 不可续 | ⚠️ 脚本内部状态 | ✅ 确定性控制流 |

---

## 与 Devin "脑机分离"的关系

Devin 的架构：推理（大脑）在沙箱外执行，执行环境（机器）无权访问大脑状态。state.json 做了更轻量的等价——==不隔离进程，但隔离状态写入权限==：

| Devin | 本 Harness |
|-------|-----------|
| 进程级隔离（大脑 vs 机器） | 文件级隔离（hook 拦截非编排层的写入） |
| 状态在沙箱外持久化 | state.json 在文件系统持久化 |
| 代价：状态管理更复杂 | 代价：更轻量但依赖 hook 执行力 |

---

## 关联知识

- [[Dispatcher 状态机与文件交接]] — state.json 是 Dispatcher 状态机的核心载体
- [[Agent 遗忘三重根因]] — state.json 三优势对应三对策
- [[G1-G8 门禁墙参考卡]] — state.json 记录门禁通过/失败状态
- [[Harness 五层架构实践]] — state.json 在角色 Agent 层的使用
- [[护栏与钩子系统]] — hook 拦截是单写者原则的运行时保障
- [[Auth Profile与FailoverError]] — 闭合枚举防止不可能状态
- [[多智能体协作]] — 状态管理是多 Agent 三大挑战之一
