---
title: Agent 间数据流转约定
description: Orchestrator-Worker 模式的工程化落地——上下文隔离后通过共享外置文件传递信息，而非会话历史。四原则（输出结构化、进度文件追踪、Worker context裁剪、临时文件清理），含 .agent/ 目录约定和 JSON 模板示例
tags: [概念卡片, 永久笔记, 方法, 智能体, 多智能体, 工程实践, Token成本]
aliases:
  - Agent Data Flow Convention
  - .agent/ 目录约定
  - Agent间数据传递
  - 文件系统共享模式
  - Orchestrator-Worker 数据流转
date: 2026-06-15
source: "[[一篇搞懂 AI Coding Agent 的 Token 成本控制]]"
---

# Agent 间数据流转约定

> 上下文既然隔离了，Agent 之间怎么传递信息？答案：**通过共享外置文件，不通过会话历史。** 会话历史是每个 Agent 私有的，但文件系统是共享的。

## 🧠 核心洞察

Orchestrator-Worker 模式（详见 [[多智能体协作]]）让每个 Agent 只看当前步骤的相关内容，大幅减少每轮的 Token 消耗。但上下文隔离带来一个必须解决的问题：**Agent 之间怎么传递信息？**

会话历史是每个 Agent 私有的，不同 Agent 的历史无法互通。但文件系统是共享的。这意味着上下文隔离和信息共享可以同时成立：

```
Agent A 完成工作
  → 把结果写入文件（.agent/step1_result.json）
  → 上下文销毁

Agent B 开始工作
  → 读取文件（.agent/step1_result.json）
  → 只看这个文件，不看 A 的历史
  → 把自己的结果写入 .agent/step2_result.json
```

> [!important] 用文件替代历史
> Orchestrator 每次唤醒时，只需要读进度文件就能知道任务进展。不需要回放任何 Agent 的历史会话。这就是用 `.agent/` 目录约定替代"会话历史传递"。

## 🔑 四原则

### 原则一：输出格式要结构化

自然语言在 Agent 之间传递时容易产生歧义，也难以精确定位所需信息。结构化的 JSON 更紧凑、更可靠，下游 Agent 只需要读它关心的字段：

```json
// .agent/findings.json
{
  "task": "locate-bug",
  "status": "completed",
  "findings": [
    {
      "file": "src/api/order.go",
      "line": 142,
      "issue": "并发场景下 inventory.Lock() 未释放",
      "severity": "high"
    }
  ],
  "next_step": "fix-bug",
  "context_needed": ["src/api/order.go:120-165", "src/model/inventory.go:30-55"]
}
```

下游 Agent 的指令只需要：

```
读取 .agent/findings.json，针对其中的 findings 数组修复代码。
修复完成后将结果写入 .agent/fix_result.json，格式参考 .agent/findings.json。
```

### 原则二：用进度文件追踪状态

复杂任务里，Orchestrator 需要知道每个步骤是否完成、是否失败、是否需要重试。这个状态本身也应该外置：

```json
// .agent/progress.json
{
  "task_id": "fix-order-bug-20260609",
  "created_at": "2026-06-09T10:00:00Z",
  "steps": [
    {
      "id": "step-1-locate",
      "status": "completed",
      "worker": "investigator",
      "output_file": ".agent/findings.json",
      "completed_at": "2026-06-09T10:02:30Z"
    },
    {
      "id": "step-2-fix",
      "status": "in_progress",
      "worker": "implementer",
      "started_at": "2026-06-09T10:02:35Z"
    },
    {
      "id": "step-3-test",
      "status": "pending",
      "depends_on": "step-2-fix"
    },
    {
      "id": "step-4-changelog",
      "status": "pending",
      "depends_on": "step-2-fix"
    }
  ]
}
```

Orchestrator 每次唤醒时，只需要读这一个文件就能知道任务进展。

### 原则三：每个 Worker 的 context 包精心裁剪

