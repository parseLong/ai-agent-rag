---
title: OpenClaw vs Hermes 架构对比
tags:
  - 方法
  - 架构对比
  - 微内核
  - 单体
  - 记忆系统
  - 上下文工程
  - 驾驭工程
sources:
  - "[[OpenClaw与Hermes：源码里的 AI Agent 架构知识大复盘]]"
  - "[[OpenClaw架构深度解析]]"
  - "[[Hermes Agent 框架]]"
---

# OpenClaw vs Hermes 架构对比

> [!quote] 核心判断
> **OpenClaw 是给平台架构师的范本，Hermes 是给个人开发者的瑞士军刀**——两者的差异不是"谁更好"，是**目标受众不同**。

---

## 为什么走了两条不同的路

OpenClaw 和 Hermes 不是同一个目标的两种实现，而是两个不同目标的成熟解。

### OpenClaw 选微内核——要做"长期演进的平台"

所有架构选择指向同一目标：让多个团队、多种语言、多个迭代周期的代码能共存而不互相破坏。

- **Plugin SDK 强契约** → 百余 extensions 是结果不是原因
- **Channel 25+ Adapter** → 加新通道不需要懂 Gateway 内核
- **Auth Profile 双轴** → 换 Provider 不需要改路由逻辑
- **Context Engine 可插拔** → 换记忆策略不需要改 Runtime
- **CLI Backend 双路径** → 换 LLM 调用方式不需要改业务逻辑

**代价**：上手陡——新人要先理解 SDK 才能写第一行代码
**回报**：可演进性——核心 src/ 几千行，能力靠插件叠出来

### Hermes 选单体——要做"个人开发者闭环"

所有架构选择指向另一目标：让一个开发者从安装到改源码到自己造工具的链路最短。

- **AIAgent 单体类** → 一个文件看完就能理解全部核心逻辑
- **技能自创建闭环** → Agent 自己根据经验创建新技能
- **MEMORY.md / USER.md 直接编辑** → 不需要懂数据库或向量索引
- **Smart Approval LLM 辅助安全评估** → 不需要写规则也能跑得稳

**代价**：难演进——改核心行为要动万行 AIAgent 类，profile 串扰是结构性问题
**回报**：个人开发者的"全栈掌控感"——反馈闭环极短

### 共同判断：记忆是长期投入的主战场

两个框架都把记忆当成一级模块，而非可选功能——各自投入的设计复杂度远超大多数 Agent 框架。

| 维度 | OpenClaw | Hermes |
|------|----------|--------|
| **目标受众** | 平台团队 / 多人协作 / 长期演进 | 个人开发者 / 重度 dogfood / 快速迭代 |
| **设计哲学** | 边界 vs 实现分离 | 一体化丰满 |
| **演进策略** | 微内核 + Plugin SDK + 百余插件生态 | 单体 + ToolRegistry 自注册 + 技能自创建 |
| **协作成本** | 低（插件不互相破坏） | 高（多人改单体类易冲突） |
| **可观测性** | 显式（FailoverError 闭合枚举 / 截断告警注入 LLM） | 隐式（错误启发式分类 / stdout 日志） |
| **适合场景** | 多通道 Bot 平台、多租户 SaaS | 个人助手、研究原型、单团队工具人 |

---

## 6 维度对比

### 维度一：执行引擎

| 维度 | OpenClaw | Hermes |
|------|----------|--------|
| 主循环 | `runEmbeddedPiAgent`（双路径：embedded vs CLI backend） | `AIAgent.run_conversation`（单一路径） |
| 迭代上限 | Auth profile 数量驱动动态计算 | 硬编码 父 90 / 子 50 |
| 凭据抽象 | Auth Profile（api_key + token + oauth，带生命周期） | Credential Pool（API Key 数组） |
| 凭据持久化 | 磁盘 store，重启保留冷却状态 | 进程内内存（重启丢失） |
| 错误分类 | FailoverError 13 闭合枚举 | 错误类型 + 计数启发式 |
| 压缩算法 | 三级（pre-request / timeout-triggered / overflow） | 四步（工具裁剪→边界→摘要→组装） |
| Context 管理 | Context Engine 可插拔契约 | 内置 `context_compressor` |
| CLI 兼容 | Claude Code / Codex CLI 可当 backend | 仅支持 Copilot ACP |
| 子 Agent | Subagent spawn（registry + control + role + depth 可配） | `delegate_tool`（阻止列表 + 默认深度 1） |
| 并发调度 | **5 车道** CommandLane | 全局单一命令队列 |
| Prompt 缓存 | Cache Trace 全链路追踪 + Provider 侧缓存 | 主动注入 `cache_control` 断点 + 冻结快照 |

