---
tags:
  - 概念
  - 智能体
  - agent-architecture
aliases:
  - 17种Agent架构演化
  - Agent控制流设计
创建日期: 2026-06-15
来源:
  - "[[从0开发大模型的17种Agent架构演进详细拆解]]"
  - all-agentic-architectures (GitHub)
  - agno框架
---

# Agent 架构演化总览：控制流设计史

> **核心命题：Agent architecture 的本质不是 prompt engineering，也不是某个框架的 DSL，而是控制流设计。**

![[07-Attachments/all-agentic-architectures/总演化图.png]]

---

## 统一分析框架：六个固定问题

后面每一种架构，都用同一套问题来拆解：

1. **它要解决什么问题？** 上一代架构哪里不够。
2. **它的 State 是什么？** 新增了哪些字段，为什么必须存在。
3. **它的拓扑是什么？** 线性链、循环、分叉汇聚、共享黑板、树搜索还是网格涌现。
4. **它的 Router 怎么工作？** 固定边、条件边、动态调度、验证回路、人工审批。
5. **它的失败模式是什么？** 架构最容易在哪个环节坏掉。
6. **什么时候该升级到下一种？** 当前模式的能力边界在哪里。

> 不再把问题描述成"这个架构更聪明"，而是描述成：它新增了什么状态字段、什么 agent 或工具、什么路由逻辑、什么验证机制。

---

## Agent 演化路径

所谓"agent 架构演化"，不是追求 AGI，而是解决一个老问题：**怎样让系统在更复杂的环境里，依然保持可控、可解释、可恢复。**

| 演化阶段 | 起点 → 终点 | 核心跨越 |
|---|---|---|
| 单次生成 → 反思闭环 | [[01 Reflection]] | 把"生成"拆成 generator + critic + refiner 三步 |
| 反思闭环 → 工具交互 | [[02 Tool Use]] | 打破参数知识边界和上下文封闭性 |
| 工具交互 → 观察-行动循环 | [[03 ReAct]] | 工具结果进入下一轮决策 |
| 局部决策 → 显式规划 | [[04 Planning]] | 控制流本身变成模型输出 |
| 无验证执行 → 验证驱动重规划 | [[06 PEV]] | 验证成为控制流一等公民 |
| 单 agent → 多 agent 编排 | [[05 Multi-Agent Systems]] | 认知分工写进图结构 |
| 固定流水线 → 动态调度 | [[07 Blackboard]] | 共享状态 + controller 动态编排 |
| 持续调度 → 入口分诊 | [[11 Meta-Controller]] | 一次性路由，而不是持续编排 |
| 单视角 → 冗余多视角 | [[13 Ensemble]] | 同一问题多独立 agent 处理 |
| 短期上下文 → 长期记忆 | [[08 Episodic + Semantic Memory]] | episodic + semantic 双记忆 |
| 相似召回 → 关系推理 | [[12 Graph Memory]] | 从 chunk 组织到关系结构 |
| 线性推理 → 搜索 | [[09 Tree of Thoughts]] | 推理转写为搜索问题 |
| 直接执行 → 模拟预演 | [[10 Mental Loop]] | 反事实执行 |
| 副作用自由 → 闸门审批 | [[14 Dry-Run Harness]] | 副作用变成可审批对象 |
| 无边界感知 → 自我建模 | [[17 Reflexive Metacognitive]] | 系统知道自己擅长什么、不擅长什么 |
| 单次优化 → 进化回路 | [[15 RLHF]] | 质量循环 + 样例积累 |
| 中心控制 → 分布式涌现 | [[16 Cellular Automata]] | LLM 退出主循环 |

---

## 架构能力增量一览