新开一个 Worker 时，Orchestrator 应该明确告诉它"只读哪些东西"，而不是让 Worker 自己去探索：

```
# Orchestrator 派遣 Worker 时的指令模板

任务：为 CreateOrder 修复编写单元测试
上下文：
  - 修复内容：读 .agent/fix_result.json 中的 diff 字段
  - 环有测试风格：读 src/api/order_test.go（前50行）
  - 不需要读其他文件

输出：
  - 新增测试写入 src/api/order_test.go
  - 把测试覆盖情况写入 .agent/test_result.json

约束：
  - 只修改 order_test.go，不动其他文件
  - 测试不超过 80 行
```

这个指令本身很短（约 150 tokens），但它让 Worker 的上下文精准到最小必要集合。

### 原则四：临时文件及时清理

`.agent/` 目录是临时工作区，任务完成后可以归档或删除：

```bash
# 任务完成后归档
mv .agent/ .agent-archive/fix-order-bug-20260609/

# 或直接清理
rm -rf .agent/
```

这样不会污染代码仓库，也不会把旧任务的上下文意外带入新任务。

## 🔑 并行执行的适用性判断

独立的子任务可以同时启动。实际加速效果：

| 并行 Worker 数 | 实际加速倍数 |
|----------------|-------------|
| 2 个 | 1.5–1.8 倍 |
| 3 个 | 2.2–2.6 倍 |
| 4 个 | 2.8–3.4 倍 |

**适合并行**：
- 不同模块的 Bug 修复（改的文件没有交集）
- 代码 + 测试 + 文档（三者可以同时生成）
- 影响分析 + 实现方案设计（可以同时推进）

**不适合并行**：
- 有顺序依赖的步骤（先定位 Bug 才能修复）
- 修改同一个文件的多个任务（会产生冲突）
- 依赖上一步输出的任务（需要等待）

## 🔑 端到端示例：API 层重构

**场景**：给一个中型 Go 项目做 API 层重构。

| 模式 | Token 消耗 | 说明 |
|------|-----------|------|
| 单 Agent 全程跑 | 800K–1.2M | 带全程历史，每轮重新处理同一批背景 |
| Orchestrator-Worker | 100K–150K | 每步只看相关内容，**节省 70–85%** |

Orchestrator-Worker 分阶段流转：

```
阶段 0：初始化
  Orchestrator 读 graph.json → 生成 .agent/plan.json
  → 消耗：～8K tokens

阶段 1：分析（并行）
  Worker A → .agent/audit.json
  Worker B → .agent/test_gap.json
  → 消耗：～12K tokens

阶段 2：实现
  Worker C → 按 .agent/audit.json 逐文件修复
  → 消耗：～6K × 文件数

阶段 3：补测（串行，依赖阶段 2）
  Worker D → 读修改后文件 + .agent/test_gap.json
  → 消耗：～10K tokens

阶段 4：汇总（并行）
  Worker E → 重构报告
  Worker F → PR 描述
  → 消耗：～8K tokens
```

## 🔗 关联知识

- [[AI Coding Agent Token 成本控制]] — 五层框架中第五层的工程化落地
- [[多智能体协作]] — Orchestrator-Worker 的理论基础
- [[任务分解]] — 识别可并行 vs 必串行的子任务
- [[Agent 工作边界]] — Worker 的自主权范围和 context 裁剪
- [[上下文重置与交接]] — 另一种 Agent 间信息传递方式（结构化 Handoff 文件）
- [[上下文工程三层Stack]] — 数据流转约定属于 Harness Engineering（第3层 runtime 控制）
- [[工具调用回路]] — 文件传递避免了会话历史在 Agent 间的回路放大
- [[减少重试是成本优化]] — 结构化 JSON + 进度文件减少 Agent 间的信息歧义重试

## 🏷️ 标签

#概念卡片 #永久笔记 #方法 #智能体 #多智能体 #工程实践 #Token成本
