---
title: 模型不是关键，Harness 才是
author: 尹John
source: 微信公众号·AGI Hunt
url: https://mp.weixin.qq.com/s/sVGeofV9uTgvhgR44q8pNA
tags: [学习笔记, 驾驭工程, Harness, 行业案例, 路线之争, 拐杖论]
date: 2026-06-11
status: 已整理
---

# 模型不是关键，Harness 才是

> 尹John（AGI Hunt）的行业观察文章，核心论题：**同一模型换 Harness，性能翻倍**。从行业数据、案例实践、路线之争三个维度，论证 Harness Engineering 的价值与争议。

---

## 文章核心论题

同一模型，换一套运行环境，编程基准成功率从 **42% → 78%**（Nate B Jones 研究）。变量只有 Harness。

![[笔记同步助手/images/167c9929fc3078d948632e3f074198fd_MD5.png]]
> Nate B Jones 研究数据图：同一模型不同 Harness 下的性能对比

---

## 三层进化时间线

| 阶段 | 时间 | 核心关注 | 隐喻 |
|------|------|---------|------|
| Prompt Engineering | 2022-2024 | 精心构造单次指令 | 写一封好邮件 |
| Context Engineering | 2025 | 为每个决策点动态构建上下文 | 把相关附件都带上 |
| Harness Engineering | 2026 年 2 月起 | 设计完整的控制系统 | 搭建整个办公室 |

![[笔记同步助手/images/c3bc40ac89c0b99fc20bd33d1b4ac846_MD5.png]]
> 三次进化层层包裹图：Prompt → Context → Harness 依次嵌套