| 阶段 | 新增能力 | 一句话解释 | 代表架构 |
| --- | --- | --- | --- |
| 单次生成优化 | critique pass | 先出版再自己挑毛病改掉 | [[01 Reflection]] |
| 与世界交互 | tool interface | 外部 API 挂成结构化工具接口 | [[02 Tool Use]] |
| 基于观察持续行动 | observation loop | Thought → Action → Observation 滚动循环 | [[03 ReAct]] |
| 先生成控制流再执行 | explicit planning | 控制流本身做成可审计对象 | [[04 Planning]] |
| 把验证接入主回路 | verification loop | 失败就回到重规划 | [[06 PEV]] |
| 把认知任务拆成角色 | role decomposition | 研究员/写作/审阅拆开 | [[05 Multi-Agent Systems]] |
| 把中间状态显式共享 | shared workspace | controller 根据黑板状态动态调度 | [[07 Blackboard]] |
| 把入口做成路由系统 | entry routing | 任务路由到最合适的专家子 agent | [[11 Meta-Controller]] |
| 用冗余换可靠性 | parallel redundancy | 多 agent 独立处理再融合投票 | [[13 Ensemble]] |
| 把历史状态纳入系统 | long-term memory | episodic 向量库 + semantic 图/KV | [[08 Episodic + Semantic Memory]] |
| 把推理变成搜索 | search tree | 展开多条思路形成树，边展开边剪枝 | [[09 Tree of Thoughts]] |
| 把行动前评估做成模拟 | counterfactual execution | 先在内部世界模型里预演 | [[10 Mental Loop]] |
| 把副作用关进闸门 | side-effect gating | dry-run + 审核后才真执行 | [[14 Dry-Run Harness]] |
| 把自我边界建模 | self-boundary reasoning | 知道自己擅长什么、不擅长什么 | [[17 Reflexive Metacognitive]] |
| 把质量改进做成循环 | iterative refinement loop | 高分样本沉淀下来用于持续改进 | [[15 RLHF]] |
| 去中心化计算 | emergence | 局部规则产生全局行为 | [[16 Cellular Automata]] |

---

## 架构演化对照表（agno 实现）

![[07-Attachments/all-agentic-architectures/架构演化表.png]]

| 架构 | 新增的关键能力 | 解决的问题 | agno 对应能力 |
| --- | --- | --- | --- |
| Reflection | critique pass | 单次生成质量不稳 | 3 × `Agent` 串联成 `Workflow` |
| Tool Use | world interface | 模型无法触达真实世界 | `Agent(tools=[...])` |
| ReAct | observation loop | 工具结果不能驱动下一步 | `Agent(tools=..., reasoning=True)` |
| Planning | explicit plan state | 缺少全局步骤控制 | `Agent(response_model=Plan)` + `Loop` |
| PEV | verification loop | 执行失败会静默传播 | `Router` + verifier Agent |
| Multi-Agent | role decomposition | 单 prompt 角色冲突 | 多 `Agent` 或 `Team(mode="coordinate")` |
| Blackboard | shared workspace + dynamic controller | 固定流水线不够灵活 | `workflow_session_state` + controller `Router` |
| Meta-Controller | entry routing | 请求类型不同需要分诊 | `Team(mode="route")` |
| Ensemble | parallel redundancy | 单一答案不够可靠 | `Workflow(Parallel(...))` + aggregator |
| Episodic/Semantic Memory | long-term recall | 系统跨轮失忆 | `Memory` + `AgentKnowledge` |
| Graph Memory | relational reasoning | 相似召回不能做关系推理 | `Neo4jTools` + Cypher agent |
| ToT | search tree | 线性推理无法回溯 | 程序化搜索 + proposer `Agent` |
| Mental Loop | counterfactual execution | 真实试错成本太高 | 双工具 `simulate_action / execute_action` |
| Dry-Run | side-effect gating | 副作用动作不能直接执行 | 工具带 `dry_run` 参数 + approval Step |
| Metacognitive | self-boundary reasoning | 系统不知道自己不会什么 | `response_model=MetacognitiveAnalysis` + `Router` |
| Self-Improvement | iterative quality loop | 一次优化不足 | `Loop(end_condition=...)` + gold memory |
| Cellular Automata | decentralized emergence | 中央控制不适合某些问题 | LLM 只设计规则，程序化并行更新 |