> [!important] 执行引擎差异最大的不是"代码多少行"
> 而是**"错误契约的闭合性"**——OpenClaw 用 13 种 FailoverError 闭合枚举把外部世界的不确定性显式吃掉，Hermes 用启发式分类应对。前者更工程化但维护成本高，后者更敏捷但黑盒风险大。

### 维度二：记忆系统

| 层级 | OpenClaw | Hermes |
|------|----------|--------|
| 静态层 | SOUL / USER / MEMORY / AGENTS → 每次 buildPrompt 注入 | MEMORY + USER → 首轮冻结快照 |
| 向量层 | memory-core（SQLite + FTS5 + sqlite-vec；可切 QMD）或 memory-lancedb | 8 插件提供者（Honcho / Mem0 / ...） |
| 搜索 | BM25 + Vector 混合，MMR 重排，时间衰减 | 依赖选定记忆提供者 |
| 后台整理 | **Dreaming 三阶段**（Light → REM → Deep） | ❌ 无等价机制 |
| 主动召回 | Active Recall 插件（pre-reply sub-agent） | Memory Nudge（每 10 轮触发后台 review） |
| 跨会话搜索 | ❌ 无内置 | **Session Search**（SQLite FTS5 + Gemini Flash 摘要） |
| 记忆安全 | 插件安装时静态扫描 | 实时检测（12 种威胁模式 + 不可见 Unicode） |

> [!note] 两层互补而非二选一
> 记忆系统其实分两层——**Session 层**（原始对话日志）和 **Memory 层**（从 Session 沉淀出的结构化记忆），两者是"原始日记"和"读书笔记"的互补关系。OpenClaw 把 Memory 层做得重（Dreaming 自动沉淀），但 Session 层只是 JSONL append-only；Hermes 反过来把 Session 层做得重（FTS5 全文搜索），但 Memory 层缺自动沉淀机制。**理想形态是两层都做好**。

### 维度三：插件/工具系统

| 维度 | OpenClaw | Hermes |
|------|----------|--------|
| 扩展机制 | npm 包 + Plugin SDK 合约 | Python 模块 + 导入时自注册 |
| 添加工具 | 创建 npm 包 + Plugin SDK | 创建工具文件 + `registry.register()` |
| 工具分发 | npm 发布 + `openclaw plugins install` | pip 安装整个包 |
| 类型安全 | TypeScript 编译时类型检查 | Python 运行时检查 |
| 技能系统 | 目录式 Markdown 指令（人工编写） | 目录式 + **Agent 自主创建/编辑/patch** |
| Toolsets | Plugin 粒度 enable/disable | 20+ 预定义 toolset，支持递归组合 |

> [!tip] 根本差异是"扩展路径的耦合度"
> OpenClaw 的扩展走**独立仓库**（独立 npm 包 + 版本化 SDK 契约），扩展者不需要动核心代码；Hermes 的扩展走**同一仓库**（新建 Python 文件 + 导入时自注册），改动直接。不存在哪个更好，只有哪个更匹配你的扩展场景。

### 维度四：安全模型

| 安全层 | OpenClaw | Hermes |
|--------|----------|--------|
| 网络层 | TLS 强制 + 证书 Pinning + SSRF 防护 | IPv4 偏好 + 代理支持 |
| 认证层 | Device Identity + Ed25519 签名 + RBAC | DM 配对验证码 |
| 命令执行 | Allowlist + 交互式审批 | 35 条 DANGEROUS_PATTERNS + **Smart Approval 三态** |
| 内容安全 | 插件安装静态扫描 | Tirith + 上下文注入检测 + 记忆扫描 + MCP 扫描 |
| 技能安全 | 路径遍历 + 文件权限 | **4 级信任** + 6 类威胁静态分析 |
| 沙箱 | Docker / SSH | 8 种沙箱后端（含 Modal / Singularity / Vercel） |
| 供应链 | npm 签名验证 | **Tirith cosign + SHA-256** 供应链证明 |

