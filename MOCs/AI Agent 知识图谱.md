
> 系统性 AI Agent 知识管理体系——从"使用者"蜕变为"构建者"。
> 按主题组织核心概念，按章节衔接学习路径，按案例连接工程实践。
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
- [[Agent架构演化总览]] — 控制流设计视角：17种架构的本质不是 prompt engineering，而是控制流设计
---

## 🧠 大语言模型

### 基础概念
- [[Transformer 架构]] — 现代 LLM 的基础架构
- [[注意力机制]] — 自注意力→Prefill/Decode拆分→CSA/HCA混合稀疏注意力→闪电索引→三层处理
- [[注意力残差机制]] — Kimi Attn-Res/Block Attn-Res，与mHC从不同侧面解决残差问题
- [[mHC（多流约束残差连接）]] — 标准残差→HC→mHC演进，双随机矩阵约束+非对称约束设计
- [[MoE架构]] — 路由/门控机制、稀疏激活原理（3%激活率）、细粒度计算通信重叠、MoE vs Dense对比
- [[Muon优化器]] — 梯度正交化数学直觉+RMSNorm前置vs LayerNorm+与分布式训练冲突
- [[提示工程]] — 与 LLM 有效沟通的艺术
- [[Token]] — 模型处理文本的最小单元
- [[上下文窗口]] — 1M上下文三大消耗场景+Prefill/Decode成本拆分
- [[长上下文]] — Agent 时代的核心基础能力
- [[GPT]] — 生成式预训练 Transformer 系列

### 模型与训练
- [[LLM 训练流程]] — 从预训练到后训练的完整流水线（预训练→SFT→微调→RLHF→GRPO→Post-train）
- [[DeepSeek V4]] — 1.6T/49B MoE + 1M上下文，三项创新互锁逻辑（mHC/CSA-HCA/Muon）+ V4 vs V3.2演进
- [[DeepSeek V4 Infra 层优化]] — Infra全景索引，指向5个子笔记
- [[FP4量化感知训练]] — 分模块精准量化（MoE→FP4、CSA索引→FP4、打分→BF16）、QAT vs PTQ对比
- [[DeepResearch]] — 端到端研究型语言模型，模型即产品典范

### 推理优化
- [[量化 (GGUF, GPTQ, AWQ) ]]
- [[服务引擎 (vLLM, TGI, llama.cpp) ]]
- [[KV 缓存]] — Prefill/Decode角色 → PagedAttention → V4异构KV Cache架构 + SWA三档取舍 + 分类持久化
- [[Flash Attention ]]
- [[推测解码]]

### 部署与工程
- [[TileLang]] — 数据流逻辑与调度策略解耦的算子开发语言，可移植多硬件
- [[批无关性与计算确定性]] — bit级一致输出 + DeepGEMM + 加法括号结构钉死
- [[DeepSeek V4 训练与推理框架优化]] — Muon兼容分布式、mHC调优(6.7%)、跨机器压缩、单tensor级重算
- [[GPU 调度 ]]
- [[云平台 (AWS Bedrock, GCP Vertex AI) ]]
- [[Docker, Kubernetes ]]
- [[FastAPI, 流式传输 (SSE)]]

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
- [[OpenClaw架构深度解析]] — TypeScript 微内核"本地优先 Agent OS"，Gateway + Plugin SDK + Dreaming 三阶段记忆晋升
- [[OpenClaw vs Hermes 架构对比]] — 微内核 vs 单体、平台架构师 vs 个人开发者，目标受众不同的成熟取舍

