---
tags: agent-architecture
aliases: [元胞自动机, Cellular Automata]
创建日期: 2026-05-19
来源: 16_cellular_automata.ipynb
---

# 16 元胞自动机 (Cellular Automata)

> 去中心化网格代理产生涌现行为
> 关键应用: 空间推理、物流、复杂系统模拟
---

## 定义

**基于网格的代理系统**是一种架构，其中大量简单代理（或“单元”）排列在空间网格中。每个代理都有一个状态，并根据仅考虑其直接邻居状态的规则集同步更新它。这些本地交互产生了复杂的高级模式和解决问题的能力。

## 工作流程

1. **网格初始化：** 创建细胞代理网格，每个网格都使用类型（例如障碍物、空）和状态（例如值）进行初始化。
2. **设置边界条件：** 一个或多个单元格被赋予一种特殊状态以开始计算（例如，“目标”单元格的值设置为 0）。
3. **同步滴答：** 系统向前“滴答”。在每个刻度中，每个单元都会根据其邻居的当前状态同时计算其下一个状态。
4. **出现：** 随着系统的运转，信息像波浪一样在网格中传播。这可以创建渐变、路径和其他复杂结构。
5. **状态稳定：**系统运行直到电网状态稳定（不再发生变化），表明计算完成。
6. **读出：** 然后直接从网格的最终状态读取问题的解决方案（例如，通过遵循计算的梯度）。

## 应用场景

* **空间推理和物流：** 动态环境中的最佳寻路（如我们的仓库示例）。
* **复杂系统模拟：** 对森林火灾、疾病传播或城市发展等突发行为的现象进行建模。
* **并行计算：** 某些算法可以映射到元胞自动机模型，以便在高度并行的硬件（如 GPU）上执行。

## 优缺点

* **优势：**
* **高并行性：** 逻辑本质上是并行的，使其在适当的硬件上速度极快。
* **适应性：**系统可以通过简单地重新传播其波来动态地对环境的变化（例如，新的障碍物）做出反应。
* **紧急复杂性：** 可以用极其简单的规则解决非常复杂的问题。
* **弱点：**
* **设计复杂性：** 设计局部规则以产生所需的全局行为可能具有挑战性且不直观。
* **较少内省：** 很难询问单个细胞“为什么”它具有某种状态；推理分布在整个系统中。

## 核心代码示例

### 示例 1

```python
import os
import numpy as np
import time
from typing import List, Dict, Any, Optional, Tuple
from dotenv import load_dotenv
from IPython.display import clear_output

# LangChain for optional final summary
from langchain_nebius import ChatNebius
from langchain_core.prompts import ChatPromptTemplate

# For pretty printing and visualization
from rich.console import Console
from rich.table import Table
from rich.panel import Panel

# --- API Key and Tracing Setup ---
load_dotenv()

os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_PROJECT"] = "Agentic Architecture - Cellular Automata (Nebius)"

required_vars = ["NEBIUS_API_KEY", "LANGCHAIN_API_KEY"]
for var in required_vars:
    if var not in os.environ:
        print(f"Warning: Environment variable {var} not set.")

print("Environment variables loaded and tracing is set up.")
```

### 示例 2

