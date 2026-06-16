---
date: 2026-06-15
tags:
  - 概念
  - 记忆系统
  - Agent框架
  - 源码分析
  - OpenClaw
source: "[[OpenClaw与Hermes源码架构深度复盘]]"
---

# Dreaming 三阶段记忆晋升

> [!quote] 核心定义
> OpenClaw 的 Dreaming 机制借鉴人类睡眠的记忆整合过程，通过三个协作阶段（Light → REM → Deep）实现 Agent 后台自主整理和晋升记忆。**默认 opt-in 关闭**——关键工程取舍：错误晋升会污染所有后续对话（MEMORY.md 每轮注入 LLM）。

---

## 三阶段职责与本质

| 阶段 | 做什么 | 写入MEMORY.md? | 本质 | 人脑类比 |
|------|--------|----------------|------|----------|
| **Light Sleep** | 读取近期短期召回、每日记忆和脱敏会话，去重整理候选条目 | ❌ 仅整理候选 | 物料准备 | 睡前快速过一遍今天的事 |
| **REM Sleep** | 提取反复主题+选候选"潜在真理"+反馈Deep排名权重 | ❌ 仅提取信号 | 抽象思考 | 做梦时大脑抽象推演"这几件事有共同模式" |
| **Deep Sleep** | 加权评分+阈值门控+回源验证后写入持久记忆 | ✅ 唯一写入路径 | 固化决策 | 皮层将稳固事实固化为长期记忆 |

> [!important] 关键洞察
> **REM 找"稳固事实"，Deep 找"值得晋升到每轮可见的稳固事实"**——两个目标不同。REM 更偏重相关性（0.45 vs 0.30），Deep 有 diversity + recency 因为不能让陈旧/单一视角永久驻场。

---

## Deep 阶段：6 信号加权评分

```
score = 0.24 × frequency      命中次数
      + 0.30 × relevance      召回质量（权重最大）
      + 0.15 × diversity      query/天 多样性
      + 0.15 × recency        时间衰减新鲜度（半衰期14天）
      + 0.10 × consolidation  多天复现强度
      + 0.06 × conceptual     概念标签密度
      + Light boost (≤ +0.06) 浅睡命中加成
      + REM   boost (≤ +0.09) REM命中加成
```

## Deep 阶段：3 重门禁（必须全部通过才晋升）

| 门禁 | 阈值 | 防什么 |
|------|------|--------|
| 总分 ≥ 0.75 | 防一次召回特别准（偶然高分） |
| 命中 ≥ 3 次 | 防同一query反复命中（偶然高频） |
| 不同query ≥ 2 | 防召回质量差（偶然广覆盖） |

> **三个都通过 = 真正"重要"** ✅ 单独任何一个通过都不够。

**防 stale 设计**：晋升前重新读 daily 文件 hydrate——删了 daily 笔记 = 自动撤回候选。

---

## REM 阶段：置信度公式

```
confidence = averageScore   × 0.45   // 召回质量（权重最大！）
           + recallStrength × 0.25   // 次线性饱和 log1p(recall)/log1p(6)
           + consolidation  × 0.20   // 3天饱和 min(1, recallDays/3)
           + conceptual     × 0.10   // 6标签饱和 min(1, conceptTags/6)
```

过滤：confidence ≥ 0.45（远宽松于Deep的0.75）
去重：相似度 0.88 阈值
截断：top 3

## REM vs Deep 公式对比

| 信号 | REM truth 置信度 | Deep 晋升评分 |
|------|-----------------|---------------|
| **Relevance** | **0.45** | 0.30 |
| **Frequency** | 0.25 | 0.24 |
| **Consolidation** | **0.20** | 0.10 |
| **Conceptual** | 0.10 | 0.06 |
| **Diversity** | **无** | 0.15 |
| **Recency** | **无** | 0.15 |
| 阈值 | ≥ 0.45（宽松） | ≥ 0.75（严格） |

**深刻差异**：REM 不看 diversity 和 recency——只关心内在质量。Deep 有这两个维度因为永久固化不能让陈旧/单一视角永久驻场。

---

## 为什么 REM boost (+0.09) > Light boost (+0.06)

- **Light** = "这条东西被看到过" → 弱信号
- **REM** = "这条东西被Agent主动识别为可能是真理" → 强信号

---

## 记忆双层流转架构

```
4路径流入                    1路径晋升
┌──────────┬──────────┬──────────┬──────────┐
│ /new     │Compaction │LLM主动   │Dreaming  │
│ 触发     │Pre-Flush  │Write     │摄入      │
│(用户)    │(自动)     │(LLM自决) │(Dreaming)│
└──────────┴──────┬───┴──────────┘
                  ▼
      【memory/YYYY-MM-DD.md】（召回层/pull）
                  │
                  ▼ Dreaming 晋升 ⭐⭐⭐
      【MEMORY.md】（静态层/push，每轮注入LLM context）
```

