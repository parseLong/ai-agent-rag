---
tags:
  - agent-architecture
aliases:
  - AI Agent架构总览
  - 17种Agent架构
创建日期: 2026-05-19
---

> 基于 [all-agentic-architectures](https://github.com/your-username/all-agentic-architectures) 项目整理
> 涵盖 17 种核心 Agent 架构模式

---

## 总览表

|   #    | 架构                         | 核心概念                                 | 关键应用场景           |                 |
| :----: | -------------------------- | ------------------------------------ | ---------------- | --------------- |
| **01** | 01 Reflection              | 反射 (Reflection)                      | 通过自我批评和修正提升输出质量  | 高质量代码生成、复杂摘要    |
| **02** | 02 Tool Use                | 工具使用 (Tool Use)                      | 调用外部 API 和函数扩展能力 | 实时研究助手、企业机器人    |
| **03** | 03 ReAct                   | 推理+行动 (ReAct)                        | 动态交错推理与行动的循环     | 多跳问答、网页导航研究     |
| **04** | 04 Planning                | 规划 (Planning)                        | 在执行前将复杂任务分解为详细计划 | 报告生成、项目管理       |
| **05** | 05 Multi-Agent Systems     | 多智能体系统 (Multi-Agent Systems)         | 多专家代理协作分工        | 软件开发流水线、创意头脑风暴  |
| **06** | 06 PEV                     | 计划执行验证 (PEV)                         | 自校正循环，验证每一步结果    | 高风险自动化、金融、不可靠工具 |
| **07** | Blackboard                 | 黑板系统 (Blackboard)                    | 通过共享中央黑板动态协作     | 复杂诊断、动态感知       |
| **08** | Episodic + Semantic Memory | 情景+语义记忆 (Episodic + Semantic Memory) | 双记忆系统：向量存储+图数据库  | 长期个人助理、个性化导师    |
| **09** | Tree of Thoughts           | 思维树 (Tree of Thoughts)               | 在树结构中探索多条推理路径    | 逻辑谜题、约束规划       |
| **10** | Mental Loop                | 心理循环 (Mental Loop)                   | 在内部模拟器中预测行动后果    | 机器人、金融交易、安全关键系统 |
| **11** | Meta-Controller            | 元控制器 (Meta-Controller)               | 监督代理路由任务到专家子代理   | 多服务 AI 平台、自适应助手 |
| **12** | Graph Memory               | 图记忆 (Graph Memory)                   | 结构化图存储实体关系       | 企业情报、高级研究       |
| **13** | Ensemble                   | 集成 (Ensemble)                        | 多代理并行分析后聚合结论     | 高风险决策支持、事实核查    |
| **14** | Dry-Run Harness            | 试运行框架 (Dry-Run Harness)              | 代理行动先模拟再审批执行     | 生产代理部署、调试       |
| **15** | RLHF                       | 基于人类反馈的强化学习 (RLHF)                   | 利用编辑反馈迭代改进并保存    | 高质量内容生成、持续学习    |
| **16** | Cellular Automata          | 元胞自动机 (Cellular Automata)            | 去中心化网格代理产生涌现行为   | 空间推理、物流、复杂系统模拟  |
| **17** | Reflexive Metacognitive    | 反射性元认知 (Reflexive Metacognitive)     | 代理自我建模，选择行动或升级   | 高风险咨询（医疗、法律、金融） |

---

## 学习路径

### Part 1: 基础模式 (Notebooks 1-4)
- [[01 Reflection|反射]] -- 通过自我批评和修正提升输出质量
- [[02 Tool Use|工具使用]] -- 调用外部 API 和函数扩展能力
- [[03 ReAct|推理+行动]] -- 动态交错推理与行动的循环
- [[04 Planning|规划]] -- 在执行前将复杂任务分解为详细计划

### Part 2: 多智能体协作 (Notebooks 5, 7, 11, 13)
- [[05 Multi-Agent Systems|多智能体系统]] -- 多专家代理协作分工
- [[07 Blackboard|黑板系统]] -- 通过共享中央黑板动态协作
- [[11 Meta-Controller|元控制器]] -- 监督代理路由任务到专家子代理
- [[13 Ensemble|集成]] -- 多代理并行分析后聚合结论

### Part 3: 高级记忆与推理 (Notebooks 8, 9, 12)
- [[08 Episodic + Semantic Memory|情景+语义记忆]] -- 双记忆系统：向量存储+图数据库
- [[09 Tree of Thoughts|思维树]] -- 在树结构中探索多条推理路径
- [[12 Graph Memory|图记忆]] -- 结构化图存储实体关系

### Part 4: 安全、可靠性与现实交互 (Notebooks 6, 10, 14, 17)
- [[06 PEV|计划执行验证]] -- 自校正循环，验证每一步结果
- [[10 Mental Loop|心理循环]] -- 在内部模拟器中预测行动后果
- [[14 Dry-Run Harness|试运行框架]] -- 代理行动先模拟再审批执行
- [[17 Reflexive Metacognitive|反射性元认知]] -- 代理自我建模，选择行动或升级

### Part 5: 学习与适应 (Notebooks 15, 16)
- [[15 RLHF|基于人类反馈的强化学习]] -- 利用编辑反馈迭代改进并保存
- [[16 Cellular Automata|元胞自动机]] -- 去中心化网格代理产生涌现行为