→ 详细分析见 [[驾驭工程#三次进化的关系]] 和 [[Prompt与Context工程（三次进化）]]

---

## Mitchell Hashimoto 的起源故事

[[驾驭工程#Mitchell Hashimoto 的采纳五阶段]]

HashiCorp 联合创始人、Terraform 缔造者，2026 年 2 月 5 日写博客记录 AI 编程进化史，将采纳之旅分为六阶段。**第五阶段名为「Engineer the Harness」**：

> 每当你发现 Agent 犯了一个错误，你就花时间去工程化一个解决方案，让它再也不会犯同样的错。

AGENTS.md 文件里每一行规则，都对应着 Agent 曾犯过的一个错：

> 那个文件里的每一行都基于一次 Agent 的不良行为，而且几乎完全解决了这些问题。

---

## Philipp Schmid 的操作系统类比

[[驾驭工程#Philipp Schmid 的操作系统类比]]

![[笔记同步助手/images/95c04ed3d0e78c7efb5dcd14a35694f2_MD5.jpg]]
> Harness 作为操作系统的架构图：Model(CPU) → Context Window(RAM) → Agent Harness(OS) → Agent(Application)

核心洞察：**过去两年一直在升级 CPU，但操作系统还停留在 DOS 时代。**

---

## 三层技术定义（arxiv 论文）

[[驾驭工程#三层技术定义（arxiv 论文 2603.05344）]]

![[笔记同步助手/images/62203db8fbb85cfb21696a5e5acff40f_MD5.png]]
> OpenDev/OpenHarness 完整系统架构分层图

![[笔记同步助手/images/9a5de36b530c5fc0aa02ab24ad32f4d2_MD5.png]]
> Agent 运行6阶段流程图（Pre-checks → Tool dispatch → Thinking → Self-critique → Action → Cap）

核心公式：**coding agent = AI model(s) + harness**

---

## OpenAI Codex 百万行代码实验

[[驾驭工程#OpenAI 百万行代码实验]]

![[笔记同步助手/images/06c92a4e21afa860ee05eaeeb32f5c77_MD5.png]]
> OpenAI 博文标题截图：Harness engineering: leveraging Codex in an agent-first world

![[笔记同步助手/images/f13d2b8255e4d3dcf8b42b70f1d8416c_MD5.png]]
> Codex 验证闭环流程图：Chrome DevTools → Snapshot → UI Trigger → Runtime Events → Fix → Loop

四个核心理念的补充细节：

### 仓库就是大脑
仓库是 Agent 唯一的知识来源。代码、markdown、schema、可执行计划全都版本化存储。Agent 看不到仓库之外的东西，所以仓库必须包含 Agent 工作所需的一切。

![[笔记同步助手/images/e08add0aa550495be7ba071044264bdf_MD5.png]]
> Agent 知识的边界：Codex 能看到的（编码在代码仓库中的 markdown）vs 看不到的（Google doc、Slack、隐性知识）

### 给 Agent 写的代码
代码不仅要对人类可读，更要对 Agent 可读。Agent 更依赖**结构化线索**：严格分层架构、一致的命名模式、显式类型定义。这是新的可读性标准：**application legibility**（应用的可读性）。

![[笔记同步助手/images/e65a566290f03811c6737ab2a4357180_MD5.png]]
> 分层架构图：Types → Config → Repo → Service → Runtime → UI（每层只能依赖下层）

### 自主性分级
从简单任务开始逐步提升自主权限。后期单个 Codex 任务可连续运行 **6 小时以上**。**自主性提升必须建立在约束系统成熟基础上。**

### 合并哲学
每个 PR 合并前经过 linter、结构化测试和自动化检查。工程师做「审查」而非「修改」。如果一个 PR 需大量修改才能合并，反思 Harness 哪里出了问题。

Ryan Lopopolo：**Agent 不难，Harness 才难。**

---

## Harness 三根支柱

### 上下文工程
AGENTS.md 控制在 ~100 行，只做「目录」。ETH Zurich 研究：CLAUDE.md / AGENTS.md 应控制在 **60 行以内**。

![[笔记同步助手/images/5b8bb8a8e557edc44af00b3a26497009_MD5.png]]
> 本地可观测性栈：Logs/Metrics/Traces → Victoria 系列 → Codex 自查自修闭环

### 架构约束
确定性 linter + 结构化测试机械执行，而非 prompt 软约束。linter 报错信息嵌入修复指引。

Cursor 团队结论：**约束比指令更有效。告诉 Agent「不要留 TODO」比「完成所有实现」效果更好。**

反直觉洞察：**约束解空间反而让 Agent 更有生产力。**

### 熵管理
定期启动专门 Agent 扫描文档不一致、架构漂移，提交修复 PR。1 分钟内审查并合并。

改进飞轮：Agent 犯错 → 人类诊断 → 改进 Harness → Agent 不再犯 → 新错误出现 → 循环。

![[笔记同步助手/images/f22674689eb78f00f006007e5b79b7e6_MD5.png]]
> Harness 改进飞轮循环图

---

## 行业实践案例

### Stripe Minions：每周 1300 个 PR

[[驾驭工程#Stripe Minions]]

**每周合并 1300+ 个 PR，全部无人值守 Agent 完成。**

Blueprint 编排系统：确定性节点（linter/推送）+ Agentic 节点（实现功能/修复CI）混合。

![[笔记同步助手/images/f9129e6505746315f22628a49a330766_MD5.jpg]]
> Blueprint 编排图：确定性与 Agentic 节点交替

**CI 最多两轮**：第一轮失败→Agent 自动修复→再跑一次→还失败→交人类。

Toolshed 挂载 ~500 MCP 工具，但每个 Agent 只给精选子集。**更多工具 ≠ 更好表现。**

### Cursor Self-Driving：每小时 1000 个 commit

[[驾驭工程#Cursor Self-Driving]]

每小时约 1000 个 commit，一周超过 **1000 万次工具调用**。

经历四版迭代：
1. 单 Agent → 复杂任务扛不住
2. 多 Agent 共享状态文件 → 锁竞争严重
3. 结构化角色分工 → 太僵硬
4. 持续执行器 → 角色过载

最终版：**递归 Planner-Worker 模型**（根级 Planner → 子 Planner → Worker 独立操作）

关键发现：**差的初始指令会在数百个 Agent 间被放大**。

### Peter Steinberger：一个人的军队

2026 年 1 月单月 **6600+ 个 commit**，同时运行 5-10 个 Agent。OpenClaw 项目 4 个月内 18 万 stars。

激进做法：**不逐行审查代码**，Code Review 变成「Prompt Review」。

反常识发现：**喜欢算法谜题的工程师反而难适应 Agent 工作流，产品导向的工程师适应更快。**

2026 年 2 月加入 OpenAI。

---

## 数据对比：壳比模型值钱

[[驾驭工程#数据对比：壳比模型值钱]]

| 来源 | 同一模型 | 变量 | 效果 |
|------|---------|------|------|
| **Nate B Jones** | 同一模型 | 只改 Harness | 42% → 78%（≈换一代模型） |
| **LangChain** | gpt-5.2-codex | 只改 Harness | Terminal Bench 2.0: 52.8% → 66.5%（#30+ → 前5） |
| **Pi Research** | 15 个不同 LLM | 只改 Harness | 一个下午提升所有 LLM 编程能力 |
| **Vercel** | 同一模型 | 工具 15→2 | 准确率 80% → 100% |
| **Terminal Bench 2.0** | Claude Opus 4.6 | 换 Harness | 排名 #33 → #5（差 28 位） |

> [!important] 模型可能对特定 Harness 过拟合
> Claude Opus 4.6 在 Claude Code Harness 上排 #33，换不同 Harness 冲到 #5。

---

## 拐杖论：反对观点

[[驾驭工程#拐杖论（Bitter Lesson 阵营）]]

OpenAI 的 Noam Brown：

> Harness 就像一根拐杖，我们终将能够超越它。

历史支撑：推理模型出现前，开发者搭建了大量 Agentic 系统模拟推理能力。推理模型一出，这些基础设施一夜之间就不需要了。

预判：OpenAI 将走向**统一模型**的未来——不应该需要在模型上面再加一个路由器。

给开发者建议：**别花六个月搭建一个可能六个月后就被淘汰的东西。**

### METR 数据

4 位活跃维护者审查 296 个 AI 生成 PR：**维护者合并率 ≈ 自动评分通过率的一半**。

Claude Sonnet 4.5：自动评分对应 50 分钟任务范围，维护者实际合并对应仅 8 分钟 → **7 倍能力高估**。

Claude Code 和 Codex 并不比基础脚手架表现更好。Harness 选择基本落在误差范围内。

---

## 路线之争：Big Model vs Big Harness

[[驾驭工程#路线之争]]

![[笔记同步助手/images/c0fcd15907487356d621f45c050aa39d_MD5.png]]
> Big Model vs Big Harness 路线之争对比图

### Big Model 阵营
**核心观点**：模型能力增长才是主旋律，Harness 只是权宜之计。

Claude Code 团队 Boris Cherny & Cat Wu：

> 所有的秘密武器都在模型本身。我们追求的是最薄的那层包装。

### Big Harness 阵营
**核心观点**：模型是引擎，Harness 是方向盘和刹车。

Jerry Liu（LlamaIndex 创始人）：

> Model Harness 就是一切。从 AI 那里获取价值的最大障碍，是你自己为模型做上下文工程和工作流工程的能力。

数据支撑：开发者 60% 工作使用 AI，但完全委托的只有 **0-20%**。Cursor $50B 估值印证 Harness 方向价值。

---

## 护栏悖论

[[驾驭工程#护栏悖论]]

![[笔记同步助手/images/e55261f52edd6e4928020309af4f06c8_MD5.png]]
> 护栏悖论：车速越快护栏越重要——自行车道(30km无护栏) → 高速公路(120km标配护栏) → 磁悬浮(300km全封闭)

**车速越快，护栏越重要。**

模型越强（引擎越强），速度越快，就越需要精心设计的约束系统确保跑在正确方向上。

Noam Brown 说得对：很多脚手架会随模型进化被淘汰。但**架构约束、反馈循环、熵管理不会消失，只会换形态**。

从马车到汽车：马鞭消失了，方向盘和刹车不会消失。

**Harness 阵营真正应该担心的**：自己搭建的 Harness 六个月后是否就过时了。

Philipp Schmid 三词建议：**Start Simple. Build to Delete.**

---

## 七个配置杠杆

[[驾驭工程#七个配置杠杆]]

![[笔记同步助手/images/e83475ebc71b370a2fcc7762569f26b4_MD5.png]]
> Harness 的七个配置杠杆分类图

| 杠杆 | 说明 | 关键要点 |
|------|------|---------|
| **AGENTS.md / CLAUDE.md** | 仓库根目录 markdown 文件，写架构约定、命名规范 | **控制在 60 行以内**，写「目录」别写「百科全书」 |
| **确定性约束** | linter、类型检查、结构化测试、pre-commit hooks | 硬约束比软性 prompt 指令更可靠 |
| **工具精简** | 别给 Agent 塞太多工具 | Vercel 15→2，准确率反而升 |
| **Sub-Agent 隔离** | 复杂任务拆成子任务，各有独立上下文 | 防止噪声累积，可用不同模型 |
| **反馈循环** | Agent 自己跑测试、看截图、查日志 | 让 Agent 自验证产出 |
| **CI 限速** | 最多两轮 CI | 跑两次还不过→交人类 |
| **垃圾回收** | 定期扫描技术债、过时文档、架构漂移 | 容易忽略但极重要 |

---

## 未来趋势

- **Harness 成为新的「服务模板」**：未来组织可能从预制 Harness 模板中选择并定制
- **技术栈收敛**：AI 友好性可能超过开发者个人偏好，成为选型首要考量
- **Harness 反哺模型训练**：Agent 失败轨迹 → 模型训练高质量数据
- **「旧代码」问题**：给几十万行代码的老项目加 Harness，可能像给从不跑测试的项目补测试一样痛苦
- **学科化**：AIE Europe 设立首个 Harness Engineering 专题赛道，arxiv 有专门论文

---

## 本质是管理

Harness Engineering 说的这些——上下文管理、架构约束、反馈循环、定期清理——**不就是管理吗？**

好的技术 leader 怎么带团队：
- 给新人写 onboarding 文档 → AGENTS.md
- 定代码规范和架构原则 → linter 和结构化测试
- 做 code review 确保质量 → CI/CD 检查
- 定期做技术债清理 → 垃圾回收
- 给工具做精简和选型 → 工具链管理
- 遇到反复出现的问题写进 wiki → 反馈循环

> **AI Agent 越强，就越像一个能力很强但需要管理的员工。**

未来最吃香的 AI 工程师，可能是最懂「管理」的那种——管理 Agent。

---

## 关联知识

- **核心概念**：[[驾驭工程]]、[[Harness Engineering深度解析——从有序驾驭无序的工程哲学]]、[[Agent技术范式演变（2023-2026）]]
- **工程实践**：[[Prompt与Context工程（三次进化）]]、[[工程设计与安全]]、[[系统提示词工程]]
- **上下文工程**：[[上下文工程]]、[[上下文注入]]、[[上下文压缩]]、[[上下文窗口管理]]
- **安全与约束**：[[护栏与钩子系统]]、[[重试与恢复机制]]、[[工程设计与安全]]
- **多智能体**：[[多智能体协作]]、[[角色分配]]、[[任务分解]]

## 标签

#学习笔记 #驾驭工程 #Harness #行业案例 #路线之争 #拐杖论