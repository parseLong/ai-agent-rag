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

**进阶优化方向**：

- **Fine-tuning Embedding**：在特定领域上下文中微调嵌入模型，增强检索内容与查询之间的相关性。特别适用于包含进化或稀有术语的专业领域，定制嵌入方法可显著提高检索相关性
- **Dynamic Embedding**：根据单词出现的上下文动态调整嵌入向量，不同于为每个单词使用单一向量的静态嵌入。理想情况下嵌入应包含尽可能多的上下文以确保语义准确

### 检索优化

单纯的向量相似度检索远远不够，需要多种策略组合。优化分为**预索引优化**（索引构建阶段）和**后索引优化**（检索结果返回后）两大方向：

#### 预索引优化

- **混合检索（Hybrid Search）**：`α×向量语义得分 + (1-α)×BM25关键词得分`，平衡精确召回与语义理解
- **假设性问题（Hypothetical Questions）**：让LLM为每个chunk生成一个问题，保存问题→chunk映射并存入向量数据库。检索时先在问题库匹配相似问题，再路由到原始文本块作为上下文——利用查询与假设性问题之间更高的语义相似性提升搜索质量
- **元数据增强**：将标题、摘要、作者、时间、实体信息嵌入chunk，提升检索过滤效率
- **多粒度分块**：滑动窗口分割、语义分块等策略解决语义断裂（详见上方[[#文档分块]]）

#### 后索引优化

- **重排序（Re-ranking）**：检索Top-50 → Cross-Encoder精排 → 取Top-5（精度提升10-30%），将最相关信息重新定位到上下文前沿
- **Prompt Compression**：压缩不相关上下文，突出关键段落，减少整体上下文长度（详见[[上下文压缩]]）

### 高级RAG模式

- **Self-RAG**：模型自评估是否需要检索、检索结果是否相关、生成是否忠实，不相关时丢弃重新检索
- **CRAG**：对检索结果质量评估——Correct则使用，Incorrect则改用Web Search，Ambiguous则两者都用
- **Agentic RAG**：将RAG嵌入Agent循环，Agent自主决定何时检索、检索什么、结果是否足够
	- Agentic RAG不再局限于**单一知识源**，可以聚合来自多个地方或服务的信息。通过代理可以访问各种工具或数据源，包括用于私有文档索引的向量搜索引擎、用于通用知识或实时信息的网络搜索API、用于计算的计算器，以及其他内部API（如数据库、电子邮件等）；
	- Agentic RAG引入了**迭代推理和验证**，不再局限于单一次的检索。Agentic RAG系统围绕Agent代理核心组织，该代理负责协调这些步骤。代理实际上位于管道的中间，决定如何路由查询和数据。这意味着Agentic RAG通常涉及反馈循环或迭代过程，而不是单次通过。
	- 此外，Agentic RAG支持**多代理架构**：你可以有一个路由代理，将复杂任务分配给多个专门的检索代理，每个代理负责不同的领域（一个代理负责内部文档，一个代理负责网络数据等），然后由一个协调（主）代理汇总发现结果；
	- **更好的适应性：**Agentic RAG体现了**“计划和执行”**的范式——它可以即时调整策略以应对新的或不断发展的查询。代理的包含记忆和规划能力意味着系统可以在没有明确重新编程的情况下适应上下文变化或不可预见的情况。从**静态查找思维模式**转变为**自适应问题**解决思维模式。Agentic RAG不受开发人员预期场景的限制；代理可以利用其一般推理能力来处理新的问题类型或数据源，使系统在需求增长时更加稳健。

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
| **子查询拆解** | LLM 将复杂 query 拆分为多个子查询、分别检索后整合上下文 | 复杂多步问题、需综合多个信息源 |
| **历史对话补齐** | 利用多轮对话历史补齐当前 query 的语义完整性 | 多轮上下文对话、当前查询信息不完整 |

> [!tip] 检索策略选择建议
> - **一般查询**：启用 MQE（多查询扩展）即可，擅长处理用词多样性问题
> - **专业领域查询**：同时启用 MQE + HyDE，HyDE 擅长处理语义鸿沟问题
> - **性能敏感场景**：仅使用基础检索或仅启用 MQE
> - 统一框架确保结果的质量和多样性

### RAG vs Long Context vs Fine-tuning — 何时用什么

| 选择 | 适合 | 不适合 |
|---|---|---|
| **[[RAG]]** | 大型/变化快/私有知识库、需要 citation | 推理要整份文档一起看、需要 multi-hop |
| **Long Context** | 200k token 以内中型文档、一次性查询、cross-doc reasoning | 知识库大/经常变化/要 citation |
| **Fine-tuning** | 风格/格式统一、特定领域语言 | 知识会变化、需要 citation |

> 先试 RAG（成本最低）→ RAG 不够再 Long Context → 都不行再 Fine-tuning。

## RAG典型应用场景

1. **智能文档处理**：使用 MarkItDown 实现 PDF→Markdown 统一转换，基于 Markdown 结构的智能分块策略，高效的向量化和索引构建
2. **高级检索问答**：多查询扩展（MQE）提升召回率 + 假设文档嵌入（HyDE）改善检索精度 + 上下文感知的智能问答
3. **多层次记忆管理**：工作记忆（当前学习任务和上下文）→ 情景记忆（学习事件和查询历史）→ 语义记忆（概念知识和理解）→ 感知记忆（文档特征和多模态信息）
4. **个性化学习支持**：基于学习历史的个性化推荐、记忆整合和选择性遗忘、学习报告生成和进度追踪

## 关联知识

- [[向量存储]] — RAG的存储基础设施
- [[上下文窗口]] — RAG检索结果的注入上限
- [[记忆系统]] — Agent长期记忆的核心实现
- [[上下文压缩]] — 处理检索结果过长的问题
- [[Context Engineering]] — RAG 属于 Context 层（决定窗口里装什么信息）
- [[短时记忆与长时记忆]] — RAG 是 Semantic memory 的技术实现
- [[DSPy]] — 自动优化 RAG prompt + retriever 配置（Path 3 范式）
- [[智能体评估]] — RAG Eval（ragas）是 eval harness 的一部分
- [[Harness Engineering 工程]] — Production RAG 需要 observability + eval + cost 优化

## 参考资料

- [[万字详解面试题库｜RAG篇]]
- [[万字详解大模型应用发展：RAG、MCP、Agent的爆发之旅]]
- [[从0到1搭建Agent：原理分析及个人助手实践]] — 分块策略+嵌入模型+检索优化+RAGAS评估