### 源码架构深度分析
- [[OpenClaw与Hermes源码架构深度复盘]] — 源码级深度解析，Gateway 5大角色 + Channel 25+ Adapter + Auth Profile + FailoverError + Dreaming算法 + GAN-like架构 + Harness空白区
- [[Dreaming三阶段记忆晋升]] — Light→REM→Deep 三阶段加权晋升，6信号评分+3重门禁+REM置信度公式+千人千面潜力+情景→语义→程序性转化
- [[Channel Plugin 25+ Adapter 契约]] — 完整IM域协作单元，25+可选槽位+5种交互模式+跨Channel会话迁移(Docking)+反向工具
- [[Auth Profile与FailoverError]] — 带健康状态凭证管理+13种闭合枚举错误分类，可恢复错误处理静态可证明
- [[Smart Approval三态审批]] — 先LLM分诊再叫人(APPROVE/DENY/ESCALATE)，LLM辅助安全审查员层
- [[CLI Backend双路径执行]] — 把CLI当可替换执行backend+反向MCP注入+双向连接(MCP/ACP/HTTP三层暴露)
- [[Session Search (Hermes)]] — SQLite FTS5双索引+LLM摘要的跨会话搜索，翻日记本式回忆
- [[Lanes分车道并发管控]] — 4车道独立队列防cron自递归死锁+权限边界，按调用来源决定安全策略
- [[Bootstrap截断策略与子Agent Allowlist]] — head70%+tail20%砍中间+子Agent5文件保留+截断告警注入LLM自感知
- [[技能自创建闭环与渐进式披露]] — Agent自主创建技能+三级渐进式披露(30+倍token节省)+4级信任+6类威胁
- [[GAN-like多智能体架构与Sprint Contract]] — Planner/Generator/Evaluator+对抗性验证+结构化Handoff5层
- [[上下文焦虑症与自我评估偏差]] — 信息丢失+注意力崩溃+提前收工+盲目自信+幻觉闭环，两个框架的空白区
- [[Hermes Prompt缓存冻结快照策略]] — system_and_3策略+4断点+~75%成本节省+记忆延迟一个会话

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
- [[Dreaming三阶段记忆晋升]] — OpenClaw Light→REM→Deep 加权晋升，6信号+3重门禁+千人千面潜力+情景→语义→程序性转化
- [[Session Search (Hermes)]] — SQLite FTS5+LLM摘要的跨会话搜索，翻日记本式回忆
- [[技能自创建闭环与渐进式披露]] — Hermes Agent自主创建技能+三级渐进式披露(30+倍token节省)+4级信任+6类威胁
- [[RAG]] — 分块策略+嵌入模型+检索优化+Self-RAG/CRAG/RAGAS评估
- [[向量存储]] — Faiss / [[Milvus]] / Pinecone / weaviate / pgvector 选型与 HNSW / IVF 算法
- [[Mermaid任务画布]] — 用Mermaid Flowchart作为Agent的无限画布——把离散摘要组织成任务拓扑图，Flowchart效果比StateDiagram好~15%
- [[TencentDB Agent Memory]] — 四级折叠架构(Raw→JSONL→MMD→Metadata)的生产化实践，最高节省61% Token、提升52%成功率

### 上下文工程
- [[Context Engineering]] — 从提示工程到上下文工程的范式演进，结合 Claude Code 和 OpenClaw 实践的总览
- [[上下文注入]] — 将外部信息注入到 Prompt 中，含 Claude Code 三管道与 OpenClaw Context Engine 契约对比；注入有两层服务对象（模型工作记忆 vs 人类审计需求）
- [[上下文压缩]] — 第一代→第二代演进 + 六大产品横向对比 + 压缩哲学光谱(三种立场) + 七条共识原则 + 滑窗stub陷阱
- [[上下文窗口管理]] — Claude Code 弹性状态机(7个Continue站点) + OpenClaw 预算贯穿 Runtime + 进度锚点 + 动态上下文发现
- [[上下文腐蚀与注意力衰减]] — Context Rot / Lost in the Middle / 上下文焦虑症，压缩系统=信号工程师
- [[上下文焦虑症与自我评估偏差]] — 信息丢失+注意力崩溃+提前收工+盲目自信+幻觉闭环，Anthropic发现两大致命挑战及对抗方案
- [[上下文重置与交接]] — Context Reset vs Compaction，结构化 Handoff，Amp/Codex handoff 策略
- [[工具调用回路]] — 请求-返回-再请求的链条，工具往返成本的底层机制
- [[减少重试是成本优化]] — 格式约束更清楚→任务边界更明确→上下文给得更准，省下的是整条失败链路
- [[上下文预算与资源分配]] — Token Budget 行为塑形 + OpenClaw 10种稀缺资源量化 + JIT 检索
- [[Bootstrap截断策略与子Agent Allowlist]] — head70%+tail20%砍中间+子Agent5文件保留+截断告警注入LLM自感知
- [[Hermes Prompt缓存冻结快照策略]] — system_and_3策略+4断点+~75%成本节省+记忆延迟一个会话
- [[GSSC流水线]] — Gather-Select-Structure-Compress 通用上下文构建模式
- [[四级水位线方案]] — MUR AI 落地方案：Tier 0-3 四级水位线 + 增量摘要 + 红线与可观测性
- [[云端Agent上下文管理]] — CLI不用管但云端必须做的：存储分离/工具差异化/ReplacementCache/多用户隔离
- [[递归摘要衰减与语义漂移]] — 全量摘要的根本缺陷：摘要的摘要导致语义逐轮漂移，增量摘要/换线程/结构化保护区三种对抗策略
- [[可逆压缩]] — 压缩后能否恢复？OpenCode可逆(时间戳标记) vs 大多数方案不可逆(硬删除) vs MUR AI存储分离 vs TencentDB四级折叠索引
- [[上下文卸载]] — 把完整信息移出上下文窗口只保留摘要和索引——卸载解决"信息太长"，压缩解决"信息变短"
- [[层次化注意力机制]] — Overview→Focus→Detail三层导航——让Agent注意力变有层次而非扁平
- [[符号化压缩]] — 为LLM设计可推理的结构化压缩符号——三条原则(通用知识/生成不复杂/表达自由)+记忆驱动vs结构驱动
- [[语言压缩哲学]] — 压缩依赖语义结构保留而非字面保留——"一语胜千言"不是包含全部细节而是抓住代表整体的结构

