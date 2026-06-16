---
tags: agent-architecture
aliases: [集成, Ensemble]
创建日期: 2026-05-19
来源: 13_ensemble.ipynb
---

# 13 集成 (Ensemble)

> 多代理并行分析后聚合结论
> 关键应用: 高风险决策支持、事实核查

---

## 定义

**并行探索+集成决策**是一种代理架构，其中问题由多个独立代理或推理路径同时处理。然后，通常由单独的代理通过投票、建立共识或综合等方法汇总各个输出，以得出最终的、更可靠的结论。

## 工作流程

1. **扇出（并行探索）：** 用户的查询被分发给 N 个独立的专家代理。至关重要的是，这些代理通常会获得不同的指示、角色或工具，以鼓励采用不同的分析方法。
2. **独立处理：** 每个智能体独立处理问题，生成自己的完整分析、结论或答案。
3. **扇入（聚合）：** 收集所有 N 个代理的输出。
4. **综合（整体决策）：** 最终的“聚合器”或“判断”代理接收所有单独的输出。它的任务是分析这些观点，找出共同点，权衡相互矛盾的证据，并综合一个全面的最终答案。

## 应用场景

* **硬推理问答：** 对于复杂、模糊的问题，单条推理可能很容易错过细微差别（例如，“2008 年金融危机的主要原因是什么？”）。
* **事实检查和验证：** 让多个代理从不同来源搜索和验证事实可以大大减少幻觉。
* **高风险决策支持：** 在医学或金融等领域，在提出建议之前从不同的人工智能角色获得“第二意见”（或第三或第四）。

## 优缺点

* **优势：**
* **提高可靠性和准确性：** 平均掉单个代理的随机错误或偏差，使最终答案更有可能是正确且全面的。
* **减少幻觉：** 如果一个智能体产生了幻觉，其他智能体就不太可能做同样的事情，并且聚合器可以轻松发现异常值。
* **弱点：**
* **成本非常高：** 这是最昂贵的架构之一，因为它将 LLM 调用的数量乘以集合中代理的数量（加上最终的聚合调用）。
* **延迟增加：** 系统必须等待所有并行路径完成才能开始最终合成。

## 核心代码示例

### 示例 1

```python
import os
from typing import List, Dict, Any, Optional
from dotenv import load_dotenv

# Pydantic for data modeling
from pydantic import BaseModel, Field

# LangChain components
from langchain_nebius import ChatNebius
from langchain_tavily import TavilySearch
from langchain_core.prompts import ChatPromptTemplate

# LangGraph components
from langgraph.graph import StateGraph, END
from typing_extensions import TypedDict

# For pretty printing
from rich.console import Console
from rich.markdown import Markdown
from rich.panel import Panel

# --- API Key and Tracing Setup ---
load_dotenv()

os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_PROJECT"] = "Agentic Architecture - Parallel Ensemble (Nebius)"

required_vars = ["NEBIUS_API_KEY", "LANGCHAIN_API_KEY", "TAVILY_API_KEY"]
for var in required_vars:
    if var not in os.environ:
        print(f"Warning: Environment variable {var} not set.")

print("Environment variables loaded and tracing is set up.")
```

### 示例 2