| 维度 | 每天 memory/YYYY-MM-DD.md | 全局 MEMORY.md |
|------|---------------------------|----------------|
| 属于哪层 | 召回层（pull） | 静态层（push） |
| 注入LLM? | ❌ 不自动注入 | ✅ 每轮注入 |
| 写入触发 | 多种来源 | **只有Dreaming Deep阶段** |
| 类比 | 海马体的短期记忆 | 皮层的长期记忆 |

> [!important] 双层之间的管道默认关闭
> 用户显式启用 Dreaming 前，每天 memory 永远不会自动晋升。Dreaming 是两层之间的"晋升管道"。

---

## 演进方向

1. **与Session轮换联动**：session使用率达60%时触发Light sweep，Handoff时作为新session暖启动
2. **REM输出关系图**：当前buildRemReflections提取扁平concept tags → 升级为实体关系三元组 → Deep晋升结构化知识而非孤立事实
3. **增加遗忘机制**：MEMORY.md中长期未被召回的条目 → 定期标记"待验证" → 下次对话主动确认 → 模拟人脑遗忘曲线

### 千人千面潜力

Dreaming 的 6 信号评分完全由用户交互行为驱动（频率/相关性/多样性），只要加一层 `user_id` 维度隔离，同一套算法 + 不同用户行为模式 = 每个用户各自的长期记忆。**算法不变，数据区分，千人千面自然涌现**。

### 情景→语义→程序性记忆转化

2026年Mem0定义三类记忆：**情景**（发生了什么）、**语义**（知道什么）、**程序性**（如何做事）。Dreaming三阶段恰好覆盖这条转化链路——Light摄入情景，REM提取跨日反复主题形成"潜在真理"（语义），Deep将通过评分门控的真理固化（语义+程序性）。如果Deep晋升时打类型标签（fact/preference/procedure），就能区分"用户住北京"（语义）和"用户习惯先列大纲再写正文"（程序性）——后者就是**用户级Skill的自动沉淀**。

---

## Auto Capture 触发规则

源码 `extensions/memory-lancedb/index.ts` 的 `MEMORY_TRIGGERS`：

```
"remember" / "记住" / "记下"     → 用户明确要求记忆
"prefer" / "like" / "hate"       → 情感偏好
"+8613800000xxxx"                → 电话号码（10位以上数字）
"user@example.com"               → 邮箱
"my X is" / "is my"              → 所有权声明
"I like / prefer / hate / want"  → 偏好动词
"always / never / important"     → 强调词
"我喜欢 / 我偏好 / 决定 / 重要"  → 中文触发词
```

触发后还有安全过滤（`shouldCapture`）：跳过 prompt injection 载荷、跳过 Agent 自己生成的内容（含 markdown/emoji 特征）、长度限制（10字符 < 内容 < maxChars）。

捕获后自动分类（`detectCategory`）为：`preference` | `decision` | `entity` | `fact` | `other`

---

## Memory 引擎三选项对比

| 引擎 | 后端 | 存储方案 | 特点 |
|------|------|----------|------|
| **memory-core**（默认） | builtin（内置） | 每 Agent 一个 SQLite：FTS5全文+可选sqlite-vec向量；CJK trigram分词 | 零依赖，开箱即用 |
| **memory-core** | qmd（可选） | 外挂 QMD sidecar（Bun+node-llama-cpp独立二进制） | 额外提供reranking、查询扩展、索引工作区外路径；不可用时自动降级到builtin |
| **memory-lancedb**（第三方） | — | LanceDB嵌入式向量数据库 | 向量检索性能更强，auto-capture+auto-recall生命周期钩子 |

三者都支持混合搜索（BM25+向量相似度），通过 `plugins.slots.memory` + `memory.backend` 两层配置切换。

---

## Active Memory Recall 插件

独立插件，每次对话前运行**阻塞式记忆子Agent**（15秒超时）：

1. 读取当前对话上下文（支持 `message` / `recent` / `full` 三种查询模式）
2. 搜索记忆库找到相关记忆
3. 生成 ≤220 字符摘要，以**隐藏Prompt Prefix**注入
4. 结果缓存15秒

支持 6 种 prompt 风格：`balanced` | `strict` | `contextual` | `recall-heavy` | `precision-heavy` | `preference-only`

---

## 相关笔记

- [[OpenClaw架构深度解析]] — OpenClaw 的完整架构剖析
- [[记忆系统]] — 短期+长期记忆的通用理论框架
- [[记忆到技能转化]] — L1记住→L2总结→L3形成技能的三级递进
- [[上下文预算与资源分配]] — OpenClaw 10种稀缺资源量化
- [[OpenClaw vs Hermes 架构对比]] — 两套框架的正面对比
- [[上下文焦虑症与自我评估偏差]] — 长周期任务的认知偏差
- [[Bootstrap截断策略与子Agent Allowlist]] — 子Agent上下文裁剪
