---
title: "OpenClaw与Hermes：源码里的 AI Agent 架构知识大复盘"
source: "https://mp.weixin.qq.com/s/49dxdMXEUoWIYlIh8fFqMQ"
author: rianli（腾讯程序员）
published: 2026-05-29
created: 2026-06-15
tags:
  - 阅读
  - Agent框架
  - 微内核
  - 源码分析
  - 架构对比
description: "对 OpenClaw (TypeScript 微内核) 和 Hermes (Python 单体) 两套 Agent 框架的源码级深度解析与架构对比"
---

# OpenClaw与Hermes：源码架构深度复盘

> [!quote] 文章定位
> 作者 rianli 在开发 QQ Bot 插件期间深入阅读源码后写出的架构复盘。Part I 拆 OpenClaw，Part II 拆 Hermes，Part III 正面对比，第 22 章延伸思考落地难题。

> [!tip] 知识链接
> 本笔记是精炼版阅读笔记，核心概念已拆解为原子笔记：
> - [[OpenClaw架构深度解析]] / [[OpenClaw vs Hermes 架构对比]] / [[Hermes Agent 框架]]
> - [[Dreaming三阶段记忆晋升]] / [[Channel Plugin 25+ Adapter 契约]] / [[Auth Profile与FailoverError]]
> - [[Smart Approval三态审批]] / [[CLI Backend双路径执行]] / [[Session Search (Hermes)]]
> - [[Lanes分车道并发管控]] / [[Bootstrap截断策略与子Agent Allowlist]] / [[技能自创建闭环与渐进式披露]]
> - [[GAN-like多智能体架构与Sprint Contract]] / [[上下文焦虑症与自我评估偏差]] / [[Hermes Prompt缓存冻结快照策略]]
> - [[上下文腐蚀与注意力衰减]] / [[上下文重置与交接]] / [[Harness Engineering]]

---

## 看山三境——认知演进

1. **看山是山**：初见惊艳——24/7常驻、跨IM流转、长期记忆、自主完成开放性任务
2. **看山不是山**：光环褪色——OpenClaw费token/健忘/交付度低，Hermes多人串扰/核心单体/记忆半自动
3. **看山又是山**：踩坑后看懂每个"不完美"背后的工程取舍

---

## Part I: OpenClaw — TypeScript 微内核架构

![[07-Attachments/OpenClaw-Hermes/design_philosophy.png]]

### 四大设计回答四个问题

| 问题 | OpenClaw 的设计 |
|------|-----------------|
| 多协议可插拔 | **Channel 25+ Adapter** 契约 → [[Channel Plugin 25+ Adapter 契约]] |
| LLM 上下文资源预算 | **可插拔 Context Engine + 三级 Compaction** → [[上下文预算与资源分配]] |
| 记忆自动沉淀不退化 | **Dreaming 三阶段加权晋升** → [[Dreaming三阶段记忆晋升]] |
| 凭证失败与业务失败分治 | **Auth Profile + FailoverError** → [[Auth Profile与FailoverError]] |

### 五层架构全景

![[07-Attachments/OpenClaw-Hermes/arch_overview_5_layers.png]]

- **触达层**：用户通过各种 IM 平台交互 → Channel Plugin
- **编排层**：Gateway 负责消息路由、Agent调度、安全控制 → [[OpenClaw架构深度解析]]
- **能力层**：所有功能以插件形式提供 → Plugin SDK
- **记忆层**：向量记忆 + Dreaming + Active Recall → [[Dreaming三阶段记忆晋升]]
- **模型层**：9种LLM API协议 + 多模型降级链

### Gateway — 5大角色

![[07-Attachments/OpenClaw-Hermes/gateway_core.png]]

| 角色 | 说明 |
|------|------|
| 唯一长驻进程 | Single Source of Truth，一个Gateway per host |
| 消息总线 | 一切流量必经 Gateway，统一WS Schema |
| 多Agent路由边界 | Multi-Agent Router物理隔离，解决上下文污染/工具链冲突/渠道风格差异 |
| 认证+信任边界 | Challenge-Response + Device Identity + RBAC |
| 嵌入式HTTP Host | Agent可主动构造UI（canvas/A2UI） |

