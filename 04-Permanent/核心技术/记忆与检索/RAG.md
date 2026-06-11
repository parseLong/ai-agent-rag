---
tags:
  - 概念
  - 记忆与检索
aliases:
  - RAG
  - 检索增强生成
  - Retrieval-Augmented Generation
---
# RAG

> 在大模型生成答案之前，先从外部知识库中检索相关信息，把检索内容作为上下文传给模型，让模型基于事实生成答案。

## 核心要点

- **解决两个问题**：大模型知识局限（知识有截止日期）和幻觉问题（一本正经胡说八道）
- **基本流程**：索引阶段（文档切分→Embedding→存入向量数据库）→查询阶段（问题向量化→相似度检索→拼接Prompt→LLM生成）
- **与微调的区别**：RAG不需要训练模型，成本低更新快；微调需要重新训练，成本高但模型能力本身提升

## RAG演进路线

| 阶段           | 特点                  |
| ------------ | ------------------- |
| Naive RAG    | 简单的索引→检索→生成三阶段      |
| Advanced RAG | 引入查询优化、混合检索、重排序     |
| Modular RAG  | 可插拔模块化组件，灵活组合       |
| GraphRAG     | 结合知识图谱，增强结构化关系推理    |
| Agentic RAG  | Agent自主决定检索策略，可多次检索 |

> [!note] Agentic RAG
> 让Agent自主决定检索策略，可以多次检索、动态调整查询、主动验证信息。代价是延迟增加、成本上升。

## RAG关键环节

### 文档分块（Chunking）

分块质量直接决定检索效果——切得太大检索精度下降，切得太小上下文信息不完整。

| 分块策略        | 原理                           | 适用场景            | 优缺点           |
| ----------- | ---------------------------- | --------------- | ------------- |
| **固定大小**    | 按字符/token数切割，设overlap        | 快速原型、通用         | 简单快速，可能切断语义单元 |
| **递归分割**    | 按分隔符层级递归（`\n\n`→`\n`→`.`→空格） | 通用（LangChain默认） | 比固定大小好，仍不语义感知 |
| **语义分块**    | 计算相邻句子embedding相似度，相似度骤降处切割  | 语义完整性要求高        | 效果最好但计算开销大    |
| **结构感知**    | 按Markdown标题/HTML标签/PDF段落结构切割 | 结构化文档           | 保留层级关系        |
| **Agentic** | 用LLM判断每句话属于哪个主题              | 最高质量要求          | 效果极佳，成本极高     |

> [!tip] 生产经验
> - Chunk size典型值：256-1024 tokens，overlap 10-20%
> - 推荐方案：递归分割 + 元数据增强（保留标题层级、页码、文件路径）

### 向量化（Embedding）

| 模型 | 维度 | 多语言 | 推荐场景 |
|------|------|--------|---------|
| **OpenAI text-embedding-3-small** | 1536 | 良好 | 英文为主、快速接入 |
| **BGE-M3** (BAAI) | 1024 | 100+语言 | 中文场景首选 |
| **GTE-Qwen2** (阿里) | 多种 | 中英 | 阿里云生态 |
| **Jina-embeddings-v3** | 1024 | 多语言 | 长文档场景 |
| **Cohere embed-v3** | 1024 | 100+语言 | 需区分查询和文档embedding |

### 检索优化

单纯的向量相似度检索远远不够，需要多种策略组合：

- **混合检索（Hybrid Search）**：`α×向量语义得分 + (1-α)×BM25关键词得分`
- **重排序（Re-ranking）**：检索Top-50 → Cross-Encoder精排 → 取Top-5（精度提升10-30%）
- **查询变换**：Query Rewriting / Query Decomposition / HyDE / Multi-query

### 高级RAG模式

- **Self-RAG**：模型自评估是否需要检索、检索结果是否相关、生成是否忠实，不相关时丢弃重新检索
- **CRAG**：对检索结果质量评估——Correct则使用，Incorrect则改用Web Search，Ambiguous则两者都用
- **Agentic RAG**：将RAG嵌入Agent循环，Agent自主决定何时检索、检索什么、结果是否足够

### RAGAS评估框架

| 指标 | 评估对象 | 含义 |
|------|---------|------|
| **Faithfulness** | 生成质量 | 回答忠实于检索内容的程度 |
| **Answer Relevancy** | 生成质量 | 是否真正在回答用户问题 |
| **Context Precision** | 检索质量 | 检索结果中有多少是有用的 |
| **Context Recall** | 检索质量 | 是否检索到了所有必要信息 |

## 2025-2026 进阶 RAG 三大主线

进阶 RAG 在 2025-2026 年集中在 **3 大主线**：

