---
tags: agent-architecture
aliases: [试运行框架, Dry-Run Harness]
创建日期: 2026-05-19
来源: 14_dry_run.ipynb
---

# 14 试运行框架 (Dry-Run Harness)

> 代理行动先模拟再审批执行
> 关键应用: 生产代理部署、调试

---

## 定义

**可观察性和试运行框架**是一种拦截代理操作的测试和部署架构。它首先以“试运行”或“沙盒”模式执行它们，模拟动作而不会造成现实世界的效果。然后将生成的计划和日志公开以供审核，只有在明确批准后才能在实时环境中执行操作。

## 工作流程

1. **代理提出操作：** 代理确定要执行的计划或特定工具调用（例如，`api.post_update(...)`）。
2. **试运行执行：** 线束通过以下命令调用代理的计划：`dry_run=True`旗帜。底层工具旨在识别此标志，并仅输出它们“将”执行的操作以及日志和跟踪。
3. **收集可观测性数据：** 该工具从模拟中捕获建议的操作、空运行日志和任何其他相关跟踪数据。
4. **人工/自动审核：** 该可观察性数据将呈现给审核者。人类可以检查正确性、安全性以及与目标的一致性。自动化系统可以检查是否违反策略或已知的不良模式。
5. **通过/不通过决定：** 审阅者做出`approve` or `reject`决定。
6. **实时执行（“开始”）：** 如果获得批准，该工具将重新执行代理的操作，这次使用`dry_run=False`，使其产生现实世界的效果。

## 应用场景

* **调试和测试：** 在开发中，准确了解代理如何解释任务以及它正在采取哪些操作，而不会产生副作用。
* **生产验证和安全：** 作为生产中的永久功能，任何代理都可以修改状态、花钱、发送通信或执行任何其他不可逆转的操作。
* **代理的 CI/CD：** 将试运行工具集成到自动化测试管道中，以在部署新版本之前验证代理行为。

## 优缺点

* **优势：**
* **最大透明度和安全性：** 提供代理操作的清晰、可审核的预览，防止代价高昂或令人尴尬的错误。
* **非常适合调试：** 可以轻松跟踪代理的逻辑和工具调用，而无需撤消实际更改。
* **弱点：**
* **延迟部署/执行：** 强制审核步骤（尤其是人工审核）会引入延迟，使其不适合实时应用程序。
* **需要工具支持：** 代理使用的工具和 API 必须设计为支持`dry_run`模式。

## 核心代码示例

### 示例 1

```python
import os
import datetime
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
os.environ["LANGCHAIN_PROJECT"] = "Agentic Architecture - Dry-Run Harness (Nebius)"

required_vars = ["NEBIUS_API_KEY", "LANGCHAIN_API_KEY"]
for var in required_vars:
    if var not in os.environ:
        print(f"Warning: Environment variable {var} not set.")

print("Environment variables loaded and tracing is set up.")
```

### 示例 2

```python
console = Console()

# Structured model for the agent's proposed post
class SocialMediaPost(BaseModel):
    content: str = Field(description="The full text content of the social media post.")
    hashtags: List[str] = Field(description="A list of relevant hashtags, without the '#'.")

# The key component: A tool with a dry_run flag
class SocialMediaAPI:
    """A mock social media API that supports a dry-run mode."""
    
    def publish_post(self, post: SocialMediaPost, dry_run: bool = True) -> Dict[str, Any]:
        """Publishes a post to the social media feed."""
        timestamp = datetime.datetime.now().isoformat()
        hashtags_str = ' '.join([f'#{h}' for h in post.hashtags])
        full_post_text = f"{post.content}\n\n{hashtags_str}"
        
        if dry_run:
            # In dry-run mode, we don't execute, we just return the plan and logs
            log_message = f"[DRY RUN] At {timestamp}, would publish the following post:\n--- PREVIEW ---\n{full_post_text}\n--- END PREVIEW ---"
            console.print(Panel(log_message, title="[yellow]Dry Run Log[/yellow]", border_style="yellow"))
            return {"status": "DRY_RUN_SUCCESS", "log": log_message, "proposed_post": full_post_text}
        else:
            # In live mode, we execute the action
            log_message = f"[LIVE] At {timestamp}, successfully published post!"
            console.print(Panel(log_message, title="[green]Live Execution Log[/green]", border_style="green"))
            # Here you would have the actual API call, e.g., twitter_client.create_tweet(...)
            return {"status": "LIVE_SUCCESS", "log": log_message, "post_id": f"post_{hash(full_post_text)}"}

social_media_tool = SocialMediaAPI()
print("Dry-run capable SocialMediaAPI tool defined successfully.")
```

---

## 结论

在此笔记本中，我们构建了完整的**可观察性和试运行线束**。这种架构不仅仅是一个功能，而且是部署与现实世界交互的代理的基本理念。通过强制执行`propose -> review -> execute`循环，我们获得了重要的好处：

- **透明度：** 我们确切地知道代理在行动之前打算做什么。
- **控制：** 我们拥有一个人机交互（或自动规则引擎），对任何行动拥有最终否决权。
- **安全：** 我们防止意外、代价高昂或有害的行为，从满怀希望的执行转向自信的部署。

虽然这种模式会带来延迟，但它提供的安全性和可靠性对于大多数现实世界的应用程序来说都是不可协商的。对于任何希望构建强大、值得信赖且可投入生产的代理系统的开发人员来说，它是一个必不可少的工具。