> [!important] 边界 vs 实现
> Gateway 是边界，不是实现——自己只做"协议+路由+信任"，其他全是插件。这是微内核保持几千行的根本原因。

### Agent执行引擎核心架构

- 底层引擎：`@mariozechner/pi-agent-core`（ReAct循环的工程级实现）
- 四层叠加：循环层(pi-agent-core) → 编排层(pi-embedded-runner) → 拦截层(hooks) → 能力层(plugins)
- 三层错误边界：内层(runEmbeddedAttempt) → 中层(runEmbeddedPiAgent) → 外层(runWithModelFallback)
- 七类分支顺序决定正确性（aborted → live model switch → timeout compaction → overflow → assistant error → success → fallback）

> 深入拆解见 → [[Auth Profile与FailoverError]] / [[上下文预算与资源分配]]

### Lanes 分车道并发管控

4条车道（Default/Nested/Subagent/Cron）各自独立队列互不阻塞，同时是权限边界。Nested lane防止Cron自递归死锁。Hermes是单一全局队列。

> 深入拆解见 → [[Lanes分车道并发管控]]

### Bootstrap 截断策略

两道过滤层：(1) 截断策略 head70%+tail20%砍中间（不是前缀截断）；(2) 子Agent allowlist只保留5个文件（保留人格连续性、剥离状态性数据）。截断告警注入LLM——让Agent自知信息不全，主动补读。

> 深入拆解见 → [[Bootstrap截断策略与子Agent Allowlist]]

### 预算贯穿 Runtime — 10种稀缺资源量化

OpenClaw把runtime里每一种稀缺资源都显式量化并配降级路径：上下文窗口、单次工具输出(30%软限+16K硬限)、启动上下文、循环迭代、Overflow压缩尝试、凭证可用性、子Agent递归、Lane并发、Steer速率等。

核心约束：reserve不能挤占minPromptBudget（min(8K, 50% context)）；SAFETY_MARGIN=1.2留20%安全边距。

### 双路径执行——把 CLI 当 Backend

OpenClaw 不把 Claude Code/Codex CLI 当竞品，而是当**可替换的执行 backend**。

三种 Provider 类型对比：

| 类型 | 协议 | 适用场景 |
|------|------|----------|
| embedded | 各厂商 HTTP API | 走自己的 API Key |
| CLI provider | 各家私有 stdout + 反向 MCP | 复用 CLI 登录态 |
| ACP provider | ACP JSON-RPC | 把别的 ACP agent 当 LLM backend |

> 深入拆解见 → [[CLI Backend双路径执行]]

### 双向连接——OpenClaw 同时是主和辅

| 路径 | 暴露粒度 | 协议 |
|------|----------|------|
| MCP Server | 工具粒度（9个工具） | stdio JSON-RPC |
| ACP Server | Agent粒度 | stdio JSON-RPC |
| Gateway API | 系统粒度 | HTTP REST/RPC |

---

## Part II: Hermes — Python 单体架构

### 五层架构

- **触达层**：30+平台适配器
- **Gateway层**：统一消息网关
- **执行引擎层**：`AIAgent`单体类（万行级，一切汇聚的枢纽）
- **能力层**：76工具文件 + 技能自创建闭环
- **记忆层**：MEMORY.md + 8插件提供者 + Session Search

### AIAgent 单体执行引擎

- `run_conversation()` 五阶段：恢复→净化→系统提示→预压缩→主循环
- 四种API模式自动选择（chat_completions / codex_responses / anthropic_messages / bedrock_converse）
- IterationBudget：线程安全迭代控制（父90/子50）
- Credential Pool + Model Fallback Chain 两级容错

### 上下文压缩——四步算法

工具裁剪 → 边界 → 摘要 → 组装。**工具输出智能摘要**（16+种工具类型专用格式）。反抖动保护：连续2次压缩效果低于10%自动跳过。

