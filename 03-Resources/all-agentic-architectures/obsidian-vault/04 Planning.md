---
tags: agent-architecture
aliases: [规划, Planning]
创建日期: 2026-05-19
来源: 04_planning.ipynb
---

# 04 规划 (Planning)

> 在执行前将复杂任务分解为详细计划
> 关键应用: 报告生成、项目管理

---

## 定义

**规划**架构涉及一个代理，该代理在*开始执行之前*将复杂的目标明确分解为一系列详细的子任务。这个初始规划阶段的输出是一个具体的、逐步的计划，然后代理有条不紊地遵循该计划以找到解决方案。

## 工作流程

1. **接收目标：** 代理被赋予一项复杂的任务。
2. **计划：** 专用的“计划器”组件分析目标并生成实现目标所需的子任务的有序列表。例如：`["Find fact A", "Find fact B", "Calculate C using A and B"]`。
3. **执行：** “执行器”组件接受计划并根据需要使用工具按顺序执行每个子任务。
4. **综合：** 计划中的所有步骤完成后，最终组件会将执行步骤的结果综合为连贯的最终答案。

## 应用场景

* **多步骤工作流程：** 非常适合操作顺序已知且关键的任务，例如生成需要获取数据、处理数据然后对其进行汇总的报告。
* **项目管理助理：** 将“推出新功能”之类的大目标分解为不同团队的子任务。
* **教育辅导：** 创建课程计划来教授学生特定的概念，从基础原理到高级应用。

## 优缺点

* **优势：**
* **结构化&可追溯：** 整个工作流程提前布局，使得代理流程透明且易于调试。
* **高效：** 对于可预测的任务，它比 ReAct 更高效，因为它避免了步骤之间不必要的推理循环。
* **弱点：**
* **难以改变：** 如果执行过程中环境发生意外变化，预先制定的计划可能会失败。它的适应性不如 ReAct 代理，ReAct 代理可能会在每一步后改变主意。

## 核心代码示例

### 示例 1

```python
import os
import re
from typing import List, Annotated, TypedDict, Optional
from dotenv import load_dotenv

# LangChain components
from langchain_nebius import ChatNebius
from langchain_core.messages import BaseMessage, ToolMessage
from pydantic import BaseModel, Field
from langchain_core.tools import tool
from langchain_core.messages import SystemMessage
from langchain_tavily import TavilySearch

# LangGraph components
from langgraph.graph import StateGraph, END
from langgraph.graph.message import AnyMessage, add_messages
from langgraph.prebuilt import ToolNode, tools_condition

# For pretty printing
from rich.console import Console
from rich.markdown import Markdown

# --- API Key and Tracing Setup ---
load_dotenv()

os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_PROJECT"] = "Agentic Architecture - Planning (Nebius)"

# Check that the keys are set
for key in ["NEBIUS_API_KEY", "LANGCHAIN_API_KEY", "TAVILY_API_KEY"]:
    if not os.environ.get(key):
        print(f"{key} not found. Please create a .env file and set it.")

print("Environment variables loaded and tracing is set up.")
```

### 示例 2

```python
console = Console()

# Define the state for our graphs
class AgentState(TypedDict):
    messages: Annotated[list[AnyMessage], add_messages]

# 1. Define the base tool from the tavily package
tavily_search_tool = TavilySearch(max_results=2)

# 2. THE FIX: Simplify the custom tool. 
#    The .invoke() method already returns a clean string, so we just pass it through.
@tool
def web_search(query: str) -> str:
    """Performs a web search using Tavily and returns the results as a string."""
    console.print(f"--- TOOL: Searching for '{query}'...")
    results = tavily_search_tool.invoke(query)
    return results

# 3. Define the LLM and bind it to our custom tool
llm = ChatNebius(model="meta-llama/Meta-Llama-3.1-8B-Instruct", temperature=0)
llm_with_tools = llm.bind_tools([web_search])

# 4. Agent node with a system prompt to force one tool call at a time
def react_agent_node(state: AgentState):
    console.print("--- REACTIVE AGENT: Thinking... ---")
    
    messages_with_system_prompt = [
        SystemMessage(content="You are a helpful research assistant. You must call one and only one tool at a time. Do not call multiple tools in a single turn. After receiving the result from a tool, you will decide on the next step.")
    ] + state["messages"]

    response = llm_with_tools.invoke(messages_with_system_prompt)
    
    return {"messages": [response]}

# 5. Use our corrected custom tool in the ToolNode
tool_node = ToolNode([web_search])

# The ReAct graph with its characteristic loop
react_graph_builder = StateGraph(AgentState)
react_graph_builder.add_node("agent", react_agent_node)
react_graph_builder.add_node("tools", tool_node)
react_graph_builder.set_entry_point("agent")
react_graph_builder.add_conditional_edges("agent", tools_condition)
react_graph_builder.add_edge("tools", "agent")

react_agent_app = react_graph_builder.compile()
print("Reactive (ReAct) agent compiled successfully.")
```

