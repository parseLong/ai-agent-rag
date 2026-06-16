---
tags: agent-architecture
aliases: [基于人类反馈的强化学习, RLHF]
创建日期: 2026-05-19
来源: 15_RLHF.ipynb
---

# 15 基于人类反馈的强化学习 (RLHF)

> 利用编辑反馈迭代改进并保存
> 关键应用: 高质量内容生成、持续学习
---

## 定义

**自我改进循环**是一种代理架构，其中代理的输出由其自身或另一个代理进行评估，并且此评估用作反馈以生成修订后的更高质量的输出。当这种反馈被存储并用于随着时间的推移提高代理的基准性能时，它就成为一种持续学习的形式。

## 工作流程

1. **生成初始输出：** 主要代理生成解决方案的第一个版本（“草案”）。
2. **批评输出：** 批评代理人（或“批评模式”中的主要代理人）根据一组预定义的标准或一般规则评估草案。
3. **决策：** 系统检查批评是否足够积极以接受输出。
4. **修订（循环）：** 如果输出不被接受，则原始草稿*和*评论家的反馈将被传递回主要代理，指示主要代理生成解决反馈的修订版本。
5. **接受：** 一旦输出满足质量标准，循环终止，并返回最终版本。

## 应用场景

* **高质量内容生成：** 适用于通用初稿不足以完成的任务，例如撰写法律文件、详细的技术报告或有说服力的营销文案。
* **持续学习和个性化：** 通过生成响应、获取隐式或显式反馈以及改进下一次交互的内部策略来学习用户偏好的代理。
* **解决复杂问题：** 代理可以提出计划，批评其缺陷或效率低下，然后在执行前修改计划。

## 优缺点

* **优势：**
* **显着提高输出质量：** 迭代细化始终能够产生比单次生成更好的结果。
* **实现持续学习：** 为代理提供一个框架，使其随着时间的推移变得更好，适应新信息或反馈。
* **弱点：**
* **强化偏见的风险：** 如果批评者的逻辑或偏见有缺陷，系统可能会陷入强化自身错误的循环中。
* **计算成本高昂：** 迭代性质意味着每个任务需要多次 LLM 调用，从而增加成本和延迟。

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
os.environ["LANGCHAIN_PROJECT"] = "Agentic Architecture - Self-Improvement Loop (Nebius)"

required_vars = ["NEBIUS_API_KEY", "LANGCHAIN_API_KEY"]
for var in required_vars:
    if var not in os.environ:
        print(f"Warning: Environment variable {var} not set.")

print("Environment variables loaded and tracing is set up.")
```

### 示例 2

```python
console = Console()
llm = ChatNebius(model="mistralai/Mixtral-8x22B-Instruct-v0.1", temperature=0.4)

# --- Pydantic Models for Structured Data ---
class MarketingEmail(BaseModel):
    """Represents a marketing email draft."""
    subject: str = Field(description="A catchy and concise subject line for the email.")
    body: str = Field(description="The full body text of the email, written in markdown.")

class Critique(BaseModel):
    """A structured critique of the marketing email draft."""
    score: int = Field(description="Overall quality score from 1 (poor) to 10 (excellent).")
    feedback_points: List[str] = Field(description="A bulleted list of specific, actionable feedback points for improvement.")
    is_approved: bool = Field(description="A boolean indicating if the draft is approved (score >= 8). This is redundant with the score but useful for routing.")

# --- 1. The Generator: Junior Copywriter ---
def get_generator_chain():
    prompt = ChatPromptTemplate.from_messages([
        ("system", "You are a junior marketing copywriter. Your task is to write a first draft of a marketing email based on the user's request. Be creative, but focus on getting the core message across."),
        ("human", "Write a marketing email about the following topic:\n\n{request}")
    ])
    return prompt | llm.with_structured_output(MarketingEmail)

# --- 2. The Critic: Senior Editor ---
def get_critic_chain():
    prompt = ChatPromptTemplate.from_messages([
        ("system", """You are a senior marketing editor and brand manager. Your job is to critique an email draft written by a junior copywriter. 
        Evaluate the draft against the following criteria:
        1.  **Catchy Subject:** Is the subject line engaging and likely to get opened?
        2.  **Clarity & Persuasiveness:** Is the body text clear, compelling, and persuasive?
        3.  **Strong Call-to-Action (CTA):** Is there a clear, single action for the user to take?
        4.  **Brand Voice:** Is the tone professional yet approachable?
        Provide a score from 1-10. A score of 8 or higher means the draft is approved for sending. Provide specific, actionable feedback to help the writer improve."""
        ),
        ("human", "Please critique the following email draft:\n\n**Subject:** {subject}\n\n**Body:**\n{body}")
    ])
    return prompt | llm.with_structured_output(Critique)

