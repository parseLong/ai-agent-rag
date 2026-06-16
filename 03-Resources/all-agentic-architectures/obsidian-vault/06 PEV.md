---
tags: agent-architecture
aliases: [计划执行验证, PEV]
创建日期: 2026-05-19
来源: 06_PEV.ipynb
---

# 06 计划执行验证 (PEV)

> 自校正循环，验证每一步结果
> 关键应用: 高风险自动化、金融、不可靠工具

---

## 定义

**规划器 → 执行器 → 验证器 (PEV)** 架构是一个三阶段工作流程，明确分离了规划、执行和验证的行为。它确保在代理继续之前验证每个步骤的输出，从而创建一个强大的自我纠正循环。

## 工作流程

1. **计划：** “计划者”代理将高级目标分解为一系列具体的可执行步骤。
2. **执行：** “执行者”代理从计划中执行*下一步*并调用适当的工具。
3. **验证：** “验证者”代理检查执行器的输出。它检查正确性、相关性和潜在错误。然后它会产生一个判断：该步骤成功还是失败？
4. **路由和迭代：** 根据验证者的判断，路由器决定下一步行动：
* 如果步骤**成功**并且计划未完成，则循环回执行器进行下一步。
* 如果步骤**失败**，则循环回规划器以创建*新*计划，通常提供失败上下文，以便新计划可以更智能。
* 如果步骤**成功**并且计划完成，则继续进行最终综合步骤。

## 应用场景

* **安全关键型应用（金融、医疗保健）：** 当错误成本很高时，PEV 提供必要的护栏来防止代理对错误数据采取行动。
* **具有不可靠工具的系统：** 在处理可能不稳定或返回不一致数据的外部 API 时，验证器可以优雅地捕获故障。
* **高精度任务（法律、科学）：** 对于需要高度事实准确性的任务，验证器确保每条检索到的信息在用于下游推理之前都是有效的。

## 优缺点

* **优势：**
* **稳健性和可靠性：** 其核心优势是检测错误并从错误中恢复的能力。
* **模块化：** 关注点分离使系统更易于调试和维护。
* **弱点：**
* **增加延迟和成本：** 每次操作后添加验证步骤都会增加更多的 LLM 调用，使其成为我们迄今为止介绍的最慢且最昂贵的架构。
* **验证器复杂性：** 设计有效的验证器可能具有挑战性。它需要足够智能来区分小问题和严重故障。

## 核心代码示例

### 示例 1

```python
import os
import re
from typing import List, Annotated, TypedDict, Optional
from dotenv import load_dotenv
import json

# LangChain components
from langchain_nebius import ChatNebius
from langchain_tavily import TavilySearch
from langchain_core.messages import BaseMessage, ToolMessage
from pydantic import BaseModel, Field

# LangGraph components
from langgraph.graph import StateGraph, END

# For pretty printing
from rich.console import Console
from rich.markdown import Markdown

# --- API Key and Tracing Setup ---
load_dotenv()

os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_PROJECT"] = "Agentic Architecture - PEV (Nebius)"

for key in ["NEBIUS_API_KEY", "LANGCHAIN_API_KEY", "TAVILY_API_KEY"]:
    if not os.environ.get(key):
        print(f"{key} not found. Please create a .env file and set it.")

print("Environment variables loaded and tracing is set up.")
```

### 示例 2

```python
console = Console()
llm = ChatNebius(model="meta-llama/Meta-Llama-3.1-8B-Instruct", temperature=0)

# Define a 'flaky' tool that will fail for a specific query
def flaky_web_search(query: str) -> str:
    """Performs a web search, but is designed to fail for a specific query."""
    console.print(f"--- TOOL: Searching for '{query}'... ---")
    if "employee count" in query.lower():
        console.print("--- TOOL: [bold red]Simulating API failure![/bold red] ---")
        return "Error: Could not retrieve data. The API endpoint is currently unavailable."
    else:
        result = TavilySearch(max_results=2).invoke(query)
        # 🔑 Ensure result is always a string
        if isinstance(result, (dict, list)):
            return json.dumps(result, indent=2)
        return str(result)

# Define the state for the basic P-E agent
class BasicPEState(TypedDict):
    user_request: str
    plan: Optional[List[str]]
    intermediate_steps: List[str]
    final_answer: Optional[str]

class Plan(BaseModel):
    steps: List[str] = Field(description="A list of tool calls to execute.")

def basic_planner_node(state: BasicPEState):
    console.print("--- (Basic) PLANNER: Creating plan... ---")
    planner_llm = llm.with_structured_output(Plan)

    prompt = f"""
    You are a planning agent. 
    Your job is to decompose the user's request into a list of clear tool queries.

    - Only return JSON that matches this schema: {{ "steps": [ "query1", "query2", ... ] }}
    - Do NOT return any prose or explanation.
    - Always use the 'flaky_web_search' tool for queries.

    User's request: "{state['user_request']}"
    """
    plan = planner_llm.invoke(prompt)
    return {"plan": plan.steps}

def basic_executor_node(state: BasicPEState):
    console.print("--- (Basic) EXECUTOR: Running next step... ---")
    next_step = state["plan"][0]
    result = flaky_web_search(next_step)
    return {"plan": state["plan"][1:], "intermediate_steps": state["intermediate_steps"] + [result]}

def basic_synthesizer_node(state: BasicPEState):
    console.print("--- (Basic) SYNTHESIZER: Generating final answer... ---")
    context = "\n".join(state["intermediate_steps"])
    prompt = f"Synthesize an answer for '{state['user_request']}' using this data:\n{context}"
    answer = llm.invoke(prompt).content
    return {"final_answer": answer}

# Build the graph
pe_graph_builder = StateGraph(BasicPEState)
pe_graph_builder.add_node("plan", basic_planner_node)
pe_graph_builder.add_node("execute", basic_executor_node)
pe_graph_builder.add_node("synthesize", basic_synthesizer_node)

pe_graph_builder.set_entry_point("plan")
pe_graph_builder.add_conditional_edges("plan", lambda s: "execute" if s["plan"] else "synthesize")
pe_graph_builder.add_conditional_edges("execute", lambda s: "execute" if s["plan"] else "synthesize")
pe_graph_builder.add_edge("synthesize", END)

basic_pe_app = pe_graph_builder.compile()
print("Basic Planner-Executor agent compiled successfully.")
```

