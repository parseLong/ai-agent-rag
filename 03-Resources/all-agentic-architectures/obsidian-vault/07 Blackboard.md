---
tags: agent-architecture
aliases: [黑板系统, Blackboard]
创建日期: 2026-05-19
来源: 07_blackboard.ipynb
---

# 07 黑板系统 (Blackboard)

> 通过共享中央黑板动态协作
> 关键应用: 复杂诊断、动态感知

---

## 定义

**Blackboard System** 是一种多代理架构，其中多个专业代理通过读取和写入称为“黑板”的共享中央数据存储库进行协作。控制器或调度程序根据黑板上解决方案的发展状态动态确定下一步应该采取行动的代理。

## 工作流程

1. **共享内存（黑板）：** 中央数据结构保存问题的当前状态，包括用户的请求、中间发现和部分解决方案。
2. **专家代理：** 一群独立代理，每个人都具有特定的专业知识，持续监控黑板。
3. **控制器：** 中央“控制器”代理还监视黑板。它的工作是分析当前状态并决定哪个专业代理最适合做出下一个贡献。
4. **机会激活：** 控制器激活所选代理。代理从黑板上读取相关数据，执行其任务，并将其结果写回黑板。
5. **迭代：** 该过程重复进行，控制器以动态顺序激活不同的代理，直到确定黑板上的解决方案已完成。

## 应用场景

* **复杂、结构不良的问题：** 非常适合提前未知解决路径且需要紧急、机会主义策略（例如复杂的诊断、科学发现）的问题。
* **多模式系统：** 协调处理不同数据类型（文本、图像、代码）的代理的好方法，因为他们都可以将他们的发现发布到共享黑板上。
* **动态意义构建：** 需要从许多不同的异步源合成信息的情况。

## 优缺点

* **优势：**
* **灵活性和适应性：** 工作流程不是硬编码的；它是根据问题而出现的，使得系统具有很强的适应性。
* **模块化：** 添加或删除专业代理非常容易，无需重新构建整个系统。
* **弱点：**
* **控制器复杂度：** 整个系统的智能很大程度上取决于控制器的复杂程度。幼稚的控制器可能会导致低效或循环行为。
* **调试挑战：** 工作流的非线性、突发性有时会使跟踪和调试比简单的顺序过程更困难。

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

# For pretty printing
from rich.console import Console
from rich.markdown import Markdown

# --- API Key and Tracing Setup ---
load_dotenv()

os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_PROJECT"] = "Agentic Architecture - Blackboard (Nebius)"

for key in ["NEBIUS_API_KEY", "LANGCHAIN_API_KEY", "TAVILY_API_KEY"]:
    if not os.environ.get(key):
        print(f"{key} not found. Please create a .env file and set it.")

print("Environment variables loaded and tracing is set up.")
```

### 示例 2

```python
console = Console()
# Using a more capable model to handle complex instructions better
llm = ChatNebius(model="mistralai/Mixtral-8x22B-Instruct-v0.1", temperature=0)
search_tool = TavilySearch(max_results=2)

# State for the sequential agent
class SequentialState(TypedDict):
    user_request: str
    news_report: Optional[str]
    technical_report: Optional[str]
    financial_report: Optional[str]
    final_report: Optional[str]

# --- CORRECTED SPECIALIST NODES FOR SEQUENTIAL AGENT ---
# The key change is that each agent now gets context from previous steps, not just the original request.

def news_analyst_node_seq(state: SequentialState):
    console.print("--- (Sequential) CALLING NEWS ANALYST ---")
    prompt = f"Your task is to act as an expert News Analyst. Find the latest major news about the topic in the user's request and provide a concise summary.\n\nUser Request: {state['user_request']}"
    agent = llm.bind_tools([search_tool])
    result = agent.invoke(prompt)
    return {"news_report": result.content}

def technical_analyst_node_seq(state: SequentialState):
    console.print("--- (Sequential) CALLING TECHNICAL ANALYST ---")
    # This agent now uses the news report as context.
    prompt = f"Your task is to act as an expert Technical Analyst. Based on the following news report, conduct a technical analysis of the company's stock.\n\nNews Report:\n{state['news_report']}"
    agent = llm.bind_tools([search_tool])
    result = agent.invoke(prompt)
    return {"technical_report": result.content}

def financial_analyst_node_seq(state: SequentialState):
    console.print("--- (Sequential) CALLING FINANCIAL ANALYST ---")
    # This agent also uses the news report as context.
    prompt = f"Your task is to act as an expert Financial Analyst. Based on the following news report, analyze the company's recent financial performance.\n\nNews Report:\n{state['news_report']}"
    agent = llm.bind_tools([search_tool])
    result = agent.invoke(prompt)
    return {"financial_report": result.content}


def report_writer_node_seq(state: SequentialState):
    console.print("--- (Sequential) CALLING REPORT WRITER ---")
    prompt = f"""You are an expert report writer. Your task is to synthesize the information from the News, Technical, and Financial analysts into a single, cohesive report that directly answers the user's original request.

User Request: {state['user_request']}

Here are the reports to combine:
---
News Report: {state['news_report']}
---
Technical Report: {state['technical_report']}
---
Financial Report: {state['financial_report']}
"""
    report = llm.invoke(prompt).content
    return {"final_report": report}

# Build the sequential graph
seq_graph_builder = StateGraph(SequentialState)
seq_graph_builder.add_node("news", news_analyst_node_seq)
seq_graph_builder.add_node("tech", technical_analyst_node_seq)
seq_graph_builder.add_node("finance", financial_analyst_node_seq)
seq_graph_builder.add_node("writer", report_writer_node_seq)

# The rigid, hardcoded sequence
seq_graph_builder.set_entry_point("news")
seq_graph_builder.add_edge("news", "tech")
seq_graph_builder.add_edge("tech", "finance")
seq_graph_builder.add_edge("finance", "writer")
seq_graph_builder.add_edge("writer", END)

sequential_app = seq_graph_builder.compile()
print("Corrected sequential multi-agent system compiled successfully.")
```

---

## 结论

在此笔记本中，我们实现并纠正了 **Blackboard System**，展示了其相对于顺序多代理架构的显着优势。通过引入共享内存（黑板）和智能的、状态感知的**控制器**，我们创建了一个不仅具有协作性、而且具有自适应性和机会性的系统。

面对面的比较表明，对于具有条件逻辑的任务，Blackboard 系统能够在正确的时间选择正确的专家，从而实现更高效且逻辑合理的流程。虽然它需要更复杂的控制器，但这种架构是解决结构不良的现实世界问题的强大工具，而刚性的线性工作流程无法有效解决。