1. **🧠 KG + Memory 融合** — 从扁平[[向量存储]]走向"结构化、可演化、可联想"的知识表示。代表：[HippoRAG 2](https://arxiv.org/abs/2502.14802)
2. **🎬 Multimodal RAG** — 从文本检索走向图像/视频/表格的本地化检索。代表：[ColPali](https://arxiv.org/abs/2407.01449)
3. **🤖 Agentic RAG** — Retrieval 从固定 Pipeline 演变为 Agent Loop 内的[[工具调用（Function Calling）]]

**另外 2 个值得关注**：
- **🛡 RAG 安全** — Corpus poisoning / prompt injection。代表：[RAGPart / RAGMask](https://arxiv.org/abs/2512.24268)
- **🔧 自动优化 Prompt** — DSPy 程序自动搜索最佳 prompt + retriever 组合。代表：[[DSPy]]

### GraphRAG — 知识图谱 + RAG

Vanilla RAG 不知道哪些 entity 是同一个东西及 entity 之间有什么关系。GraphRAG 在 ingest 阶段用 LLM 将文档抽取成 **(entity, relation, entity)** 三元组构建知识图谱，检索时除向量比对还会 graph traversal。

**何时使用**：需要 multi-hop reasoning（A→B→C）、跨文档实体互引（财报、论文、法律案例）

**何时不使用**：文档间没有实体-关系链接（纯 FAQ）、知识库小（<1k chunks）、预算紧张（KG 构建 token 成本 10-50x）

### Contextual Retrieval — Anthropic 的 Prompt Caching 方案

Vanilla chunk 丢失原始文档上下文——"Q3 revenue grew 15%"的 chunk 你不知道是哪家公司哪一年。Anthropic 在 ingest 阶段用 LLM 为每个 chunk 编写 50-100 token 的上下文头部，结合 **prompt caching** 可节省 ~90% 成本。

**何时使用**：chunk 字面意思与文档主题相差较远（财务报告、研究论文）

### Hybrid Search & Reranking — Production RAG 性价比最高的两个改动

- **Hybrid Search** = vector 相似度 + BM25 keyword 搜索 + RRF 融合分数
- **Reranking** = 第一阶段检索 top-50 → cross-encoder 重新评分取 top-5

Production RAG 评估一致表明：添加 hybrid search + reranker 后，recall@5 通常从 70% 提升到 85-90%，**边际成本低**。

### RAPTOR — 阶层式递归检索

Vanilla chunking 把文档切成扁平 chunk——但整本书的主旨不存在任何单个 chunk 中。RAPTOR 递归聚类和摘要 chunk，构建多层树：底层=原始 chunk、中层=摘要、顶层=全文摘要。

**何时使用**：长文档需要不同抽象层级的查询

### Query Transformations

| 技巧 | 如何改写 | 何时使用 |
|---|---|---|
| **HyDE** | 先让 LLM 为 query 生成"假设答案"、用其 embedding 检索 | 查询用词/风格与文档差异大 |
| **Multi-Query** | LLM 将 query 改写成 N 个变体、分别检索合并 | 查询过短/模糊/多义 |
| **RAG Fusion** | Multi-Query + RRF 融合 | 同 Multi-Query、排名更稳定 |

### RAG vs Long Context vs Fine-tuning — 何时用什么

| 选择 | 适合 | 不适合 |
|---|---|---|
| **[[RAG]]** | 大型/变化快/私有知识库、需要 citation | 推理要整份文档一起看、需要 multi-hop |
| **Long Context** | 200k token 以内中型文档、一次性查询、cross-doc reasoning | 知识库大/经常变化/要 citation |
| **Fine-tuning** | 风格/格式统一、特定领域语言 | 知识会变化、需要 citation |

> 先试 RAG（成本最低）→ RAG 不够再 Long Context → 都不行再 Fine-tuning。

## 关联知识

- [[向量存储]] — RAG的存储基础设施
- [[上下文窗口]] — RAG检索结果的注入上限
- [[记忆系统]] — Agent长期记忆的核心实现
- [[上下文压缩]] — 处理检索结果过长的问题
- [[上下文工程]] — RAG 属于 Context 层（决定窗口里装什么信息）
- [[短时记忆与长时记忆]] — RAG 是 Semantic memory 的技术实现
- [[DSPy]] — 自动优化 RAG prompt + retriever 配置（Path 3 范式）
- [[智能体评估]] — RAG Eval（ragas）是 eval harness 的一部分
- [[Harness Engineering 工程]] — Production RAG 需要 observability + eval + cost 优化

## 参考资料

- [[万字详解面试题库｜RAG篇]]
- [[万字详解大模型应用发展：RAG、MCP、Agent的爆发之旅]]
- [[从0到1搭建Agent：原理分析及个人助手实践]] — 分块策略+嵌入模型+检索优化+RAGAS评估