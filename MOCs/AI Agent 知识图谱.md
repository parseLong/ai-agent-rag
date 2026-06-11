
> 系统性 AI Agent 知识管理体系——从"使用者"蜕变为"构建者"。
> 按主题组织核心概念，按章节衔接学习路径，按案例连接工程实践。

---

## 🗺️ 学习路线

> 系统化学习 AI Agent 的完整路线——从零基础到多智能体系统构建。

- [[Awesome Agentic AI 知识地图]] — 8阶段学习路线导航（Stage 0-8）
  - [[Stage 0 — 基础准备（Foundations）]] → [[LLM 基础]] → [[Stage 2 — 提示工程（Prompt Engineering）]] → [[Stage 3 — 工具使用与第一个 Agent]] → [[Stage 4 — Agent 框架（Agent Frameworks）]] → [[Stage 5 — Claude Code 生态]] → [[Stage 6 — 记忆与 RAG]] → [[Stage 7 — 多智能体与生产部署]] → [[Stage 8 — Agent Interfaces]]
- [[环境搭建指南]] — 30-45分钟从零搭建开发环境
- [[Agent 课程精选]] — 系统化课程推荐

---

## 🤖 智能体基础

### 定义与分类
- [[智能体（Agent）]] — AI Agent 的定义、核心组件与与传统 AI 的区别
- [[智能体循环]] — While 循环机制：LLM Call → 工具调用 → 结果反馈
### 通用Agent产品
- [[Manus]] — Monica，多智能体协同+虚拟机沙盒+人机协作
- [[本地Agent（Claw）]] — 运行在本地客户端的Agent，直接操控工作环境
### 智能体范式
- [[CoT]] — 思维链推理，显式推理步骤提升复杂任务性能
- [[ToT]] — 思维树探索，从线性推理到树状搜索
- [[ReAct 范式]] — Reasoning + Acting，Thought-Action-Observation 循环
- [[Reflection 范式]] — Self-Refine 与 Reflexion，反思驱动改进
- [[LATS]] — 语言Agent树搜索，统一推理+规划+反思
- [[ReWoo]] — 无需观察的规划，一次性生成工具链
- [[Plan-and-Execute]] — 规划+重规划，动态调整计划
- [[LLM Compiler]] — DAG并行执行，类似处理器乱序执行
- [[Plan-and-Solve]] — Plan-Execute-Replan 模式
- [[智能体发展史]] — 从符号主义到 LLM 驱动的智能体演进
- [[Agent技术范式演变（2023-2026）]] — 近代Agent四阶段演进+六维度范式重构
- [[亚符号主义 AI]] — 隐式知识表示
- [[神经符号主义 AI]] — 符号推理与神经网络的优势互补
- [[Agent 设计架构总览]] — 涵盖 17 种核心 Agent 架构模式
---

## 🧠 大语言模型

### 基础概念
- [[Transformer 架构]] — 现代 LLM 的基础架构
- [[注意力机制]] — 自注意力、CSA/HCA 混合稀疏注意力
- [[提示工程]] — 与 LLM 有效沟通的艺术
- [[Token]] — 模型处理文本的最小单元
- [[上下文窗口]] — 模型能处理的最大 Token 数
- [[长上下文]] — Agent 时代的核心基础能力
- [[GPT]] — 生成式预训练 Transformer 系列

### 模型与训练
- [[LLM 训练流程]] — 从预训练到后训练的完整流水线（预训练→SFT→微调→RLHF→GRPO→Post-train）
- [[DeepResearch]] — 端到端研究型语言模型，模型即产品典范

### 推理优化 
一旦系统运行起来，它们就需要快速且经济。此步骤侧重于学习性能和成本效率。 
- [[量化 (GGUF, GPTQ, AWQ) ]]
- [[服务引擎 (vLLM, TGI, llama.cpp) ]]
- [[KV 缓存 ]]
- [[Flash Attention ]]
- [[推测解码]]

### 部署 
如果模型只停留在笔记本中，它们就毫无用处。
• [[GPU 调度 ]]
• [[云平台 (AWS Bedrock, GCP Vertex AI) ]]
• [[Docker, Kubernetes ]]
• [[FastAPI, 流式传输 (SSE)]]

