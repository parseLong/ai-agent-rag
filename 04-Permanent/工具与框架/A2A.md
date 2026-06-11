---
title: A2A
description: Agent-to-Agent Protocol，由 Google 提出的智能体间通信协议，实现智能体之间的点对点协作
tags: [概念, 工具与框架, 概念卡片, 永久笔记, 协议, 智能体, 通信, A2A]
aliases:
  - A2A
  - Agent-to-Agent Protocol
  - A2A 协议
date: 2025-05-19
---

# A2A

> Google提出的Agent间协作协议，实现跨平台跨框架的Agent安全、标准化协作。如果说 MCP 解决的是"智能体如何访问工具"，那么 A2A 解决的就是"智能体如何与其他智能体对话"。

## 核心要点

- **四大功能特性**：
  - Capability discovery：Agent用JSON格式的"Agent Card"宣传自身能力
  - Task management：通信以任务完成为导向，Task对象有生命周期
  - Collaboration：Agent间传递上下文、回复、Artifact，支持认证/授权
  - UX negotiation：协商内容格式和用户界面呈现方式

- **定位**：面向Agent与Agent的连接，区别于MCP面向Agent与外部世界的连接

- **设计哲学**：对等通信——智能体既是服务提供者也是消费者，避免了中心化协调器的瓶颈

## A2A 核心概念

| 概念 | 说明 |
|------|------|
| **任务（Task）** | A2A 的核心抽象，代表智能体之间需要协作完成的工作 |
| **工件（Artifact）** | 任务的产出物，可以是文档、代码、数据等 |
| **对等通信** | 智能体既是服务提供者也是消费者 |
| **动态发现** | 智能体可以动态发现网络中的其他智能体 |
| **任务协商** | 智能体之间可以协商任务分工和执行计划 |

## 与 MCP 的对比

| 特性 | MCP | A2A |
|------|-----|-----|
| **通信对象** | 智能体 ↔ 工具 | 智能体 ↔ 智能体 |
| **设计哲学** | 上下文共享 | 对等通信 |
| **核心功能** | 工具发现、调用 | 智能体发现、任务协商 |
| **典型场景** | 访问外部服务 | 多智能体协作 |
| **提出者** | Anthropic | Google |

## 💡 代码示例

### 创建 A2A 智能体

```python
from hello_agents.protocols.a2a.implementation import A2AServer, A2A_AVAILABLE

def create_calculator_agent():
    if not A2A_AVAILABLE:
        print("A2A SDK 未安装，请运行: pip install a2a-sdk")
        return None

    # 创建 A2A 服务器
    calculator = A2AServer(
        name="calculator-agent",
        description="专业的数学计算智能体",
        version="1.0.0",
        capabilities={
            "math": ["addition", "subtraction", "multiplication", "division"],
            "advanced": ["power", "sqrt", "factorial"]
        }
    )

    # 添加基础计算技能
    @calculator.skill("add")
    def add_numbers(query: str) -> str:
        """加法计算"""
        try:
            parts = query.replace("计算", "").replace("加", "+")
            if "+" in parts:
                numbers = [float(x.strip()) for x in parts.split("+")]
                result = sum(numbers)
                return f"计算结果: {' + '.join(map(str, numbers))} = {result}"
        except Exception as e:
            return f"计算错误: {e}"

    return calculator

# 创建并测试智能体
calc_agent = create_calculator_agent()
if calc_agent:
    result = calc_agent.skills["add"]("计算 10 + 5")
    print(result)  # 计算结果: 10.0 + 5.0 = 15.0
```

## 💡 应用场景

- **多智能体协作**：研究员、撰写员、编辑等角色智能体协同完成任务
- **任务分配**：复杂任务自动分解并分配给合适的智能体
- **能力互补**：不同智能体发挥各自专长，共同完成复杂目标

## 关联知识

- **相关协议**：[[MCP]]（智能体-工具通信）、[[ANP]]（智能体网络协议）
- **核心概念**：[[智能体通信协议]]、[[多智能体]]
- [[万字详解大模型应用发展：RAG、MCP、Agent的爆发之旅]]

## 🏷️ 标签

#概念 #工具与框架 #概念卡片 #永久笔记 #协议 #智能体 #通信 #A2A