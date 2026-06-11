---
tags: agent-architecture
aliases: [工具使用, Tool Use]
创建日期: 2026-05-19
来源: 02_tool_use.ipynb
---

# 02 工具使用 (Tool Use)

> 调用外部 API 和函数扩展能力
> 关键应用: 实时研究助手、企业机器人

---

## 定义

**工具使用**架构使 LLM 支持的代理能够调用外部函数或 API（“工具”）。代理自主决定何时仅靠其内部知识无法回答用户的查询，并确定适合调用哪个工具来查找必要的信息。

## 工作流程

1. **接收查询：** 代理接收来自用户的请求。
2. **决策：** 代理分析查询及其可用工具。它决定是否需要一个工具来准确回答问题。
3. **操作：** 如果需要工具，代理会格式化对该工具的调用（例如，具有正确参数的特定函数）。
4. **观察：** 系统执行工具调用，并将结果（“观察”）返回给代理。
5. **综合：** 代理将工具的输出集成到其推理过程中，为用户生成最终的、有根据的答案。

## 应用场景

* **研究助理：** 使用网络搜索 API 回答需要最新信息的问题。
* **企业助理：** 查询公司内部数据库以回答诸如“上周有多少新用户注册？”之类的问题。
* **科学和数学任务：** 使用计算器或像 WolframAlpha 这样的计算引擎进行法学硕士经常遇到的精确计算。

## 优缺点

* **优势：**
* **事实依据：** 通过获取真实的实时数据，大大减少幻觉。
* **可扩展性：** 只需添加新工具即可不断扩展代理的功能。
* **弱点：**
* **集成开销：** 需要仔细的“管道”来定义工具、处理 API 密钥并管理潜在的工具故障。
* **工具信任：** 代理答案的质量取决于其使用的工具的可靠性和准确性。代理必须相信其工具提供正确的信息。

## 核心代码示例

### 示例 1

```python
import os
import json
from typing import List, Annotated, TypedDict, Optional
from dotenv import load_dotenv

# LangChain components
from langchain_nebius import ChatNebius
from langchain_community.tools.tavily_search import TavilySearchResults
from langchain_core.messages import BaseMessage, ToolMessage
from pydantic import BaseModel, Field

# LangGraph components
from langgraph.graph import StateGraph, END
from langgraph.graph.message import AnyMessage, add_messages
from langgraph.prebuilt import ToolNode

# For pretty printing
from rich.console import Console
from rich.markdown import Markdown

# --- API Key and Tracing Setup ---
load_dotenv()

os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_PROJECT"] = "Agentic Architecture - Tool Use (Nebius)"

# Check that the keys are set
for key in ["NEBIUS_API_KEY", "LANGCHAIN_API_KEY", "TAVILY_API_KEY"]:
    if not os.environ.get(key):
        print(f"{key} not found. Please create a .env file and set it.")

print("Environment variables loaded and tracing is set up.")
```

### 示例 2

```python
# Initialize the tool. We can set the max number of results to keep the context concise.
search_tool = TavilySearchResults(max_results=2)

# It's crucial to give the tool a clear name and description for the agent
search_tool.name = "web_search"
search_tool.description = "A tool that can be used to search the internet for up-to-date information on any topic, including news, events, and current affairs."

tools = [search_tool]
print(f"Tool '{search_tool.name}' created with description: '{search_tool.description}'")

console = Console()

# Let's test the tool directly to see its output format
print("\n--- Testing the tool directly ---")
test_query = "What was the score of the last Super Bowl?"
test_result = search_tool.invoke({"query": test_query})
console.print(f"[bold green]Query:[/bold green] {test_query}")
console.print("\n[bold green]Result:[/bold green]")
console.print(test_result)
```

---

## 结论

在此笔记本中，我们基于**工具使用**架构构建了一个完整的功能代理。我们成功地为 Nebius 支持的 LLM 配备了网络搜索工具，并使用 LangGraph 创建了一个强大的推理循环，允许代理决定何时以及如何使用它。

端到端的执行和随后的评估证明了这种模式的巨大价值。通过将我们的代理连接到实时的外部信息，我们从根本上克服了静态训练数据的限制。代理人不再只是一个推理者；它是一位研究人员，能够提供有根据的、事实的和最新的答案。该架构是创建几乎任何实用的现实世界人工智能助手的基础构建块。

