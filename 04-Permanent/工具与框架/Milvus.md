---
title: Milvus 向量数据库
date: 2026-03
tags:
  - 工具与框架
  - 向量数据库
  - 概念卡片
  - Milvus
aliases:
  - Milvus
---

# Milvus 向量数据库 — 核心知识点全解析

架构原理 · 核心概念 · 索引算法 · 底层实现 · 实战指南

## 目录

- 一、Milvus 概述
- 二、核心概念详解
- 三、系统架构（存算分离四层架构）
- 四、索引算法全解析
- 五、相似度度量标准
- 六、数据模型与 Schema 设计
- 七、检索功能详解
- 八、一致性模型
- 九、Python SDK 实战指南
- 十、性能优化最佳实践
- 十一、典型应用场景
- 十二、Milvus vs 其他向量数据库
- 十三、部署方案
- 十四、常见问题与排查
- 十五、底层原理深入解析
- 十六、待深入探索的方向

---

# 一、Milvus 概述

## 1.1 什么是 Milvus

Milvus 是一款开源的、云原生的向量数据库，专为海量向量数据的高性能相似性搜索（Similarity Search）而设计。它能够存储、管理和检索来自深度学习模型、NLP 模型等产生的高维向量数据（Embedding），广泛应用于 AI 应用和非结构化数据检索场景。

## 1.2 核心特性

- **开源云原生：** 完全开源（Apache 2.0 许可证），支持 Kubernetes 部署，天然支持存算分离。
- **高性能：** 基于 Faiss、HNSW、DiskANN、SCANN 等业界领先的向量搜索库构建。
- **高可扩展：** 分布式架构，支持水平扩展至数十亿甚至万亿级向量规模。
- **多模态支持：** 支持稠密向量、稀疏向量、二进制向量，支持多向量字段混合检索。
- **丰富接口：** 提供 Python、Java、Go、Node.js、C# 等多语言 SDK，以及 RESTful API。
- **混合检索：** 支持向量检索与标量条件过滤相结合的混合搜索。
- **多租户：** 支持 Partition Key 和多 Collection 隔离，实现租户级数据隔离。

## 1.3 发展历程

- 2019 年：Milvus 0.x 发布，单机版本，基于 Faiss。
- 2021 年：Milvus 2.0 发布，全面重构为云原生存算分离架构。
- 2023—2025 年：持续迭代，新增 Streaming Node、稀疏向量、多向量混合检索、GPU 索引等能力。
- 当前版本：Milvus 2.6.x，已成为 AI 领域最主流的开源向量数据库之一。

## 1.4 为什么需要向量数据库

传统关系型数据库（如 MySQL）和 NoSQL 数据库（如 MongoDB）擅长结构化数据的精确匹配和范围查询，但无法高效完成「语义相似性搜索」。例如：

- 「找到与这段文字语义最相似的 10 篇文章」
- 「在这 1 亿张图片中找到与这张最相似的」
- 「推荐与该用户兴趣向量最接近的商品」

这些场景都需要将数据转换为高维向量（通常 128~4096 维），然后进行最近邻（Nearest Neighbor）搜索。向量数据库就是为此而生的专用基础设施。

---

# 二、核心概念详解

## 2.1 Collection（集合）

Collection 是 Milvus 中最基本的数据组织单位，类似于关系型数据库中的「表」。一个 Collection 包含一组具有相同 Schema（模式）的数据实体。

- **Schema 定义：** 包含主键字段、向量字段、标量字段。
- **动态 Schema：** 支持在创建时启用动态字段，允许插入未预定义的字段。
- **属性设置：** 可设置一致性级别、TTL（数据过期时间）等。

## 2.2 Entity（实体）

Entity 是 Collection 中的一条记录，类似于数据库中的一「行」。每个 Entity 包含：

- **主键：** 唯一标识，支持 INT64（自增或手动指定）和 VARCHAR。
- **向量字段：** 存储高维向量数据，支持 FLOAT_VECTOR、BINARY_VECTOR、FLOAT16_VECTOR、BFLOAT16_VECTOR、SPARSE_FLOAT_VECTOR。
- **标量字段：** 存储元数据，如 INT8/16/32/64、VARCHAR、BOOL、FLOAT、DOUBLE、JSON、ARRAY。

## 2.3 Partition（分区）

Partition 是 Collection 的逻辑子集，用于将数据按特定维度划分，提升查询效率和数据管理便利性。

- 最多支持 1024 个分区。
- 查询时可指定搜索特定分区，减少扫描范围。
- Partition Key：Milvus 2.2+ 支持，自动按指定字段的哈希值路由到不同分区，实现多租户隔离。