---

## 🛠️ 工具与框架

### 低代码平台
- [[Dify]]
- [[Coze]]
- [[n8n]]

### 开发框架
- [[LangGraph]] — State/Node/Edge 图结构工作流
- [[AutoGen]]
- [[AgentScope]]
- [[CrewAI]]
- [[MetaGPT]]
- [[Hermes Agent 框架]] — Python 单体"自我改进型 AI Agent"，技能自创建闭环 + Smart Approval + 8 种沙箱

### 工具与协议
- [[工具调用（Function Calling）]] — 让LLM从"生成文本"到"调用工具"的关键技术
- [[MCP]] — Host-Client-Server 架构，连接外部工具（Function Calling的标准化升级版）
- [[A2A]] — Agent-to-Agent 通信协议
- [[AG-UI]] — Agent↔用户界面的交互协议，实时事件流+人机协作
- [[ANP]]

### ECC 深度参考
- [[ECC使用指南]] — Skills、Hooks、Subagents、MCPs、Plugins 配置与高级模式
- [[ECC智能体设计模式]] — 60+ 专业智能体的设计范式与编排模式
- [[Claude Code安全实践]] — 智能体安全完整指南：攻击向量、沙箱、清理、审批、可观测性
- [[Claude Code Skills开发指南]] — Skill 创建架构、分类、写作技巧、测试验证
- [[Claude Code Hooks与Rules]] — Hooks 生命周期、Rule 规则体系、记忆持久化
- [[ECC架构设计与Token优化]] — 参考架构、Token 优化、跨平台适配、自改进循环

---

## 🧩 核心技术

### 记忆与检索
- [[记忆系统]] — 短期5种策略+长期3类架构+市场方案对比+核心挑战
- [[记忆到技能转化]] — L1记住事实→L2总结规则→L3形成技能的三级递进学习框架
- [[RAG]] — 分块策略+嵌入模型+检索优化+Self-RAG/CRAG/RAGAS评估
- [[向量存储]] — Faiss / [[Milvus]] / Pinecone / weaviate / pgvector 选型与 HNSW / IVF 算法
- [[短时记忆]]
- [[长时记忆]]

### 上下文工程
- [[上下文工程]] — 从提示工程到上下文工程的范式演进，结合 Claude Code 和 OpenClaw 实践的总览
- [[上下文注入]] — 将外部信息注入到 Prompt 中，含 Claude Code 三管道与 OpenClaw Context Engine 契约对比
- [[上下文压缩]] — Claude Code 五层递进压缩 + OpenClaw 三级 Compaction，从预防到紧急的分层响应
- [[上下文窗口管理]] — Claude Code 弹性状态机(7个Continue站点) + OpenClaw 预算贯穿 Runtime
- [[上下文腐蚀与注意力衰减]] — Context Rot / Lost in the Middle / 上下文焦虑症的理论根基
- [[上下文重置与交接]] — Context Reset vs Compaction，结构化 Handoff，两阶段 Session 轮换
- [[上下文预算与资源分配]] — Token Budget 行为塑形 + OpenClaw 10种稀缺资源量化 + JIT 检索
- [[GSSC流水线]] — Gather-Select-Structure-Compress 通用上下文构建模式

### 多智能体
- [[多智能体协作]] — 专业化分工、三大挑战、三大协作架构
- [[多智能体协作模式]] — 7种核心协作模式（主从/接力/讨论/层级/竞争/评估/路由）+组合策略
- [[智能体通信协议]] — MCP / A2A / ANP / Function Calling 对比与三层定位
- [[角色分配]] — Multi-Agent 角色设计原则
- [[任务分解]] — 任务拆解策略与依赖处理

---

## 📊 评估与优化

### 可观测性 
此步骤帮助您跟踪质量、延迟和成本。
• [[追踪 (LangSmith, Langfuse, Arize Phoenix) ]]
• [[延迟 (TTFT) ]]
• [[令牌使用 ]]
• [[成本跟踪]]

### 度量驱动演进
- [[度量驱动演进闭环]] — Harness 的自进化机制：采集→标注→回流→验证闭环，与记忆到技能转化的精确对应