### Prompt 缓存冻结快照

`system_and_3` 策略——4个cache_control断点（system prompt + 最后3条消息）。首轮构建后冻结系统提示，新记忆延迟一个会话才注入。~75%输入token成本节省。

> 深入拆解见 → [[Hermes Prompt缓存冻结快照策略]]

### 技能自创建闭环

经验积累 → Nudge触发 → review评估 → 创建技能 → 安全扫描 → 后续发现不足 → patch更新 → 持续优化

> [!important] 渐进式披露
> 三级访问：skills_list（元数据）→ skill_view（完整说明）→ skill_view+file_path（支撑文件）。把技能token成本从 O(N) 降到 O(被实际用到的)。元数据约束做成硬校验（name≤64, description≤1024）。

> 深入拆解见 → [[技能自创建闭环与渐进式披露]]

### Smart Approval — 先LLM分诊再叫人

| 结果 | 处理 |
|------|------|
| APPROVE | 自动批准 + 会话级免审 |
| DENY | 直接阻止 + 禁止重试 |
| ESCALATE | 降级为手动审批 |

> 深入拆解见 → [[Smart Approval三态审批]]

### Session Search — 跨会话搜索

SQLite FTS5 双索引（标准分词 + trigram）→ 辅助LLM摘要（max 10K tokens）→ 截断策略25%前+75%后 → 子session溯源到根session

> 深入拆解见 → [[Session Search (Hermes)]]

---

## Part III: 架构对比与设计启示

### 核心判断

> **OpenClaw 是给平台架构师的范本，Hermes 是给个人开发者的瑞士军刀**——差异不是"谁更好"，是目标受众不同。

| 维度 | OpenClaw | Hermes |
|------|----------|--------|
| 目标受众 | 平台团队/长期演进 | 个人开发者/快速迭代 |
| 设计哲学 | 边界vs实现分离 | 一体化丰满 |
| 演进策略 | 微内核+Plugin SDK | 单体+ToolRegistry+技能自创建 |
| 可观测性 | 显式(FailoverError 13种闭合枚举) | 隐式(错误启发式分类) |

### 执行引擎最大差异

**错误契约的闭合性**——OpenClaw 用 13 种 FailoverError 闭合枚举把外部世界不确定性显式吃掉，Hermes 用启发式分类应对。前者更工程化但维护成本高，后者更敏捷但黑盒风险大。

### 记忆系统互补

| 层级 | OpenClaw做得重 | Hermes做得重 |
|------|----------------|--------------|
| Memory层 | Dreaming三阶段加权晋升 | ❌ 无等价机制 |
| Session层 | ❌ 无跨会话搜索 | FTS5+LLM摘要 |

> 理想形态是两层都做好——Session层保证"找得到原始出处"，Memory层保证"不用每次都重读原始"。

---

## 第22章：面向落地的延伸思考

### 22.1 插件化+协议互通

一个进程同时扮演四种协议角色——MCP Server + ACP Server + HTTP API + WebSocket。这种"任意位置可插拔"的灵活性，根源是把循环层和能力层彻底拆开。

### 22.2 记忆系统演进方向

- Dreaming三阶段恰好覆盖情景→语义→程序性记忆转化链路
- **千人千面潜力**：6信号评分完全由用户交互行为驱动，加一层user_id维度隔离即可实现个性化
- 三个演进：Session轮换联动 / REM输出关系图 / 增加遗忘机制

### 22.3 Context Engineering → 上下文焦虑症

> [[上下文焦虑症与自我评估偏差]] / [[上下文腐蚀与注意力衰减]] / [[上下文重置与交接]] 已有详细笔记

Anthropic 发现的两大致命挑战：
1. **上下文焦虑症**（信息丢失 + 注意力崩溃 + 提前收工倾向）——约35分钟/80K-150K tokens开始出现
2. **自我评估偏差**（盲目自信 + 拒绝查证 + 幻觉闭环）——模型既当裁判又当运动员

