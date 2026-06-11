---
tags: agent-architecture
aliases: [推理+行动, ReAct]
创建日期: 2026-05-19
来源: 03_ReAct.ipynb
---

# 03 推理+行动 (ReAct)

> 动态交错推理与行动的循环
> 关键应用: 多跳问答、网页导航研究

---

## 定义

**ReAct** 架构是一种设计模式，其中代理将推理步骤与操作交织在一起。智能体不是预先计划所有步骤，而是生成有关下一步的想法，采取行动（例如调用工具），观察结果，然后使用新信息生成下一个想法和行动。这创建了一个动态和自适应循环。

## 工作流程

1. **接收目标：** 代理被赋予一项复杂的任务。
2. **思考（原因）：** 智能体产生一个内部思考，例如：*“要回答这个问题，我首先需要找到一条信息X。”*
3. **行动：** 基于其思想，代理执行一个操作，通常调用一个工具（例如，`search_api('X')`）。
4. **观察：** 代理从工具接收结果。
5. **重复：** 代理将观察结果合并到其上下文中并返回到步骤 2，产生新的想法（例如，*“好吧，现在我有了 X，我需要用它来找到 Y。”*）。这个循环一直持续到总体目标得到满足。

## 应用场景

* **多跳问答：** 当回答问题需要按顺序查找多条信息时（例如，“谁是制造 iPhone 的公司的首席执行官？”）。
* **网络导航和研究：** 代理可以搜索起点，读取结果，然后根据学到的内容决定新的搜索查询。
* **交互式工作流程：** 环境是动态的并且无法提前知道解决方案的完整路径的任何任务。

## 优缺点

* **优势：**
* **自适应和动态：** 可以根据新信息动态调整其计划。
* **处理复杂性：** 擅长解决需要链接多个相关步骤的问题。
* **弱点：**
* **更高的延迟和成本：** 涉及多个连续的 LLM 调用，使其比单次方法更慢且更昂贵。
* **循环的风险：** 指导不善的代理可能会陷入重复、低效的思想和行动循环。

## 核心代码示例

### 示例 1

```python
import os
from typing import Annotated
from dotenv import load_dotenv

# LangChain components
from langchain_nebius import ChatNebius
from langchain_community.tools.tavily_search import TavilySearchResults
from langchain_core.messages import BaseMessage
from pydantic import BaseModel, Field

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
os.environ["LANGCHAIN_PROJECT"] = "Agentic Architecture - ReAct (Nebius)"

# Check that the keys are set
for key in ["NEBIUS_API_KEY", "LANGCHAIN_API_KEY", "TAVILY_API_KEY"]:
    if not os.environ.get(key):
        print(f"{key} not found. Please create a .env file and set it.")

print("Environment variables loaded and tracing is set up.")
```

### 示例 2

```python
from typing import TypedDict

console = Console()

# Define the state for our graphs
class AgentState(TypedDict):
    messages: Annotated[list[AnyMessage], add_messages]

# Define the tool and LLM
search_tool = TavilySearchResults(max_results=2, name="web_search")
llm = ChatNebius(model="meta-llama/Meta-Llama-3.1-8B-Instruct", temperature=0)
llm_with_tools = llm.bind_tools([search_tool])

# Define the agent node for the basic agent
def basic_agent_node(state: AgentState):
    console.print("--- BASIC AGENT: Thinking... ---")
    # Note: We provide a system prompt to encourage it to answer directly after one tool call
    system_prompt = "You are a helpful assistant. You have access to a web search tool. Answer the user's question based on the tool's results. You must provide a final answer after one tool call."
    messages = [("system", system_prompt)] + state["messages"]
    response = llm_with_tools.invoke(messages)
    return {"messages": [response]}

# Define the basic, linear graph
basic_graph_builder = StateGraph(AgentState)
basic_graph_builder.add_node("agent", basic_agent_node)
basic_graph_builder.add_node("tools", ToolNode([search_tool]))

basic_graph_builder.set_entry_point("agent")
# After the agent, it can only go to tools, and after tools, it MUST end.
basic_graph_builder.add_conditional_edges("agent", tools_condition, {"tools": "tools", "__end__": "__end__"})
basic_graph_builder.add_edge("tools", END)

basic_tool_agent_app = basic_graph_builder.compile()

print("Basic single-shot tool-using agent compiled successfully.")
```

---

## 结论

在本笔记本中，我们不仅实现了 **ReAct** 架构，而且还展示了其相对于更基本的单次方法的明显优势。通过构建一个允许代理循环推理和行动的工作流程，我们使其能够解决复杂的、多步骤的问题，否则这些问题将变得棘手。

观察行动结果并利用该信息指导下一步的能力是智能行为的基本组成部分。ReAct 模式提供了一种简单但极其有效的方法来将此功能构建到我们的 AI 代理中，使它们更强大、适应性更强，并且对现实世界的任务更有用。