## 2.4 Index（索引）

索引是加速向量相似性搜索的关键数据结构。Milvus 支持十余种索引类型，通过牺牲少量精度换取检索速度的大幅提升。索引创建通常在数据写入完成后进行（但也可在写入前预建）。

- **索引时机：** 数据插入 → flush → create_index。
- **加载机制：** 索引创建后需调用 `load()` 将数据加载到内存才能查询。

## 2.5 Segment（段）

Segment 是 Milvus 内部的数据存储和管理单元。一个 Collection 的数据会被划分为多个 Segment：

- **Growing Segment（增长段）：** 实时接收新写入的数据，支持实时查询。
- **Sealed Segment（封存段）：** 当 Growing Segment 达到阈值后封存，由 Data Node 进行压缩和索引构建。封存后数据不可修改。

## 2.6 概念对比（与关系型数据库）

| Milvus 概念 | 关系型数据库 | 说明 |
|-------------|-------------|------|
| Collection  | Table       | 数据表 |
| Entity      | Row         | 一行数据 |
| Field       | Column      | 列/字段 |
| Schema      | Schema      | 表结构定义 |
| Partition   | Partition   | 分区 |
| Index       | Index       | 索引（类型不同） |
| Load        | Cache       | 将数据加载到内存供查询 |

---

# 三、系统架构（存算分离四层架构）

Milvus 2.x 采用云原生存算分离架构，遵循「数据平面与控制平面分离」的设计原则，整体分为四个层次。

## 3.1 第一层：接入层（Access Layer）

- **组件：** 一组无状态的 Proxy 节点。
- **职责：** 客户端请求的统一入口，负责请求验证、参数校验、结果聚合与后处理。
- **负载均衡：** 通过 Nginx、Kubernetes Ingress 等组件实现流量分发。
- **特点：** 完全无状态，可水平扩展，是系统吞吐量的关键入口。

## 3.2 第二层：协调服务层（Coordinator Service）

协调层是系统的「大脑」，负责集群拓扑管理、任务调度和一致性维护。包含四个组件：

- **Root Coordinator（根协调器）：** 管理 DDL/DCL 操作（创建/删除 Collection、Partition、Index），维护全局时间戳（TSO）。
- **Query Coordinator（查询协调器）：** 管理 Query Node 的拓扑，负责查询任务分发和负载均衡。
- **Data Coordinator（数据协调器）：** 管理 Data Node，负责数据压缩、索引构建等离线任务的调度。
- **Streaming Coordinator（流协调器）：** 管理 WAL 与 Streaming Node 的绑定，负责流数据的消费进度跟踪和服务发现。

## 3.3 第三层：工作节点层（Worker Nodes）

工作节点是实际执行数据处理和查询的层，全部为无状态设计，可弹性扩缩：

- **Query Node（查询节点）：** 从对象存储加载 Sealed Segment，执行历史数据的向量检索。支持 GPU 加速。
- **Data Node（数据节点）：** 负责离线任务，包括 Growing → Sealed 的数据转换、索引构建、数据压缩等。
- **Streaming Node（流节点，Milvus 2.5+）：** 处理实时增量数据，生成查询计划，将增量数据转为历史数据。提供分片级一致性保证和故障恢复。

## 3.4 第四层：存储层（Storage）

| 存储类型 | 技术选型 | 存储内容 |
|----------|----------|----------|
| 元存储（Meta Storage） | etcd | Collection Schema、消费位点、节点状态等元数据 |
| 对象存储（Object Storage） | MinIO / AWS S3 / Azure Blob | 日志快照、索引文件、标量与向量数据文件 |
| WAL 存储 | Woodpecker / Kafka / Pulsar | 预写日志，保证数据持久性和故障恢复 |

## 3.5 数据流转示例

### 搜索流程

```
客户端 → Proxy（请求验证）→ Streaming Node（查询 Growing Segment）→ Query Node（查询 Sealed Segment）→ Proxy（结果归并）→ 客户端
```

### 写入流程

```
客户端 → Proxy → Streaming Node → WAL（持久化）→ Growing Segment（实时可查）→ 封存 → Data Node（压缩+建索引）→ Query Node（加载查询）
```

## 3.6 架构核心优势

- **弹性扩展：** 计算节点（Query/Data/Streaming Node）无状态，可按需扩缩容。
- **高可用：** 任何组件故障均可通过重启或替换恢复，不丢失数据。
- **存储解耦：** 数据持久化在对象存储中，与计算进程完全分离。
- **混合查询：** 统一处理实时增量数据和封存历史数据。

---

# 四、索引算法全解析

