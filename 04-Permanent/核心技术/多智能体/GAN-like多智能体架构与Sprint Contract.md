---
date: 2026-06-15
tags:
  - 方法
  - 多智能体
  - 驭驭工程
  - 架构模式
source: "[[OpenClaw与Hermes源码架构深度复盘]]"
  - "Anthropic Harness Engineering实践(2026.03)"
---

# GAN-like 多智能体架构与 Sprint Contract

> [!quote] 核心定义
> Anthropic 在 Harness Engineering 实践中提出的一种受 GAN（生成对抗网络）启发的多智能体协作范式——将单体 Agent 分解为 Planner/Generator/Evaluator 三个职责清晰的角色，通过角色分离实现"生产"与"验收"的职责隔离，再用 Sprint Contract 定义可验证的验收标准。

---

## 三角色架构

| 角色 | 核心职责 | 交互机制 |
|------|---------|----------|
| **Planner（规划者）** | 将简短需求扩展为详细产品规格，拆分任务为可执行的冲刺 | 输出 Sprint Contract |
| **Generator（生成器）** | 逐步实现每个冲刺，编写代码和设计 | 接收 Planner 的计划，按 Sprint Contract 交付 |
| **Evaluator（评估者）** | 像 QA 团队一样测试应用，寻找缺陷和改进点 | 通过 Playwright 等工具对**运行中的应用**进行动态测试 |

---

## 核心洞见

### 角色分离 — 不让模型自评

> **Generator 不能评估自己的输出**（避免自我评估偏差）
> **Evaluator 被提示词设计为"寻找漏洞的挑剔者"**而非"友好用户"

### 每个 Role 可以独立做 Context Reset

Planner 完成规划后 → Generator 以**全新 session**启动 → 只接收结构化任务描述 → 不背负规划过程中的推理噪声。这是 [[上下文重置与交接]] 在多 Agent 场景的应用。

---

## Sprint Contract — 明确任务边界与验收标准

在每个冲刺开始前，Planner 与 Generator 就"完成"的定义达成一致——将主观的"完成标准"转化为**可验证的客观条件**。

### 为什么需要 Sprint Contract

防止长周期任务中常见的**规范漂移**：用户故事与实现细节之间的落差逐步积累成 bug。Sprint Contract 让 Evaluator 有明确的验收标准可执行，而非凭"感觉"打分。

### 结构化 Handoff（Anthropic 的 `claude-progress.txt` 模式）

session 切换时不是简单"摘要前文"，而是写入 5 层结构：

```
1. 状态快照 — 代码仓库当前状态、已完成的任务
2. 叙事上下文 — "我们为什么这样做"
3. 决策日志 — "选择了X而非Y的原因"
4. 优先队列 — "下一步该做什么"
5. 警告与陷阱 — "踩过的坑、要注意的事"
```

新 session warm start 时读取这个文件，立刻知道"上次做到哪、下一步该干什么"。

---

## 两个框架的空白区

| 框架 | 现状 | 缺什么 |
|------|------|--------|
| OpenClaw `sessions_spawn` | 可以 spawn 子Agent并行 | ❌ 无 Evaluator 角色 — 结果直接回传父Agent |
| Hermes `delegate_tool` | 子Agent有阻止列表+迭代预算 | ❌ 无独立评估角色 — 父Agent既是委派者又是验收者 |

---

## 落地思路

在 OpenClaw Subagent 机制上叠加 Evaluator Agent：

- `subagentRole` 新增 `evaluator` 类型
- 该角色拥有 Playwright MCP 工具但没有代码编辑权限
- 按 Sprint Contract 定义 Rubric 对 Generator 输出打分
- 分数不过 → 触发 Generator 在新 session 中修复（又一次 Context Reset）
- 直到验收通过

这比当前的"spawn → 收结果 → 信任结果"更可靠。

---

## 多维度 Rubric 评分

Anthropic 将主观质量转化为可量化的四个维度：

| 维度 | 评分什么 |
|------|----------|
| **设计质量** | 整体性而非零件堆砌 |
| **原创性** | 严惩"AI套路" |
| **工艺** | 排版间距一致性 |
| **功能性** | 用户能否完成任务 |

---

## 与其他多 Agent 架构对比

| 模式 | 来源 | 核心思想 |
|------|------|----------|
| **GAN-like** | Anthropic Harness | 生成→对抗验证→修复循环 |
| **CrewAI** | CrewAI | 角色分工+Sequential/Hierarchical |
| **DeepAgent** | Eino(字节) | 主Agent拆分+子Agent各自执行+Checkpoint |
| **Sprint Contract** | Anthropic | 显式验收标准 + 结构化Handoff |

---

## 相关笔记

- [[Harness Engineering]] — 驾驭工程全景
- [[上下文重置与交接]] — Context Reset 的工程实现
- [[上下文腐蚀与注意力衰减]] — 长周期任务失败的根因分析
- [[多智能体协作]] — 通用多Agent理论
- [[Smart Approval三态审批]] — 安全审批中的角色分离思想
- [[技能自创建闭环与渐进式披露]] — Hermes的自我改进机制（另一种"对抗"思路）
