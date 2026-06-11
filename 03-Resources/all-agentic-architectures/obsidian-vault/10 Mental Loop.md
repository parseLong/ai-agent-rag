---
tags: agent-architecture
aliases: [心理循环, Mental Loop]
创建日期: 2026-05-19
来源: 10_mental_loop.ipynb
---

# 10 心理循环 (Mental Loop)

> 在内部模拟器中预测行动后果
> 关键应用: 机器人、金融交易、安全关键系统

---

## 定义

**模拟器**或**心智模型在环**架构涉及一个代理，该代理使用其环境的内部模型来模拟潜在操作的结果，然后再执行任何操作。这使得代理能够执行假设分析、预测后果并完善其安全性和有效性计划。

## 工作流程

1. **观察：**代理观察真实环境的当前状态。
2. **建议行动：** 根据其目标和当前状态，代理的规划模块生成高级建议行动或策略。
3. **模拟：** 代理将环境的当前状态分叉到沙盒模拟中。它应用建议的行动并向前运行模拟以观察一系列可能的结果。
4. **评估和优化：** 代理分析模拟结果。该行动是否达到了预期的结果？是否有不可预见的负面后果？根据这一评估，它将最初的建议细化为最终的具体行动。
5. **执行：** 代理在*真实*环境中执行最终的、完善的操作。
6. **重复：** 从真实环境的新状态再次开始循环。

## 应用场景

* **机器人：** 在移动物理手臂之前模拟抓握或路径，以避免碰撞或损坏。
* **高风险决策：** 在金融领域，模拟不同市场条件下交易对投资组合的影响。在医疗保健领域，模拟治疗计划的潜在效果。
* **复杂的游戏人工智能：** 策略游戏中的人工智能，模拟未来的几个动作以选择最佳的一个。

## 优缺点

* **优势：**
* **安全和风险降低：** 通过首先在安全环境中审查操作，大大减少发生有害或代价高昂的错误的可能性。
* **提高性能：** 通过允许前瞻和规划，可以做出更稳健、经过深思熟虑的决策。
* **弱点：**
* **模拟与现实差距：** 有效性完全取决于模拟器的保真度。如果世界模型不准确，智能体的计划可能基于错误的假设。
* **计算成本：** 运行模拟，尤其是多个场景，计算成本昂贵且比直接操作慢。

## 核心代码示例

### 示例 1

```python
import os
import random
import numpy as np
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
from rich.table import Table

# --- API Key and Tracing Setup ---
load_dotenv()

os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_PROJECT"] = "Agentic Architecture - Simulator (Nebius)"

required_vars = ["NEBIUS_API_KEY", "LANGCHAIN_API_KEY"]
for var in required_vars:
    if var not in os.environ:
        print(f"Warning: Environment variable {var} not set.")

print("Environment variables loaded and tracing is set up.")
```

### 示例 2

```python
console = Console()

class Portfolio(BaseModel):
    cash: float = 10000.0
    shares: int = 0
    
    def value(self, current_price: float) -> float:
        return self.cash + self.shares * current_price

class MarketSimulator(BaseModel):
    """A simple simulation of a stock market for one asset."""
    day: int = 0
    price: float = 100.0
    volatility: float = 0.1 # Standard deviation for price changes
    drift: float = 0.01 # General trend
    market_news: str = "Market is stable."
    portfolio: Portfolio = Field(default_factory=Portfolio)

    def step(self, action: str, amount: float = 0.0):
        """Advance the simulation by one day, executing a trade first."""
        # 1. Execute trade
        if action == "buy": # amount is number of shares
            shares_to_buy = int(amount)
            cost = shares_to_buy * self.price
            if self.portfolio.cash >= cost:
                self.portfolio.shares += shares_to_buy
                self.portfolio.cash -= cost
        elif action == "sell": # amount is number of shares
            shares_to_sell = int(amount)
            if self.portfolio.shares >= shares_to_sell:
                self.portfolio.shares -= shares_to_sell
                self.portfolio.cash += shares_to_sell * self.price
        
        # 2. Update market price (Geometric Brownian Motion)
        daily_return = np.random.normal(self.drift, self.volatility)
        self.price *= (1 + daily_return)
        
        # 3. Advance time
        self.day += 1
        
        # 4. Potentially update news
        if random.random() < 0.1: # 10% chance of new news
            self.market_news = random.choice(["Positive earnings report expected.", "New competitor enters the market.", "Macroeconomic outlook is strong.", "Regulatory concerns are growing."])
            # News affects drift
            if "Positive" in self.market_news or "strong" in self.market_news:
                self.drift = 0.05
            else:
                self.drift = -0.05
        else:
             self.drift = 0.01 # Revert to normal drift

    def get_state_string(self) -> str:
        return f"Day {self.day}: Price=${self.price:.2f}, News: {self.market_news}\nPortfolio: ${self.portfolio.value(self.price):.2f} ({self.portfolio.shares} shares, ${self.portfolio.cash:.2f} cash)"

print("Market simulator environment defined successfully.")
```

---

## 结论

在此笔记本中，我们构建了一个强大的代理架构，该架构使用内部**模拟器**在提交之前测试和完善其操作。通过创建一个循环`Propose -> Simulate -> Refine -> Execute`，我们使我们的代理能够执行复杂的风险分析，并在动态环境中做出更细致、更安全的决策。

这种模式是构建能够在现实世界中安全有效运行的代理的基石。对内部“心理模型”进行假设分析的能力使代理能够预测后果，避免代价高昂的错误，并制定更稳健的策略。虽然模拟器的保真度是一个关键的依赖项（“模拟与现实差距”），但该架构为构建负责任的、采取行动的人工智能提供了一个清晰且可扩展的框架。