### 多智能体
- [[多智能体协作]] — 专业化分工、三大挑战、三大协作架构
- [[多智能体协作模式]] — 7种核心协作模式（主从/接力/讨论/层级/竞争/评估/路由）+组合策略
- [[多智能体形态分类]] — Sub-agent/Agent Team/Dynamic Workflows/Agent OS 四种核心形态区分 + 2026爆发三条线索
- [[GAN-like多智能体架构与Sprint Contract]] — Anthropic Planner/Generator/Evaluator+对抗性验证+结构化Handoff5层+Rubric4维度
- [[智能体通信协议]] — MCP / A2A / ANP / Function Calling 对比与三层定位
- [[角色分配]] — Multi-Agent 角色设计原则
- [[任务分解]] — 任务拆解策略与依赖处理
- [[Agent间数据流转约定]] — Orchestrator-Worker的工程化落地：.agent/目录约定+四原则+JSON模板

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
- [[七维确定性评分]] — Harness 评测体系：流程完整性/产物质量/代码正确性/效率/安全合规/迭代能力/接口验收，100%确定性逻辑 + honesty gap
- [[Honesty Gap 诚实度差距]] — AI自报vs真实验证的差距量化，G3门禁实测7分差距，失败必须响亮绝不静默兜底
- [[Effective Feedback Compute 验证反馈质量]] — 验证反馈质量R²=0.94~0.99远超token消耗R²=0.33~0.42，检查做得多好几乎完全决定成功率

---

## 🧬 工程实践与案例

### 成本优化
- [[AI Coding Agent Token 成本控制]] — 五层成本优化框架：使用习惯→模型路由→上下文压缩→代码图谱→多Agent架构。核心洞察：成本不是"你问了什么"，而是"系统为了回答你重复搬运了多少上下文"
- [[模型路由]] — 任务→模型匹配策略：复杂任务用强模型、简单任务用便宜模型、Skill/Agent/Command都应绑定模型
- [[RTK]] — 终端命令输出压缩器，30分钟会话节省80% Token
- [[Caveman]] — AI 回复输出废话压缩器，平均节省65-75% 输出 Token
- [[headroom]] — 通用上下文压缩代理层，可逆压缩+按需还原，47-92%节省
- [[context-mode]] — MCP工具输出沙箱化+跨compact会话连续性，工具输出压缩98%
- [[Graphify]] — Skill形态代码图谱，比直接读文件减少71.5倍Token消耗
- [[CodeGraph]] — MCP Server形态代码图谱，7个仓库平均47%更少Token