### 评估指标与基准测试
- [[智能体评估]] — 综合评估指南：准确率/召回率/F1/用户满意度 + AgentBench/WebArena/SWE-bench

---

## 🧬 工程实践与案例

### 驾驭工程
- [[驾驭工程]] — 控制层全景：LLM 是引擎，驾驭层是方向盘和安全带，三次进化历程+起源故事+三层技术定义+行业案例+数据对比+路线之争+护栏悖论+七个杠杆
- [[成为 Harness Engineering 时代的工程师]] — 四步进阶路线图：Prompt基础 → Context工程 → Agent系统设计 → 动态Harness思维
- [[工程设计与安全]] — 构建者信条、代码-文档同步、工具描述工程、输出解析、重试恢复、护栏钩子、流式渲染
- [[系统提示词工程]] — 静态段+动态段+缓存边界+行为塑形
- [[MCP 协议实现]] — 外部工具连接的实现层
- [[MCP 服务器集成]] — MCP 服务器配置与集成指南
- [[Skill 开发指南]] — Skill 创建架构与分类
- [[Skills技能]] — 人类经验沉淀为智能体能力
- [[运行时环境（Runtime）]] — Agent的专属Workspace：从无状态到有状态隔离运行时
- [[Prompt与Context工程（三次进化）]] — Prompt 与 Context 工程的三次进化历程
- [[Harness Engineering深度解析——从有序驾驭无序的工程哲学]] — 六层深度解析：核心哲学、关键悖论、概念谱系、交叉验证、批判阅读与实践启示
- [[长程Coding自动化实践经验]] — 四轮迭代实战：两条直觉经验、上下文甜区、Contract握手、Rubric四层评测(L1-L4)、quality-vision品质锚定、Skills开发Harness
- [[工程技术：在智能体优先的世界中利用 Codex]] — OpenAI百万行代码实验：人类掌舵/智能体执行、AGENTS.md渐进式披露、品味即代码、熵与垃圾回收
- [[模型不是关键，Harness 才是]] — 行业数据对比+起源故事+拐杖论+路线之争+护栏悖论+七个配置杠杆

### Claude Code 源码架构
- [[Claude Code 源码架构]] — 生产级智能体的完整架构剖析
  - 整体架构 → System Prompt 装配 → Query Pipeline → 工具系统 → 子代理系统 → Bridge 桥接
  - 命令系统 → 状态管理 → 功能开关 → 记忆压缩 → 权限与安全 → 守护进程模式
  - 序言组合根模式 → 分层安全防御 → 扩展子系统与十层防御 → 工具 Prompt 与自修复
- [[Claude Code 架构解析]]

### 架构模式
- [[MTP架构]] — 多词元预测架构

### 项目案例
- [[gstack 知识地图]] — AI 辅助开发工具栈的完整知识体系


---

## 🏗️ 第二大脑结构

```
04-Permanent/           # 永久笔记、概念卡片
├── 智能体基础/         # Agent定义、分类、发展史、产品
├── 智能体范式/         # ReAct、Reflection、CoT、Plan-and-Solve等
├── 核心技术/           # 核心技术体系
│   ├── 记忆与检索/     # 记忆系统、RAG、向量存储
│   ├── 上下文工程/     # 注入、压缩、窗口管理
│   └── 多智能体/       # 协作、通信、角色、任务分解
├── 工具与框架/         # LangGraph、AutoGen、MCP、A2A等
├── 大语言模型/         # Transformer、LLM训练流程、提示工程
├── 评估与优化/         # 智能体评估（指标+基准测试）
├── 工程实践/           # 驾驭工程、Claude Code源码架构、MCP实现
├── 应用场景/           # OpenClaw、Super App
├── 知识基础/           # ML/DL/NLP基础、BERT/CNN、术语表
├── 独立概念/           # AI伦理、范式转变、组织文化
└── 综合/               # 访谈合集、深度整理、工程进化、范式演变
```

---

## 🏷️ 标签体系

### 按知识类型
- `#概念` - 核心概念定义
- `#原理` - 底层原理
- `#方法` - 方法论、技巧
- `#工具` - 工具、框架、平台
- `#实践` - 实战经验、项目
- `#案例` - 具体案例分析

