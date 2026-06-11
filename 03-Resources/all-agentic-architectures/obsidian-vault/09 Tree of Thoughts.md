---
tags: agent-architecture
aliases: [思维树, Tree of Thoughts]
创建日期: 2026-05-19
来源: 09_tree_of_thoughts.ipynb
---

# 09 思维树 (Tree of Thoughts)

> 在树结构中探索多条推理路径
> 关键应用: 逻辑谜题、约束规划

---

## 定义

**思想树 (ToT)** 是一种代理推理框架，其中问题解决被建模为通过树进行搜索。代理同时探索多个推理路径（分支）。在每一步中，它都会生成潜在的后续步骤（“想法”），评估其可行性，并决定继续探索哪些路径，从而有效地修剪搜索空间。

## 工作流程

1. **分解：** 将问题分解为一系列步骤或想法。
2. **想法生成：** 对于问题的当前状态，代理生成多个潜在的后续步骤或想法。这会在搜索树中创建分支。
3. **状态评估：** 每个新想法（导致新状态）都由“批评家”或验证功能进行评估。该评估可以评估：
* **有效性：** 问题规则是否允许此举动？
* **进展：** 这一举措是否让我们更接近解决方案？
* **启发式：** 这条道路有可能成功吗？
4. **修剪和扩展：** 修剪无效或无前途的分支。然后，智能体从最有希望的活跃分支出发，重复思想生成过程。
5. **解决方案：** 该过程持续进行，直到达到目标状态。解决之道是从根源到目标的思想路径。

## 应用场景

* **逻辑谜题和数学问题：** 具有明确规则和目标状态，需要多步骤、非线性推理的问题（例如数独、过河谜题）。
* **复杂计划：** 当任务需要详细计划时，其中操作顺序很重要并且必须遵守限制（例如，计划具有多条航段和预算限制的复杂旅行）。
* **创意写作或代码生成：** 在致力于一个故事分支或实施策略之前探索多个故事分支或实施策略。

## 优缺点

* **优势：**
* **稳健性：** 系统地探索问题空间，与单遍方法相比，它不太可能陷入困境或产生错误答案。
* **处理组合复杂性：** 非常适合可能序列数量巨大的问题。
* **弱点：**
* **计算量大：** 与简单的思想链提示相比，需要更多的 LLM 调用和状态管理，从而使其更慢且更昂贵。
* **需要一个好的评估器：** 搜索的有效性在很大程度上取决于状态评估逻辑的质量。

## 核心代码示例

### 示例 1

```python
import os
import re
from typing import List, Dict, Any, Optional
from dotenv import load_dotenv
from collections import defaultdict

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
from rich.tree import Tree

# --- API Key and Tracing Setup ---
load_dotenv()

os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_PROJECT"] = "Agentic Architecture - Tree-of-Thoughts (Nebius)"

# Check for required environment variables
required_vars = ["NEBIUS_API_KEY", "LANGCHAIN_API_KEY"]
for var in required_vars:
    if var not in os.environ:
        print(f"Warning: Environment variable {var} not set.")

print("Environment variables loaded and tracing is set up.")
```

### 示例 2

```python
console = Console()

class PuzzleState(BaseModel):
    "Represents the state of the Wolf, Goat, and Cabbage puzzle."
    # Using sets for unordered collections of items on each bank
    left_bank: set[str] = Field(default_factory=lambda: {"wolf", "goat", "cabbage"})
    right_bank: set[str] = Field(default_factory=set)
    boat_location: str = "left"
    move_description: str = "Initial state."

    def is_valid(self) -> bool:
        """Checks if the current state is valid (no one gets eaten)."""
        # Check left bank
        if self.boat_location == "right":
            if "wolf" in self.left_bank and "goat" in self.left_bank:
                return False
            if "goat" in self.left_bank and "cabbage" in self.left_bank:
                return False
        # Check right bank
        if self.boat_location == "left":
            if "wolf" in self.right_bank and "goat" in self.right_bank:
                return False
            if "goat" in self.right_bank and "cabbage" in self.right_bank:
                return False
        return True

    def is_goal(self) -> bool:
        """Checks if the goal state has been reached."""
        return self.right_bank == {"wolf", "goat", "cabbage"}
    
    def __hash__(self):
        # Make the state hashable to check for visited states
        return hash((frozenset(self.left_bank), frozenset(self.right_bank), self.boat_location))
    
    def __eq__(self, other):
        return self.__hash__() == other.__hash__()

def get_possible_moves(state: PuzzleState) -> list[PuzzleState]:
    """Generates all possible valid next states from the current state."""
    moves = []
    current_bank = state.left_bank if state.boat_location == "left" else state.right_bank
    
    # Option 1: Move one item in the boat
    for item in current_bank:
        new_state = state.model_copy(deep=True)
        if state.boat_location == "left":
            new_state.left_bank.remove(item)
            new_state.right_bank.add(item)
            new_state.boat_location = "right"
            new_state.move_description = f"Move {item} to the right bank."
        else:
            new_state.right_bank.remove(item)
            new_state.left_bank.add(item)
            new_state.boat_location = "left"
            new_state.move_description = f"Move {item} to the left bank."
        if new_state.is_valid():
            moves.append(new_state)
            
    # Option 2: Move the boat empty
    empty_move_state = state.model_copy(deep=True)
    if state.boat_location == "left":
        empty_move_state.boat_location = "right"
        empty_move_state.move_description = "Move the boat empty to the right bank."
    else:
        empty_move_state.boat_location = "left"
        empty_move_state.move_description = "Move the boat empty to the left bank."
    if empty_move_state.is_valid():
        moves.append(empty_move_state)
        
    return moves

print("Puzzle environment defined successfully.")
```

---

## 结论

在本笔记本中，我们实现了一个**思想树**代理来解决经典的逻辑难题。我们证明，通过将问题转换为状态空间并系统地搜索它，代理可以达到简单的单遍推理方法不可能达到的鲁棒性和准确性水平。

ToT 的核心组件——**思想生成（扩展）**、**状态评估（修剪）**和**搜索**——创建了一个强大的框架来处理复杂的规划和推理任务。虽然它带来了更高的计算成本，但代价是可靠性和解决问题的能力显着提高。这种架构是构建能够进行故意推理并找到具有挑战性的多步骤问题的解决方案的代理的关键一步。

