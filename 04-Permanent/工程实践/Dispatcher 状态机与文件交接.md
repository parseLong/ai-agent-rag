---
title: Dispatcher 状态机与文件交接
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

# Dispatcher 状态机与文件交接

> Harness 本质上是==控制平面==，不是计算平面。三种编排机制正交互补——Workflow 管计算平面、Team 管协作平面、dispatcher + 文件交接管控制平面。

![[07-Attachments/harness-practice/三条路对比图.jpg]]

---

## 三条路对比

### ① Claude Code Workflow（JS 脚本编排）

| 强项 | 确定性控制流（循环/条件/扇出）、高并行 `pipeline()` / `parallel()`、schema 强校验 |
|------|------|

**三个硬伤**：
1. **超时机制**——Bash 命令默认 120s 超时（最大 10 分钟），Workflow 子 agent 本身也有执行时长上限；TDD 全循环或 Maven 长构建经常被静默杀死，脚本层拿到的只是 null 返回
2. **无 `askUser` 交互原语**——19 节点链有 6 个人工确认点，Workflow 脚本内无法暂停等待用户决策
3. **跨 session 不可续**——同 session 内可 `resumeFromRunId` 恢复，但 HIGH 需求可能跨 2-3 天，换 session 后状态接不上

> 适合场景：单阶段、无人工交互、可在超时窗口内完成的计算任务（如三角色并行评审）

### ② Agent Team（消息驱动团队）

| 强项 | 多人独立并行改多模块 |
|------|------|

**三个缺陷**：
1. **松散协调无确定性工序保证**——成员 idle 后靠消息唤醒
2. **状态散落在 TaskList 中**——无统一 state.json，中断后恢复靠推断
3. **SendMessage 是"通知"不是"阻断"**——无法做到 hook 级硬围栏

> 适合场景：多人并行改多模块，不适合严格工序链

### ③ Dispatcher 状态机 + 文件系统交接（最终选择）

| 硬优势 | 说明 |
|--------|------|
| **天然持久化** | 进程崩了文件还在，跨天需求 Read state.json 即续 |
| **可审计** | 每步产物都是人可读 markdown，git diff 一眼看清谁在哪步写了什么 |
| **强一致性** | state-keeper 单写者（hook 拦截其他写者）+ ajv schema 校验前置，从架构层面消除多 agent 写冲突 |

**代价**：
- 每次 agent 切换需 Read 上一步产物（~2-5K tokens IO 开销）
- 调试链路跨多个 agent 的 transcript
- 并行能力受限于文件交接的序列化特性

---

## 核心架构：Dispatcher 状态机

```
主会话 → dispatcher(读 state.json，返回"下一步调谁")
→ intent-classifier 判定意图×风险
→ dispatcher → 三角色并行评审 → orchestrator 合成 → 用户确认
→ dispatcher → plan-generator 出实施计划
→ dispatcher → developer 按 TDD 编码 → dispatcher → verifier 跑门禁
→ dispatcher → deployer 部署预发 → dispatcher → tester 接口测试 → 验收报告
```

> [!important] 主会话是"笨执行器"
> 全程主会话没"思考"过任何业务细节，它只是 dispatcher 指令的执行器；每个 agent 从干净上下文启动、只装自己那一段的规则和输入。

---

## 三种机制的定位

```
Workflow → 计算平面（高并行单阶段）
Team     → 协作平面（多人独立任务）
Dispatcher + 文件交接 → 控制平面（有状态工序链 + 人工门禁 + 跨天续跑）
```

> [!tip] 混合编排方向
> 当前实验方向：dispatcher 管控制流，Workflow 加速三角色评审等纯计算环节。三种机制==正交互补==，不必选一弃二。

---

## 与 [[多智能体形态分类]] 的对照

| 本框架概念 | 多智能体形态分类 |
|-----------|----------------|
| Dispatcher 状态机 | Agent OS 形态（有状态编排） |
| Workflow | Dynamic Workflows 形态（JS脚本编排） |
| Agent Team | Agent Team 形态（消息驱动） |
| 文件交接 | Sub-agent 的结构化 Handoff |

> [!note] 与 [[多智能体协作模式]] 的关系
> Dispatcher 模式是"主从模式"的进化——不是单一 Master 指挥，而是状态机驱动的==角色轮换==：dispatcher 指出下一步该谁上场，主会话只负责调用，不负责决策。

---

## 与业界方案的对照

| 方案 | 核心思路 | 与 Dispatcher 模式的关系 |
|------|---------|------------------------|
| **Devin "脑机分离"** | 推理在沙箱外执行，执行环境无权访问大脑状态 | Dispatcher 走了更轻量的路：不隔离进程，agent 职责隔离 + 文件交接 |
| **Apache Burr** | 状态机节点 + 可插拔持久化 + 实时追踪 UI | Dispatcher 的学术化框架版本 |
| **sd0x-dev-flow** | hook-enforced dual review, state-machine gates that survive context compaction | 核心共识一致：门禁外置 + fail-closed |
| **Stripe Minions Blueprint** | 确定性节点 + Agentic 节点混合编排 | Dispatcher 的生产级实现 |

---

## 关联知识

- [[Harness 五层架构实践]] — Dispatcher 在角色 Agent 层的具体角色
- [[多智能体协作模式]] — 7 种协作模式
- [[多智能体形态分类]] — 四种核心形态
- [[多智能体协作]] — 专业化分工与三大挑战
- [[Harness Engineering]] — 驭驭工程全景
- [[Agent 遗忘三重根因]] — 文件交接解决检索失败根因
- [[长程Coding自动化实践经验]] — 另一套 Agent 编排实践