---

## 控制流视角（agno 实现）

> 来自 [[Agent架构演化总览]] 的控制流分析框架

### 六问拆解

| 问题 | 回答 |
|---|---|
| 它要解决什么问题？ | ReAct 本质是在线贪心策略，对需要顺序约束、步骤依赖和过程可追踪的任务不够用 |
| 它的 State 是什么？ | 新增了 `Plan` 字段（有序步骤列表），以及 `session_state` 中的中间结果。**控制流本身被对象化了** |
| 它的拓扑是什么？ | 线性链 + Loop：Plan → Loop(Execute each step) → Synthesize |
| 它的 Router 怎么工作？ | 路由逻辑从"LLM 在 prompt 里决定下一步"变成了**"一个数据结构是否还有剩余项"**——这才是 Planning 真正的结构变化 |
| 它的失败模式是什么？ | **过于乐观**——一旦 plan 错了，后面每一步都可能是错的。可预测性增强了，适应性下降了 |
| 什么时候该升级？ | 当你不再相信工具会稳定成功 → [[06 PEV]] |

![[07-Attachments/all-agentic-architectures/Planning拓扑.png]]

### 它真正新增了什么能力？

不是"更聪明的思考"，而是：**把未来控制流提前 materialize 成一个数据结构。** 你现在可以可视化计划、检查计划、修改计划、追踪执行进度。

### agno 实现

```python
from agno.workflow.v2 import Workflow, Step, Loop

planner = Agent(name="planner", model=OpenAIChat(id="gpt-5-mini"),
    response_model=Plan,
    instructions="Decompose the user request into a list of atomic tool-queryable steps.")

executor = Agent(name="executor", model=OpenAIChat(id="gpt-5-mini"),
    tools=[DuckDuckGoTools()],
    instructions="Answer exactly one sub-question using tools.")

def plan_is_empty(_outputs) -> bool:
    return len(planning_wf.session_state["plan"]) == 0

planning_wf = Workflow(
    name="planning",
    session_state={"plan": [], "intermediate": []},
    steps=[
        Step(name="plan", executor=plan_step),
        Loop(
            name="execute_all",
            steps=[Step(name="execute_one", executor=execute_step)],
            end_condition=plan_is_empty,
        ),
        Step(name="synthesize", executor=synth_step),
    ],
)
```

`Loop.end_condition` 检查的是数据结构（plan 是否空），而不是 LLM 输出——路由逻辑从非确定性变成确定性。

---

## 结论

在此笔记本中，我们实现了 **Planning** 架构，并将其直接与 **ReAct** 模式进行对比。通过强制代理在执行之前首先构建全面的计划，我们在明确定义的多步骤任务的透明度、稳健性和效率方面获得了显着的好处。

虽然 ReAct 擅长于下一步未知的探索性场景，但当可以提前绘制解决方案的路径时，Planning 就会大放异彩。理解这种权衡对于系统设计者来说至关重要。为正确的问题选择正确的架构是构建有效且智能的人工智能代理的关键技能。规划模式是该工具包中的一个重要工具，提供复杂、可预测的工作流程所需的结构。

---