### 驭驭工程
- [[Harness Engineering]] — 控制层全景：LLM 是引擎，驾驭层是方向盘和安全带，三次进化历程+起源故事+三层技术定义+行业案例+数据对比+路线之争+护栏悖论+七个杠杆
- [[Harness 五层架构实践]] — 五层具体架构（常驻入口→原子规则→角色Agent→按需上下文→执行支撑）+ 19节点链 × G1-G8门禁 × intent×risk动态裁剪
- [[Agent 遗忘三重根因]] — 压缩丢失/检索失败/指令遵循失败 → harness 三层设计（规则外置/状态持久化/门禁阻断）逐一堵漏
- [[Dispatcher 状态机与文件交接]] — 三路编排对比（Workflow/Agent Team/Dispatcher+文件交接）+ 控制平面 vs 计算平面 vs 协作平面
- [[Harness 四阶段演进实践]] — 拿来主义→重prompt→减负分层→Agent编排，四阶段个人实践路径
- [[经验三级进化]] — lesson→pattern→instinct 三级晋升，踩坑驱动的规则进化机制
- [[G1-G8 门禁墙参考卡]] — 8个确定性门禁定义（产物存在→编译→单测→覆盖率→ATDD→接口→上线）+ fail-closed + intent×risk裁剪
- [[state.json 契约]] — Dispatcher状态机的单一真相来源：单写者+ajv校验+文件交接，对应遗忘三重根因的三对策
- [[Harness Engineering 核心概念]] — 四步进阶路线图：Prompt基础 → Context工程 → Agent系统设计 → 动态Harness思维
- [[工程设计与安全]] — 构建者信条、代码-文档同步、工具描述工程、输出解析、重试恢复、护栏钩子、流式渲染
- [[系统提示词工程]] — 静态段+动态段+缓存边界+行为塑形
- [[MCP 协议实现]] — 外部工具连接的实现层
- [[MCP 服务器集成]] — MCP 服务器配置与集成指南
- [[Skill 开发指南]] — Skill 创建架构与分类
- [[Skills技能]] — 人类经验沉淀为智能体能力
- [[运行时环境（Runtime）]] — Agent的专属Workspace：从无状态到有状态隔离运行时
- [[Prompt与Context工程（三次进化）]] — Prompt 与 Context 工程的三次进化历程
- [[Harness Engineering深度解析]] — 六层深度解析：核心哲学、关键悖论、概念谱系、交叉验证、批判阅读与实践启示
- [[长程Coding自动化实践经验]] — 四轮迭代实战：两条直觉经验、上下文甜区、Contract握手、Rubric四层评测(L1-L4)、quality-vision品质锚定、Skills开发Harness
- [[工程技术：在智能体优先的世界中利用 Codex]] — OpenAI百万行代码实验：人类掌舵/智能体执行、AGENTS.md渐进式披露、品味即代码、熵与垃圾回收
- [[模型不是关键，Harness 才是]] — 行业数据对比+起源故事+拐杖论+路线之争+护栏悖论+七个配置杠杆

### 知识工程与沉淀
- [[知识沉淀护城河]] — 工作流是管道，私域与团队知识才是流过管道的活水——知识是真正的技术护城河
- [[知识分层架构]] — 五层存储×五种类型×三级成熟度的三维正交知识体系
- [[知识成熟度与自动衰减]] — draft→verified→proven 三级成熟度 + 自动衰减 + Lint 治理
- [[团队知识库共建共享]] — 独立Git仓库 + 三种角色 + 区块链思想贡献模式 + 自动冲突解决
- [[工作流知识沉淀闭环]] — INIT注入→各阶段按需消费→ARCHIVE自动提取，知识闭环
- [[三级渐进式索引]] — 全景→分类→完整，上下文效率提升一个数量级
- [[人机交互瓶颈与远程操控]] — 在场依赖问题 + 远程操控 + 异步审批 + 四条设计原则

### Agent 工程实践案例
- [[十年老技术开发的 AI Agent 探索之路]] — 从手动管终端到24h无人值守：人在瓶颈→80%需求不需要AI→Vibe Coding翻车→SDD→Agent自举→脚手架>模型→Goal-Driven
- [[SDD]] — Spec-Driven Development：把模糊想法逐步转成可执行单元，留痕是为了进化而非debug

### 产品案例
- [[TencentDB Agent Memory]] — 四级折叠架构(Raw→JSONL→MMD→Metadata)生产化实践，短期记忆压缩最高节省61% Token+提升52%成功率，长期个性化记忆相对提升59%准确率

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

---

## 📡 行业访谈精选

> 来自一线AI研究者的深度访谈，提取核心洞见。

### 杨植麟 (月之暗面/Kimi) — 访谈合集
- [[杨植麟访谈合集]] — 5篇访谈深度整理：AI产品范式、Agent定义与核心能力、泛化评估与训练、多智能体与AGI分级、推理与Test-Time Scaling

### 姚顺雨 (OpenAI) — Language Agent先驱
- [[Language Agent 概念]] — 语言智能体的核心定义
- [[ReAct 范式]] — 协同推理与行动
- [[泛化能力与语言]] — 语言作为泛化工具

### 罗福莉 (小米) — Agent时代亲历者
- [[AI范式转变]] — 从Chat到Agent的时代转变
- [[OpenClaw架构深度解析]] — 开源 Agent 框架全解析（核心价值 + 源码架构 + 部署指南 + 横向对比）
- [[Skills技能]] — 人类经验沉淀
- [[长上下文]] — Agent时代核心基础
- [[MTP架构]] — 多词元预测架构