```python
console = Console()
# A powerful model is needed for this complex task
llm = ChatNebius(model="mistralai/Mixtral-8x22B-Instruct-v0.1", temperature=0.3)
search_tool = TavilySearch(max_results=5)

# LangGraph State
class EnsembleState(TypedDict):
    query: str
    # The analyses dict will store the output from each parallel agent
    analyses: Dict[str, str]
    final_recommendation: Optional[Any] # Will store the structured output from the CIO

# Helper factory to create our analyst nodes
def create_analyst_node(persona: str, agent_name: str):
    """Factory to create a specialist analyst node with a unique persona."""
    system_prompt = f"You are an expert financial analyst. Your persona is '{persona}'. You must use your search tool to gather up-to-date information. Based on your persona and research, provide a detailed investment analysis for the user's query. Conclude with a clear 'Recommendation' (e.g., Buy, Hold, Sell) and a 'Confidence Score' (1-10)."
    
    prompt = ChatPromptTemplate.from_messages([
        ("system", system_prompt),
        ("human", "{query}")
    ])
    chain = prompt | llm.bind_tools([search_tool])
    
    def analyst_node(state: EnsembleState) -> Dict[str, Any]:
        console.print(f"--- 👨‍💻 Calling {agent_name} --- ")
        result = chain.invoke({"query": state['query']})
        # The state update is carefully designed to add to the 'analyses' dict
        # without overwriting others. This is key for parallel execution.
        current_analyses = state.get('analyses', {})
        current_analyses[agent_name] = result.content
        return {"analyses": current_analyses}
    
    return analyst_node

# 1. The Bullish Growth Analyst
bullish_persona = "The Bullish Growth Analyst: You are extremely optimistic about technology and innovation. You focus on Total Addressable Market (TAM), visionary leadership, technological moats, and future growth potential. Downplay short-term volatility and valuation concerns in favor of the long-term disruptive story."
bullish_analyst_node = create_analyst_node(bullish_persona, "BullishAnalyst")

# 2. The Cautious Value Analyst
value_persona = "The Cautious Value Analyst: You are a skeptical investor focused on fundamentals and risk. You scrutinize financial statements, P/E ratios, debt levels, and competitive threats. You are wary of hype and market bubbles. Highlight potential risks, downside scenarios, and reasons for caution."
value_analyst_node = create_analyst_node(value_persona, "ValueAnalyst")

# 3. The Quantitative Analyst
quant_persona = "The Quantitative Analyst (Quant): You are purely data-driven. You ignore narratives and focus on hard numbers. Report on key financial metrics (YoY revenue growth, EPS, margins), valuation multiples (P/E, P/S), and technical indicators (RSI, moving averages). Your analysis must be objective and based on the data you find."
quant_analyst_node = create_analyst_node(quant_persona, "QuantAnalyst")

print("Specialist analyst agents defined successfully.")
```

---

## 控制流视角（agno 实现）

> 来自 [[Agent架构演化总览]] 的控制流分析框架

### 六问拆解

| 问题 | 回答 |
|---|---|
| 它要解决什么问题？ | 前面的多 agent 基本都在解决"分工"问题。Ensemble 解决的是：**同一个问题，单个 agent 的结论不够可靠** |
| 它的 State 是什么？ | `views` dict——各分支并发写入各自的视角，aggregator 融合后输出结构化的 `FinalRecommendation` |
| 它的拓扑是什么？ | fan-out / fan-in：多个分析师并行 → CIO 融合 |
| 它的 Router 怎么工作？ | 没有 router——所有分析师同时收到同一问题，结果自然汇聚 |
| 它的失败模式是什么？ | 成本线性增长、多 agent 可能共享同样偏见、aggregator 可能强行合并不该合并的冲突、"综合意见"可能掩盖关键分歧 |
| 什么时候用？ | 高风险判断、事实核查、投资建议——凡是不想把决定押在一次输出上的场景 |

![[07-Attachments/all-agentic-architectures/Ensemble拓扑.png]]

### 它和 Multi-Agent 到底差在哪？

- **Multi-Agent**：不同 agent 做不同子任务 → 分工
- **Ensemble**：不同 agent 分析同一个问题 → 冗余

### 它真正新增了什么能力？

**用多视角和冗余来降低单次推理偏差。** ensemble 的关键不是"取平均"，而是**保留冲突信息并解释冲突**——这就是为什么 `FinalRecommendation` 里必须有 `identified_risks`。

### agno 实现

```python
from agno.workflow.v2 import Workflow, Step, Parallel

ensemble_wf = Workflow(
    name="ensemble",
    session_state={"views": {}},
    steps=[
        Parallel(
            name="analysts",
            steps=[
                Step(name="bullish", executor=run_one(bullish)),
                Step(name="value", executor=run_one(value)),
                Step(name="quant", executor=run_one(quant)),
            ],
        ),
        Step(name="cio_synth", executor=aggregate_step),
    ],
)
```

`Parallel` 做 fan-out，多个 agent 并发执行；aggregator step 做 fan-in，结构化输出确保冲突不被掩盖。

---

## 结论

在本笔记本中，我们实现了一个全面且复杂的**并行探索+集成决策**代理。通过模拟由不同专家和最终决策者组成的委员会，我们建立了一个擅长解​​决模糊、高风险问题的系统。

核心原则——**产生多样化、独立的推理者**，然后**综合其输出**——创建一个强大的机制，以减轻偏见、减少错误和增加分析深度。虽然这是计算成本最高的代理架构之一，但它提供稳健、可靠和细致入微的结论的能力使其成为任何最终决策的质量和可信度至关重要的应用程序不可或缺的工具。
