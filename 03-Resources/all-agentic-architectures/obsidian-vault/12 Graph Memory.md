---
tags: agent-architecture
aliases: [图记忆, Graph Memory]
创建日期: 2026-05-19
来源: 12_graph.ipynb
---

# 12 图记忆 (Graph Memory)

> 结构化图存储实体关系
> 关键应用: 企业情报、高级研究

---

## 定义

**图形/世界模型内存**是一种代理架构，其中知识存储在结构化图形数据库中。信息被表示为节点（诸如人物、地点、概念之类的实体）和边缘（它们之间的关系）。这创建了一个动态的“世界模型”，代理可以对其进行推理。

## 工作流程

1. **信息摄取：** 代理接收非结构化或半结构化数据（文本、文档、API 响应）。
2. **知识提取：** 由法学硕士支持的流程解析信息，识别关键实体以及连接它们的关系。
3. **图更新：** 提取的节点和边被添加到持久图数据库（如 Neo4j）中或在其中更新。
4. **问答/推理：** 当被问到问题时，代理：
a.将自然语言问题转换为正式的图形查询语言（例如 Neo4j 的 Cypher）。
b.对图执行查询以检索相关子图或事实。
c.将查询结果合成为自然语言答案。

## 应用场景

* **企业知识助手：** 从内部文档构建公司项目、员工和客户的可查询模型。
* **高级研究助理：** 通过吸收研究论文创建科学领域的动态知识库。
* **复杂系统诊断：** 对系统组件及其依赖关系进行建模以诊断故障。

## 优缺点

* **优势：**
* **结构化且可解释：** 知识是高度组织化的。可以通过在图表中显示通向该答案的确切路径来解释答案。
* **启用复杂推理：** 擅长回答需要通过关系连接不同信息的“多跳”问题。
* **弱点：**
*   **Upfront Complexity:** Requires a well-defined schema and a robust extraction process.
* **保持图表更新：** 随着时间的推移，管理更新、解决冲突信息和修剪过时的事实（知识生命周期管理）可能具有挑战性。

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
from langchain_community.graphs import Neo4jGraph
from langchain.chains import GraphCypherQAChain
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.pydantic_v1 import BaseModel as V1BaseModel

# For pretty printing
from rich.console import Console
from rich.markdown import Markdown

# --- API Key and Tracing Setup ---
load_dotenv()

os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_PROJECT"] = "Agentic Architecture - Graph Memory (Nebius)"

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

# Connect to our Neo4j database
try:
    graph = Neo4jGraph()
    # Clear the graph for a clean run
    graph.query("MATCH (n) DETACH DELETE n")
except Exception as e:
    console.print(f"[bold red]Failed to connect to Neo4j: {e}. Please check your credentials and connection.[/bold red]")
    graph = None

# Pydantic models for structured extraction (using LangChain's v1 BaseModel for compatibility with older structured output methods)
class Node(V1BaseModel):
    id: str = Field(description="Unique name or identifier for the entity.")
    type: str = Field(description="The type or label of the entity (e.g., Person, Company, Product).")

class Relationship(V1BaseModel):
    source: Node
    target: Node
    type: str = Field(description="The type of relationship (e.g., WORKS_FOR, ACQUIRED).")

class KnowledgeGraph(V1BaseModel):
    """A graph of nodes and relationships."""
    relationships: List[Relationship]

# The Graph Maker Agent
def get_graph_maker_chain():
    extractor_llm = llm.with_structured_output(KnowledgeGraph)
    prompt = ChatPromptTemplate.from_messages([
        ("system", "You are an expert at extracting information from text and building a knowledge graph. Extract all entities (nodes) and relationships from the provided text. The relationship type should be a verb in all caps, like 'WORKS_FOR' or 'ACQUIRED'."),
        ("human", "Extract a knowledge graph from the following text:\n\n{text}")
    ])
    return prompt | extractor_llm

graph_maker_agent = get_graph_maker_chain()
print("Successfully connected to Neo4j and defined the Graph Maker Agent.")
```

---

## 结论

在本笔记本中，我们构建了一个围绕**图/世界模型内存**构建的完整代理系统。我们演示了完整的生命周期：摄取非结构化数据，使用法学硕士构建结构化知识图，然后使用该图来回答需要真正推理的复杂、多跳问题。

与更简单的内存系统相比，该架构代表了功能上的重大飞跃。通过创建一个明确的、可查询的世界模型，我们使代理能够连接不同的事实并发现隐藏的见解。虽然随着时间的推移维护这个图的挑战是真实存在的，但构建知识渊博且可解释的人工智能助手的潜力使其成为现代代理设计中最令人兴奋和最强大的模式之一。
