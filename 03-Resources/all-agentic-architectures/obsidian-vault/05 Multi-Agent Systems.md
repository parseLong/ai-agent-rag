---
tags: agent-architecture
aliases: [多智能体系统, Multi-Agent Systems]
创建日期: 2026-05-19
来源: 05_multi_agent.ipynb
---

# 05 多智能体系统 (Multi-Agent Systems)

> 多专家代理协作分工
> 关键应用: 软件开发流水线、创意头脑风暴

---

## 定义

**多代理系统**是一种架构，其中一组不同的专业代理协作（或有时竞争）以实现共同目标。中央控制器或定义的工作流协议用于管理代理之间的通信和路由任务。

## 工作流程

1. **分解：** 一个主控制器或用户提供一个复杂的任务。
2. **角色定义：** 系统根据定义的角色（例如“研究员”、“编码员”、“评论家”、“作家”）将子任务分配给专门的代理。
3. **协作：** 代理通常并行或顺序执行任务。他们将输出相互传递或传递到中央“黑板”。
4. **综合：** 最终的“管理器”或“合成器”代理收集专业代理的输出并组装最终的综合响应。

## 应用场景

* **复杂报告生成：** 创建需要多个领域专业知识（例如财务分析、科学研究）的详细报告。
* **软件开发流程：** 模拟一个由程序员、代码审查员、测试员和项目经理组成的开发团队。
* **创意头脑风暴：** 具有不同“个性”的代理人团队（例如，一个乐观，一个谨慎，一个极富创造力）可以产生一组更加多样化的想法。

## 优缺点

* **优势：**
* **专业化和深度：** 每个代理都可以使用特定的角色和工具进行微调，从而在其领域内实现更高质量的工作。
* **模块化和可扩展性：** 可以轻松添加、删除或升级单个代理，而无需重新设计整个系统。
* **并行性：** 多个代理可以同时处理其子任务，从而可能减少总体任务时间。
* **弱点：**
* **协调开销：** 管理代理之间的通信和工作流程增加了系统设计的复杂性。
* **成本和延迟增加：** 运行多个代理涉及更多的 LLM 调用，这可能比单代理方法更昂贵且更慢。

## 核心代码示例

### 示例 1

```python
import os
from typing import List, Annotated, TypedDict, Optional
from dotenv import load_dotenv

# LangChain components
from langchain_nebius import ChatNebius
from langchain_tavily import TavilySearch
from langchain_core.messages import BaseMessage, SystemMessage, HumanMessage
from pydantic import BaseModel, Field
from langchain_core.prompts import ChatPromptTemplate

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
os.environ["LANGCHAIN_PROJECT"] = "Agentic Architecture - Multi-Agent (Nebius)"

for key in ["NEBIUS_API_KEY", "LANGCHAIN_API_KEY", "TAVILY_API_KEY"]:
    if not os.environ.get(key):
        print(f"{key} not found. Please create a .env file and set it.")

print("Environment variables loaded and tracing is set up.")
```

### 示例 2

```python
console = Console()

# Define the shared state for both agents
class AgentState(TypedDict):
    messages: Annotated[list[AnyMessage], add_messages]

# Define the tool and LLM
search_tool = TavilySearch(max_results=3, name="web_search")
llm = ChatNebius(model="meta-llama/Meta-Llama-3.1-8B-Instruct", temperature=0)
llm_with_tools = llm.bind_tools([search_tool])

# Define the monolithic agent node
def monolithic_agent_node(state: AgentState):
    console.print("--- MONOLITHIC AGENT: Thinking... ---")
    response = llm_with_tools.invoke(state["messages"])
    return {"messages": [response]}

tool_node = ToolNode([search_tool])

# Build the ReAct graph for the monolithic agent
mono_graph_builder = StateGraph(AgentState)
mono_graph_builder.add_node("agent", monolithic_agent_node)
mono_graph_builder.add_node("tools", tool_node)
mono_graph_builder.set_entry_point("agent")

def tools_condition_with_end(state):
    result = tools_condition(state)
    if isinstance(result, str):
        # Older versions return just "tools" or "agent"
        return {result: "tools", "__default__": END}
    elif isinstance(result, dict):
        # Newer versions return a mapping
        result["__default__"] = END
        return result
    else:
        raise TypeError(f"Unexpected type from tools_condition: {type(result)}")

mono_graph_builder.add_conditional_edges("agent", tools_condition_with_end)
mono_graph_builder.add_edge("tools", "agent")

monolithic_agent_app = mono_graph_builder.compile()

print("Monolithic 'generalist' agent compiled successfully.")
```

---

## 结论

在本笔记本中，我们展示了**多代理系统**相对于单个整体代理在复杂、多方面任务方面的明显优势。通过创建一个由专业代理组成的团队，每个代理都有一个专注的角色和角色，并由一名经理来综合他们的工作，我们产生了明显更高质量的最终输出。

关键要点是**专业化**的力量。就像在人类组织中一样，分解一个大问题并将其部分分配给专家会产生更好的结果。虽然这种架构在编排方面引入了更多的复杂性，但最终输出的结构、深度和专业性的显着改进使其成为任何需要跨多个领域提供专家级性能的严肃代理应用程序不可或缺的模式。

