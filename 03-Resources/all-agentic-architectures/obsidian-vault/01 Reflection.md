---
tags: agent-architecture
aliases: [反射, Reflection]
创建日期: 2026-05-19
来源: 01_reflection.ipynb
---

# 01 反射 (Reflection)

> 通过自我批评和修正提升输出质量
> 关键应用: 高质量代码生成、复杂摘要

---

## 定义

**反射**架构涉及代理在返回最终答案之前批评和修改自己的输出。它不是单遍生成，而是进行多步骤的内部独白：生产、评估和改进。这模仿了人类起草、审查和编辑的过程，以发现错误并提高质量。

## 工作流程

1. **生成：** 代理根据用户的提示生成初始草案或解决方案。
2. **批评：** 然后代理转换角色成为批评者。它会问自己这样的问题：*“这个答案可能有什么问题？”*，*“缺少什么？”*，*“这个解决方案是最佳的吗？”*，或*“是否有任何逻辑缺陷或错误？”*。
3. **优化：** 利用自我批评的见解，代理生成输出的最终改进版本。

## 应用场景

* **代码生成：** 初始代码可能存在错误、效率低下或缺少注释。反射允许代理充当自己的代码审查者，在呈现最终脚本之前捕获错误并改进风格。
* **复杂的总结：** 当总结密集的文档时，第一次可能会错过细微差别或省略关键细节。反思步骤有助于确保摘要全面且准确。
* **创意写作和内容创作：** 电子邮件、博客文章或故事的初稿始终可以改进。反思可以让代理改进其语气、清晰度和影响力。

## 优缺点

* **优势：**
* **提高质量：** 直接解决和纠正错误，从而产生更准确、稳健和合理的输出。
* **低开销：** 这是一个概念上简单的模式，可以使用单个 LLM 来实现，并且不需要复杂的外部工具。
* **弱点：**
* **自我偏见：** 代理仍然受到其自身知识和偏见的限制。如果它不知道解决问题的更好方法，它就无法批评其寻求更好解决方案的方法。它可以修复它认识到的缺陷，但无法发明它所缺乏的知识。
* **增加延迟和成本：** 该过程至少涉及两次 LLM 调用（生成 + 批评/细化），使其比单遍方法更慢且更昂贵。

## 控制流视角（agno 实现）

> 来自 [[Agent架构演化总览]] 的控制流分析框架

### 六问拆解

| 问题 | 回答 |
|---|---|
| 它要解决什么问题？ | 单次 LLM 生成质量不稳定。Reflection 就是最小修复：把单步生成改成三阶段控制流 |
| 它的 State 是什么？ | `draft / critique / refined_code`——系统第一次把”中间思考结果”显式写进 state |
| 它的拓扑是什么？ | 纯线性链：Generator → Critic → Refiner |
| 它的 Router 怎么工作？ | **没有 router。** 没有条件分支，没有失败恢复，系统默认三步走完直接结束 |
| 它的失败模式是什么？ | **不能验证 refiner 是否真的修好了 critic 提到的问题。** 有 critique，但没有闭环 |
| 什么时候该升级？ | 当你需要系统根据中间结果继续行动，就必须进入下一代：[[02 Tool Use]] 或 [[03 ReAct]] |

![[07-Attachments/all-agentic-architectures/Reflection拓扑.png]]

### 它真正新增了什么能力？

不是”模型会反思”，而是：**把单步生成拆成三个更明确的子任务**。核心经验判断是：LLM 作为 critic 往往比作为 generator 更稳定。

### agno 实现

```python
from pydantic import BaseModel, Field
from typing import List

class DraftCode(BaseModel):
    code: str = Field(description=”Python code to solve the user's request.”)
    explanation: str = Field(description=”A brief explanation of how the code works.”)

class Critique(BaseModel):
    has_errors: bool
    is_efficient: bool
    suggested_improvements: List[str]
    critique_summary: str

class RefinedCode(BaseModel):
    refined_code: str
    refinement_summary: str

generator = Agent(name=”generator”, model=OpenAIChat(id=”gpt-5-mini”),
    response_model=DraftCode,
    instructions=”You are an expert Python programmer. Write code and a brief explanation.”)

critic = Agent(name=”critic”, model=OpenAIChat(id=”gpt-5-mini”),
    response_model=Critique,
    instructions=”You are a senior code reviewer. Analyze for bugs, inefficiencies and PEP8 issues.”)

refiner = Agent(name=”refiner”, model=OpenAIChat(id=”gpt-5-mini”),
    response_model=RefinedCode,
    instructions=”Rewrite the code, incorporating every suggestion from the critique.”)

reflection_wf = Workflow(
    name=”reflection”,
    session_state={“draft”: None},
    steps=[
        Step(name=”generator_step”, executor=generator_step),
        Step(name=”critic_step”, executor=critic_step),
        Step(name=”refiner_step”, executor=refiner_step),
    ],
)
```

关键设计：`response_model` 让中间结果结构化，`workflow_session_state` 传递跨步骤数据，`previous_step_output.content` 自动传递相邻步骤数据。

---

## 结论

在此笔记本中，我们使用 **Reflection** 架构和 Nebius AI Studio 模型成功构建、执行和评估了一个完整的端到端代理。我们亲眼目睹了这种简单而强大的模式如何将基本的 LLM 生成器转变为更复杂、更可靠的问题解决器。

通过将流程构建为不同的”生成”、”批判”和”优化”步骤并使用 LangGraph 进行编排，我们创建了一个强大的系统，可以识别并纠正其自身的重大缺陷。从低效的递归解决方案到最佳迭代解决方案的切实改进表明，反射是超越琐碎代理任务并构建具有更深层次质量和深思熟虑的人工智能系统的基础技术。

---