索引是 Milvus 性能的核心。选择合适的索引类型和参数，对查询速度、召回率和内存占用有决定性影响。

## 4.1 浮点向量索引（最常用）

| 索引类型 | 原理 | 优点 | 缺点 | 适用场景 |
|----------|------|------|------|----------|
| FLAT | 暴力穷举，逐一比较所有向量 | 100% 召回率，精度最高 | 速度慢，O(N) | 百万级以下小数据集；作为基准测试 |
| IVF_FLAT | K-Means 聚类后只搜索最近的 nprobe 个簇 | 速度快，召回率高 | 内存占用大 | 百万~千万级；需要高精度 |
| IVF_SQ8 | IVF + 标量量化（float32→uint8） | 内存减少 75%，速度快 | 精度略降 | 内存有限；可接受少量精度损失 |
| IVF_PQ | IVF + 乘积量化（分解为子向量量化） | 内存极小，速度快 | 精度损失较大 | 亿级数据；内存极度有限 |
| HNSW | 多层图结构，从顶层逐层逼近 | 极高速，高召回 | 内存占用大，构建慢 | 低延迟高召回场景；内存充足 |
| HNSW_SQ | HNSW + 标量量化 | 速度+内存平衡 | 构建慢 | 兼顾速度和内存 |
| HNSW_PQ | HNSW + 乘积量化 | 内存小 | 精度有妥协 | 内存有限且需要图索引 |
| SCANN | 类似 IVF_PQ + SIMD 优化 | 极速，高召回 | 内存大 | 大规模高精度场景 |
| DiskANN | 基于 SSD 的 Vamana 图索引 | 内存占用极低 | 依赖 SSD I/O | 十亿级以上超大数据 |

## 4.2 二进制向量索引

| 索引类型 | 原理 | 适用场景 |
|----------|------|----------|
| BIN_FLAT | 二进制暴力穷举 | 小规模二进制向量；100% 精度 |
| BIN_IVF_FLAT | 二进制向量 + 聚类 | 大规模二进制向量；速度与精度平衡 |

## 4.3 稀疏向量索引

| 索引类型 | 原理 | 适用场景 |
|----------|------|----------|
| SPARSE_INVERTED_INDEX | 倒排索引，支持 BM25 评分 | 全文检索、BM25 排序 |
| SPARSE_WAND | WAND 算法优化的倒排索引 | 大规模稀疏向量；高性能全文检索 |

## 4.4 索引选择指南

根据数据规模和需求选择：

- **< 100 万条：** FLAT（精确）或 HNSW（极速）。
- **100 万 ~ 1000 万条：** IVF_FLAT 或 HNSW。
- **1000 万 ~ 1 亿条：** IVF_SQ8（内存优化）或 HNSW（速度优先）。
- **> 1 亿条：** IVF_PQ 或 DiskANN。
- **GPU 可用：** GPU_IVF_FLAT、GPU_CAGRA，可大幅加速索引构建和查询。

---

# 五、相似度度量标准

Milvus 支持多种相似度度量方式，不同度量适用于不同的业务场景：

| 度量类型 | 标识符 | 计算公式 | 值域 | 说明 |
|----------|--------|----------|------|------|
| 欧氏距离 | L2 | $\sqrt{\sum(x_i - y_i)^2}$ | [0, +∞) | 越小越相似；最常用的度量 |
| 内积 | IP | $\sum(x_i \cdot y_i)$ | (-∞, +∞) | 越大越相似；需先归一化向量 |
| 余弦相似度 | COSINE | $\cos(\theta) = \frac{A \cdot B}{|A| \cdot |B|}$ | [-1, 1] | 越大越相似；文本检索首选 |
| 汉明距离 | HAMMING | 不同位的个数 | [0, dim] | 二进制向量专用 |
| 杰卡德相似度 | JACCARD | $\frac{|A \cap B|}{|A \cup B|}$ | [0, 1] | 二进制向量；集合相似度 |

> [!tip] 选型建议
> 文本 Embedding 推荐使用 **COSINE**（余弦相似度）或 **IP**（归一化后等价）；图像特征推荐 **L2** 或 **COSINE**。

---

# 六、数据模型与 Schema 设计

## 6.1 字段类型

| 类型分类 | DataType | 说明 |
|----------|----------|------|
| 主键 | INT64 | 64 位整数，可自增 |
| 主键 | VARCHAR | 字符串主键 |
| 向量 | FLOAT_VECTOR | 32 位浮点向量（最常用） |
| 向量 | FLOAT16_VECTOR | 16 位浮点向量（省内存） |
| 向量 | BFLOAT16_VECTOR | Brain Float 16 向量 |
| 向量 | BINARY_VECTOR | 二进制向量 |
| 向量 | SPARSE_FLOAT_VECTOR | 稀疏浮点向量 |
| 标量 | BOOL / INT8/16/32/64 | 数值标量 |
| 标量 | FLOAT / DOUBLE | 浮点标量 |
| 标量 | VARCHAR | 字符串（需指定 max_length） |
| 标量 | JSON | JSON 对象字段 |
| 标量 | ARRAY | 数组字段 |