```python
console = Console()

class CellAgent:
    """A single agent in our grid. Its only job is to update its value based on neighbors."""
    def __init__(self, cell_type: str, item: Optional[str] = None):
        self.type = cell_type # 'EMPTY', 'OBSTACLE', 'SHELF', 'PACKING_STATION'
        self.item = item
        self.pathfinding_value = float('inf')

    def update_value(self, neighbors: List['CellAgent']):
        """The core local rule: my new value is 1 + the minimum value of my non-obstacle neighbors."""
        if self.type == 'OBSTACLE':
            return float('inf')
        
        min_neighbor_value = float('inf')
        for neighbor in neighbors:
            if neighbor.pathfinding_value < min_neighbor_value:
                min_neighbor_value = neighbor.pathfinding_value
        
        # The +1 represents the cost of moving from a neighbor to this cell
        return min(self.pathfinding_value, min_neighbor_value + 1)

class WarehouseGrid:
    """Manages the entire grid of CellAgents and the simulation loop."""
    def __init__(self, layout: List[str]):
        self.height = len(layout)
        self.width = len(layout[0])
        self.grid = self._create_grid_from_layout(layout)
        self.item_locations = self._get_item_locations()

    def _create_grid_from_layout(self, layout):
        grid = np.empty((self.height, self.width), dtype=object)
        for r, row_str in enumerate(layout):
            for c, char in enumerate(row_str):
                if char == ' ':
                    grid[r, c] = CellAgent('EMPTY')
                elif char == '#':
                    grid[r, c] = CellAgent('OBSTACLE')
                elif char == 'P':
                    grid[r, c] = CellAgent('PACKING_STATION')
                else: # It's an item
                    grid[r, c] = CellAgent('SHELF', item=char)
        return grid

    def _get_item_locations(self) -> Dict[str, Tuple[int, int]]:
        locations = {}
        for r in range(self.height):
            for c in range(self.width):
                if self.grid[r, c].type == 'SHELF':
                    locations[self.grid[r, c].item] = (r, c)
                if self.grid[r, c].type == 'PACKING_STATION':
                    locations['P'] = (r, c)
        return locations

    def get_neighbors(self, r: int, c: int) -> List[CellAgent]:
        neighbors = []
        for dr, dc in [(0, 1), (0, -1), (1, 0), (-1, 0)]: # N, S, E, W
            nr, nc = r + dr, c + dc
            if 0 <= nr < self.height and 0 <= nc < self.width:
                neighbors.append(self.grid[nr, nc])
        return neighbors

    def tick(self) -> bool:
        """Performs one synchronous update of all cells. Returns True if any value changed."""
        changed = False
        # First, calculate all new values based on the current state
        new_values = np.empty((self.height, self.width))
        for r in range(self.height):
            for c in range(self.width):
                neighbors = self.get_neighbors(r, c)
                new_values[r, c] = self.grid[r, c].update_value(neighbors)
        
        # Then, apply all the new values
        for r in range(self.height):
            for c in range(self.width):
                if self.grid[r, c].pathfinding_value != new_values[r, c]:
                    self.grid[r, c].pathfinding_value = new_values[r, c]
                    changed = True
        return changed

    def visualize(self, show_values: bool = False, title: str = "Warehouse Grid"):
        """Displays the grid state using Rich."""
        table = Table(title=title, show_header=False, show_edge=True, padding=0)
        for _ in range(self.width):
            table.add_column(justify="center")
        
        for r in range(self.height):
            row_renderables = []
            for c in range(self.width):
                cell = self.grid[r, c]
                val = cell.pathfinding_value
                display_char = ''
                if cell.type == 'EMPTY': display_char = '[grey70]·[/grey70]'
                elif cell.type == 'OBSTACLE': display_char = '[red]█[/red]'
                elif cell.type == 'PACKING_STATION': display_char = '[bold green]P[/bold green]'
                elif cell.type == 'SHELF': display_char = f'[bold blue]{cell.item}[/bold blue]'

                if show_values and val != float('inf'):
                    # Color code the path values
                    color = int(255 - (val * 5) % 255)
                    row_renderables.append(f"[rgb({color},{color},{color}) on rgb(30,30,60)]{int(val):^3}[/]")
                else:
                    row_renderables.append(f" {display_char} ")
            table.add_row(*row_renderables)
        console.print(table)

print("Cellular Automata environment defined successfully.")
```

---

## 控制流视角（agno 实现）

> 来自 [[Agent架构演化总览]] 的控制流分析框架

### 六问拆解

| 问题 | 回答 |
|---|---|
| 它要解决什么问题？ | 前面所有架构都默认存在一个中心 agent、一个主控制流、一个 orchestrator。CA 彻底打破这个前提：**有些问题根本不需要中心规划** |
| 它的 State 是什么？ | 每个 CellAgent 的局部状态（type + pathfinding_value），没有全局 state |
| 它的拓扑是什么？ | 网格：每个单元只看四邻，同步演化 |
| 它的 Router 怎么工作？ | 没有 router——每个格子根据邻居状态独立执行局部规则 |
| 它的失败模式是什么？ | 局部规则设计不当、系统收敛慢、涌现出错误的全局结构 |
| 什么时候该用？ | 寻路、扩散、空间优化等适合用局部规则产生全局行为的场景 |

### 它和前面所有架构最大的区别

LLM 退出了执行主回路，它最多负责规则设计、参数选择、系统解释，真正的求解过程由大量局部节点同步演化完成。整个范式从**中心控制转向分布式涌现**。

### agno 实现

```python
class CellAgent(BaseModel):
    type: str  # 'EMPTY' | 'OBSTACLE' | 'GOAL'
    pathfinding_value: float = float("inf")

    def update(self, neighbors: List["CellAgent"]):
        if self.type == "OBSTACLE": return
        if self.type == "GOAL":
            self.pathfinding_value = 0
            return
        m = min((n.pathfinding_value for n in neighbors), default=float("inf"))
        self.pathfinding_value = min(self.pathfinding_value, m + 1)

def run_ca(grid, steps=50):
    for _ in range(steps):
        snapshot = [[copy.deepcopy(c) for c in row] for row in grid]
        for r in range(len(grid)):
            for c in range(len(grid[0])):
                grid[r][c].update(neighbors_of(snapshot, r, c))

# LLM 只负责规则设计
rule_designer = Agent(name="rule_designer", model=OpenAIChat(id="gpt-5-mini"),
    instructions="You design local update rules for a cellular automaton. Given a high-level objective, propose a minimal Python update function that each cell will execute based only on its neighbors.")
```

LLM 退到纯"规则设计者/解释者"的位置——真正的求解由程序化并行更新完成。

---

## 结论

在本笔记本中，我们构建了一个完全实现的**元胞自动机/基于网格的代理系统**。我们超越了理论，为复杂的空间推理问题——仓库物流——实现了一个实用的解决方案。

我们亲眼目睹了通过迷你代理网格同步执行简单的本地规则可以产生复杂的、面向目标的行为。**波传播**和**梯度下降**的概念并不是以自上而下的方式明确编程的，而是元胞自动机进化的自然结果。

这种架构虽然并不适合所有问题，但对于涉及动态环境中的空间推理、模拟和优化的任务来说非常强大。它鼓励我们将代理系统视为一个**可编程的计算环境**，而不是单个的“机器人”，可以配置为以大规模并行和自适应的方式解决问题。