### 按学习阶段
- `#学习笔记` - 学习过程中的记录
- `#待整理` - 需要进一步整理的笔记
- `#概念卡片` - 精炼的核心概念
- `#永久笔记` - 经过验证的知识

### 按主题领域
- `#智能体` - Agent 通用知识
- `#LLM` - 大语言模型
- `#记忆系统` - Memory & RAG
- `#多智能体` - Multi-Agent
- `#评估` - Evaluation
- `#训练` - Training

---

## 💡 使用指南

1. **新建笔记**：从 Templates/ 中选择合适的模板
2. **记录学习**：每学完一章，在对应章节 MOC 下创建笔记
3. **建立链接**：通过 `[[双链]]` 建立知识之间的关联
4. **定期回顾**：利用 Graph View 查看知识图谱
5. **每日复盘**：使用每日笔记模板记录学习进度

---

## 🔭 扩展视野：AI路线对比

> 与主流LLM路线不同的观点视角，帮助理解AI发展的多元路径。

### 路线对比
- [[世界模型与LLM对比]] — 谢赛宁/ Yann LeCun 的 World Model 路线 vs 主流 LLM 路线
  - 核心观点：LLM是"反Bitter Lesson"，语言是"鸦片"
  - 世界模型出口：AI眼镜、机器人
  - AMI Labs：10亿美元融资的"草根联盟"模式

---

## 📚 知识基础

> AI大模型的必备基础知识与核心术语，从"使用者"到"构建者"的知识根基。

### 基础学科
- [[机器学习基础]] — 监督/无监督/强化学习的核心概念
- [[深度学习]] — 神经网络的核心架构与训练方法
- [[自然语言处理（NLP）]] — NLP 的核心任务与技术演进
- [[AI大模型基础知识]] — 数学基础（线性代数/微积分/概率统计）与CS基础

### 模型基础
- [[BERT]] — 双向编码器表示，预训练-微调范式奠基者
- [[CNN]] — 卷积神经网络，视觉领域的基石
- [[大模型核心技术]] — Transformer、分布式训练、高效推理
- [[大模型核心术语]] — AI大模型领域完整术语表

## 🛠️ 动手练习

- [[Stage 1 练习 — 跨provider调用与错误处理]]
- [[Stage 3 练习 — 工具使用与 ReAct]]
- [[Stage 4 练习 — Agent框架对比]]
- [[Stage 5 练习 — Tool Calling Tutor]]
- [[Stage 6 练习 — 记忆与RAG]]
- [[Stage 7 练习 — 多智能体与部署]]

---

## 📡 行业访谈精选

> 来自一线AI研究者的深度访谈，提取核心洞见。

### 杨植麟 (月之暗面/Kimi) — 访谈合集
- [[杨植麟访谈合集]] — 5篇访谈深度整理：AI产品范式、Agent定义与核心能力、泛化评估与训练、多智能体与AGI分级、推理与Test-Time Scaling

### 姚顺雨 (OpenAI) — Language Agent先驱
- [[Language Agent 概念]] — 语言智能体的核心定义
- [[ReAct 范式]] — 协同推理与行动
- [[泛化能力与语言]] — 语言作为泛化工具
- [[Super App与交互方式]] — 创业机会与未来形态

### 罗福莉 (小米) — Agent时代亲历者
- [[AI范式转变]] — 从Chat到Agent的时代转变
- [[OpenClaw架构深度解析]] — 开源 Agent 框架全解析（核心价值 + 源码架构 + 部署指南 + 横向对比）
- [[Skills技能]] — 人类经验沉淀
- [[长上下文]] — Agent时代核心基础
- [[MTP架构]] — 多词元预测架构
- [[平权文化与组织创新]] — 组织创新实践

### 姚顺宇 (Anthropic/Google) — RL训练实践
- [[Claude3.7与后训练Scaling]] — 后训练分水岭
- [[AI研究员特质]] — 行业最重要的特质
- [[Anthropic组织文化]] — Top-down组织与执行力

### 访谈索引
- [[访谈概念卡片索引]] — 所有访谈概念卡片汇总