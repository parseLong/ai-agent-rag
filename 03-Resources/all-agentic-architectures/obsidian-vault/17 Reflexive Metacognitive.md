---
tags: agent-architecture
aliases: [反射性元认知, Reflexive Metacognitive]
创建日期: 2026-05-19
来源: 17_reflexive_metacognitive.ipynb
---

# 17 反射性元认知 (Reflexive Metacognitive)

> 代理自我建模，选择行动或升级
> 关键应用: 高风险咨询（医疗、法律、金融）

---

## 定义

**反射元认知代理**是一个维护并使用其自身能力、知识边界和置信水平的显式模型来为给定任务选择最合适策略的代理。这种自我建模使其能够更加安全可靠地运行，特别是在错误信息有害的领域。

## 工作流程

1. **感知任务：**代理接收用户请求。
2. **元认知分析（自我反思）：** 代理的核心推理引擎*针对其自己的自我模型*分析请求。它评估其置信度、工具的相关性以及查询是否属于其预定义的操作域。
3. **策略选择：** 根据分析，代理选择以下几种策略之一：
* **直接推理：** 用于其知识库内的高置信度、低风险查询。
* **使用工具：** 当查询需要代理通过工具拥有的特定功能时。
* **升级/拒绝：** 对于低置信度、高风险或超出范围的查询。
4. **执行策略：** 执行选择的路径。
5. **回复：** 代理提供结果，可以是直接答案、工具增强答案或带有咨询专家指示的安全拒绝。

## 应用场景

* **高风险咨询系统：** 提供医疗保健、法律或金融等领域信息的任何系统，其中代理必须能够说“我不知道”或“您应该咨询专业人士”。
* **自主系统：** 机器人在尝试之前必须评估其自身安全执行物理任务的能力。
* **复杂工具协调器：** 代理必须从庞大的库中选择正确的 API，并了解某些 API 比其他 API 更危险或成本更高。

## 优缺点

* **优势：**
* **增强安全性和可靠性：** 主要好处。该代理被明确设计为避免在它不是专家的领域做出自信的断言。
* **改进决策：** 通过强制深思熟虑地选择策略而不是天真的直接尝试，导致更稳健的行为。
* **弱点：**
* **自我模型的复杂性：** 定义和维护准确的自我模型可能很复杂。
* **元认知开销：** 初始分析步骤会增加每个请求的延迟和计算成本。

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
os.environ["LANGCHAIN_PROJECT"] = "Agentic Architecture - Metacognitive Agent (Nebius)"

required_vars = ["NEBIUS_API_KEY", "LANGCHAIN_API_KEY"]
for var in required_vars:
    if var not in os.environ:
        print(f"Warning: Environment variable {var} not set.")

print("Environment variables loaded and tracing is set up.")
```

### 示例 2

```python
console = Console()

# --- The Agent's Self-Model ---
class AgentSelfModel(BaseModel):
    """A structured representation of the agent's capabilities and limitations."""
    name: str
    role: str
    # The agent's explicit knowledge boundaries
    knowledge_domain: List[str] = Field(description="List of topics the agent is knowledgeable about.")
    # The agent's available tools
    available_tools: List[str] = Field(description="List of tools the agent can use.")
    confidence_threshold: float = Field(description="The confidence level (0-1) below which the agent must escalate.", default=0.6)

# Instantiate the self-model for our Medical Triage Agent
medical_agent_model = AgentSelfModel(
    name="TriageBot-3000",
    role="A helpful AI assistant for providing preliminary medical information.",
    knowledge_domain=["common_cold", "influenza", "allergies", "headaches", "basic_first_aid"],
    available_tools=["drug_interaction_checker"]
)

# --- Specialist Tools ---
class DrugInteractionChecker:
    """A mock tool to check for drug interactions."""
    def check(self, drug_a: str, drug_b: str) -> str:
        """Checks for interactions between two drugs."""
        # In a real system, this would query a medical database.
        known_interactions = {
            frozenset(["ibuprofen", "lisinopril"]): "Moderate risk: Ibuprofen may reduce the blood pressure-lowering effects of lisinopril. Monitor blood pressure.",
            frozenset(["aspirin", "warfarin"]): "High risk: Increased risk of bleeding. This combination should be avoided unless directed by a doctor."
        }
        interaction = known_interactions.get(frozenset([drug_a.lower(), drug_b.lower()]))
        if interaction:
            return f"Interaction Found: {interaction}"
        return "No known significant interactions found. However, always consult a pharmacist or doctor."