## 6.2 多向量字段

Milvus 2.4+ 支持在一个 Collection 中定义多个向量字段，实现多模态混合检索。例如同时存储文本向量和图像向量：

```python
fields = [
    FieldSchema(name="id", dtype=DataType.INT64, is_primary=True),
    FieldSchema(name="text_vec", dtype=DataType.FLOAT_VECTOR, dim=768),
    FieldSchema(name="image_vec", dtype=DataType.FLOAT_VECTOR, dim=512),
    FieldSchema(name="title", dtype=DataType.VARCHAR, max_length=200),
]
```

## 6.3 动态 Schema

创建 Collection 时启用 `enable_dynamic_field=True`，即可插入未在 Schema 中预定义的字段，数据以 JSON 格式存储在 `$meta` 字段中。

---

# 七、检索功能详解

## 7.1 ANN Search（近似最近邻搜索）

最基础的向量检索方式，给定一个或多个查询向量，返回 Top-K 个最相似的结果。

## 7.2 混合搜索（Hybrid Search）

将向量检索与标量条件过滤相结合，例如：「查找与目标向量相似且价格在 100~500 之间、分类为电子产品的前 10 条结果」。

```python
results = collection.search(
    data=[query_vector],
    anns_field="embedding",
    param=search_params,
    limit=10,
    expr='price >= 100 and price <= 500 and category == "electronics"'
)
```

## 7.3 范围搜索（Range Search）

按相似度阈值返回结果，适用于人脸验证（阈值判定是否同一人）、异常检测等场景。

## 7.4 多向量混合检索

同时对多个向量字段发起搜索，通过 Ranker（重排器）融合多个检索结果：

- **WeightedRanker：** 按权重加权融合多个分数。
- **RRFRanker（Reciprocal Rank Fusion）：** 基于排名倒数融合，无需归一化。

## 7.5 分组搜索（Grouping Search）

确保返回结果来自不同分组（如不同文档、不同用户），提升结果多样性。通过 `group_by_field` 参数指定。

## 7.6 全文检索

Milvus 2.5+ 支持原生全文检索功能，可使用 BM25 算法对标量文本字段进行关键词搜索，无需额外的搜索引擎。

## 7.7 删除与 Upsert

- **Delete：** 支持按主键或表达式删除数据。
- **Upsert：** Milvus 2.3+ 支持，对已有主键的数据执行插入或更新。

---

# 八、一致性模型

Milvus 支持四种一致性级别，适用于不同场景：

| 一致性级别 | 说明 | 延迟 | 适用场景 |
|------------|------|------|----------|
| Strong（强一致） | 读操作保证看到最新写入 | 最高 | 对数据一致性要求极高 |
| Session（会话一致） | 同一客户端保证读到自己写入的数据 | 中等 | 推荐默认选项；大多数场景 |
| Bounded（有界一致） | 读操作可能在一定时间窗口内看不到最新数据 | 较低 | 可容忍短暂延迟 |
| Eventually（最终一致） | 最终会一致，但不保证时间 | 最低 | 对实时性不敏感 |

---

# 九、Python SDK 实战指南

## 9.1 安装与连接

```python
# 安装
pip install pymilvus

# 连接 Milvus
from pymilvus import connections
connections.connect(alias="default", host="localhost", port="19530")

# 使用 Milvus Lite（本地轻量模式，无需部署服务端）
from pymilvus import MilvusClient
client = MilvusClient("./milvus_demo.db")
```

## 9.2 创建 Collection

```python
from pymilvus import FieldSchema, CollectionSchema, DataType, Collection

fields = [
    FieldSchema(name="id", dtype=DataType.INT64, is_primary=True, auto_id=True),
    FieldSchema(name="title", dtype=DataType.VARCHAR, max_length=200),
    FieldSchema(name="embedding", dtype=DataType.FLOAT_VECTOR, dim=768),
]

schema = CollectionSchema(fields, description="商品向量集合")
collection = Collection(name="products", schema=schema)
```

## 9.3 插入数据

```python
import random

titles = ["iPhone 15 Pro", "MacBook Air M3", "AirPods Pro 2"]
vectors = [[random.random() for _ in range(768)] for _ in range(3)]

collection.insert([titles, vectors])
collection.flush()  # 确保数据持久化到存储
```

