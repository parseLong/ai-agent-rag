---
tags: agent-architecture
aliases: [情景+语义记忆, Episodic + Semantic Memory]
创建日期: 2026-05-19
来源: 08_episodic_with_semantic.ipynb
---

# 08 情景+语义记忆 (Episodic + Semantic Memory)

> 双记忆系统：向量存储+图数据库
> 关键应用: 长期个人助理、个性化导师

---

## 定义

**情景+语义记忆堆栈**是一种代理架构，可维护两种类型的长期记忆。**情景记忆**存储按时间顺序排列的经历日志（例如聊天历史摘要），并且通常根据语义相似性进行搜索。**语义记忆**将提取的结构化知识（事实、实体、关系）存储在知识库（通常是图表）中。

## 工作流程

1. **交互：** 代理与用户进行对话。
2. **内存检索（Recall）：** 对于新用户查询，代理首先查询两个内存系统。
* 它在**情景**矢量存储中搜索类似的过去对话。
* 它在**语义**图形数据库中查询与查询相关的实体和事实。
3. **增强生成：** 检索到的记忆被添加到提示的上下文中，允许法学硕士生成了解过去交互和学到的事实的响应。
4. **内存创建（编码）：** 交互完成后，后台进程会分析对话。
* 它创建了回合的简洁摘要（新的**情景**记忆）。
* 它提取关键实体和关系（新的**语义**记忆）。
5. **内存存储：** 新的情景摘要被嵌入并保存到矢量存储中。新的语义事实被写为图数据库中的节点和边。

## 应用场景

* **长期个人助理：** 能够在数周或数月内记住您的偏好、项目和个人详细信息的助理。
* **个性化系统：** 记住您的风格的电子商务机器人，或记住您的学习进度和弱点的教育导师。
* **复杂研究代理：** 在探索文档时构建主题知识图的代理，使其能够回答复杂的多跳问题。

## 优缺点

* **优势：**
* **真正的个性化：** 实现无限期持续的情境和学习，远远超出单个会话的情境窗口。
* **丰富的理解：** 图数据库允许代理理解和推理实体之间的复杂关系。
* **弱点：**
* **复杂性：** 这是一个比简单的无状态代理更复杂的构建和维护架构。
* **内存膨胀和修剪：** 随着时间的推移，内存存储可能会变得庞大。总结、巩固或修剪旧的/不相关的记忆的策略对于长期表现至关重要。

## 核心代码示例

### 示例 1

```python
import os
import uuid
from typing import List, Dict, Any, Optional, Tuple
from dotenv import load_dotenv

# Pydantic for data modeling
from pydantic import BaseModel, Field

# LangChain components
from langchain_nebius import ChatNebius, NebiusEmbeddings
from langchain_community.graphs import Neo4jGraph
from langchain_community.vectorstores import FAISS
from langchain.docstore.document import Document
from langchain_core.prompts import ChatPromptTemplate

# LangGraph components
from langgraph.graph import StateGraph, END
from typing_extensions import TypedDict

# For pretty printing
from rich.console import Console
from rich.markdown import Markdown

# --- API Key and Tracing Setup ---
load_dotenv()

os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_PROJECT"] = "Agentic Architecture - Memory Stack (Nebius)"

# Check for required environment variables
required_vars = ["NEBIUS_API_KEY", "LANGCHAIN_API_KEY", "NEO4J_URI", "NEO4J_USERNAME", "NEO4J_PASSWORD"]
for var in required_vars:
    if var not in os.environ:
        print(f"Warning: Environment variable {var} not set.")

print("Environment variables loaded and tracing is set up.")
```

### 示例 2

```python
console = Console()
llm = ChatNebius(model="mistralai/Mixtral-8x22B-Instruct-v0.1", temperature=0)
embeddings = NebiusEmbeddings()

# --- 1. Vector Store for Episodic Memory ---
# In a real application, you'd persist this. For this example, it's in-memory.
try:
    episodic_vector_store = FAISS.from_texts(["Initial document to bootstrap the store"], embeddings)
except ImportError:
    console.print("[bold red]FAISS not installed. Please run `pip install faiss-cpu`.")[/bold red]
    episodic_vector_store = None

# --- 2. Graph DB for Semantic Memory ---
try:
    graph = Neo4jGraph(
        url=os.environ.get("NEO4J_URI"),
        username=os.environ.get("NEO4J_USERNAME"),
        password=os.environ.get("NEO4J_PASSWORD")
    )
    # Clear the graph for a clean run
    graph.query("MATCH (n) DETACH DELETE n")
except Exception as e:
    console.print(f"[bold red]Failed to connect to Neo4j: {e}. Please check your credentials and connection.[/bold red]")
    graph = None

# --- 3. Pydantic Models for the "Memory Maker" ---
# Define the structure of knowledge we want to extract.
class Node(BaseModel):
    id: str = Field(description="Unique identifier for the node, which can be a person's name, a company ticker, or a concept.")
    type: str = Field(description="The type of the node (e.g., 'User', 'Company', 'InvestmentPhilosophy').")
    properties: Dict[str, Any] = Field(description="A dictionary of properties for the node.")

class Relationship(BaseModel):
    source: Node = Field(description="The source node of the relationship.")
    target: Node = Field(description="The target node of the relationship.")
    type: str = Field(description="The type of the relationship (e.g., 'IS_A', 'INTERESTED_IN').")
    properties: Dict[str, Any] = Field(description="A dictionary of properties for the relationship.")

class KnowledgeGraph(BaseModel):
    """Represents the structured knowledge extracted from a conversation."""
    relationships: List[Relationship] = Field(description="A list of relationships to be added to the knowledge graph.")

# --- 4. The "Memory Maker" Agent ---
def create_memories(user_input: str, assistant_output: str):
    conversation = f"User: {user_input}\nAssistant: {assistant_output}"
    
    # 4a. Create Episodic Memory (Summarization)
    console.print("--- Creating Episodic Memory (Summary) ---")
    summary_prompt = ChatPromptTemplate.from_messages([
        ("system", "You are a summarization expert. Create a concise, one-sentence summary of the following user-assistant interaction. This summary will be used as a memory for future recall."),
        ("human", "Interaction:\n{interaction}")
    ])
    summarizer = summary_prompt | llm
    episodic_summary = summarizer.invoke({"interaction": conversation}).content
    
    new_doc = Document(page_content=episodic_summary, metadata={"created_at": uuid.uuid4().hex})
    episodic_vector_store.add_documents([new_doc])
    console.print(f"[green]Episodic memory created:[/green] '{episodic_summary}'")
    
    # 4b. Create Semantic Memory (Fact Extraction)
    console.print("--- Creating Semantic Memory (Graph) ---")
    extraction_llm = llm.with_structured_output(KnowledgeGraph)
    extraction_prompt = ChatPromptTemplate.from_messages([
        ("system", "You are a knowledge extraction expert. Your task is to identify key entities and their relationships from a conversation and model them as a graph. Focus on user preferences, goals, and stated facts."),
        ("human", "Extract all relationships from this interaction:\n{interaction}")
    ])
    extractor = extraction_prompt | extraction_llm
    try:
        kg_data = extractor.invoke({"interaction": conversation})
        if kg_data.relationships:
            for rel in kg_data.relationships:
                graph.add_graph_documents([rel], include_source=True)
            console.print(f"[green]Semantic memory created:[/green] Added {len(kg_data.relationships)} relationships to the graph.")
        else:
            console.print("[yellow]No new semantic memories identified in this interaction.[/yellow]")
    except Exception as e:
        console.print(f"[red]Could not extract or save semantic memory: {e}[/red]")

if episodic_vector_store and graph:
    print("Memory components initialized successfully.")
```