> [!note] 安全重心落在不同层
> **OpenClaw 偏底层**（TLS Pinning + Ed25519 签名 + RBAC），**Hermes 偏应用层**（Tirith + cosign + Smart Approval + 8 种沙箱）。对沙箱隔离/内容扫描要求高 → Hermes；对跨机器身份认证/RPC 安全要求高 → OpenClaw。

### 维度五：子 Agent / 委派

| 维度 | OpenClaw | Hermes |
|------|----------|--------|
| 机制 | 多 Agent 路由 + Session 隔离 | `delegate_tool` 子 Agent 并行 |
| 并发 | Session 级并行 | ThreadPoolExecutor（最大 3 并发） |
| 嵌套 | 无限制（Session 隔离） | 默认深度 1（可配置） |
| 工具隔离 | 插件级隔离 | 阻止列表 |
| 记忆共享 | 通过记忆插件 | 子 Agent skip_memory，共享 session DB |

> [!important] 两种任务分解哲学
> OpenClaw 用 Multi-Agent 路由实现**"职责分解"**（不同 Agent 服务不同用户/角色，物理隔离），Hermes 用 delegate_tool 实现**"任务分解"**（同一 Agent 把复杂任务拆给子 Agent 并行）。前者解决"多人协作"，后者解决"单任务复杂度"。

### 维度六：Prompt 缓存

| 维度 | OpenClaw | Hermes |
|------|----------|--------|
| 策略 | 依赖 Provider 侧缓存 | 主动 `system_and_3`（4 个 cache_control 断点） |
| 系统提示 | 每次 `buildPrompt()` 动态构建 | 首轮构建后冻结 |
| 记忆注入时机 | 每次 Prompt 构建时 | 仅新会话开始时 |
| 成本节省 | 取决于 Provider | ~75% 输入 token 成本 |
| 动态性 | 高（记忆实时反映） | 低（记忆延迟一个会话） |

> [!tip] "动态性 vs 命中率"的取舍
> **没有"正确答案"的工程取舍**——成本敏感选 Hermes（冻结快照），记忆驱动场景选 OpenClaw（动态构建）。

---

## 记忆+检索方案的三种范式定位

| 维度 | Static RAG | Agentic RAG | LCM |
|------|------------|-------------|-----|
| 一句话 | 一次检索 + 一次生成 | 多次检索 + 反思循环 | DAG 建模 + 按需下钻 |
| 检索次数 | 1 次（固定） | **多次**（LLM 自主决定） | 按需沿因果链下钻 |
| 数据建模 | 扁平索引 | 扁平索引 | **DAG（保留因果/时序）** |
| 信息保留 | 有损 | 有损 | **无损** |
| 能否反思 | ❌ | ✅ | ✅ |
| 典型实现 | LangChain RetrievalQA | Self-RAG / Multi-Hop | lossless-claw |

三者不是递进替代关系——Static RAG 和 Agentic RAG 关注"怎么检索"，LCM 关注"怎么建模上下文"。成熟 Agent 可能同时需要两者。

**两个框架在三种范式中的位置**：

- **Static RAG**：两框架都已具备
- **Agentic RAG**：两框架有潜力但尚未成体系
- **LCM**：两框架核心未覆盖——真正实现的是 OpenClaw 第三方插件 **lossless-claw**（DAG + SQLite + `lcm_expand_query`）
- **自动记忆整理**：OpenClaw 独有 Dreaming 三阶段

lossless-claw 的存在就是"微内核 + 插件"长期红利——核心不够的能力，生态可以补。

---

## 上下文焦虑症与上下文重置

### 上下文焦虑症——长周期任务失败的首要原因

Anthropic 在 Harness Engineering 实践中发现，长周期任务失败的首要原因不是模型能力不足，而是**上下文焦虑症（Context Anxiety）**：

- **信息丢失**：早期对话中的关键决策被逐渐遗忘或边缘化
- **注意力崩溃**：模型无法有效关联先前上下文，设计一致性丧失
- **提前收工倾向**：模型感知到上下文接近极限时，下意识提前结束

