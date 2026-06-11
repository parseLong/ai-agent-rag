---
title: AutoGen
description: 微软推出的多智能体对话框架，通过角色定义和自动化对话实现智能体协作
tags: [概念卡片, 永久笔记, 框架, 多智能体, 协作]
date: 2026-05-19
---

# AutoGen

## 📖 定义

AutoGen 是微软推出的多智能体对话框架，其核心思想是通过**对话实现协作**。它将多智能体系统抽象为一个由多个"可对话"智能体组成的群聊。开发者可以定义不同角色（如 `Coder`、`ProductManager`、`Tester`），并设定它们之间的交互规则，任务的解决过程就是智能体在群聊中通过自动化消息传递不断协作的过程。

## 🧠 我的理解

AutoGen 的设计哲学是"以对话驱动协作"。它模拟了人类团队的工作方式：不同角色的人通过对话交流信息、分工合作、解决问题。AutoGen 将这种协作模式形式化，让智能体能够像人类团队一样高效协作。

## 🎯 为什么重要

AutoGen 代表了多智能体协作的一种自然范式。它不依赖复杂的工作流定义，而是通过角色定义和对话规则，让协作行为"涌现"出来。这种方式更贴近人类协作模式，在某些场景下比硬编码工作流更灵活。

## 🔑 核心组件

| 组件 | 说明 |
|------|------|
| **AssistantAgent** | 任务的主要解决者，封装 LLM，根据对话历史生成回复 |
| **UserProxyAgent** | 人类用户的"代言人"，可执行代码或调用工具 |
| **RoundRobinGroupChat** | 轮询群聊，按预定义顺序让智能体依次发言 |
| **Team** | 智能体团队的协调机制 |

## 🔑 关键特性

- **对话驱动**：通过消息传递实现协作，自然模拟人类团队
- **角色专业化**：通过系统消息定义高度专业化的角色
- **流程自动化**：`RoundRobinGroupChat` 自动管理对话流程
- **人类在环**：`UserProxyAgent` 提供天然的人类介入接口

## ⚠️ 局限性

- **不确定性**：LLM 对话本质不确定，可能偏离预期
- **调试困难**：对话历史长，调试复杂
- **循环风险**：对话可能陷入无限循环

## 💡 实例/案例

在 Hello-Agents 教程中，使用 AutoGen 构建了一个"软件开发团队"：
- **ProductManager**：将需求转化为可执行的开发计划
- **Engineer**：编写具体的应用程序代码
- **CodeReviewer**：审查代码质量、可读性和健壮性
- **UserProxy**：代表用户，执行和验证最终代码

### 案例详解：软件开发团队

**1. 模型客户端配置**：
```python
from autogen_ext.models.openai import OpenAIChatCompletionClient

def create_openai_model_client():
    return OpenAIChatCompletionClient(
        model=os.getenv("LLM_MODEL_ID", "gpt-4o"),
        api_key=os.getenv("LLM_API_KEY"),
        base_url=os.getenv("LLM_BASE_URL", "https://api.openai.com/v1")
    )
```

**2. 智能体角色定义**（以工程师为例）：
```python
def create_engineer(model_client):
    system_message = """你是一位资深的软件工程师，擅长 Python 开发和 Web 应用构建。

当收到开发任务时，请：
1. 仔细分析技术需求
2. 选择合适的技术方案
3. 编写完整的代码实现
4. 添加必要的注释和说明
5. 考虑边界情况和异常处理

请提供完整的可运行代码，并在完成后说"请代码审查员检查"。"""

    return AssistantAgent(
        name="Engineer",
        model_client=model_client,
        system_message=system_message,
    )
```

**3. 定义团队协作流程**：
```python
from autogen_agentchat.teams import RoundRobinGroupChat
from autogen_agentchat.conditions import TextMentionTermination

# 定义团队聊天和协作规则
team_chat = RoundRobinGroupChat(
    participants=[
        product_manager,
        engineer,
        code_reviewer,
        user_proxy
    ],
    termination_condition=TextMentionTermination("TERMINATE"),
    max_turns=20,
)
```

**4. 启动与运行**：
```python
async def run_software_development_team():
    task = """我们需要开发一个比特币价格显示应用，具体要求如下：
    - 实时显示比特币当前价格（USD）
    - 显示24小时价格变化趋势
    - 使用 Streamlit 框架创建 Web 应用
    请团队协作完成这个任务，从需求分析到最终实现。"""
    
    # 异步执行团队协作，并流式输出对话过程
    result = await Console(team_chat.run_stream(task=task))
    return result

# 主程序入口
if __name__ == "__main__":
    result = asyncio.run(run_software_development_team())
```

**5. 预期协作效果**：
```
============================================================
---------- TextMessage (ProductManager) ----------
### 1. 需求理解与分析
...
请工程师开始实现。
---------- TextMessage (Engineer) ----------
### 技术方案实施
...
请代码审查员检查。
---------- TextMessage (CodeReviewer) ----------
### 代码审查
...
代码审查完成，请用户代理测试。
---------- TextMessage (UserProxy) ----------
Enter your response: TERMINATE
============================================================
✅ 团队协作完成！
```

## 🔗 关联知识

- **相似框架**：[[AgentScope]]（消息驱动）、[[CAMEL]]（角色扮演）
- **对比框架**：[[LangGraph]]（图结构控制）
- **核心概念**：[[多智能体]]、[[对话系统]]、[[ReAct]]
- **应用场景**：[[第六章 框架开发实践]]

## 📝 个人笔记

AutoGen 的最大价值在于"降低多智能体协作的建模门槛"。开发者只需定义"谁（角色）"和"做什么（职责）"，而不必关心"如何做（流程控制）"。但要注意，对话的不确定性可能带来稳定性问题，适合创意类、探索性任务，而非严格的流程控制。

## 🏷️ 标签

#概念卡片 #永久笔记 #框架 #多智能体 #协作 #AutoGen