## 9.4 创建索引

```python
# HNSW 索引
index_params = {
    "index_type": "HNSW",
    "metric_type": "COSINE",
    "params": {"M": 16, "efConstruction": 256}
}
collection.create_index(field_name="embedding", index_params=index_params)

# IVF_FLAT 索引
index_params_ivf = {
    "index_type": "IVF_FLAT",
    "metric_type": "L2",
    "params": {"nlist": 128}
}
collection.create_index(field_name="embedding", index_params=index_params_ivf)
```

## 9.5 加载数据并搜索

```python
# 加载到内存（必须！）
collection.load()

# 向量搜索
query_vector = [[random.random() for _ in range(768)]]
search_params = {"metric_type": "COSINE", "params": {"ef": 64}}

results = collection.search(
    data=query_vector,
    anns_field="embedding",
    param=search_params,
    limit=5,
    expr='title like "Mac%"'  # 标量过滤
)

for hits in results:
    for hit in hits:
        print(f"ID: {hit.id}, Distance: {hit.distance:.4f}")
```

## 9.6 使用 ORM 风格（MilvusClient）

```python
from pymilvus import MilvusClient

client = MilvusClient("http://localhost:19530")

# 快速创建
client.create_collection(
    collection_name="demo",
    dimension=768,
    metric_type="COSINE"
)

# 插入
client.insert("demo", data=[
    {"id": 1, "vector": [0.1]*768, "text": "hello"},
    {"id": 2, "vector": [0.2]*768, "text": "world"},
])

# 搜索
results = client.search(
    "demo",
    data=[[0.15]*768],
    limit=5,
    filter='text == "hello"'
)
```

---

# 十、性能优化最佳实践

## 10.1 索引优化

1. HNSW 是大多数场景的首选，设置 M=16~64，efConstruction=200~500。
2. IVF 系列适合超大规模数据，nlist 通常设为 $\sqrt{N} \times 4$（N 为数据量）。
3. 搜索参数 nprobe（IVF）或 ef（HNSW）越大精度越高但越慢，需要调优平衡。
4. 数据更新频繁时，避免使用 HNSW（建索引慢），可用 IVF_FLAT。

## 10.2 数据加载优化

1. 使用 `collection.load(partition_names=[...])` 仅加载需要的分区。
2. 使用 `collection.release()` 释放不再使用的 Collection 内存。
3. 大 Collection 可设置 `replica_number > 1`，提升查询并发能力。

## 10.3 写入优化

1. 批量插入（batch insert）远快于逐条插入，建议每批 1000~10000 条。
2. 合理设置 Collection 的 `shard_num`（分片数），通常等于 Query Node 数量。
3. 写入前不需要先建索引，可先大量写入再统一建索引。

## 10.4 内存规划

估算公式：

```python
# FLOAT_VECTOR 内存估算
memory_gb = 数据条数 × 向量维度 × 4 / (1024³)

# 例如：1000万条 × 768维
# = 10,000,000 × 768 × 4 / 1,073,741,824 ≈ 28.6 GB
# 使用 IVF_SQ8 可减少到约 7.2 GB
# 使用 IVF_PQ(m=128, nbits=8) 可减少到约 3.6 GB
```

---

# 十一、典型应用场景

## 11.1 RAG（检索增强生成）

最核心的应用场景。将文档切分、向量化后存入 Milvus，用户提问时检索最相关的文档片段，送入 LLM 生成答案。

- **典型流程：** 文档 → 切分 → Embedding 模型 → Milvus → 用户查询 → 检索 → LLM → 回答。
- **技术栈：** LangChain/LlamaIndex + Embedding 模型（BGE/OpenAI/text2vec）+ Milvus。

## 11.2 图像/视频检索

将图像/视频帧通过 CNN/ViT 模型提取特征向量，存入 Milvus 实现以图搜图。

## 11.3 推荐系统

将用户行为和物品特征向量化，通过向量相似性实现个性化推荐。

## 11.4 智能问答

将 FAQ 问答对向量化，用户提问时检索最相似的已有人工回答。

## 11.5 药物分子搜索

将分子指纹转为向量，实现相似分子结构的快速检索。

## 11.6 音频检索

将音频特征（如 MFCC、声纹）向量化，实现声音相似性搜索。

---

# 十二、Milvus vs 其他向量数据库