---

## 怎么选？问你缺哪种控制能力

| 你缺的能力 | 优先架构 | 为什么 |
|---|---|---|
| 输出质量不稳 | [[01 Reflection]] | 最小质量闭环 |
| 多步工具推理 | [[03 ReAct]] | 观察-行动循环最实用 |
| 全局步骤控制 | [[04 Planning]] | 把控制流显式化 |
| 工具容错 | [[06 PEV]] | 把验证接进主回路 |
| 角色分工 | [[05 Multi-Agent Systems]] | 把认知拆开 |
| 动态编排 | [[07 Blackboard]] | 基于共享状态调度 |
| 请求分诊 | [[11 Meta-Controller]] | 一次路由最省复杂度 |
| 高可靠结论 | [[13 Ensemble]] | 用冗余降低偏差 |
| 跨轮记忆 | [[08 Episodic + Semantic Memory]] | 把历史纳入系统 |
| 关系推理 | [[12 Graph Memory]] | 支持多跳查询 |
| 回溯搜索 | [[09 Tree of Thoughts]] | 适合分支型解空间 |
| 行动前模拟 | [[10 Mental Loop]] | 降低真实试错成本 |
| 副作用审批 | [[14 Dry-Run Harness]] | 先预演再执行 |
| 边界感知 | [[17 Reflexive Metacognitive]] | 先判断能不能做 |
| 长期自我改进 | [[15 RLHF]] | 质量循环 + 样例积累 |
| 去中心化求解 | [[16 Cellular Automata]] | 用局部规则换全局行为 |

### 缺输出质量 → Reflection / Self-Improvement

- 先上 [[01 Reflection]]；需要多轮逼近和长期改进，上 [[15 RLHF]]

### 缺与世界交互 → Tool Use / ReAct

- 简单任务上 [[02 Tool Use]]；多步动态任务上 [[03 ReAct]]

### 缺显式步骤控制 → Planning / PEV

- 上 [[04 Planning]]；工具不可靠，再升级到 [[06 PEV]]

### 缺角色分工 → Multi-Agent / Blackboard / Meta-Controller / Ensemble

- 固定分工：[[05 Multi-Agent Systems]]
- 动态持续调度：[[07 Blackboard]]
- 入口分诊：[[11 Meta-Controller]]
- 同题多视角冗余：[[13 Ensemble]]

### 缺长期状态 → Episodic Memory / Graph Memory

- 记历史事件：[[08 Episodic + Semantic Memory]]
- 做关系推理：[[12 Graph Memory]]

### 缺求解范式 → ToT / Mental Loop / Cellular Automata

- 需要回溯搜索：[[09 Tree of Thoughts]]
- 需要先模拟后执行：[[10 Mental Loop]]
- 需要去中心化求解：[[16 Cellular Automata]]

### 缺安全边界 → Dry-Run / Metacognitive

- 需要副作用审批：[[14 Dry-Run Harness]]
- 需要知道自己不能做什么：[[17 Reflexive Metacognitive]]

---

## Evaluator 不是可选项

agent 不是只要能跑就行，必须能评估，至少有五类 evaluator：

1. **LLM-as-a-Judge**：用独立 Agent 打分
2. **内置 critic**：直接控制循环是否继续（`Loop.end_condition`）
3. **程序化验证**：像 `is_valid()` / `is_goal()` 这种硬约束
4. **Human-in-the-Loop**：用人工审批做最后闸门（Dry-Run）
5. **演示式验证**：用多场景运行验证系统行为

> **没有 evaluator 的 agent，大概率只是一个会循环的 prompt，不是一个可靠的系统。**

---

## agno 最小数学结构