关键洞见：解法不是更大窗口或更强模型，而是预防性上下文隔离、上下文重置和对抗性验证。

**Context Reset vs Compaction**：Compaction在原会话压缩（有损但连续），Reset彻底起新Agent+结构化Handoff（选择性但完整重启）。不是所有内存泄漏都能靠清理缓存解决。

**两阶段Session轮换**（前沿实践）：60-70%使用率触发记忆同步，80%触发优雅Handoff。不等overflow才压缩。

**结构化Handoff**（5层）：状态快照/叙事上下文/决策日志/优先队列/警告与陷阱

### 22.4 Skill 渐进式披露

云端Skill体系两层红利：
1. 官方Skill快速迭代（Skill Registry按需加载）
2. 用户侧经验自动沉淀（识别反复出现的操作链，编排为专属Skill）

### 22.5 确定性编排

已验证的流程从Skill（指引式）固化为Workflow——固定步骤直接编排执行，只在需要判断的节点才调用LLM。

### 22.6 GAN-like 多智能体架构

Anthropic 的 Planner/Generator/Evaluator 三角色架构——Generator不能评估自己的输出（避免自我评估偏差），Evaluator用Playwright对运行中的应用做**动态测试**而非阅读静态代码。每个Role可独立做Context Reset。

**Sprint Contract**：在每个冲刺开始前就定义可验证的验收标准（4维度Rubric：设计质量/原创性/工艺/功能性）。

**结构化Handoff**：`claude-progress.txt` 5层结构——状态快照/叙事上下文/决策日志/优先队列/警告与陷阱。

> 深入拆解见 → [[GAN-like多智能体架构与Sprint Contract]]

### 22.7 Harness Engineering

> [[Harness Engineering]] / [[Harness Engineering深度解析]] 已有详细笔记

核心洞见：**自我评估偏差**是Agent失控的根源——模型既当裁判又当运动员，不可能产出客观评估。Harness的本质是系统性地消除这种偏差。

三阶段治理模型：执行前（PreToolUse拦截+权限检查+Sprint Contract）→ 执行中（预算约束+沙箱隔离+Context Reset）→ 执行后（对抗性评估+循环检测+审计日志）

**模型能力与脚手架复杂度的动态平衡**：模型弱→需要更复杂脚手架（频繁Reset+详细Contract+严格验证）；模型强→可以简化（减少Reset+简化协议+单次构建+最终QA）。核心原则不变：角色分离、对抗性验证、上下文管理。

两个框架的空白区：
1. **对抗性评估**——独立Evaluator Agent用动态测试验证输出
2. **显式循环检测**——同一步骤重试3次自动中断
3. **Sprint Contract机制**——委派任务时生成可验证的验收标准

**OpenHarness参考实现**（港大HKUDS）：Agent=LLM（智能）+Harness（工具、技能、记忆、治理、协调）——"模型提供智能，Harness提供手、眼、记忆和安全边界"。

---

## 参考引用

| 项目 | 链接 |
|------|------|
| OpenClaw | https://github.com/openclaw/openclaw |
| Hermes Agent | https://github.com/NousResearch/hermes-agent |
| Agentic Design Patterns（中文版） | https://github.com/xindoo/agentic-design-patterns |
| Eino（字节CloudWeGo） | https://github.com/cloudwego/eino |
| OpenHarness（港大HKUDS） | https://github.com/HKUDS/OpenHarness |
| QQ Bot 插件 | https://github.com/tencent-connect/openclaw-qqbot |
| pi-agent-core | https://www.npmjs.com/package/@mariozechner/pi-agent-core |
| lossless-claw | OpenClaw第三方LCM插件 |
| QMD | https://github.com/tobi/qmd |

| 研究 | 关联章节 |
|------|----------|
| Anthropic Harness Design (2026.03) | §22.3/§22.6/§22.7 |
| Chroma Research Context Rot (2025) | §22.3 |
| Mem0 Engineering Memory 2026 | §22.2 |