这与 Chroma Research 的 Context Rot 发现互为印证——微观（注意力架构级退化）和宏观（工程实践级焦虑）视角描述同一个根因：**上下文长度超过有效注意力范围后，推理质量不可避免地退化**。

### Context Reset vs Context Compaction

| 策略 | 做法 | 适用 | 信息保留 |
|------|------|------|----------|
| **Context Compaction** | 在原会话中压缩历史 | 短-中期任务 | 有损但连续 |
| **Context Reset** | 彻底起新 Agent，通过结构化工件进行工作交接 | 极长或焦虑明显任务 | 选择性但完整重启 |

类比：不是所有内存泄漏都能靠"清理缓存"解决，有时候得重启进程。

**两个框架现状**：

- **OpenClaw**：Compaction 是主要手段，Context Engine `compact()` 支持可插拔压缩策略，但没有内置 session 级 reset 机制
- **Hermes**：四步压缩 + 首轮冻结快照（隐式 reset——新会话不继承旧会话完整历史）

> [!important] 结合两者的演进方向
> 把 Reset 作为 Compaction 失败后的**兜底策略**纳入 Context Engine 契约——当压缩后的 context 仍超过有效注意力范围，触发 reset：写入结构化 Handoff 文件 → 关闭当前 session → 启动新 Agent 实例 → 读取 Handoff 文件暖启动。OpenClaw 的 Context Engine `assemble()` 接口已为此留好扩展点。

---

## Harness 全链路治理

> [!quote] 核心理念
> **模型能力决定上限，Harness 设计决定能否落地**。Anthropic 的核心理念是"缩小依赖模型自觉性的面积"——不依赖模型"记得住"上下文（用 Context Reset 兜底）、不依赖模型"评得准"自己的输出（用对抗性评估兜底）、不依赖模型"知道何时停"（用预算约束兜底）。

### 自我评估偏差——Agent 失控的根源

- **盲目自信**：忽略明显缺陷，给低质量输出打高分
- **拒绝查证**：依赖自己的记忆，而非外部验证
- **幻觉闭环**：构建自我强化的正向反馈循环，逐渐脱离现实

> "既当裁判又当运动员"不可能产出客观评估。

### 对抗性评估——用"物理现实锚点"粉碎幻觉

1. **评估者提示词设计为"寻找漏洞的挑剔者"**——消除模型的讨好倾向
2. **动态测试而非静态检查**——通过 Playwright 对运行中的应用操作测试，用"物理现实"锚定评估
3. **多维度 Rubric 评分**——设计质量 / 原创性 / 工艺 / 功能性四个量化维度

**关键洞见**：对抗性评估的价值不只是"找 bug"，而是**打破幻觉闭环**。当 Generator 产出必须通过 Playwright 真实运行验证时，"代码看起来对"这种幻觉就无处藏身。

### Harness 的三阶段治理模型

| 阶段 | 机制 | 解决的问题 |
|------|------|------------|
| **执行前** | PreToolUse Hook 拦截、权限检查、输入校验、Sprint Contract | 危险操作未执行就挡住 |
| **执行中** | 预算约束、沙箱隔离、Sub-agent 上下文隔离、Context Reset | 执行不失控、不互相污染 |
| **执行后** | PostToolUse Hook 质检、对抗性评估、循环检测、审计日志 | 做错了能发现 |

> LangChain 仅调整 Harness（未换模型）就将 Terminal Bench 2.0 得分从 52.8% 提升到 66.5%。

### 模型能力与 Harness 复杂度的动态平衡

- **模型能力弱时**（Sonnet 4.5）→ 更复杂脚手架（频繁 Reset + 严格迭代验证）
- **模型能力增强时**（Opus 4.6）→ 可简化机制（减少 Reset 频率 + 单次构建 + 最终 QA）
- **核心原则不变**：角色分离 + 对抗性验证 + 上下文管理

Harness 不是一成不变的重量级框架，而是随模型能力演进而持续校准的治理体系——设计时应预留"旋钮"。

### OpenClaw 和 Hermes 已有 vs 缺失的 Harness 能力

