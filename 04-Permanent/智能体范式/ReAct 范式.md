---
tags:
  - 概念
  - 智能体范式
aliases:
  - ReAct
  - Reasoning and Acting
---
# ReAct 范式

> 让模型交替进行推理和行动，形成"思考→行动→观察"循环直到任务完成。谷歌2022年提出。

## 核心要点

- **三个标签**：Thought（思考）→Action（行动）→Observation（观察），每轮迭代更新
- **优势**：实现简单，Prompt里给几个示例就能跑，不需要额外训练，适合动态环境
- **局限**：缺乏全局规划能力，容易陷入局部最优；成本高昂（多次LLM调用），耗时长，已有思考过程无法固化；在早期模型阶段，==能做好3轮以上Reasoning的模型都不多==

> [!example] 示例
> 用户问"今天北京天气"：
> 1. Thought: 需要查询实时天气
> 2. Action: call_weather_api(city="北京")
> 3. Observation: {"weather": "晴", "temperature": "28°C"}
> 4. Thought: 已获取天气，可以回答
> 5. Answer: 今天北京晴，28°C

## 与CoT的区别

[[提示工程]]中的CoT只推理不行动，输出思维链但没有外部工具调用。ReAct推理和行动结合，能通过工具获取外部信息。

## ReAct 的历史定位：Agent发展的第一阶段

> 来源：[[Agent技术范式演变（2023-2026）]]

ReAct 是 2023 年 Agent 概念启蒙期的代表性范式，属于**"被动式 ReAct"阶段**：

| 特征 | 说明 |
|------|------|
| **交互形态** | 增强版 Chatbot，"一问一答"或"指令-执行" |
| **能力边界** | 依赖用户明确指令，只能完成单点、短链路小任务 |
| **局限性** | 缺乏长期规划能力，逻辑链条过长时易偏离/中断 |
| **受限于模型** | 当时能做好3轮以上Reasoning的模型不多 |

ReAct 本质上符合单步的"Reasoning → Observe → Response"过程链条，是 Agent 最基础的运行范式。随着基座模型推理能力升级，Agent 在 ReAct 基础上逐步发展出 [[Plan-and-Solve]]、[[Plan-and-Execute]] 等更复杂的规划范式，最终走向自主 Agent 阶段。

ReAct 循环中通过 **`todo` 工具** 嵌入 Plan 能力，简单任务零开销，复杂任务自动启用计划。

> [!note] 阶段递进
> ReAct → 工作流 Agent → 自主 Agent → 自进化 Agent —— 四个阶段并存互补，ReAct 在简单任务场景中仍有价值。

详见 [[Agent技术范式演变（2023-2026）]]

## 关联知识

- [[Plan-and-Solve]] — 先规划后执行的另一种范式
- [[Reflection 范式]] — 反思改进的补充机制
- [[智能体循环]] — ReAct的底层循环机制
- [[Function Calling]] — ReAct中行动的具体实现方式
- **访谈洞察**：
  - [[Language Agent 概念]] — 姚顺雨关于ReAct是"最喜欢的"工作的阐述
  - [[ReAct 范式]] — 姚顺雨访谈中的详细解读

## 参考资料

- [[万字详解面试题库｜Agent篇]]
- [[详尽地带你从零开始设计实现一个AI Agent框架]]
- [[万字详解大模型应用发展：RAG、MCP、Agent的爆发之旅]]