| 特性 | Milvus | Pinecone | Weaviate | Qdrant | Chroma |
|------|---------|----------|----------|---------|--------|
| 开源 | 是 | 否 | 是 | 是 | 是 |
| 存算分离 | 是 | 托管 | 否 | 否 | 否 |
| 分布式 | 是 | 托管 | 可选 | 是 | 否 |
| GPU 加速 | 是 | 否 | 否 | 否 | 否 |
| 稀疏向量 | 是 | 否 | 是 | 是 | 否 |
| 多向量混合检索 | 是 | 是 | 是 | 是 | 否 |
| 全文检索（BM25） | 是（2.5+） | 否 | 是 | 是 | 否 |
| Upsert | 是（2.3+） | 是 | 是 | 是 | 是 |
| 主要优势 | 性能强，功能全 | 易用 | 语义丰富 | 轻量高效 | 极简开发 |

---

# 十三、部署方案

## 13.1 Milvus Lite

本地轻量模式，嵌入到 Python 应用中，无需额外部署。适合开发测试和原型验证。

```python
from pymilvus import MilvusClient
client = MilvusClient("./milvus_demo.db")
```

## 13.2 Docker Compose

单机开发环境，包含所有组件的 Standalone 模式。

```bash
# 下载配置
wget https://github.com/milvus-io/milvus/releases/download/v2.6.0/milvus-standalone-docker-compose.yml -O docker-compose.yml

# 启动
docker compose up -d
```

## 13.3 Kubernetes（生产推荐）

使用 Helm Chart 部署集群模式，支持自动扩缩容和高可用。适用于生产环境。

```bash
# 添加 Helm 仓库
helm repo add milvus https://zilliztech.github.io/milvus-helm/
helm repo update

# 安装
helm install milvus milvus/milvus -f values.yaml
```

## 13.4 Zilliz Cloud（全托管）

Milvus 团队的商业全托管服务，零运维，自动扩缩容，提供免费层。适合不想管理基础设施的团队。

---

# 十四、常见问题与排查

## Q1：搜索前忘记 load() 怎么办？

会报错提示 Collection 未加载。调用 `collection.load()` 即可。也可以在创建 Collection 后立即 load，后续插入的数据会自动增量加载。

## Q2：如何查看数据量？

```python
collection.num_entities  # 总实体数（含未 flush）
```

## Q3：向量维度不一致会怎样？

插入时维度与 Schema 定义不匹配会直接报错。务必确保所有向量维度一致。

## Q4：Milvus 支持删除和更新吗？

Milvus 2.3+ 支持 Upsert 操作（按主键更新或插入）。Delete 操作支持按主键或表达式删除。但频繁的删除和更新会产生大量「墓碑标记」，影响性能。

## Q5：如何监控 Milvus？

Milvus 内置 Prometheus 指标暴露端点（默认 9091），可通过 Grafana Dashboard 可视化。关键指标包括：查询 QPS、延迟 P99、内存使用率、Segment 数量等。

## Q6：Milvus 的数据会丢失吗？

数据通过 WAL 持久化到对象存储，即使所有计算节点宕机，重启后数据可恢复。对象存储本身提供高持久性（如 S3 提供 99.999999999%）。

## Q7：如何备份数据？

使用 Milvus Backup 工具（milvus-backup），支持 Collection 级别的全量和增量备份，可备份到 S3 等对象存储。

---

# 十五、底层原理深入解析

本节深入 Milvus 的底层实现机制，补充前面章节中概念层面的介绍，揭示数据持久化、删除机制、一致性保证、时间戳系统和索引引擎的核心实现细节。

## 15.1 Segment 数据持久化机制

前面介绍了 Growing/Sealed Segment 的概念，这里深入 Sealed Segment 的物理存储结构。

Sealed Segment 被持久化到对象存储（MinIO/S3/Azure Blob）时，数据以**三种日志文件类型**组织：

| 文件类型 | 作用 | 内容 |
|----------|------|------|
| **binlog** | 列级二进制数据文件 | 每个 Segment 的每个字段（列）各生成一个 binlog 文件，存储向量数据和标量数据 |
| **deltalog** | 删除操作记录 | 记录该 Segment 内被删除的行的主键（Primary Key）和时间戳，即「墓碑」（tombstone） |
| **statslog** | 元数据统计信息 | 包含行数统计、主键布隆过滤器（Bloom Filter）、各字段的 min/max 统计值，用于查询剪枝优化 |

> [!important] 关键设计要点
> - **列级存储：** binlog 按字段/列拆分（而非按行存储），这使向量检索时只需加载向量列的 binlog，无需加载所有标量列，极大减少 I/O。
> - **索引文件独立：** 索引文件由 IndexCoord 异步构建，存储在对象存储中但不属于上述三种日志文件类型，是独立的文件类别。
> - **布隆过滤器加速：** statslog 中的主键布隆过滤器允许 Query Node 在执行标量过滤时快速判断某个主键「可能不存在」或「一定不存在」，避免不必要的 binlog 加载。