| Harness 能力 | OpenClaw | Hermes |
|--------------|----------|--------|
| 执行前拦截 | Exec Approval | Smart Approval |
| 权限模式 | 二态（allow/deny） | 三态（Default/Auto/Plan） |
| 输入校验 | Zod schema | Pydantic model_validate |
| 预算约束 | Bootstrap + Context + Lanes | IterationBudget |
| 沙箱隔离 | Docker + SSH | 8 种后端 |
| **对抗性评估** | ❌ | ❌ |
| **循环检测** | ❌ | ❌ |
| **输出质检** | ❌ | ❌ |
| 截断告警 | Bootstrap 截断注入 LLM 自知 | — |
| 审计可观测 | Cache Trace 全链路记录 | — |

**空白区**：对抗性评估 / 显式循环检测 / Sprint Contract 机制——这是两个框架共同缺失的 Harness 组件，也是落地最关键的下一步。

---

## 多 Agent 协作编排

### GAN-like 多智能体架构

Anthropic 提出受 GAN 启发的多智能体协作范式：

| 角色 | 核心职责 | 交互机制 |
|------|----------|----------|
| **Planner** | 将简短需求扩展为详细产品规格，拆分任务为可执行冲刺 | 输出 Sprint Contract |
| **Generator** | 逐步实现每个冲刺 | 接收 Planner 的计划，按 Contract 交付 |
| **Evaluator** | 像 QA 团队测试应用，寻找缺陷 | 通过 Playwright **动态测试** |

**核心洞见**：通过角色分离实现"生产"与"验收"的职责隔离——Generator 不能评估自己的输出，Evaluator 被设计为"挑剔者"而非"友好用户"。每个角色可以独立做 Context Reset。

### Sprint Contract——明确任务边界与验收标准

将主观"完成标准"转化为可验证客观条件，防止长周期任务中的**规范漂移**。Evaluator 有明确验收标准可执行，而非凭"感觉"打分。

> [!tip] 落地思路
> 在 OpenClaw 的 Subagent 机制上叠加"Evaluator Agent"——`subagentRole` 新增 `evaluator` 类型，拥有 Playwright MCP 工具但没有代码编辑权限，按 Sprint Contract 定义 Rubric 对 Generator 输出打分。分数不过则触发 Generator 在新 session 中修复。

### 完整的架构层次

```
协议层：ACP, MCP, A2A, HTTP（让不同框架 Agent 互通）
     ↓
编排层：显式 DAG 工作流 + Sprint Contract
     ↓
执行层：各个 Agent 各自跑 ReAct 循环
     ↓
状态层：Checkpoint + Context Reset + Handoff
     ↓
Harness 层：约束 + 对抗性验证 + 纠错（贯穿以上所有层）
```

---

## 两个框架共同的空白区与演进方向

| 方向 | 当前空白 | 演进思路 |
|------|----------|----------|
| **记忆** | Session 层与 Memory 层的断层 | Dreaming 与 Session 轮换联动；REM 输出关系图；增加遗忘机制 |
| **上下文** | 两阶段预防性轮换缺失 | 60-70% 触发记忆同步 → 80% 触发优雅 Handoff |
| **检索** | 静态 RAG 为主 | Self-RAG hook（每次召回后 LLM 自检"够不够"） |
| **能力管理** | 渐进式披露已解决 token 问题 | 确定性编排——已验证流程从 Skill 固化为 Workflow |
| **评估** | 自我评估偏差 | 对抗性评估 + Sprint Contract + 显式循环检测 |
| **沙箱** | 服务端保护 | 用户侧隐私沙箱（数据不出设备） |
| **协作** | 单框架内调度 | 跨架构协议互通 + 显式 DAG 编排 |

---

## 相关笔记

- [[OpenClaw架构深度解析]] — OpenClaw 源码级架构深度解析
- [[Hermes Agent 框架]] — Hermes 单体架构全貌
- [[OpenClaw框架]] — OpenClaw 实用部署和使用指南
- [[驾驭工程]] — 控制层全景：LLM 是引擎，驾驭层是方向盘和安全带
- [[Harness Engineering深度解析——从有序驾驭无序的工程哲学]] — 六层深度解析
- [[记忆系统]] — 短期5种策略+长期3类架构
- [[上下文压缩]] — Map-Reduce / Refine / 相关性过滤 / LLM 压缩
- [[多智能体协作模式]] — 7种核心协作模式
- [[Agent技术范式演变（2023-2026）]] — 近代Agent四阶段演进