```python
from agno.agent import Agent
from agno.workflow.v2 import Workflow, Step, Router, Loop

agent = Agent(
    model=OpenAIChat(id="gpt-5-mini"),
    tools=[...],
    instructions="...",
    response_model=SomePydanticModel,  # 结构化输出
)

wf = Workflow(
    name="my_flow",
    steps=[
        Step(name="plan", agent=planner_agent),
        Loop(
            name="execute_and_verify",
            steps=[executor_step, verifier_step],
            end_condition=lambda outputs: outputs[-1].content.is_done,
        ),
        Step(name="synthesize", agent=synthesizer_agent),
    ],
)
```

这段代码背后包含了 agent 系统的最小数学结构：

- `response_model` → 状态空间
- `Agent / tools / Step` → 状态变换
- `steps` 列表 → 确定性转移
- `Router / Condition` → 条件转移
- `Loop.end_condition` → 终止条件
- `wf.run()` → 可执行系统

---

## 最终结论

### Agent 架构的演化，本质上是在设计更好的控制流

它在不断回答同一组问题：什么时候该停？什么时候该继续？什么时候该重试？什么时候该换角色？什么时候该查工具？什么时候该调用历史？什么时候该先模拟？什么时候该拒绝？什么时候该让人类接管？

这些问题不依赖某一个框架。只要开始做真实的 agent 系统，就一定会亲手长出同一组抽象：

- `Workflow.steps` → 确定性边
- `Router` → 条件路由
- `Loop` → 循环与终止条件
- `Parallel` → 并行冗余
- `workflow_session_state` → 显式共享状态
- `Agent(tools=...)` → 内置 tool loop
- `Team(mode="route")` → 分诊逻辑

### 三句话总结

1. **先别迷信"万能 agent"，先把状态和控制流画清楚。**
2. **大多数系统从 ReAct 起步，但可靠系统一定会引入验证、记忆和边界控制。**
3. **真正高级的 agent，不是更敢做事，而是更知道什么时候不该做。**

### 验证新架构的三个问题

当你看到任何新的"agent 架构名词"，先问三个问题：

- 它新增了什么 **state**？
- 它新增了什么 **router**？
- 它新增了什么 **evaluator**？

只要这三个问题答不出来，它大概率就不是一种新架构，只是旧架构换了个名字。

---

## 落地系统的五问自查

真正决定一个 agent 系统能不能落地的，通常不是模型回答是不是够好，而是：

- 状态有没有被正确建模
- 控制流有没有被显式表达
- 错误能不能被局部截断
- 副作用能不能被关进闸门
- 系统知不知道自己什么时候该停

---

## 相关笔记

- [[01 Reflection]] — 最小质量闭环
- [[02 Tool Use]] — 文本世界到结构化世界的跨越
- [[03 ReAct]] — Agent 真正成形的地方
- [[04 Planning]] — 把控制流变成模型输出
- [[06 PEV]] — 验证提升为控制流一等公民
- [[05 Multi-Agent Systems]] — 认知分工写进图
- [[07 Blackboard]] — 共享状态成为系统中心
- [[11 Meta-Controller]] — 一次性路由，而不是持续编排
- [[13 Ensemble]] — 不是分工，而是冗余
- [[08 Episodic + Semantic Memory]] — 记忆不是塞回上下文
- [[12 Graph Memory]] — 关系推理而非相似召回
- [[09 Tree of Thoughts]] — 推理变成搜索
- [[10 Mental Loop]] — 行动前先模拟
- [[14 Dry-Run Harness]] — 副作用关进闸门
- [[17 Reflexive Metacognitive]] — 系统思考自己的边界
- [[15 RLHF]] — 进化回路
- [[16 Cellular Automata]] — LLM 退出主循环

## 参考来源

- [all-agentic-architectures (GitHub)](https://github.com/FareedKhan-dev/all-agentic-architectures) — 原项目（LangChain/LangGraph 实现）
- [agno (GitHub)](https://github.com/agno-agi/agno) — agno 框架，更简洁的实现
- [[从0开发大模型的17种Agent架构演进详细拆解]] — 腾讯技术工程原文