# --- 3. The Reviser (Generator in 'Revise' Mode) ---
def get_reviser_chain():
    prompt = ChatPromptTemplate.from_messages([
        ("system", "You are the junior marketing copywriter who wrote the original draft. You have just received feedback from your senior editor. Your task is to carefully revise your draft to address every single point of feedback. Produce a new, improved version of the email."),
        ("human", "Original Request: {request}\n\nHere is your original draft:\n**Subject:** {original_subject}\n**Body:**\n{original_body}\n\nHere is the feedback from your editor:\n{feedback}\n\nPlease provide the revised email.")
    ])
    return prompt | llm.with_structured_output(MarketingEmail)

print("Generator and Critic components defined successfully.")
```

---

## 控制流视角（agno 实现）

> 来自 [[Agent架构演化总览]] 的控制流分析框架

### 六问拆解

| 问题 | 回答 |
|---|---|
| 它要解决什么问题？ | Reflection 只有一次 critique pass。追求高质量输出需要：生成 → 评估 → 修订 → 再评估 → 不达标继续，即**进化回路** |
| 它的 State 是什么？ | `last_email / last_critique / revision`，以及跨任务的 `GoldStandardMemory`——批准过的结果注入未来生成 |
| 它的拓扑是什么？ | Loop(gen → critic) + 终止条件（critic 批准或达到最大修订次数） |
| 它的 Router 怎么工作？ | `Loop.end_condition` 天然支持：critic 批准 → 终止；revision ≥ 3 → 强制终止 |
| 它的失败模式是什么？ | critic 标准不稳、revision 收益递减、记忆库积累低质量样本会反向污染未来生成 |
| 什么时候该升级？ | Self-Improve 不是"会自动越来越好"，而是"有机会在严格约束下越来越好" |

### 它和 Reflection 的本质区别

两层能力：**单次任务内迭代优化** + **跨任务积累高质量样例**（GoldStandardMemory 把批准过的结果注入未来生成）。

### agno 实现

```python
class EmailCritique(BaseModel):
    is_approved: bool
    feedback: str

class MarketingEmail(BaseModel):
    subject: str
    body: str

# 跨任务：高质量样例库
class GoldStandardMemory:
    def __init__(self):
        self.examples: List[MarketingEmail] = []
    def few_shot_block(self) -> str:
        if not self.examples: return "No gold examples yet."
        return "\n\n---\n\n".join(f"Subject: {e.subject}\nBody:\n{e.body}" for e in self.examples[-3:])
    def add(self, e: MarketingEmail):
        self.examples.append(e)

def should_stop(_outputs) -> bool:
    """Loop 终止：critic 批准，或达到最大修订次数。"""
    state = self_improve_wf.session_state
    last = state.get("last_critique")
    if last is not None and last.is_approved: return True
    if state.get("revision", 0) >= 3: return True
    return False

self_improve_wf = Workflow(
    name="self_improve",
    session_state={"revision": 0},
    steps=[
        Loop(
            name="refine_loop",
            steps=[Step(name="gen", executor=gen_step), Step(name="critic", executor=critic_step)],
            end_condition=should_stop,
        ),
    ],
)
```

关键设计：`Loop.end_condition` 实现了终止条件，`GoldStandardMemory` 实现了跨任务样例积累——这是它和 Reflection 的本质区别。

---

## 结论

在本笔记本中，我们实现了全面且复杂的**自我改进循环**。我们已经证明，这种架构不仅仅是改进单个工作，而且是创建能够随着时间的推移真正学习和改进的代理的强大范例。

通过分离**生成者*​​和**评论家**的角色，我们创建了反馈和修订的动态，不断提高代理输出的质量。通过添加**持久内存**以获得高质量结果，我们创建了一个积极的反馈循环，可以提高代理的基线能力，使其在未来的任务中更加高效和有效。

虽然批评者强化自身偏见的风险是真实存在的，需要谨慎管理，但构建从经验中学习的智能体的潜力是迈向更自主、更强大、更智能的人工智能系统的变革性一步。