---

## 控制流视角（agno 实现）

> 来自 [[Agent架构演化总览]] 的控制流分析框架

### 六问拆解

| 问题 | 回答 |
|---|---|
| 它要解决什么问题？ | 前面所有架构默认当前对话结束后系统基本失忆。用户需要**跨轮记忆**——偏好、讨论历史、长期稳定事实 |
| 它的 State 是什么？ | 图内状态 + 图外可检索历史状态。Episodic：用户层长期记忆（向量库）；Semantic：结构化知识库（图/KV） |
| 它的拓扑是什么？ | 生成 → 检索 episodic → 检索 semantic → 增强生成 → 写回 episodic |
| 它的 Router 怎么工作？ | agno 内置路由：`enable_agentic_memory=True` 自动接上写回链路 |
| 它的失败模式是什么？ | 错误记忆写入造成长期污染、episodic recall 召回相似但不相关的信息、semantic graph 存入过时事实、记忆让系统更强但也让错误变得持久 |
| 什么时候该升级？ | 当向量检索只能"找相似"但不能做"关系推理" → [[12 Graph Memory]] |

![[07-Attachments/all-agentic-architectures/Memory工作流.png]]

### 它真正新增了什么能力？

系统的 state 从"图内字段"扩展成：**图内状态 + 图外可检索历史状态。** agent 能力不再只依赖当前上下文窗口，而是依赖它如何从历史里提取相关信息。

### agno 实现

```python
from agno.memory.v2.memory import Memory
from agno.memory.v2.db.sqlite import SqliteMemoryDb
from agno.knowledge.text import TextKnowledgeBase
from agno.vectordb.lancedb import LanceDb, SearchType
from agno.embedder.openai import OpenAIEmbedder

# Episodic：用户层长期记忆
memory = Memory(
    db=SqliteMemoryDb(table_name="user_memories", db_file="tmp/memory.db"),
    model=OpenAIChat(id="gpt-5-mini"),
)

# Semantic：结构化知识库
knowledge = TextKnowledgeBase(
    path="data/facts",
    vector_db=LanceDb(table_name="facts", uri="tmp/lancedb",
                      search_type=SearchType.hybrid,
                      embedder=OpenAIEmbedder(id="text-embedding-3-small")),
)

mem_agent = Agent(
    model=OpenAIChat(id="gpt-5-mini"),
    memory=memory,
    enable_agentic_memory=True,      # agent 可主动写入/检索 episodic
    enable_user_memories=True,
    knowledge=knowledge,             # semantic
    search_knowledge=True,
)
```

memory 不再是附加模块，而是主控制流的一部分。`enable_agentic_memory=True` 把整条写回链路自动接上。

---

## 结论

在这个笔记本中，我们成功构建了一个具有复杂的长期记忆系统的代理。该演示清楚地展示了该架构的强大功能：

- **无状态失败：** 当被问到“根据我的目标，什么是好的替代方案？”时，标准代理会失败，因为它不记得用户的目标。
- **记忆增强成功：** 我们的代理成功了，因为它可以：
1. **回忆情景：** 它检索了第一次对话的摘要：“用户 Alex 介绍自己是一名保守派投资者......”
2.  **Recall Semantically:** It queried the graph and found the structured fact:`(User: Alex) -[HAS_GOAL]-> (InvestmentPhilosophy: Conservative)`。
3. **综合：** 它使用这种组合上下文来提供高度相关和个性化的推荐（Microsoft），明确引用用户的保守目标。

这种回忆“发生了什么”（情景）和“已知的事情”（语义）的结合是超越简单的事务性代理以创建真正的学习伙伴的强大范例。虽然大规模管理内存面临着修剪和整合等挑战，但我们在这里构建的基础架构是迈向更加智能和个性化的人工智能系统的重要一步。