> 📌 来源：[Milvus 官方文档 - Segment](https://milvus.io/docs/segment.md)、[GitHub 源码 internal/storage/](https://github.com/milvus-io/milvus)

## 15.2 惰性删除与墓碑机制

Milvus 的删除操作采用**惰性删除**（Lazy Deletion）策略，而非即时物理删除。核心原因：Sealed Segment 是**不可变的**（immutable），一旦封存就不能原地修改。

### 删除流程

```
用户发起 Delete 请求
    ↓
删除记录写入 deltalog（包含被删除行的主键 + 时间戳）
    ↓
Query Node 加载 Sealed Segment 的 binlog 数据
    ↓
将 deltalog 转换为 delete bitmap（删除位图）
    ↓
搜索时：在结果中过滤掉 delete bitmap 标记的行
    ↓
Compaction 操作：物理合并删除行，生成新的 binlog（不含已删除数据）
```

### 核心机制

1. **墓碑记录：** 删除操作不直接修改 Sealed Segment 的 binlog，而是在 deltalog 中记录「墓碑」——被删除行的主键和时间戳。
2. **Delete Bitmap：** Query Node 在查询执行时，将 deltalog 中的墓碑转换为位图过滤器，搜索结果经过位图过滤后才返回给用户。官方术语为「delete bitmap」。
3. **Compaction 合并：** deltalog 中的墓碑通过定期 Compaction 操作被物理合并——生成新的 binlog（不含已删除数据行）和新的 deltalog，旧 Segment 被标记丢弃。

> [!warning] 性能影响
> 频繁的删除和更新会产生大量墓碑记录，增加 deltalog 体积和查询时的过滤开销。定期触发 Compaction 是维持性能的关键。

> 📌 来源：[Milvus 官方文档 - Segment](https://milvus.io/docs/segment.md)、[Milvus Deep Dive - Delete 机制](https://milvus.io/blog/deep-dive-5-how-delete-works-in-milvus.md)、[GitHub Issue #15764](https://github.com/milvus-io/milvus/issues/15764)

## 15.3 GuaranteeTs 一致性实现原理

前面第八节介绍了四种一致性级别的概念，这里揭示其底层实现机制——**GuaranteeTs（保证时间戳）**。

### 核心机制

每个搜索/查询请求都携带一个 **GuaranteeTs** 值。Query Node 在执行搜索前，将自己的**服务时间戳**（Service Timestamp，表示当前已处理数据的时间点）与 GuaranteeTs 对比：

- **GuaranteeTs > Service Timestamp：** Query Node 的数据尚未处理到请求要求的时间点，**等待**直到数据追上再执行搜索。
- **GuaranteeTs ≤ Service Timestamp：** Query Node 已有足够的数据，**立即执行**搜索。

### 一致性级别与 GuaranteeTs 的映射

| 一致性级别 | GuaranteeTs 取值 | 行为 |
|------------|------------------|------|
| **Strong** | 最新的系统时间戳 | 等待所有已写入数据被处理完毕再搜索 |
| **Bounded Staleness** | 当前时间戳 - 容忍窗口 | 允许一定时间窗口内的数据延迟 |
| **Eventually** | 极小值（如 1） | 不等待，直接在当前可用数据上搜索 |
| **Session** | 同一客户端最近写入的时间戳 | 保证客户端能读到自己刚写入的数据 |

> 📌 来源：[Milvus 官方文档 - Consistency](https://milvus.io/docs/consistency.md)

## 15.4 TSO 时间戳机制

Milvus 集群的全局时间戳由 **RootCoord** 作为 **Timestamp Oracle（TSO）** 提供，是实现 MVCC 快照隔离和分布式一致性的基础。

### 混合时间戳格式

TSO 采用**混合时间戳**（Hybrid Timestamp）格式，由物理部分和逻辑部分组成：

$$
\text{TSO} = (\text{physical\_millis} \ll 18) + \text{logical}
$$

| 组成部分 | 位数 | 说明 |
|----------|------|------|
| **physical_millis** | 高位 | Unix 毫秒时间戳 |
| **logical** | 18 位 | 同一毫秒内的逻辑计数器 |

- **逻辑部分 18 位：** 最大计数器值为 $2^{18} - 1 = 262143$，意味着同一毫秒内最多可分配 262143 个唯一时间戳。
- **ComposeTS 函数：** 源码中通过 `ComposeTS(physical, logical)` 函数组合时间戳，定义为 `(physical << logicalBits) + logical`，其中 `logicalBits = 18`。
- **全局单调递增：** TSO 保证在整个集群中时间戳单调递增且唯一，所有 DML 操作（插入、删除、搜索）都携带 TSO 时间戳。

### 设计意义

- **MVCC 快照隔离：** 搜索请求携带的时间戳 T 保证「看到所有 ts ≤ T 的写入，看不到 ts > T 的写入」。
- **分布式因果一致性：** 不同节点间的操作通过 TSO 建立因果顺序。
- **高吞吐：** 每毫秒 26 万+ 的唯一时间戳分配能力，满足高并发写入需求。

> 📌 来源：[GitHub 源码 pkg/common / typeutil](https://github.com/milvus-io/milvus)，常量定义：`logicalBits=18`、`logicalBitsMask=(1<<18)-1`

## 15.5 Knowhere 索引引擎架构

Milvus 的所有向量索引操作（构建、搜索、序列化）都通过 **Knowhere** 执行。Knowhere 是一个独立的 C++ 库，作为 Milvus 与底层索引库之间的桥梁。

### 架构定位

```
用户请求 → Milvus Proxy → QueryNode/DataNode
                              ↓
                         Knowhere（C++ 统一接口层）
                              ↓
              ┌───────────────┼───────────────┐
              ↓               ↓               ↓
           FAISS          hnswlib         DiskANN        cuVS（GPU）
         (IVF系列)       (HNSW图)      (Vamana图+SSD)   (GPU加速)
```

### IndexFactory 注册机制

Knowhere 通过统一的 **IndexFactory** 注册 API 封装所有第三方索引库：

- **IndexFactory** 是单例模式（`Instance()` 方法），提供 `Register()` 方法注册新索引类型。
- **注册宏：** `KNOWHERE_SIMPLE_REGISTER_GLOBAL` 展开为两个注册步骤：
  - `KNOWHERE_REGISTER_STATIC`：注册绑定到 indexType 的静态方法。
  - `KNOWHERE_REGISTER_GLOBAL`：注册索引适配器的工厂创建函数。
- **统一接口：** 所有索引类型对外暴露相同的 `Build()`、`Search()`、`Serialize()`、`Deserialize()` 接口，上层无需关心底层库的差异。

### 封装的第三方库

| 库 | 目录 | 负责的索引类型 |
|----|------|---------------|
| FAISS | `src/index/faiss/` | IVF_FLAT、IVF_SQ8、IVF_PQ、SCANN 等 IVF 系列 |
| hnswlib | `src/index/hnsw/` | HNSW、HNSW_SQ、HNSW_PQ 等图索引 |
| DiskANN | `src/index/diskann/` | DiskANN（Vamana 图 + SSD 存储） |
| cuVS | `src/index/gpu_cuvs/` | GPU_IVF_FLAT、GPU_CAGRA 等 GPU 加速索引 |

> 📌 来源：[Knowhere GitHub 仓库](https://github.com/zilliztech/knowhere)

---

# 十六、待深入探索的方向

以下主题在本次深度研究中未能获得足够权威的确认（声称被反驳或未验证），值得后续进一步探索：

| 方向 | 当前状态 | 说明 |
|------|----------|------|
| **Woodpecker WAL 实现** | 文档尚未权威 | Milvus 新原生 WAL（替代 Kafka/Pulsar），关于其复制模型、持久化机制、崩溃恢复的细节尚无权威文档确认，可能仍在演进中 |
| **Segment 生命周期状态** | 描述存在争议 | Segment 的精确状态转换（Growing → Sealed → Flushed/Dropped）及封存阈值条件在不同版本间有变化，需结合源码验证 |
| **Compaction 触发条件** | 版本敏感 | L0/Mixed/Clustering Compaction 的具体触发参数（minDeltaSize、minDeltaCount 等）随版本变化，且配置复杂 |
| **RootCoord TSO 持久化** | 未确认 | GlobalTSOAllocator 将时间戳窗口持久化到 etcd/TiKV 的保存间隔和重启单调性保证细节尚不明确 |
| **SIMD/GPU 搜索加速** | 未覆盖 | Knowhere 搜索路径中的 SIMD 指令优化和 GPU 加速具体实现细节未被本次研究确认 |
| **查询路由与分片机制** | 未覆盖 | Proxy 层如何将搜索请求路由到多个 QueryNode 和分片 Segment 的底层机制需进一步研究 |

> [!info] 研究方法论说明
> 十五、十六节内容基于多源对抗性验证研究（105 个搜索代理、23 个源、98 个声称提取、25 个声称验证），仅保留经 2/3 以上验证者确认的高置信度结论。被反驳的声称未纳入，以确保信息准确性。