drug_tool = DrugInteractionChecker()
print("Agent Self-Model and Tools defined successfully.")
```

---

## 控制流视角（agno 实现）

> 来自 [[Agent架构演化总览]] 的控制流分析框架

### 六问拆解

| 问题 | 回答 |
|---|---|
| 它要解决什么问题？ | 前面的系统都在问"怎么把任务做完"，Metacognitive 先问：**这个任务我到底该不该做？** |
| 它的 State 是什么？ | `MetacognitiveAnalysis`（confidence + strategy + reasoning + tool_to_use），维护一份 AGENT_SELF_MODEL（knowledge_domains / tools_available / confidence_threshold / high_risk_topics） |
| 它的拓扑是什么？ | Self-model → Router → (reason_directly | use_tool | escalate)，三叉路由 |
| 它的 Router 怎么工作？ | 根据置信度 + 风险级别选择策略：≥阈值且低风险 → reason_directly；有匹配工具 → use_tool；否则 → escalate |
| 它的失败模式是什么？ | 置信度估计不准——低估自己会过度保守，高估自己会在高风险场景下危险地自信 |
| 什么时候搭配使用？ | 在医疗、法律、金融这种领域，agent 最强的能力不是"回答"，而是"拒绝"。最好与 [[14 Dry-Run Harness]] 一起出现 |

![[07-Attachments/all-agentic-architectures/Metacognitive拓扑.png]]

### 它真正新增了什么能力？

**边界感知。** 系统开始知道：我知道什么、不知道什么、需要什么工具、什么时候该把问题交给人类。

### agno 实现

```python
class MetacognitiveAnalysis(BaseModel):
    confidence: float = Field(description="0.0～1.0, confidence in safely answering.")
    strategy: str = Field(description="'reason_directly' | 'use_tool' | 'escalate'.")
    reasoning: str
    tool_to_use: Optional[str] = None

AGENT_SELF_MODEL = {
    "knowledge_domains": ["general health", "nutrition", "exercise"],
    "tools_available": ["symptom_checker"],
    "confidence_threshold": 0.7,
    "high_risk_topics": ["prescription dosage", "emergency medical advice"],
}

self_model_agent = Agent(name="self_model", model=OpenAIChat(id="gpt-5-mini"),
    response_model=MetacognitiveAnalysis,
    instructions=f"Your self-model is: {AGENT_SELF_MODEL}. Estimate confidence and pick a strategy.")

def meta_router(si):
    analysis = si.workflow_session_state["analysis"]
    if analysis.strategy == "reason_directly":
        return [Step(name="answer", executor=lambda s: responder.run(s.message).content)]
    if analysis.strategy == "use_tool":
        return [Step(name="tool_answer", executor=lambda s: responder.run(f"Use the {analysis.tool_to_use} tool to help answer: {s.message}").content)]
    return [Step(name="escalate", executor=lambda s: "I'm not confident or allowed to answer this. Escalating to a human expert.")]

metacog_wf = Workflow(
    name="metacognitive",
    steps=[
        Step(name="self_model", executor=meta_step),
        Router(name="route_strategy", selector=meta_router),
    ],
)
```

关键设计：`response_model=MetacognitiveAnalysis` 让边界感知结构化，`Router` 根据策略做三叉路由——系统不再只对外部世界建模，也开始对自身能力边界建模。

---

## 结论

在这个详细的笔记本中，我们实现了**反射元认知代理**，这是一种复杂的架构，通过赋予代理自我意识来优先考虑安全性和可靠性。通过建立一个明确的`self-model`并强制将元认知分析作为任何任务的第一步，我们创建了一个能够理解自身边界的系统。

关键的创新是代理的最初目标从“我如何回答这个问题？”的转变。到“*我应该*回答这个问题吗？如果是，怎么回答？”这种内省步骤允许代理动态选择最安全、最合适的策略——无论是直接推理、专门工具的使用，还是向人类专家进行关键升级。

这种架构不仅仅是一种技术；更是一种技术。这是一种设计理念。这对于创建负责任的人工智能代理来说是绝对必要的，这些代理可以被信任在高风险的现实世界领域中运行，在这些领域中，了解你“不”知道的事情与你所做的事情一样重要。