---

## 控制流视角（agno 实现）

> 来自 [[Agent架构演化总览]] 的控制流分析框架

### 六问拆解

| 问题 | 回答 |
|---|---|
| 它要解决什么问题？ | Planning 默认世界是稳定的，但真实世界不是。PEV 要**不要把执行结果默认为真，而要显式验证** |
| 它的 State 是什么？ | 新增 `VerificationResult`（is_successful + reasoning），图结构从 `plan → execute → next step` 变成 `plan → execute → verify → (continue | replan | finish)` |
| 它的拓扑是什么？ | Plan → Loop(Execute → Verify → Router) → Synthesize，Router 可重定向到 replan |
| 它的 Router 怎么工作？ | `pev_router`：根据 verdict 决定——成功继续、失败重试、多次失败则 replan |
| 它的失败模式是什么？ | 额外成本高、verifier 本身可能误判、过度验证会拖慢系统、某些任务里"验证"本身比"执行"更难 |
| 什么时候该升级？ | 如果问题不是"执行步骤会不会失败"，而是"一个 agent 不该包揽所有认知角色" → [[05 Multi-Agent Systems]] |

![[07-Attachments/all-agentic-architectures/PEV拓扑.png]]

### 它真正带来的系统收益

PEV 的本质是：**让错误不再静默传播。** 在普通 Planning 里，一次失败可能污染整个剩余流程；在 PEV 里，失败在局部被识别，并被重定向回 re-plan 路径。

### agno 实现

```python
from agno.workflow.v2 import Router

class VerificationResult(BaseModel):
    is_successful: bool = Field(description="True if the tool execution was successful and the data is valid.")
    reasoning: str = Field(description="Reasoning for the verification decision.")

verifier = Agent(name="verifier", model=OpenAIChat(id="gpt-5-mini"),
    response_model=VerificationResult,
    instructions="Given a sub-question and the raw tool observation, decide if the observation actually answers the sub-question.")

def pev_router(step_input):
    state = step_input.workflow_session_state
    if not state["plan"]:
        return [noop_step]
    if not state["last_verdict"].is_successful and state["retries"] >= 2:
        state["retries"] = 0
        return [replan_step]
    return [noop_step]

pev_wf = Workflow(
    name="pev",
    session_state={"plan": [], "intermediate": [], "retries": 0,
                   "last_q": "", "last_obs": "", "last_verdict": None},
    steps=[
        Step(name="plan", executor=plan_step),
        Loop(
            name="pev_loop",
            steps=[
                Step(name="pev_execute", executor=pev_execute),
                Step(name="pev_verify", executor=pev_verify),
                Router(name="decide_next", selector=pev_router),
            ],
            end_condition=pev_loop_done,
        ),
        Step(name="synthesize", executor=synth_step),
    ],
)
```

关键设计：`Router` 根据结构化 `VerificationResult` 做条件路由，这是验证被提升为控制流一等公民的具体体现。

---

## 结论

在此笔记本中，我们实现了 **Planner → Executor → Verifier** 架构，并展示了与简单的 Planner-Executor 模型相比其卓越的鲁棒性。通过引入专用的验证器节点，我们为代理提供了一个关键的“免疫系统”，可以检测并从可能对任务造成致命影响的故障中恢复。

这种模式更加耗费资源，但对于可靠性和准确性至关重要的应用程序来说，权衡是至关重要的。PEV 架构代表着朝着构建真正可靠的 AI 代理迈出的重要一步，这些代理可以在外部工具和 API 的不可预测的现实环境中安全有效地运行。

