---
author: Vayne-LW
source: 微信公众号
url: https://mp.weixin.qq.com/s?__biz=MzU0NDE2NDU1NA==&mid=2247483830&idx=1&sn=bb97e6813ef592ae0ba8e46ebedde4e5&chksm=faffae2b170958bda694a35d4c977889d6b52eb9be0c03e8401637ab9b80eb22448f939e3c1a&mpshare=1&scene=1&srcid=0611Z0I0NXVlCJTOXhAXVVeY&sharer_shareinfo=a225c3d9442d332bacf64c57b7a1305a&sharer_shareinfo_first=a225c3d9442d332bacf64c57b7a1305a#rd
saved: 2026-06-11 17:18:49
tags:
  - 笔记同步助手
id: 3ee3f8a4-7169-4d60-a03d-c2e158481252
---

公众号名称：manisfast

作者名称：Vayne-LW

发布时间：2026-05-25 02:13

> 本文是 Hermes Agent 系列的进阶实战篇。上一篇我们解决了"Hermes 怎么安装、Memory / Skill / Nudge Engine 是什么"。这篇要解决的问题是：**怎么把 Hermes 从一个能聊天、能执行命令的终端 Agent，配置成一个真正能长期运行、持续学习、主动观察外部世界、沉淀个人能力的 Agent 系统。**

---

## 先聊聊"满配"到底是什么

很多人理解的满配，是装尽可能多的插件、接尽可能多的 MCP、开尽可能多的工具、配尽可能复杂的多 Agent，让它看起来像一个很酷的 AI 控制台。

但真正用一段时间后你会发现，这种满配很容易变成另一种负担：

-   工具太多，Agent 不知道什么时候该用
    
-   Skill 太多，选择成本变高
    
-   Memory 太乱，长期偏好被临时信息污染
    
-   MCP 太多，Token 和权限风险都变大
    
-   Profile 太多，自己都不知道哪个 Agent 记住了什么
    
-   Gateway 开了，但安全边界没想清楚
    
-   Cron 跑了，但它到底做了什么你看不见
    

所以我理解的 Hermes 满配不是"装满"，而是：

> **围绕你的真实工作模式，把输入、记忆、技能、工具、自动化、可视化、Token 成本和多 Agent 协同这些能力，组合成一个可长期维护、可持续进化的个人 Agent 系统。**

全文 20 章，分四个部分：

**第一部分：系统总览和基础（第 1-4 章）**先搞清楚满配到底要配什么，然后把安装、模型、输入系统和 Memory 这四个地基打稳。

**第二部分：能力模块逐个上手（第 5-12 章）**Skills 技能系统 → Tools 工具控制 → MCP 外部连接 → Gateway 飞书接入 → Cron 定时任务 → Token 优化 → 可视化 → 语音/网页/搜索。每个模块单独一章，可以按顺序读，也可以跳到你需要的章节。

**第三部分：多实例与多 Agent（第 13-16 章）**Profile 多实例隔离 → 24h Agent 团队架构 → 生态导航工具 → 推荐满配路线。这部分是从"单兵"到"协作系统"的升级。

**第四部分：两个完整实战（第 17-18 章）**AI 工具雷达 Agent（自动搜集 + 每日报告）、头脑风暴聊天室 Agent（多角色讨论 + 知识卡片）。把前面所有模块串起来，跑通一个完整的闭环。

最后是满配清单和总结（第 19-20 章）。

---

> 这篇文章内容比较多，建议先收藏，跟着自己的节奏慢慢上手。细节上如果有纰漏，希望谅解——遇到不确定的地方，可以结合 Hermes 官方文档和 AI 工具搞清楚，也欢迎私信沟通。

---

# 一、先给结论：Hermes 满配分成 12 层

一个裸装 Hermes 长这样：

```
hermes
```

它能对话、能调用工具、能执行命令。但一个真正"满配"的 Hermes，应该至少考虑下面 12 层：

| 层级 | 模块 | 解决的问题 | 推荐程度 |
| --- | --- | --- | --- |
| L1 | 安装与模型 Provider | 先让 Hermes 稳定跑起来 | 必配 |
| L2 | 输入系统：SOUL.md / AGENTS.md / CLAUDE.md | 让 Hermes 理解你和当前项目 | 必配 |
| L3 | Memory 长期记忆 | 跨会话记住偏好、目标、环境和经验 | 必配 |
| L4 | Skills 技能系统 | 把稳定流程沉淀成可复用能力 | 必配 |
| L5 | Tools / Toolsets | 控制 Hermes 能调用哪些本地能力 | 必配 |
| L6 | MCP 外部工具连接 | 接 GitHub、文件系统、搜索、数据库、浏览器等 | 进阶必配 |
| L7 | Gateway 消息入口 | 让 Hermes 出现在 Telegram / 飞书 / Discord / 邮件 | 按需配置 |
| L8 | Cron 自动化 | 定时做日报、雷达、巡检、周报 | 强烈推荐 |
| L9 | Profiles 多实例隔离 | 为不同场景创建独立 Agent | 强烈推荐 |
| L10 | 可视化与可观测 | 看见它做了什么、哪里失败、花了多少 Token | 进阶必配 |
| L11 | Token 精简与上下文管理 | 降成本、提速度、减少上下文污染 | 进阶必配 |
| L12 | 多 Agent / 24h Agent 团队 | 让多个角色长期协作 | 高阶玩法 |

这 12 层不是让你一天全部装完，而是给你一张路线图。

我的建议是：

> 先跑通 → 再记住你 → 再沉淀技能 → 再接工具 → 再定时运行 → 再多入口触达 → 最后再多 Agent 协同。

不要一开始就搭 24h Agent 团队。那样很酷，但如果基础的 Memory、Skill、MCP、权限和可观测都没做好，最后只是制造一个更复杂的失控系统。

---

# 二、安装基线：先保证 Hermes 本体稳定

## 新手最容易犯的错

很多人安装 Hermes 后，第一件事就是搜资源：

```
hermes plugins install xxx
hermes skills install xxx
hermes gateway setup
hermes cron create ...
```

但官方 Quickstart 里有一个非常重要的原则：

> 如果 Hermes 连一个普通聊天都不能稳定完成，就不要继续叠加 Gateway、Cron、Skills、Voice 或 Routing。

满配的第一步不是满配，而是"干净可用"。

## 推荐安装方式

**方式 A：PyPI**（适合追求稳定的读者）

```
pip install hermes-agent
hermes setup
hermes model
hermes doctor
hermes status
```

**方式 B：官方安装脚本**（适合希望更接近主分支的读者）

```
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

Windows 用户建议在 WSL2 里运行。

安装完成后：

```
source ～/.bashrc
# 或
source ～/.zshrc
```

## 选择模型 Provider

```
hermes model
```

如果你已经熟悉某个模型平台，就先用你最熟悉的。不要一开始就搞多 Provider、多 Key 池、多模型路由。先保证一个 Provider 能稳定响应。

## 最小验证

```
hermes
```

输入：

> 你好，请用一句话说明你现在能正常工作。

再运行：

```
hermes doctor
```

如果 `hermes doctor` 还有明显错误，先修环境，不要继续装插件。

## 常用排查命令

```
hermes doctor
hermes model
hermes setup
hermes config show
hermes sessions list
hermes --continue
```

一句话总结：

> 满配前先学会回到"已知可用状态"。否则后面装得越多，越不知道哪里坏了。

---

# 三、输入系统：让 Hermes 真正懂你

很多人觉得 Agent 不聪明，其实不是模型不聪明，而是输入系统太差。

Hermes 的输入系统像一个分层协议栈，每个文件解决一个层面的问题：

```
SOUL.md
  → 这个 Agent 应该怎么工作（立场、职责、边界、风格）

USER.md
  → 我是谁、我喜欢什么、我讨厌什么（用户画像和偏好）

MEMORY.md
  → 我在做什么项目、环境事实、踩坑经验、工作流约定（Agent 工作笔记）

AGENTS.md / .hermes.md
  → 当前目录的规则、代码规范、目录说明、运行方式

SKILL.md
  → 可复用任务流程，比如周报生成、访客画像抽取、工具分析

当前对话
  → 临时任务、一次性要求、当前上下文
```

这六层不是随便分的。Hermes 内置记忆由两个文件组成：`MEMORY.md`（约 2200 字符）偏 Agent 的工作笔记，`USER.md`（约 1375 字符）偏你的画像和偏好。它们会在每次 session 开始时作为快照注入 system prompt；会话中写入的新记忆一般要到下次 session 才真正生效。

下面一个个讲怎么写。

---

## 3.1 SOUL.md：不要写玄学人格，要写工作协议

### 先看一个反面教材

```
你是一个聪明、强大、无所不能的 AI 助手。
```

这种写法的问题不是语气不对，而是它没有给 Agent 任何可以执行的规则。Agent 不知道什么时候该主动、什么时候该刹车、什么时候可以不同意你。

### 一个更好的写法

SOUL.md 应该回答的不是"你是什么性格"，而是"你应该怎么和我协作"。下面是一个可以直接改的模板：

```
# SOUL

你是我的个人 Agent。你的职责是优化我的工作流，保护我的注意力，
推进最有价值的工作，把意图转化为有组织的执行。

---

## 立场

保持直接、务实、有判断力、高主动性。

当我表达模糊、不现实、分心或正在制造混乱时，你要提出反对。

有用比顺耳重要。锋利比润色重要。诚实比显得厉害重要。

说重点，然后停止。

---

## 职责

你不要等待完美指令。主动发现机会、指出问题、识别停滞循环，推动事情向前。

当直接执行最快时，直接执行。
当拆分任务、专家视角或并行推进能带来更好结果时，进行委派。

你的职责不是生产一堆最后进坟场的材料。你的职责是制造推进。

---

## 自主性边界

你拥有较大的自主决策空间，但有一条硬边界：

以下事项没有我的明确批准绝不能执行：
- 公开发布内容或对外上线
- 购买东西、注册付费服务
- 向真实的人发送消息
- 删除重要工作或做不可逆修改
- 暴露私人信息、修改凭证或安全设置

除此之外，如果你对判断有信心，就推进。不要为低风险工作反复请求许可。

---

## 反对与纠偏

你可以直接不同意我，但需要先赢得反对的资格。

每一个反对意见都需要：数据、例子、推理，或一个更好的替代方案。

不要为了保护我的自尊而隐瞒有用的真相。

当你提出反对时，说明：
- 弱点在哪里
- 哪个假设尚未被证明
- 你会怎么做

---

## 沟通风格

任务简单时，简短回答。
任务复杂时，结构化回答。
任务有风险时，明确写出权衡。

优先使用朴素语言。避免企业黑话、虚假兴奋和"在当今快节奏的世界里"这类表达。

面向公众的内容应该像一个真实的人写出来的：有品味、有伤痕、有观点。

---

## 自我改进

当我纠正你时，把纠正保存到合适的位置。

当某个工作流重复出现时，考虑它是否应该变成检查清单、模板、脚本或可复用 Skill。

不要让重复摩擦保持隐形。
```

### 关键设计思路

这个模板的核心不是"让 Agent 听起来更厉害"，而是解决几个实际问题：

1.  **立场**：告诉 Agent 可以主动、可以反对，不需要事事请示
    
2.  **自主性边界**：明确哪些事可以自己做、哪些必须确认，减少无效确认
    
3.  **反对与纠偏**：让 Agent 知道什么时候该说"不"，以及怎么说
    
4.  **自我改进**：把重复出现的流程沉淀下来
    

> **SOUL.md 是 Agent 的"工作协议"，不是人设卡。好的 SOUL.md 不是让 Agent 听起来更像人，而是让它做事更像一个靠谱的搭档。**

你可以根据自己情况增减维度。比如如果你经常让 Agent 帮你写公众号，可以在"沟通风格"下加一条面向公众写作的具体要求。

---

## 3.2 USER.md + MEMORY.md：你是谁 vs 你在做什么

很多人把所有东西都往 Memory 里塞，结果 Agent 记住了一堆临时想法，反而记不住你真正稳定的工作偏好。

Hermes 的内置记忆分成两个文件是有原因的：

| 文件 | 定位 | 容量 | 适合写什么 |
| --- | --- | --- | --- |
| USER.md | 你的长期画像 | ～1375 字符 | 身份、偏好、沟通风格、期望 |
| MEMORY.md | Agent 的工作笔记 | ～2200 字符 | 项目、环境、踩坑、工作流、决策原则 |

下面各给一个模板，你可以直接改。

### USER.md 模板

```
# USER

## Profile

- 你的职业身份
- 你的技术背景
- 你当前最关注的 3 个方向

## Communication Preferences

- 默认使用什么语言
- 喜欢简短回答还是结构化长回答
- 是否接受 Agent 直接反驳
- 讨厌什么类型的回答（比如空泛概念堆砌、过度解释基础概念）

## Output Preferences

- 技术方案要包含什么（目标、架构、模块、MVP、风险、验收标准）
- 业务方案要包含什么（业务价值、资源需求、预期结果、风险）
- 写作任务要包含什么（标题、观点、读者、结构、案例、配图建议）

## Collaboration Style

- 你是否经常在多个想法之间切换（需要 Agent 帮你收敛优先级）
- 当你持续开新任务不关旧任务时，是否希望 Agent 提醒你聚焦
- 你更看重真实判断还是情绪安慰
```

### MEMORY.md 模板

```
# MEMORY

## Active Context

- 你当前正在围绕什么方向做长期探索
- 当前重点不是什么（避免 Agent 误解方向）

## Current Priorities

1. 当前最重要的 1-3 件事
2. 每件事的目标是什么

## Active Projects

### 项目 A
- 目标：
- 当前进展：
- 最大阻塞：
- 下一步：

### 项目 B
- （同上）

## Decision Principles

- 遇到复杂问题，优先拆成：目标、现状、问题、方案、MVP、风险、下一步
- 遇到 Agent 类任务，优先用：Memory、Skill、Eval、Tools、Human-in-the-loop
- 遇到工程任务，先查看文件和项目结构，不要臆测

## Known Pitfalls

- 不要把临时想法写成长期承诺
- 不要在没有验证时编造命令、路径或工具能力
- 不要让项目无限发散

## Environment Notes

- 你常用什么本地工具（Hermes、Claude Code 等）
- 有什么环境特殊性（比如某些工具只在特定 profile 下可用）

## Memory Maintenance Rules

- 只保存长期稳定、会反复影响回答质量的信息
- 过期项目、临时情绪、一次性任务不要长期保留
- 当 memory 变长时，优先压缩成原则和项目摘要
- 当你纠正某个长期偏好时，更新对应条目而不是重复追加
```

### 两个文件的核心区别

一句话：

> **USER.md 写"我是谁"，MEMORY.md 写"我在做什么"。**

你的职业身份、沟通偏好、输出期望——这些很少变，放 USER.md。 你当前的项目、环境、踩坑经验——这些会随时间更新，放 MEMORY.md。

---

## 3.3 记忆初始化：让 Hermes 通过访谈帮你写

上面的模板看着不难，但很多人不知道该往里填什么。

一个实用的做法是：**直接让 Hermes 通过访谈帮你生成。**

在 Hermes 里输入：

```
请帮我初始化 Memory。先通过访谈了解我，然后自动生成 USER.md 和 MEMORY.md。

访谈问题请覆盖以下 6 个方面，每个方面问 2-3 个问题：

1. 用户画像：职业身份、技术背景、当前最关注的方向
2. 沟通偏好：默认语言、回答风格、是否接受反驳、讨厌什么回答
3. 当前项目：最重要的 1-3 个项目、目标、进展、阻塞
4. 工作流：有哪些重复性任务、哪些适合沉淀成 Skill
5. 决策原则：做决策时最看重什么、容易犯什么错误、什么情况需要提醒
6. 工具环境：常用什么 Agent 工具、有什么环境特殊性

访谈结束后，输出两份内容：
- USER.md：只放我的长期画像和偏好
- MEMORY.md：只放项目、环境、工作流、经验和注意事项

注意：
- 不要写入 API Key、密码、隐私信息、临时情绪和一次性任务
- 如果某条信息更适合做成 Skill，单独列出，不要塞进 Memory
- 内容要压缩，不要超过 Hermes 内置 memory 的字符限制
```

Hermes 会依次问你问题，然后自动生成两个文件的内容。你审核一遍、改改措辞、确认没有敏感信息，就可以写入。

> **不要一次写完就再也不动了。Memory 应该像你的工作笔记一样，定期清理和更新。当你发现 Agent 反复犯同一个错误，说明那条信息该写进 Memory 了；当你发现 Memory 里有过期项目，说明该清理了。**

---

## 3.4 记忆写入原则

不是所有信息都应该写进 Memory。

### 适合写入

-   长期项目背景
    
-   稳定偏好
    
-   本地环境事实
    
-   常用路径和工具配置
    
-   反复踩过的坑
    
-   当前活跃目标
    
-   已验证有效的决策原则
    
-   工作流经验
    

### 不适合写入

-   人设口号（已经写在 SOUL.md 了）
    
-   临时任务（放在当前对话就够）
    
-   大段资料和文章全文（放文件系统）
    
-   API Key / Token / 密码（永远不要写）
    
-   过期计划、临时情绪、一次性聊天内容
    

### 信息分层

更准确地说，Hermes 的信息应该分成四层：

| 信息类型 | 放哪里 | 生命周期 |
| --- | --- | --- |
| 稳定偏好、长期目标 | USER.md / MEMORY.md | 长期，定期更新 |
| 反复执行的流程 | Skill | 长期，按需调用 |
| 一次性任务过程 | Session | 当前会话 |
| 大量资料、文章、日志 | 文件系统 / 知识库 | 按需检索 |

一句话：

> **USER.md 记录"我是谁"，MEMORY.md 记录"我在做什么"，Skill 记录"我怎么做事"，Session 记录"这次发生了什么"，文件系统保存"大量原始资料"。**

### 一个反例：不要把整篇文章写进 Memory

你读了一篇关于 Agent 自进化的文章。

不应该写入：

> 全文 8000 字内容……

应该写入：

> 用户关注"Agent 自进化中的外部信息筛选机制"。他认为关键不在于收藏更多信息，而在于让新信息经过评估、讨论、沉淀，进入 Memory / Skill / Eval 系统。

> **Memory 不是越完整越好，而是越高频、稳定、可复用、能改变回答质量越好。**

---

## 3.5 AGENTS.md / CLAUDE.md：让它理解当前目录

SOUL.md、USER.md、MEMORY.md 解决的是"你是谁"和"你在做什么"。但 Hermes 还需要知道"当前这个目录是干什么的"。

比如你准备做一个 Hermes 实验目录：

```
mkdir -p ～/hermes-lab
cd ～/hermes-lab
```

放一个 `AGENTS.md`：

```
# AGENTS.md

## 项目目标

这是我的个人 Agent 实验目录，用于测试 Hermes 的 Memory、Skill、MCP、Cron、Profiles 和多 Agent 能力。

## 目录结构

- ai-radar/：AI 工具雷达
- brainstorm-room/：头脑风暴聊天室
- reports/：所有输出报告
- skills/：自定义 Skill 草稿

## 工作规则

- 所有报告输出为 Markdown；
- 所有测试数据使用虚构数据；
- 不读取公司文件；
- 不上传密钥、真实聊天记录和隐私数据；
- 安装工具前必须先生成风险说明；
- 删除文件、安装依赖、发送外部消息都需要人工确认。
```

这类文件可以让 Hermes 进入某个目录后，快速理解当前工作空间的目标和边界。

> **如果你同时在多个项目间切换，每个项目放一个 AGENTS.md 比把所有项目信息都塞进 MEMORY.md 更有效。因为 MEMORY.md 每次都会加载，而 AGENTS.md 只在你进入那个目录时才加载。**

---

## 3.6 好的输入模板

前面讲了怎么配置系统级的输入。最后说说日常对话时，怎么把单次任务说清楚。

坏输入：

> 帮我看看这个工具值不值得装。

好输入：

> 任务：帮我分析这个 GitHub 项目是否值得加入我的个人 Agent 系统。
> 
> 背景：
> 
> -   我关注 Memory、Skill、Eval、MCP、Agent 自进化；
>     
> -   我倾向于低维护、本地优先、可迁移；
>     
> -   我不希望系统变得太复杂。
>     
> 
> 输入：
> 
> -   GitHub 链接：xxx
>     
> -   README 摘要：xxx
>     
> 
> 请输出：
> 
> 1.  这个项目解决什么问题；
>     
> 2.  和我现有系统的关系；
>     
> 3.  是否值得安装；
>     
> 4.  如果不安装，是否值得吸收设计思想；
>     
> 5.  风险和依赖；
>     
> 6.  最终建议：安装 / 试用 / 观察 / 忽略。
>     

好的输入不是把话说长，而是把这四件事说清楚：

> **背景 → 目标 → 约束 → 输出格式**

> **如果你发现自己在反复用同一种方式描述任务，说明这个模式应该沉淀成 Skill。这就是第五章要讲的事情。**

---

# 四、Memory：记忆系统选型

上一章讲了 USER.md 和 MEMORY.md 怎么写。这章讲记忆系统本身的选型。

## 4.1 Memory 常用命令

```
hermes memory status
hermes memory setup
hermes memory off
```

如果你不确定当前记忆后端是什么，先看：

```
hermes memory status
```

## 4.2 内置记忆的优点与局限

内置 Memory 的优点：

-   零配置
    
-   和 Hermes 原生流程集成
    
-   适合轻量使用
    
-   不需要额外云服务
    

局限：

-   容量有限（USER.md ～1375 字符，MEMORY.md ～2200 字符）
    
-   对大规模个人知识库不够
    
-   对语义召回、冲突更新、时间衰减的表现需要实际测试
    
-   不适合把大量文章、日志、历史记录都塞进去
    

> 新手先用原生 Memory，重度用户再评估外部 Memory Provider。

---

# 五、Skills Hub：Hermes 的"程序化记忆"

## Skill 是什么？

Skill 不是插件，也不是普通 Prompt。更准确地说：

> **Skill 是一种程序化记忆：当你反复用同一种方式完成任务时，可以把这套流程写成文档，让 Hermes 以后自动复用。**

但很多读者分不清 Skill 和 Memory、MCP Tool、普通 Prompt 的区别。一张表讲清楚：

| 概念 | 解决什么问题 | 举个例子 |
| --- | --- | --- |
| **Memory** | 记住"你是谁"和"你在做什么" | 你的偏好、项目背景、踩坑经验 |
| **Skill** | 记住"你怎么做事" | 分析工具是否值得装的固定流程 |
| **MCP Tool** | 连接"你能用什么外部工具" | GitHub 搜索、网页读取、数据库查询 |
| **Prompt** | 这次对话的临时指令 | "帮我看看这个链接" |

简单说：**Memory 是你的知识，Skill 是你的方法论，MCP Tool 是你的工具箱，Prompt 是你这次说的话。**

这些适合写成 Skill：

-   分析一个 GitHub 工具是否值得安装
    
-   把文章转成知识卡片
    
-   组织多角色头脑风暴
    
-   生成周报
    
-   检查 MCP Server 权限风险
    

特征是一样的：你会反复做，而且每次的判断逻辑差不多。

## 常用命令

```
# 看你现在装了哪些 Skill
hermes skills list

# 搜社区有没有你需要的 Skill（比如搜 memory、planning、writing）
hermes skills search 

# 安装社区 Skill
hermes skills install 

# 安装前先看看它里面写了什么（关键步骤，不要跳过）
hermes skills inspect 

# 浏览技能市场
hermes skills browse

# 检查已安装的 Skill 是否有更新
hermes skills check

# 更新全部 Skill
hermes skills update
```

在聊天中也可以用：

```
/skills
```

> **建议的工作流：先 `search` 找到感兴趣的 → 再 `inspect` 看内容 → 确认安全后再 `install`。不要直接装。**

---

## 从零写一个 Skill：手把手实操

这是本章最核心的部分。我们用一个最简单的 Skill 来走一遍完整流程。

### Step 1：创建 Skill 文件

Skill 文件就是一个 `.md` 文件，放在 Hermes 的 skills 目录下：

```
mkdir -p ～/.hermes/skills
```

### Step 2：写 Skill 文件

一个 Skill 文件分两部分：**frontmatter**（元信息）和 **body**（流程说明）。

```
---
name: article-summarizer
description: 把长文章压缩成结构化摘要卡片。
---

# Article Summarizer

## 什么时候使用

当用户提供一篇长文章、论文、博客或 md 文件，需要生成结构化摘要时使用。

## 输出格式

请输出以下结构：

### 一句话总结
（50 字以内）

### 核心观点
（3-5 个要点，每个一句话）

### 和我的关系
（结合 USER.md 中的长期目标，判断这篇文章对我的价值）

### 下一步
（是否值得深入读？是否沉淀为 Memory？是否触发某个 Skill？）
```

这就是一个完整的 Skill。总共 20 多行，结构清晰，Hermes 能直接理解。

### Step 3：安装到 Hermes

```
hermes skills install ～/.hermes/skills/article-summarizer.md
```

或者如果 Hermes 支持自动发现 skills 目录，重启后自动加载。

### Step 4：验证生效

在 Hermes 中输入：

```
请使用 article-summarizer 技能，帮我总结这篇文章：～/hermes-lab/inputs/xxx.md
```

如果 Hermes 正确输出了"一句话总结 / 核心观点 / 和我的关系 / 下一步"这个结构，说明 Skill 生效了。

### Step 5：迭代

如果输出不对，改 `.md` 文件，重新安装。Skill 是纯文本，改起来很快。

> **写 Skill 的核心原则：不要一次写得太复杂。先让它跑起来，再逐步加维度和约束。一个 20 行的可用 Skill，比一个 200 行但没人用的 Skill 有价值得多。**

---

## Skill 文件结构详解

上面那个例子用了最简单的结构。如果你想让 Skill 更精确，可以加更多维度。一个完整的 Skill 文件结构：

```
---
name:           # 必填，英文短名，用于命令调用
description: <一句话描述>   # 必填，说明这个 Skill 做什么
---

# 

## 什么时候使用
（触发条件：什么场景下该用这个 Skill）

## 分析维度
（判断逻辑：按什么维度分析）

## 输出格式
（结构化输出：用什么格式呈现结果）

## 约束
（可选：边界条件、禁止事项、特殊情况处理）
```

> 完整的真实案例见第十七章"实战一：AI 工具雷达 Agent"，那里有一个 8 个维度的工具分析 Skill。

---

## Skill 安装前的 5 个问题

不管是社区 Skill 还是自己写的，装之前先问：

1.  它是否服务我的长期目标？
    
2.  它是否能减少重复劳动？
    
3.  它是否和已有 Skill 重复？
    
4.  它是否需要危险权限？
    
5.  它适合常驻，还是只适合临时启用？
    

> **如果你发现自己犹豫了超过 30 秒，说明这个 Skill 先不装。等下次真的遇到那个场景时再装也不迟。**

---

## 五大王牌 Skill：社区验证过的必备能力

Hermes 社区已有数千个 Skill，但真正经过大规模验证、适合大多数用户的，是下面五个：

| Skill | 星标 | 一句话定位 | 解决什么问题 |
| --- | --- | --- | --- |
| **gstack** | 85K | 高效的工作流与技能栈 | 解决“Agent 如何高质量做事” |
| **gbrain** | 11K | Agent 的共享外部大脑 | 解决“Agent 如何长期记住并召回上下文” |
| **hermes-webui** | 4.6K | 轻量化可视化控制台 | 给 Hermes 加一个 Web 界面，降低使用门槛 |
| **self-evolution** | 2.4K | 自进化核心模块 | 让 AI 持续自主迭代，越用越聪明 |
| **awesome-hermes** | 1.9K | 全生态项目合集导航 | 快速查找 Hermes 生态的工具、教程、最佳实践 |

### gstack：一套高强度 Agent 工作流与技能栈

Hermes / Claude Code 这类工具默认只是一个通用 Agent，而 gstack 更像是一套成体系的“AI 工作方法包”：它通过一组 skills、slash commands 和角色化流程，把 AI 拆成产品、工程、设计、评审、QA 等不同工作视角，帮助你完成从思考、规划、编码、审查到交付的完整链路。

适合：想把 Claude Code / Hermes 用成高强度个人研发团队的人；需要结构化规划、代码审查、产品判断、QA、交付流程的开发者或独立创作者。

### gbrain：Agent 的共享外部大脑

gbrain 不是多 Agent 调度器，而是一个记忆与知识检索层。它把项目文档、代码、笔记、人物、公司和历史决策沉淀成可检索的长期上下文，让 Hermes 或多个 Agent 在执行任务时能从同一个“外部大脑”里读取背景信息。

适合：长期项目、个人知识库、代码仓库理解、人物/公司关系沉淀、多个 Agent 需要共享背景信息的场景。

### hermes-webui：轻量化可视化控制台

不是所有人都喜欢命令行。hermes-webui 提供了一个简洁的 Web 界面，让你用浏览器就能和 Hermes 对话、查看 Memory、管理 Skill 和 MCP。

适合：想要更低门槛使用 Hermes 的用户，或者想让团队成员也能用上的场景。

### self-evolution：自进化核心模块

这个 Skill 让 Hermes 能自主检测自己的不足（比如反复犯同一类错误、输出质量下降），然后自动调整 prompt、更新 Memory、优化 Skill。本质上是让 AI 具备"元认知"能力。

适合：希望 Hermes 越用越顺手、减少人工调教的高频用户。

### awesome-hermes：全生态导航

一个持续更新的资源合集，涵盖 Hermes 的工具、教程、模板、最佳实践、社区项目。相当于 Hermes 生态的"黄页"。

适合：想快速了解 Hermes 还能做什么、找具体方案的探索型用户。

### 安装方式

```
# 先 inspect 看看内容，确认安全后再 install
hermes skills inspect gstack
hermes skills install gstack

# 或者一次性安装多个
hermes skills install gstack gbrain awesome-hermes
```

> **不要一次全装。** 先装 1-2 个你当前最需要的，跑通一个再装下一个。每个 Skill 都会占用 system prompt 空间，装太多反而会降低 Agent 的注意力。

---

## Skill 生命周期管理

很多人装了一堆 Skill 就再也不管了。但 Skill 和代码一样，需要维护。

### 更新

```
# 检查是否有更新
hermes skills check

# 更新全部
hermes skills update

# 更新单个
hermes skills update 
```

### 禁用和删除

```
# 禁用（不删除文件，但不再加载）
hermes skills disable 

# 删除
hermes skills remove 
```

如果你是自定义 Skill，直接删除或移走 `.md` 文件即可。

### 每周清理：问自己 3 个问题

建议每周花 5 分钟做一次 Skill 清理：

1.  **过去 7 天我实际用了哪些 Skill？**——没用的，考虑禁用。
    
2.  **有没有两个 Skill 做的事重叠？**——合并成一个。
    
3.  **有没有 Skill 的输出质量下降了？**——更新或重写。
    

### 整理原则

> **常驻 Skill 不超过 5 个。**如果超过 5 个，说明你在用 Skill 数量代替 Skill 质量。

一个好的信号是：你不需要刻意记自己有哪些 Skill，因为在日常工作中它们会自然地被触发。如果你经常需要翻 `hermes skills list` 才能想起来"我还有这个 Skill"，说明它可能该禁用了。

---

# 六、Tools 与 Toolsets：控制 Hermes 的手

上一章讲了 Skill——Agent "怎么做事"的流程。这章讲 Tool——Agent "能做什么"的原子能力。

一个类比：**Tool 是锤子、螺丝刀，每个只做一件事。Skill 是组装书架的工序，编排多个工具的调用顺序。**

Hermes 内置了 47 个工具。你不需要全部记住，只需要知道三件事：哪几类工具、哪些是高频的、哪些需要谨慎。

## 6.1 常用命令

```
# 看你有哪些工具（名称 + 类别 + 是否需要审批）
hermes tools list

# 查看某个工具的参数和用法
hermes tools info 

# 启用 / 禁用某个工具
hermes tools enable 
hermes tools disable 
```

什么时候用什么命令：

-   刚装好 Hermes → `hermes tools list` 扫一眼
    
-   Agent 执行了你不想要的操作 → `hermes tools disable` 关掉它
    

## 6.2 工具分类速查

47 个工具分成 7 类。下面只列每类最高频的 2-3 个：

| 类别 | 高频工具 | 做什么 | 风险 |
| --- | --- | --- | --- |
| **文件系统** | `read_file`/ `edit_file` / `list_directory` | 读写文件、目录列表 | 安全 ～ 低风险 |
| **代码执行** | `execute_python`/ `execute_shell` | 运行代码或命令 | 中 ～ 高风险 |
| **Web** | `web_search`/ `web_extract` | 搜索和抓取网页 | 安全 |
| **终端** | `run_command`/ `system_info` | 执行系统命令 | 高风险 |
| **记忆管理** | `memory_read`/ `memory_write` | 读写记忆 | 安全 ～ 低风险 |
| **技能管理** | `skill_create`/ `skill_install` | 管理 Skill | 低风险 |
| **多 Agent** | `delegate_task`/ `agent_spawn` | 子任务委托 | 中风险 |

> 完整的工具列表和每个工具的参数说明，用 `hermes tools list` 和 `hermes tools info` 查看。不需要在这篇文章里全部列出来。

几个实用原则：

-   改文件优先用 `edit_file`（差异替换），不要动不动 `write_file`（全量覆盖）
    
-   数据分析用 `execute_python`，不要用 `execute_shell`——前者更安全
    
-   信息获取用 `web_search` + `web_extract` 组合，覆盖大部分场景
    

## 6.3 安全审批：不是想干什么就干什么

每个工具按风险分为四级：

| 等级 | 典型工具 | 审批方式 |
| --- | --- | --- |
| 安全 | `read_file`、`web_search`、`list_directory` | 无需审批 |
| 低风险 | `write_file`、`edit_file`、`memory_write` | 首次审批，可记住 |
| 中风险 | `execute_python`、`delegate_task` | 每次审批 |
| 高风险 | `execute_shell`、`kill_process` | 强制审批，无法跳过 |

当 Agent 调用需要审批的工具时，你会看到确认提示，选择 Y（允许）、N（拒绝）或 A（始终允许，高风险不可选）。

如果默认策略不合适，可以在 `～/.hermes/config.yaml` 自定义：

```
tools:
  approval:
    per_tool:
      write_file:
        strategy: "remember"    # 首次确认，后续记住
      execute_shell:
        strategy: "prompt"      # 每次都确认
    command_blacklist:
      - "rm -rf /"
```

> 如果不小心点了"始终允许"，用 `hermes approval reset` 撤销所有已记住的审批决策。

## 6.4 跟做：验证你的工具箱

在 Hermes 里跑一次快速验证，确认工具系统正常工作。

### Step 1：查看工具列表

```
帮我列出当前所有可用的工具，按类别分组。
```

检查返回结果是否和你的预期一致。

### Step 2：测试安全工具

```
帮我读取 ～/hermes-lab/AGENTS.md 的内容。
```

`read_file` 是安全工具，不需要审批，应该直接返回内容。

### Step 3：测试需要审批的工具

```
帮我在 ～/hermes-lab 下创建 test.md，内容写"工具测试成功"。
```

`write_file` 是低风险工具，你会看到审批提示。选 Y 允许，确认文件创建成功。

### Step 4：关掉不需要的高风险工具

```
hermes tools disable execute_shell
hermes tools disable kill_process
```

> **一句话：工具系统的核心不是"全开"，而是"按需开放、分级审批"。先用安全工具跑通，再逐步放开写入和外部操作。**

---

# 七、MCP：连接外部工具，但要控制边界

上一章讲了 Hermes 内置的 47 个工具。但 Agent 不可能把所有能力都内置——你需要连 GitHub、读 Notion、查数据库、发 Slack 消息。

过去每接一个外部服务，都要写一套专门的对接代码。**MCP（Model Context Protocol）** 就是来解决这个问题的——Anthropic 提出的开放协议，给 Agent 和外部工具之间建立统一的通信标准。

一句话：**MCP 让 Agent 接入外部服务，像插 USB-C 一样简单。**

| 概念 | 类比 | 说明 |
| --- | --- | --- |
| MCP Client | 笔记本电脑 | Agent 内负责连接外部工具的组件 |
| MCP Server | USB 外设 | 提供具体功能的服务端（如 GitHub MCP Server） |
| Tool | 外设功能 | Server 暴露给 Agent 可调用的功能（如"创建 Issue"） |
| allowed\_tools | USB 口限流器 | 控制哪些功能可以给 Agent 用 |

## 7.1 常用命令

```
# 查看当前已配置的 MCP Server
hermes mcp list

# 测试某个 MCP Server 是否能正常连接
hermes mcp test github

# 查看某个 Server 暴露了哪些工具
hermes mcp configure github

# 重载 MCP 配置（修改 config.yaml 后不用重启）
/reload-mcp
```

添加 MCP Server 推荐直接编辑 `～/.hermes/config.yaml`，比 `hermes mcp add` 命令更直观可靠。

## 7.2 实战：接入 GitHub

GitHub 是 MCP 生态中最成熟的 Server。我们从它开始跟做。

### Step 1：获取 GitHub Token

1.  登录 GitHub → 右上角头像 → Settings
    
2.  左侧菜单拉到底 → Developer settings → Personal access tokens
    
3.  选 "Tokens (classic)" 或 "Fine-grained tokens"
    
4.  Generate new token，权限勾选：
    

```
✅ repo          —— 访问仓库（如需私有仓库）
✅ read:org      —— 读取组织信息
✅ read:user     —— 读取用户信息
```

5.  复制 Token（页面关了就看不到了），保存到环境变量：
    

```
# 写入 ～/.zshrc 或 ～/.bashrc
export GITHUB_TOKEN="ghp_xxxxxxxxxxxxxxxxxxxx"
```

> 用 Fine-grained tokens 可以限定只访问特定仓库。Token 每 90 天轮换一次。不要把 Token 写进 config.yaml 明文。

### Step 2：写入配置

编辑 `～/.hermes/config.yaml`：

```
mcp_servers:
  github:
    command:"npx"
    args:["-y","@modelcontextprotocol/server-github"]
    env:
      GITHUB_PERSONAL_ACCESS_TOKEN:"${GITHUB_TOKEN}"
    allowed_tools:
      -"search_repositories"
      -"get_file_contents"
      -"search_issues"
      -"create_issue"
      -"search_code"
```

配置项说明：

| 配置项 | 说明 |
| --- | --- |
| `command` | 要执行的命令，一般用 `npx` |
| `args` | `-y`确认安装 + MCP Server 包名 |
| `env` | 环境变量，放 Token，用 `${变量名}` 引用 |
| `allowed_tools` | 工具白名单，不写 = 允许所有（不推荐） |

### Step 3：验证并使用

```
hermes mcp test github
```

返回工具列表说明连接成功。然后在 Hermes 里直接用自然语言：

```
你: 搜索 GitHub 上 Hermes 项目里关于 memory 的实现

Hermes: 找到 12 个相关文件，其中最核心的是：
  1. src/memory/short_term.py - 短期记忆实现
  2. src/memory/long_term.py - 长期记忆实现
```

> 如果 Agent 报"没有权限"，检查 `allowed_tools` 里有没有这个工具，以及 GitHub Token 有没有对应权限。

## 7.3 工具过滤：allowed\_tools

MCP Server 可能暴露几十个工具，`allowed_tools` 让你只开需要的那几扇门。

**白名单模式（推荐）**——只列出允许的工具，其余屏蔽：

```
allowed_tools:
  - "search_repositories"
  - "get_file_contents"
```

**不写或留空 = 允许所有**（不推荐，Server 可能暴露删除仓库等高危工具）。

**读写分离**——同一个服务配两个实例，只读版日常用，读写版按需开启：

```
mcp_servers:
  github-read:
    command:"npx"
    args:["-y","@modelcontextprotocol/server-github"]
    env:
      GITHUB_PERSONAL_ACCESS_TOKEN:"${GITHUB_TOKEN}"
    allowed_tools:
      -"search_repositories"
      -"get_file_contents"
      -"search_code"

github-write:
    command:"npx"
    args:["-y","@modelcontextprotocol/server-github"]
    env:
      GITHUB_PERSONAL_ACCESS_TOKEN:"${GITHUB_TOKEN}"
    allowed_tools:
      -"create_issue"
      -"create_pull_request"
    enabled:false    # 默认禁用，需要时手动开启
```

## 7.4 安装前的六个问题

不要看到一个 MCP Server 就装。装之前过一遍：

| 问题 | 判断 |
| --- | --- |
| 它是否真的补足内置工具做不到的事？ | 是才装 |
| 它是否需要高权限（删除、发送、支付）？ | 高权限谨慎，用 allowed\_tools 收窄 |
| 它是否会暴露隐私数据？ | 会就先不上，或用只读副本 |
| 它是否能限定目录或只读模式？ | 优先选择有限定的 |
| 它是否会显著增加 Token 开销？ | 会就用 allowed\_tools 过滤 |
| 它是否只在某个场景需要？ | 只装到对应 Profile（第十三章） |

## 7.5 生态速查

MCP 社区已有 6000+ 个 Server，安装格式统一：

```
command: "npx"
args: ["-y", "@scope/server-name"]
```

高频 Server：

| 分类 | Server | 用途 |
| --- | --- | --- |
| 代码协作 | server-github | 搜仓库、看 PR、管 Issue |
| 文件系统 | server-filesystem | 访问指定目录（零配置） |
| 数据库 | server-postgres | 查数据库 |
| 知识管理 | server-notion | 读写 Notion |
| 浏览器 | server-puppeteer | 网页自动化和截图 |

> **安全提醒**：MCP Server 运行在本地，可以访问你的文件系统、环境变量和网络。建议只用官方（`@modelcontextprotocol`）或社区广泛验证的 Server。

> **一句话：MCP 的关键不是多，而是窄。只暴露当前任务真正需要的工具。先接一个跑通，再逐步加。**

---

# 八、Gateway：把 Hermes 接到飞书里

前面几章我们主要在终端里使用 Hermes。但日常使用时，你不一定总想打开终端。更自然的方式是：把 Hermes 接到飞书里，像和一个普通同事聊天一样使用它。

本章只做一件事：**让读者把 Hermes 接入飞书，并知道日常怎么用。**

---

## 8.1 Gateway 是什么？

Gateway 可以理解成 Hermes 的“消息入口”。

你在飞书里发消息，Gateway 负责把消息转给 Hermes；Hermes 生成回复后，Gateway 再把回复发回飞书。

简单说：

```
你在飞书发消息
      ↓
飞书 Bot 收到消息
      ↓
Hermes Gateway 转发给 Hermes
      ↓
Hermes 思考、调用工具、生成回复
      ↓
Gateway 把回复发回飞书
```

所以 Gateway 的价值不是“炫技”，而是让 Hermes 变成一个随时可用的助手：

-   手机上可以随时问它问题；
    
-   可以把临时想法发给它整理；
    
-   可以把日常报告、定时任务结果推送到飞书；
    
-   可以在群里 @ 它，让它帮忙总结、分析、拆解任务。
    

---

## 8.2 先确认 Hermes 可用

配置飞书之前，先确认你本地 Hermes 已经能正常运行。

```
hermes --version
```

再启动一次普通对话：

```
hermes
```

如果 Hermes 本身还不能正常对话，先不要配置飞书。因为飞书只是入口，真正回答问题的还是 Hermes 本体。

---

## 8.3 推荐方式：用 `hermes gateway setup` 接入飞书

现在最推荐的方式是直接使用官方的交互式配置向导：

```
hermes gateway setup
```

进入向导后，选择：

```
Feishu / Lark
```

如果你的版本支持扫码创建，终端会出现二维码。用飞书手机 App 扫码后，Hermes 会自动帮你创建 Bot 应用并保存凭证。

这是最适合普通读者的方式，因为不需要手动去飞书开放平台配置一堆权限。

整体流程可以理解为：

```
运行 hermes gateway setup
        ↓
选择 Feishu / Lark
        ↓
用飞书扫码
        ↓
Hermes 自动创建应用并保存配置
        ↓
启动 Gateway
        ↓
在飞书里给 Bot 发消息测试
```

---

## 8.4 连接模式建议：优先选 WebSocket

配置过程中，如果让你选择连接模式，建议优先选择：

```
WebSocket
```

原因很简单：**WebSocket 不需要公网服务器，也不需要 ngrok。**

这对大多数个人用户最友好。你只要在自己的电脑上运行 Hermes，Hermes 会主动连接飞书服务端，然后等待消息。

可以在 `～/.hermes/.env` 里看到或手动设置：

```
FEISHU_CONNECTION_MODE=websocket
```

一般读者不需要理解太多网络细节，只要记住：

| 模式 | 是否推荐 | 适合谁 |
| --- | --- | --- |
| WebSocket | 推荐 | 本地电脑、个人用户、刚开始配置 |
| Webhook | 不优先推荐 | 已经有公网服务器、熟悉回调地址配置的人 |

---

## 8.5 如果扫码不可用：手动配置飞书应用

如果你的 Hermes 版本不支持扫码创建，或者扫码失败，可以走手动配置。

### 第一步：创建飞书自建应用

打开飞书开放平台：

```
https://open.feishu.cn/
```

进入开发者后台，创建一个企业自建应用。

可以这样填写：

```
应用名称：Hermes 助手
应用描述：用于在飞书中使用 Hermes Agent
```

创建完成后，进入应用详情页，找到：

```
凭证与基础信息
```

复制：

```
App ID
App Secret
```

### 第二步：开启机器人能力

在应用后台找到“添加应用能力”，开启：

```
机器人
```

否则你在飞书里找不到这个 Bot，也无法和它聊天。

### 第三步：写入 Hermes 环境变量

打开或创建文件：

```
nano ～/.hermes/.env
```

写入：

```
FEISHU_APP_ID=cli_xxxxxxxxxxxxxxxx
FEISHU_APP_SECRET=xxxxxxxxxxxxxxxxxxxxxxxx
FEISHU_DOMAIN=feishu
FEISHU_CONNECTION_MODE=websocket
```

说明：

| 配置项 | 含义 |
| --- | --- |
| `FEISHU_APP_ID` | 飞书应用的 App ID |
| `FEISHU_APP_SECRET` | 飞书应用的 App Secret |
| `FEISHU_DOMAIN` | 国内飞书写 `feishu`，海外 Lark 写 `lark` |
| `FEISHU_CONNECTION_MODE` | 建议写 `websocket` |

保存后，再运行：

```
hermes gateway setup
```

选择 Feishu / Lark，并按提示完成配置。

---

## 8.6 启动 Gateway

配置完成后，可以先用前台方式启动，方便看报错：

```
hermes gateway
```

有些版本也支持：

```
hermes gateway run
```

如果启动正常，你就可以打开飞书，找到刚刚创建的 Bot，发一条消息：

```
你好，帮我用一句话介绍你自己
```

如果 Bot 能回复，说明飞书接入成功。

---

## 8.7 后台运行 Gateway

前台运行适合调试，但关闭终端后 Gateway 就停了。

如果你希望 Gateway 常驻，可以先安装为系统服务：

```
hermes gateway install
```

然后启动服务：

```
hermes gateway start
```

查看状态：

```
hermes gateway status
```

停止服务：

```
hermes gateway stop
```

重启服务：

```
hermes gateway restart
```

常用命令可以记成下面这张表：

| 目的 | 命令 |
| --- | --- |
| 配置飞书等消息平台 | `hermes gateway setup` |
| 前台运行，方便调试 | `hermes gateway`或 `hermes gateway run` |
| 安装为后台服务 | `hermes gateway install` |
| 启动后台服务 | `hermes gateway start` |
| 查看状态 | `hermes gateway status` |
| 停止服务 | `hermes gateway stop` |
| 重启服务 | `hermes gateway restart` |

---

## 8.8 飞书里怎么用 Hermes？

配置完成后，你就可以像和同事聊天一样使用 Hermes。

### 1\. 私聊 Bot

适合个人使用。

你可以直接发：

```
帮我总结一下今天要做的三件事
```

或者：

```
把下面这段想法整理成公众号文章大纲：
……
```

私聊里，Hermes 通常会对每条消息直接回复。

### 2\. 群里 @ Bot

适合团队协作。

比如在群里发：

```
@Hermes 助手 帮我总结一下上面这段讨论，提炼成待办事项
```

群聊里建议用 @ 的方式唤起 Bot，避免它对所有消息都响应。

### 3\. 使用常用指令

在飞书聊天里，你也可以使用一些 Hermes 指令。

常用的有：

| 指令 | 用途 |
| --- | --- |
| `/new`或 `/reset` | 开启一段新对话 |
| `/status` | 查看当前会话状态 |
| `/model` | 查看或切换当前模型 |
| `/retry` | 让 Hermes 重新回答上一条 |
| `/undo` | 撤销上一轮对话 |
| `/sethome` | 把当前飞书聊天设为默认推送位置 |
| `/help` | 查看可用指令 |

日常最常用的是：

```
/reset
```

当你觉得 Hermes 上下文混乱、答非所问时，可以先重置会话。

---

## 8.9 最实用的 5 个日常场景

### 场景一：随手记录想法

你在路上突然想到一个选题，可以直接发给飞书 Bot：

```
记录一个想法：个人 Agent 的关键不是工作流，而是 Memory、Skill、Eval 三件事。
```

然后让它继续整理：

```
把刚才这个想法整理成 3 个公众号标题
```

### 场景二：文章大纲助手

```
我想写一篇关于 Hermes 飞书接入的文章，请帮我整理一个适合新手的教程大纲。
```

### 场景三：会议纪要整理

把会议记录粘给它：

```
帮我把下面的会议记录整理成：背景、结论、待办、风险。
……
```

### 场景四：日报 / 周报草稿

```
根据下面这些工作内容，帮我整理成一份周报：
……
```

### 场景五：定时推送结果

后面如果配置了 Cron，你可以让 Hermes 把每日总结、AI 新闻、工具雷达等内容推送到飞书。

这时可以先在目标飞书聊天里发送：

```
/sethome
```

这样 Hermes 就知道以后把默认通知推送到哪里。

---

## 8.10 安全设置：至少配置允许用户

飞书接入成功后，不要忽略安全问题。

因为一旦 Bot 能访问你的 Hermes，就意味着别人可能通过 Bot 间接使用你的本地 Agent。

建议至少设置允许用户列表。

在飞书里可以先问：

```
/whoami
```

查看自己的身份信息。

然后在 `～/.hermes/.env` 里配置允许访问的用户：

```
FEISHU_ALLOWED_USERS=ou_xxxxxxxxxxxxxxxx
```

多个用户用英文逗号分隔：

```
FEISHU_ALLOWED_USERS=ou_xxx,ou_yyy
```

如果你不确定自己的 ID 是哪个，可以先看 Hermes Gateway 的启动日志，或者在飞书里使用 `/whoami` 查看。

最低限度要记住：

| 安全建议 | 原因 |
| --- | --- |
| 不要把 App Secret 发给别人 | 泄露后别人可能冒充你的应用 |
| 配置 `FEISHU_ALLOWED_USERS` | 避免陌生人使用你的 Bot |
| 群聊里尽量 @ Bot 使用 | 避免 Bot 被无关消息频繁触发 |
| 高危操作要人工确认 | 不要让消息入口直接执行危险命令 |

---

## 8.12 本章小结

这一章你只需要完成三个动作：

```
hermes gateway setup
```

选择：

```
Feishu / Lark
```

然后启动：

```
hermes gateway
```

能在飞书里收到 Hermes 的回复，就说明配置成功。

对个人用户来说，Gateway 最重要的价值不是复杂部署，而是让 Hermes 进入你的日常消息入口。先把飞书私聊跑通，再慢慢尝试群聊、定时推送和更复杂的自动化。

---

# 九、Cron：让 Hermes 从被动工具变成主动系统

前面八章，Hermes 都是在你提问后才工作——你发一条消息，它响应一次。这是"被动模式"。

**Cron 让 Hermes 变成"主动模式"：在固定时间自动执行任务，不需要你盯着。**

适合 Cron 的场景：

| 频率 | 示例任务 |
| --- | --- |
| 每天 | 搜集 AI 新闻生成日报、检查 GitHub 关注项目有无 release |
| 每周 | 检查 Skill / MCP 是否需要清理、生成本周知识卡片汇总 |
| 每月 | 整理 Memory 是否有过期信息、测评报告汇总 |
| 按需 | 监控磁盘空间、检测某个 API 是否正常 |

## 9.1 常用命令

```
# 创建定时任务
hermes cron create "0 9 * * *""搜索今日 AI 新闻并生成摘要报告"

# 查看所有定时任务
hermes cron list

# 手动触发一次（不等定时）
hermes cron run 

# 暂停 / 恢复
hermes cron pause 
hermes cron resume 

# 编辑已有任务
hermes cron edit 

# 删除任务
hermes cron remove 

# 查看调度器状态
hermes cron status
```

什么时候用什么命令：

-   想定时执行某个任务 → `hermes cron create`
    
-   忘了自己建了哪些任务 → `hermes cron list`
    
-   任务到期了但想提前跑一次 → `hermes cron run`
    
-   某个任务暂时不需要但不想删 → `hermes cron pause`
    
-   想改某个任务的执行时间或内容 → `hermes cron edit`
    

## 9.2 Cron 表达式速查

`hermes cron create` 支持两种格式：Cron 表达式和自然语言。

### Cron 表达式

格式：`分 时 日 月 周`

| 表达式 | 含义 |
| --- | --- |
| `0 9 * * *` | 每天 9:00 |
| `0 9 * * 1` | 每周一 9:00 |
| `0 9 1 * *` | 每月 1 日 9:00 |
| `*/15 * * * *` | 每 15 分钟 |
| `0 9-17 * * 1-5` | 工作日 9:00-17:00 每小时 |
| `30 8 * * 1-5` | 工作日 8:30 |

### 自然语言

| 写法 | 含义 |
| --- | --- |
| `every 2 hours` | 每 2 小时 |
| `every day 9am` | 每天 9 点 |
| `30m` | 每 30 分钟 |

> **Cron 表达式更精确，自然语言更直观。** 复杂的定时用 Cron 表达式，简单的可以用自然语言。

## 9.3 create 命令详解

`hermes cron create` 是最核心的命令，参数比想象中丰富：

```
hermes cron create  [prompt] [选项]
```

| 参数 | 说明 |
| --- | --- |
| `schedule` | Cron 表达式或自然语言（必填） |
| `prompt` | 任务指令，告诉 Agent 要做什么 |
| `--name` | 给任务起个名字，方便管理 |
| `--deliver` | 结果推送到哪里：`origin`（当前平台）/ `telegram` / `discord` / `lark` 等 |
| `--skill` | 绑定一个 Skill，任务执行时自动加载 |
| `--script` | 绑定一个脚本，脚本输出作为 Agent 的输入 |
| `--no-agent` | 不经过 LLM，直接执行脚本并推送输出 |
| `--workdir` | 指定工作目录（会加载该目录的 AGENTS.md） |
| `--profile` | 指定用哪个 Profile 执行 |
| `--repeat` | 重复次数（比如只跑 3 次就自动删除） |

这节先掌握最基本的用法，高级参数后面实战中用到再讲。

## 9.4 实战一：每天早上推送 AI 新闻日报

这是 Cron 最经典的使用场景——每天 9 点，Hermes 自动搜集 AI 新闻，生成一份摘要报告，推送到飞书。

### Step 1：确认前提条件

-   飞书 Gateway 已配置并运行（第八章）
    
-   Hermes 的搜索工具可用
    

### Step 2：创建定时任务

```
hermes cron create "0 9 * * *" \
  "搜索最近 24 小时内最重要的 AI 新闻。只读取最多 5 个高质量来源，优先选择官方博客、arXiv、OpenAI、Anthropic、Google DeepMind、Meta AI、Hugging Face、The Decoder、TechCrunch、The Verge。不要打开明显需要登录、反爬、广告墙或加载很慢的网站；如果某个网页超过 20 秒无法读取，跳过它，不要重试。每条新闻输出：标题、来源、发布时间、3 句话摘要、对 AI 产品/算法/行业的影响。最后给出今日最值得关注的 3 个趋势。总字数控制在 1200 中文字以内。请额外将完整报告保存为 Markdown 文件到 ～/hermes-lab/reports/ 目录，文件名格式为 ai-daily-report-YYYY-MM-DD.md。最终回复只输出报告正文。" \
  --name "ai-daily-report" \
  --deliver "lark" \
  --workdir "～/hermes-lab/reports"
```

参数解读：

-   `"0 9 * * *"` → 每天 9:00 触发
    
-   `--name "ai-daily-report"` → 任务名，方便识别
    
-   `--deliver "lark"` → 结果推送到飞书
    
-   `--workdir "～/hermes-lab/reports"` → 结果推送到本地的目录下
    
-   后面的长文本 → 告诉 Agent 具体要做什么
    

### Step 3：验证任务已创建

```
hermes cron list
```

输出类似：

```
ID    Name               Schedule    Status    Next Run
────────────────────────────────────────────────────────
c001  ai-daily-report    0 9 * * *   active    2026-05-25 09:00
```

### Step 4：手动跑一次测试

不等明天 9 点，现在就跑一次看效果：

```
hermes cron run c001
```

Hermes 会立刻执行这个任务：搜索 AI 新闻 → 生成报告 → 保存到文件 → 推送到飞书。

检查报告是否生成(可以在命令中`--workdir ～/hermes-lab/reports/`指定存储目录)：

```
ls ～/hermes-lab/reports/
```

检查飞书是否收到推送。如果都正常，说明配置成功。

### Step 5：调整（可选）

如果 9 点太早，改成 10 点：

```
hermes cron edit c001 --schedule "0 10 * * *"
```

如果周末不想收报告，改成只在工作日：

```
hermes cron edit c001 --schedule "0 9 * * 1-5"
```

## 9.5 实战二：每周检查 MCP / Skill 是否需要清理

随着使用时间增长，你可能会装很多 MCP Server 和 Skill，有些已经不用了。定期清理是好习惯。

```
hermes cron create "0 10 * * 1" \
  "检查我的 MCP Server 列表和 Skill 列表，分析以下内容：
  1. 哪些 MCP Server 过去 30 天没有被调用过
  2. 哪些 Skill 已经和当前工作流不相关
  3. 哪些 MCP Server 的 allowed_tools 设置过于宽松
  给出清理建议，但不要自动删除任何东西。" \
  --name "weekly-cleanup-review" \
  --deliver "lark" \
  --workdir "～/hermes-lab/reports"
```

-   `"0 10 * * 1"` → 每周一 10:00
    
-   `--deliver "lark"` → 推到飞书，周一上班就能看到
    

> **注意命令里说了"不要自动删除任何东西"。** Cron 任务应该只做"发现和建议"，不做"擅自改变系统"。这是下面 9.6 节要讲的原则。

## 9.6 自动执行 vs 人工确认

Cron 上线后，Hermes 可以在你不在的时候自动做事。但不是所有事情都应该自动执行。

**可以自动执行：**

-   搜集公开信息
    
-   生成报告并保存到本地
    
-   发送通知到自己的飞书 / Telegram
    
-   做只读检查（磁盘空间、GitHub release、API 状态）
    

**必须人工确认：**

-   安装 / 卸载工具
    
-   修改 Memory 或配置文件
    
-   提交代码或部署服务
    
-   发送对外消息（给客户、给群组）
    
-   删除任何文件
    

一条原则：

> **Cron 只负责"发现和建议"，不负责"擅自改变系统"。** 如果某个 Cron 任务需要执行写操作，在 prompt 里明确加上"列出建议，等我确认后再执行"。

## 9.7 注意事项

| 问题 | 说明 |
| --- | --- |
| 系统睡眠 | 如果 Mac 在任务触发时处于睡眠状态，唤醒后任务会补执行 |
| 任务重叠 | 如果一个任务执行时间超过间隔，下一次会等当前执行完再开始 |
| 任务持久化 | 任务保存在 `～/.hermes/scheduled_tasks/`，Hermes 重启后不会丢失 |
| 时区 | 默认用系统时区。如果你的 Mac 时区不对，Cron 执行时间也会偏 |
| 推送失败 | 如果 Gateway 没启动，`--deliver` 会静默失败。报告仍然保存在本地 |

> **建议第一次设置 Cron 时，先用 `hermes cron run` 手动跑一次，确认效果没问题再让它自动跑。**

## 9.8 Cron + Gateway + Skill 的组合

Cron 真正强大的地方在于和其他模块组合：

```
Cron（定时触发）
  → Skill（标准化的任务流程）
    → Tool / MCP（执行具体操作）
      → Gateway（推送到飞书/Telegram）
```

举个具体例子：

```
# 用第五章的 ai-tool-update-advisor Skill 做每周工具雷达
hermes cron create "0 9 * * 1" \
  "执行 AI 工具雷达扫描，生成本周值得关注的新工具报告。 " \
  --name "tool-radar" \
  --skill "ai-tool-update-advisor" \
  --deliver "lark" \
  --workdir "～/hermes-lab/reports"
```

-   Cron 负责每周一 9 点触发
    
-   Skill 定义了"工具雷达"的标准流程（搜什么、怎么分析、输出格式）
    
-   MCP 的搜索工具负责实际搜索
    
-   Gateway 把结果推到飞书
    
-   workdir 把结果推到本地目录
    

你周一上班打开飞书，就有一份工具雷达等着你。

> **一句话：Cron 解决的是"什么时候做"，Skill 解决的是"怎么做"，MCP 解决的是"用什么做"，Gateway 解决的是"结果送到哪"。四个模块组合起来，就是一个自动运转的 Agent 系统。**

---

# 十、上下文引擎与 Token 精简：用得更久、更省、更稳

满配之后，Hermes 会接触到越来越多的信息：Memory、Skills、项目规则、MCP 工具、会话历史、工具输出……但上下文窗口不是无限的，Token 也不是免费的。

**这一章解决一个问题：Hermes 满配之后，怎么避免越来越慢、越来越贵、越来越容易"撑爆上下文"。**

需要先澄清一点：Hermes 会保存 session 历史，方便后续恢复和跨会话搜索；但这不等于每一轮都会把过去处理过的所有内容原封不动塞回模型。真正容易撑爆上下文的，通常是长对话、超长日志、大文件内容、冗长工具输出、反复粘贴的材料，以及过多常驻工具/schema。

## 10.1 上下文为什么会爆

一个满配 session 中，可能进入模型上下文的内容大致包括：

| 组成部分 | 说明 | Token 风险 |
| --- | --- | --- |
| 系统提示 + SOUL.md | Agent 身份和行为准则 | 固定，通常不大 |
| MEMORY.md + USER.md | 长期记忆与用户信息 | 中等，随长期积累变大 |
| 项目规则文件 | 如 `.hermes.md`、`HERMES.md`、`AGENTS.md`、`CLAUDE.md` 等 | 中等，取决于项目规则长度 |
| 已启用 Skills | Skill 的说明、流程和约束 | 看数量和单个 Skill 长度 |
| MCP / Tool schema | 工具名称、用途、参数定义 | 工具多时会变成大头 |
| 当前会话历史 | 你和 Agent 的对话记录 | 随时间增长 |
| 工具输出 | 搜索结果、文件内容、日志、diff、终端输出等 | 最不可控，最容易突然暴涨 |

越满配，上下文越容易变大。而且上下文还有一个反直觉的特性：

> **越大不等于越好。** 不是所有信息都应该常驻上下文。真正重要的是让 Agent 在正确时间看到正确的信息。

长上下文会带来几个问题：

1.  **更慢**：每轮都要处理更多输入。
    
2.  **更贵**：Input tokens 增加，成本自然增加。
    
3.  **更不稳**：无关旧信息越多，越容易干扰当前任务。
    
4.  **更容易丢重点**：上下文太长时，模型可能对中间信息注意力下降，也就是常说的 Lost in the Middle。
    

所以，Hermes 的上下文管理不是"无限记住所有内容"，而是尽量把原始流水账变成可继续工作的任务状态。

## 10.2 先诊断：看你的 Token 花在哪

优化之前先诊断。Hermes 提供了几个命令帮你了解上下文、Token 和历史会话情况。

### hermes insights：全局消耗概览

```
# 查看最近 7 天的 Token / 成本 / 工具调用情况
hermes insights --days 7

# 只看某个平台来源，比如 CLI / Telegram / Discord
hermes insights --days 7 --source cli
```

输出大致会包含：

```
📊 Hermes Insights — Last 7 days

📋 Overview
  Sessions:          19
  Messages:          695
  Input tokens:      705,861
  Output tokens:     80,775

🤖 Models Used
  Model                          Sessions       Tokens
  deepseek-chat                         9       747,221
  glm-5.1                               4       320,114

🔧 Top Tools
  Tool                            Calls        %
  terminal                          100     31.2%
  web_search                         35     10.9%
  read_file                          29      9.1%
```

重点看三件事：

-   **Input tokens vs Output tokens**：Input 大，通常说明上下文、工具输出、历史内容或常驻材料偏重；Output 大，说明 Agent 输出太啰嗦。
    
-   **Models Used**：看不同模型消耗是否异常，避免日常小任务长期跑在昂贵模型上。
    
-   **Top Tools**：这里反映的是工具调用频率，不等于工具 schema 大小；但它能帮助你判断哪些工具在任务中最活跃，是否需要限制读取范围或拆分任务。
    

### hermes sessions：会话管理

```
# 查看最近会话
hermes sessions list

# 查看 session 存储统计
hermes sessions stats

# 清理 30 天以前的已结束会话
hermes sessions prune --older-than 30

# 跳过确认
hermes sessions prune --older-than 30 --yes

# 清理前先导出备份
hermes sessions export backup.jsonl
```

注意：

-   `hermes sessions prune --before 30d` 是错误写法。
    
-   正确写法是 `hermes sessions prune --older-than 30`，这里的 `30` 是天数，不需要写 `d`。
    
-   `prune` 清理的是**已结束的旧 session 存储**，不是压缩当前活跃会话的上下文。
    
-   活跃 session 不会被 prune。当前会话太长时，应该用 `/compress`、`/new`，或者等待自动 compression 触发。
    

可以把它们理解成两类命令：

```
# 管当前上下文：让当前对话变短
/compress
/new

# 管历史存储：删除旧的 ended sessions
hermes sessions prune --older-than 30
```

## 10.3 内置压缩：自动精简上下文

Hermes 内置了上下文压缩机制。它不是简单截断聊天记录，而是在上下文接近阈值时，把较旧的中间历史整理成摘要，同时保留最近对话，尽量保证当前任务不断线。

Hermes 当前有两层压缩/保护机制：

| 层级 | 触发位置 | 作用 |
| --- | --- | --- |
| Gateway Session Hygiene | Agent 处理消息前 | 安全网，防止长时间网关会话在进入 Agent 前就爆上下文 |
| Agent ContextCompressor | Agent 内部循环中 | 主要压缩机制，基于更准确的 token 使用情况进行摘要压缩 |

### 推荐配置

```
# ～/.hermes/config.yaml
compression:
enabled:true              # 开启上下文压缩，默认 true
threshold:0.50            # 达到上下文窗口 50% 时触发压缩
target_ratio:0.20         # 压缩后保留 threshold 的 20% 作为近期尾部上下文
protect_last_n:20         # 至少保护最近 20 条消息不被压缩
hygiene_hard_message_limit:400# 网关安全阈值：消息数过多时强制压缩
```

查看当前配置：

```
hermes config show
```

也可以直接打开配置文件修改：

```
hermes config edit
```

### 关键参数解读

| 参数 | 默认值 | 说明 |
| --- | --- | --- |
| `compression.enabled` | `true` | 是否启用上下文压缩 |
| `compression.threshold` | `0.50` | 达到模型上下文窗口多少比例时触发压缩 |
| `compression.target_ratio` | `0.20` | 保留多少近期尾部内容；注意它是相对 threshold 的比例，不是"保留原文 20%" |
| `compression.protect_last_n` | `20` | 至少保留最近 N 条消息不压缩，保证当前对话连贯 |
| `compression.hygiene_hard_message_limit` | `400` | Gateway 层的硬保护，防止消息数过多时还没按 token 阈值触发 |

> **建议新手先保持默认值。** 如果你感觉长任务后半段经常"忘当前进度"，可以适当调高 `protect_last_n`；如果你的小上下文模型经常快爆，可以适当降低 `threshold`，让压缩更早触发。

### 压缩模型也要注意

默认情况下，Hermes 可以使用主模型做压缩摘要。你也可以给压缩单独指定更便宜、更快的辅助模型：

```
auxiliary:
  compression:
    provider: openrouter
    model: google/gemini-2.5-flash
```

但这里有一个坑：**压缩模型的上下文窗口最好不要小于主模型。** 因为压缩时要把中间历史交给摘要模型，如果摘要模型上下文太小，可能压缩失败，甚至导致中间内容没有被有效摘要，从而出现"突然失忆"的感觉。

## 10.4 预算机制：主要先管 max\_turns

当前更稳妥的写法是只讲官方明确的 **iteration budget**：

```
# ～/.hermes/config.yaml
agent:
  max_turns: 90      # 单次任务中 Agent 最多迭代多少轮，默认 90
```

它解决的是：Agent 做复杂任务时，不会无限循环。Hermes 会在接近迭代上限时提醒模型收束；达到上限后，会停止并尽量总结已经完成的工作。

什么时候调它：

-   **Agent 经常跑飞、不停调用工具** → 适当调低 `agent.max_turns`
    
-   **代码任务/长调研经常没做完就停** → 适当调高 `agent.max_turns`
    
-   **只是普通问答、写文章、轻量任务** → 默认值通常够用
    

如果你使用 `/goal` 这种持续目标能力，它还有自己的预算：

```
goals:
  max_turns: 20      # /goal 自动续跑的最大轮数，默认 20
```

至于"工具调用总次数"和"单次成本上限"，建议这一章先不要写成固定配置项。更稳妥的表达是：通过 `hermes insights` 观察成本，通过工具精简、Profile 隔离、限制文件读取范围、选择便宜模型来控制成本。

## 10.5 日常精简技巧

除了自动 compression，日常使用中更重要的是减少不必要的信息进入上下文。

### 1\. Profile 隔离：不要把所有工具都塞进 default

Hermes 的 Profile 更像多个隔离的 Hermes 实例。不同 Profile 可以有自己的 config、sessions、skills、memory、gateway setup。

建议按场景拆：

```
# 创建日常轻量 Profile
hermes profile create daily --clone

# 创建开发 Profile
hermes profile create dev --clone

# 切换默认 Profile
hermes profile use daily

# 或者单次指定 Profile 运行
hermes -p dev chat
```

然后分别进入对应 Profile 配工具：

```
# 配置当前 Profile 的工具
hermes tools

# 查看当前启用工具概览
hermes tools --summary

# 给 dev Profile 配工具
hermes -p dev tools
```

### 2\. MCP / Tool 按需启用

MCP 工具和内置 toolset 都可能带来 schema 成本。工具越多，模型每轮看到的工具说明越多，选择工具时也更容易分心。

建议：

-   日常聊天 Profile：只保留最常用工具。
    
-   开发 Profile：启用 filesystem、terminal、git、github 等开发相关工具。
    
-   自动化/Cron Profile：只启用任务真正需要的工具。
    
-   不要为了"满配感"把所有 MCP Server 都常驻。
    

全局禁用某些 toolset，可以考虑：

```
# ～/.hermes/config.yaml
agent:
  disabled_toolsets:
    - web
    - memory
```

这个适合你明确知道某个 Profile 不需要某类工具时使用。普通用户优先用 `hermes tools` 交互式配置即可。

### 3\. 控制文件读取范围

不要让 Agent 一次读完整日志或大文件：

```
❌ 帮我看看日志里有什么错误
   → Agent 可能读取大量日志，上下文暴涨

✅ 帮我查看 app.log 最后 200 行，找出所有 ERROR，并按错误类型归类
   → 范围明确，Token 可控
```

Hermes 也有文件读取安全限制：

```
# ～/.hermes/config.yaml
file_read_max_chars: 100000  # 默认值，限制单次 read_file 返回的最大字符数
```

小上下文模型可以调低，例如：

```
file_read_max_chars: 30000
```

大上下文模型可以适当调高，例如：

```
file_read_max_chars: 200000
```

### 4\. Prompt 控制输出长度

在 Cron、Skill、长任务 prompt 里，明确控制输出长度：

```
总字数控制在 1200 中文字以内。
只输出报告正文，不要重复我的指令。
每条新闻最多 3 句话摘要。
如果信息不足，先列出缺口，不要展开猜测。
```

这能减少 Output token，也能减少下一轮继续对话时的历史负担。

### 5\. 定期清理旧会话，但别指望它解决当前上下文

```
# 清理 30 天以前的已结束会话
hermes sessions prune --older-than 30

# 清理前备份
hermes sessions export backup.jsonl
```

`prune` 主要解决的是历史 session 存储膨胀，不是当前对话变长。当前对话太长时，用：

```
/compress
/new 新任务名
```

## 10.6 社区上下文工具：可以了解，但先别急

Hermes 的上下文引擎是可插拔的。默认引擎是内置的 `compressor`，也可以通过 plugin 替换成其他上下文管理方案。

```
context:
  engine: "compressor"   # 默认：内置摘要压缩

# 如果未来安装了某个上下文引擎插件，可以切换为插件名
# context:
#   engine: "lcm"
```

可以把社区上下文工具分成三类理解：

| 工具类型 | 解决问题 | 适合场景 |
| --- | --- | --- |
| 压缩器类 | 把旧上下文摘要化 | 长对话、长调研 |
| LCM / 结构化上下文管理类 | 把上下文重组成可检索结构 | 资料多、反复引用、项目周期长 |
| 可观测工具类 | 看上下文构成和压缩效果 | 调试上下文污染、分析 token 浪费 |

> **建议新手先用 Hermes 内置 compression、Profile 隔离、MCP/Tool 精简来控制上下文。** 第三方上下文引擎可以作为进阶探索，不要作为新手必装路径。

## 10.7 Token 优化总表

| 问题 | 优化方式 |
| --- | --- |
| 当前对话太长 | `/compress`、自动 compression、必要时 `/new` |
| 不知道 Token 花在哪 | `hermes insights --days 7` |
| 旧会话存储太多 | `hermes sessions prune --older-than 30` |
| 工具 schema 太多 | `hermes tools`精简工具，Profile 隔离 |
| Agent 跑飞不停 | 调低 `agent.max_turns` |
| `/goal`自动续跑太久 | 调低 `goals.max_turns` |
| 单次对话太贵 | 换便宜模型、限制工具、限制输出长度 |
| 文件太大 | 明确读取范围，必要时调低 `file_read_max_chars` |
| Skill 太多 | 常驻 Skill 保持少而精，不常用的按需调用 |
| Cron 输出太长 | 固定报告模板 + 字数限制 |
| 多 Agent 成本高 | 子 Agent 用轻量模型 + 限制任务范围 |

> **一句话：满配不是让 Hermes 永远带着所有东西跑，而是让它在正确时间看到正确东西。**

---

# 十一、可视化与可观测：看得见，才能长期用

一个长期运行的 Agent 系统，最怕变成黑箱。你需要知道：

-   今天它做了什么
    
-   哪个 Cron 成功了，哪个失败了
    
-   Gateway 是否在线
    
-   Token 花在哪里
    
-   Memory 是否该清理
    
-   哪些工具有问题
    

第十章讲的是"上下文和 Token 怎么管"，这一章讲的是"系统健不健康、出了问题怎么排查"。

## 11.1 三大诊断命令

Hermes 提供三个层级的诊断命令，从快速概览到深度排障：

| 命令 | 用途 | 什么时候用 |
| --- | --- | --- |
| `hermes status` | 快速概览：环境、API Key、Gateway、Cron 状态 | 每天看一眼，确认系统正常 |
| `hermes doctor` | 深度诊断：配置、连通性、工具可用性 | 感觉哪里不对，想全面体检 |
| `hermes logs` | 日志追踪：实时看 Agent 运行细节 | 排障时定位具体问题 |

### hermes status：快速概览

```
hermes status          # 快速概览
hermes status --all    # 显示全部详情
```

输出大致长这样：

```
⚕ Hermes Agent Status

◆ Environment
  Project:      /Users/you/hermes-agent
  Python:       3.11.15
  Model:        deepseek-v4-pro
  Provider:     DeepSeek

◆ API Keys
  OpenRouter    ✓ sk-o...171f
  DeepSeek      ✓ sk-0...4614
  Z.AI / GLM    ✓ 88cb...5wGv
  Anthropic     ✗ (not set)

◆ Messaging Platforms
  Feishu        ✓ configured
  Telegram      ✗ not configured

◆ Gateway Service
  Status:       ✓ running
  Manager:      launchd
  PID(s):       39161

◆ Scheduled Jobs
  Jobs:         2 active, 2 total

◆ Sessions
  Active:       1 session(s)
```

重点关注：

-   **API Keys**：哪些配了、哪些没配。没配的不会影响运行，但对应工具不可用
    
-   **Gateway Service**：是否在运行。如果 `✗ stopped`，飞书/Telegram 收不到消息
    
-   **Scheduled Jobs**：Cron 任务是否正常
    

### hermes doctor：深度诊断

```
hermes doctor           # 完整诊断
hermes doctor --fix     # 自动修复能修的问题
```

`hermes doctor` 会检查配置文件、API 连通性、工具可用性等，和 `status` 有重叠但更深入。重点看它独有的部分：

```
◆ Security Advisories
  ✓ No active security advisories

◆ Configuration Files
  ✓ ～/.hermes/config.yaml exists
  ⚠ Config version outdated (v22 → v23)

◆ Tool Availability
  ✓ terminal, file, web, memory, skills, vision
  ⚠ browser (system dependency not met)
  ⚠ discord (missing DISCORD_BOT_TOKEN)
```

常见 `⚠` 的处理方式：

-   `⚠ Config version outdated` → 运行 `hermes doctor --fix` 自动迁移
    
-   `⚠ browser (system dependency not met)` → 缺少依赖，按提示安装
    
-   `✗ API Key` → 需要配置对应 Key 才能使用
    

> **建议每次升级 Hermes 后都跑一次 `hermes doctor`，检查配置是否需要迁移、依赖是否完整。**

### hermes logs：日志追踪

```
hermes logs                     # 查看最近 50 行 Agent 日志
hermes logs --follow            # 实时跟踪（排障时最常用）
hermes logs errors              # 只看错误日志
hermes logs gateway             # 查看 Gateway 日志（飞书/Telegram 消息进出）

# 常用过滤
hermes logs --level WARNING     # 只看 WARNING 及以上
hermes logs --since 1h          # 最近 1 小时
hermes logs --component tools   # 只看工具相关
hermes logs --session abc123    # 按会话 ID 过滤
```

什么时候用什么：

| 场景 | 命令 |
| --- | --- |
| 飞书收不到回复 | `hermes logs gateway --follow` |
| Agent 调用工具报错 | `hermes logs --level WARNING --component tools` |
| Cron 任务执行失败 | `hermes logs --since 1h --level ERROR` |
| 想看完整的实时对话流 | `hermes logs --follow` |

## 11.2 用 Cron 自动生成系统健康报告

个人使用不需要 Dashboard，一份每天自动生成的 Markdown 报告就够了。

用第九章的 Cron 思路，创建一个每日健康报告任务：

```
hermes cron create "0 21 * * *" \
  "执行 Hermes 系统健康检查，生成一份日报，包含以下内容：
  1. 今日 Cron 执行情况（成功/失败）
  2. Gateway 是否正常运行
  3. 今日 Token 消耗估算（调用 hermes insights 数据）
  4. Memory 使用率是否需要清理
  5. 是否有 doctor 报告的问题
  6. 一句话总结今日系统状态
  总字数控制在 500 字以内。将报告保存到 ～/hermes-lab/reports/ 目录，文件名 health-report-YYYY-MM-DD.md。" \
  --name "daily-health-report" \
  --deliver "lark" \
  --workdir "～/hermes-lab/reports"
```

每天 21 点，Hermes 自动体检并推送报告到飞书。你睡前看一眼就知道系统有没有问题。

## 11.3 可视化工具：两个社区项目

终端命令覆盖了大部分可观测需求，但如果你想要图形界面，社区有两个成熟项目可选。

### hermes-webui

hermes-webui（GitHub 1,200+ stars）是一个三栏布局的 Web UI，左侧会话列表、中间聊天、右侧文件浏览器，和 CLI 功能几乎 1:1 对齐。

核心能力：

-   **Chat**：多模型对话，实时工具调用卡片（看到 Agent 正在调哪个工具、传了什么参数），流式输出
    
-   **Tasks**：图形化管理 Cron 任务，创建、编辑、运行、查看历史
    
-   **Skills / Memory / Profiles**：可视化浏览和管理，不用手改 YAML
    
-   **文件浏览器**：直接在 Web 里浏览、预览、编辑工作区文件
    
-   **手机访问**：完全响应式布局，通过 Tailscale 或 SSH 隧道从手机打开
    

安装一条命令：

```
git clone https://github.com/nesquena/hermes-webui.git
cd hermes-webui
python3 bootstrap.py
```

默认运行在 `localhost:8787`，也支持 Docker 部署。

### hermes-workspace

hermes-workspace（GitHub `outsourc-e/hermes-workspace`，3,400+ stars）定位更偏向"控制中心"，特色是 Conductor（并行 Agent 编排）和 Dashboard（Token 消耗、会话数量、工具调用统计的趋势图）。

核心能力：

-   **Conductor**：同时启动多个 Agent 执行不同任务
    
-   **Dashboard**：系统指标面板，可视化 Token 消耗趋势
    
-   **Memory**：可视化浏览和编辑 MEMORY.md / USER.md
    
-   **PWA 支持**：手机浏览器添加到主屏幕，类 App 体验
    

安装：

```
npx hermes-workspace
```

默认运行在 `localhost:3000`。

### 怎么选

| 需求 | 推荐 |
| --- | --- |
| 想要和 CLI 完全对等的 Web 体验 | hermes-webui |
| 想要同时跑多个 Agent、看趋势图 | hermes-workspace |
| 想在手机上管理 Agent | 两个都支持，都值得试 |

> **个人建议：终端命令是日常标配，可视化工具是锦上添花。** 先把 `hermes status` / `hermes doctor` / `hermes logs` 用熟，再考虑加 Web UI。毕竟多一个 Web 服务意味着多一个需要维护的东西。

> **一句话总结这一章：`hermes status` + `hermes doctor` + `hermes logs` + Cron 自动报告，解决 90% 的可观测需求。**

---

# 十二、语音、网页抓取、搜索：让输入更自然

前面几章把 Hermes 的核心能力配齐了。这一章讲两个"让交互更自然"的能力：语音（动口不动手）和搜索（让它自己上网找信息）。

语音和搜索都不是"满配必装"，但各有刚需场景：开车、做饭、通勤时适合语音；做行业调研、竞品分析、AI 工具雷达、公众号选题时适合搜索。

## 12.1 语音交互

### 安装

先确认普通文本模式可以正常使用：

```
hermes
```

进入后随便问一句：

```
你现在有哪些可用工具？
```

确认文本对话没问题后，再安装语音依赖。

如果你是通过 pip 安装 Hermes，推荐：

```
pip install "hermes-agent[voice]"
```

macOS 还需要安装系统音频依赖：

```
brew install portaudio ffmpeg opus
```

如果你是从源码目录运行，例如已经有 `～/.hermes/hermes-agent`，也可以使用：

```
cd ～/.hermes/hermes-agent
uv pip install -e ".[voice]"
```

> 注意：`uv pip install -e ".[voice]"` 更适合源码安装/开发安装场景；普通用户写教程时，优先给 `pip install "hermes-agent[voice]"`。

### 国内用户必做：HuggingFace 镜像配置

> 这是国内用户**最容易卡住的一步**。如果跳过，后面语音转录会永远卡在 "Transcribing..."。

本地 STT 使用 faster-whisper 模型，**首次运行时需要从 HuggingFace 下载模型文件**。国内网络大概率连不上 `huggingface.co`，导致模型下载超时，表现为：录音正常，但转写环节一直转圈。

解决方法很简单，在 `～/.hermes/.env` 末尾加一行：

```
HF_ENDPOINT=https://hf-mirror.com
```

可以用终端直接追加：

```
echo 'HF_ENDPOINT=https://hf-mirror.com' >> ～/.hermes/.env
```

添加后重启 Hermes 即可。模型只需要下载一次，之后会缓存到本地（`～/.cache/huggingface/hub/`），后续转录不再依赖网络。

> 怎么确认生效？重启 Hermes 后用 `/voice on` 开启语音，按 `Ctrl+B` 录一句话，松手后 2-3 秒内应该出现转写文字。如果还是卡住，在终端运行 `echo $HF_ENDPOINT` 确认环境变量已设置。

### 启用方式

安装完成后，不是"自动开始语音对话"，而是需要进入 Hermes CLI 后手动开启：

```
hermes
```

在 Hermes 交互界面里输入：

```
/voice on
```

常用语音命令：

```
/voice          # 开关语音模式
/voice on       # 开启语音模式
/voice off      # 关闭语音模式
/voice tts      # 开关语音回复
/voice status   # 查看当前语音状态（推荐先跑这个确认环境就绪）
```

默认按 `Ctrl+B` 开始录音。你说完后，Hermes 会根据静音检测自动停止录音，然后把语音转成文本发给 Agent；如果开启 TTS，它还会把回复读出来。

### 推荐配置

Hermes 的语音配置分三块：

-   `voice`：录音按键、静音检测、是否自动语音回复
    
-   `stt`：Speech-to-Text，语音识别，把你说的话转成文字
    
-   `tts`：Text-to-Speech，语音合成，把 Hermes 的回复读出来
    

推荐从"本地 STT + Edge TTS"起步：成本低、中文可用、配置简单。

```
# ～/.hermes/config.yaml

voice:
record_key:"ctrl+b"
max_recording_seconds:120
auto_tts:false
beep_enabled:true
silence_threshold:200
silence_duration:3.0

stt:
provider:"local"
local:
    model:"base"

tts:
provider:"edge"
edge:
    voice:"zh-CN-XiaoxiaoNeural"
```

如果希望男声，可以把 Edge TTS 的 voice 换成 `zh-CN-YunjianNeural`。

### 引擎怎么选

语音系统分两半：STT（你说的话 → 文字）和 TTS（文字 → Hermes 说的话）。两半可以独立选择。

**STT（语音识别）**：

| 引擎 | 类型 | 成本 | 适合场景 |
| --- | --- | --- | --- |
| `local` | 本地 | 免费，首次需下载模型（需网络） | 个人使用、隐私优先。国内用户需配置 HuggingFace 镜像 |
| `groq` | 云端 | 需要 Key，有免费额度 | 追求速度，接受云端转写 |
| `openai` | 云端 | 按量计费 | 作为稳定云端备选 |

本地 STT 使用 faster-whisper 模型，首次运行会自动下载。模型大小从 75MB 到 3GB 不等，选太大意味着下载更久、更可能中途失败：

| 模型 | 大小 | 中文识别 | 适合场景 |
| --- | --- | --- | --- |
| `tiny` | ～75 MB | 可用，精度一般 | 快速验证环境 |
| `base` | ～150 MB | 日常够用 | **推荐起步** |
| `small` | ～500 MB | 较好 | 追求准确度 |
| `medium` | ～1.5 GB | 好 | 高精度需求 |
| `large-v3` | ～3 GB | 最好 | 极致精度 |

日常中文口述建议先用 `base`，如果识别不准再升级到 `small`。

**TTS（语音合成）**：

| 引擎 | 类型 | 成本 | 适合场景 |
| --- | --- | --- | --- |
| `edge` | 云端 | 免费 | 中文日常使用，推荐起步 |
| `neutts` | 本地/开源 | 免费，消耗本地算力 | 更本地化、可控的语音输出 |
| `openai` | 云端 | 按量计费 | 追求稳定质量 |
| `elevenlabs` | 云端 | 付费 | 追求更自然的英文/多语种音色 |

国内用户起步建议：**本地 STT + Edge TTS 中文音色**，不需要折腾 Key，中文交互也够用。

如果你想接入国产语音服务（阿里云智能语音、讯飞、百度、火山引擎等），更适合通过自定义 MCP 或脚本工具接入，不建议写成 Hermes 内置 provider。

### 常见问题排查

用 `/voice status` 可以看到当前语音系统的完整状态。常见问题：

\*\*卡在 "Transcribing..."\*\*：本地 STT 大概率是模型下载失败，检查 `～/.hermes/.env` 是否设置了 `HF_ENDPOINT=https://hf-mirror.com`；Groq STT 可能是 Key 失效（403），到 console.groq.com 检查。

**按 Ctrl+B 没反应**：检查 macOS 麦克风权限（"系统设置 → 隐私与安全性 → 麦克风"），运行 `/voice status` 确认 Audio capture 是否 OK。

**录了音但没有结果**：录音时长不足 0.3 秒或音量太低会被丢弃，说话时间长一点、声音大一点。

### 国内场景示例

```
场景 1：通勤路上做今日规划
  你（语音）："Hermes，帮我整理今天飞书日程，按重要性排一下。"
  AI（语音）："今天优先处理三件事：上午的客户方案评审、下午的话术 Agent 迭代复盘、晚上整理公众号草稿。"

场景 2：睡前口述复盘
  你（语音）："记录一下，今天测试了医美客服 Agent 的三轮模拟对话，发现访客画像还不够细。"
  AI（文字+语音）："已整理成复盘：问题是访客画像粒度不足，明天建议补充渠道来源、消费预算、顾虑点和成交阻力。"

场景 3：公众号选题脑暴
  你（语音）："帮我基于最近 AI Agent 的新闻，想 5 个适合公众号的选题。"
  AI（语音）："可以从 Agent OS、自进化 Harness、企业客服 Agent、个人记忆系统、AI 工具工作流五个方向展开。"
```

### 注意事项

-   中文语音识别受口音、噪声、专业术语影响较大，重要事项建议让 Hermes 复述确认。
    
-   语音回复要短，建议控制在 100-200 字以内；人在听的时候没法像看文字一样回看。
    
-   口述任务时尽量给清楚约束，例如："保存成 Markdown""只列 5 条""每条不超过 50 字"。
    

## 12.2 网页搜索与信息获取

Hermes 默认只能处理你给它的信息。接上搜索和网页读取能力后，它可以自己上网找资料——做行业调研、竞品分析、搜集公众号素材、跟踪机器之心/量子位/InfoQ 中文站等媒体动态。

### 搜索链路和后端

大多数信息搜集任务遵循同一条链路：**搜索 → 提取正文 → 保存结果**。你可以直接在 prompt 里说"搜索 xxx 并保存到文件"，Hermes 会根据可用工具自动完成。

当前比较稳的搜索后端：

| 后端 | 需要 Key | 适合场景 |
| --- | --- | --- |
| Firecrawl | 需要 | 默认推荐，search / extract / crawl 较完整 |
| DDGS / DuckDuckGo | 不需要 | 免 Key，起步测试、轻量搜索 |
| Tavily | 需要 | 调研型搜索 |

> 注意：百度、夸克、秘塔、微信搜一搜等不是 Hermes 内置后端。想用国产搜索入口，更稳的做法是通过 SearXNG 自建聚合、自定义 MCP 封装，或者直接把链接给 Hermes 让它提取整理。

网页提取也有两种方式：`web_extract` 直接抓正文（快，适合大部分文章页），浏览器方式打开页面读取渲染结果（慢，适合需要 JS 渲染的页面）。**国内平台（微信文章、小红书、知乎）可能有登录限制或反爬，最稳的做法是你给链接，Hermes 负责提取和整理。**

### 实战 Prompt 示例

```
# 中文 AI 资讯雷达
搜索最近 7 天中文互联网关于"Agent OS""自进化 Agent""AI 工作流"的文章，
优先参考机器之心、量子位、智东西、InfoQ 中文站、少数派、知乎专栏。
输出：标题、来源、核心观点、适合我公众号引用的角度。
保存到 ～/hermes-lab/reports/chinese-agent-radar.md。

# 竞品分析
搜索并整理扣子、Dify、FastGPT、MaxKB、RAGFlow 最近 3 个月的新功能，
重点分析它们在 Agent 编排、知识库、评估、工作流、企业部署上的差异。
输出成 Markdown 表格。

# 公众号素材整理
我会给你 5 篇微信文章链接或正文，请你提取每篇的核心观点、可引用金句、适合做配图的位置，
最后整理成一份选题素材库。

# 飞书会议纪要处理
读取我给你的飞书会议纪要文本，提取已做决策、待办事项、负责人、风险点和下次会前必须完成的事项，
保存到 ～/notes/meeting-action-items.md。

# 音频转文字
请使用当前可用的语音识别/STT能力，转写 ～/recordings/meeting.mp3，
然后提取会议中的决策事项、待办事项、负责人和截止时间，
保存到 ～/notes/meeting-summary.md。
```

## 12.3 这一章的正确心法

语音和搜索不是为了"炫技"，而是为了降低输入成本和扩大信息来源。

对国内用户来说，更推荐这条路线：

1.  **语音先用本地 STT + Edge TTS 中文音色跑通。** 别忘了配 HuggingFace 镜像。
    
2.  **搜索先用 Hermes 官方支持的 Web backend 跑通。**
    
3.  **国内信息源先从 prompt 约束和人工给链接开始。**
    
4.  **需要深度国产化时，再用 MCP/脚本接入飞书、企业微信、国产搜索、国产语音服务。**
    

> **一句话：语音解放双手，搜索解放信息源；但新手教程里，配置要写官方支持的，例子可以尽量贴近国内真实场景。**

---

# 十三、Profiles：多实例隔离是长期使用的基础

> 本章目标：让你学会用 Profile 把不同用途的 Hermes Agent 隔离开，避免聊天、写作、自动任务、工具雷达等能力全部混在一起。
> 
> 适合读者：已经能正常运行 Hermes，并且至少完成过一次基础模型配置。

---

## 13.1 Profile 解决什么问题

假设你已经按照前面的章节，把 Hermes 配成了一个"AI 工具雷达"——每天自动搜集 GitHub 新项目、生成报告。然后你又开始用它聊天、写公众号、整理飞书纪要。

问题来了：**所有东西都混在同一个 Hermes 实例里。**

-   AI 雷达的自动任务和你的日常聊天共享同一套 Memory，记忆越写越乱
    
-   你想给雷达 Agent 关掉某些工具，但一关，你的日常聊天也受影响
    
-   雷达跑定时任务、Gateway 收消息、你手动聊天，可能互相干扰
    
-   不同任务的会话历史、配置、日志、Skills 混在一起，后期很难维护
    

Profile 就是为了解决这个问题。

一句话：**Profile = 一套独立的 Hermes 工作环境。**

每个 Profile 可以拥有自己独立的：

-   config 配置
    
-   API Key / `.env`
    
-   SOUL.md
    
-   memory
    
-   sessions
    
-   skills
    
-   gateway 状态
    
-   cron 任务
    

你可以把它理解成：

```
一个 Profile ≈ 一个专用 Agent 的工作空间
```

比如：

```
default         → 日常聊天
radar-agent     → AI 工具雷达
writing-agent   → 公众号写作
cleaner-agent   → 每周系统清理
commander       → 统一入口 / 总控助手
```

---

## 13.2 一个重要提醒：Profile 不是安全沙箱

Profile 能解决的是 **Hermes 内部状态隔离**，比如配置、记忆、会话、技能、Gateway 状态等。

但它不等于安全沙箱。

如果你使用的是本地 terminal backend，那么不同 Profile 里的 Agent 仍然可能访问当前系统用户有权限访问的文件。也就是说：

```
Profile 隔离的是 Hermes 状态，不是操作系统权限。
```

所以：

-   不要把 Profile 当成"安全容器"
    
-   不要只靠 SOUL.md 限制 Agent 不能访问某些目录
    
-   真正的文件权限隔离，需要额外使用 sandbox、容器、单独系统用户或受限目录
    

后面我们会在 SOUL.md 里写安全边界，但你要理解：**SOUL.md 是行为约束，不是系统权限控制。**

---

## 13.3 快速上手

### 1\. 创建第一个 Profile

建议从 `ai-radar` 开始。这个 Profile 后面可以用来做 AI 工具雷达：

```
hermes profile create ai-radar --clone
```

这里的 `--clone` 很重要。

它的意思是：从当前 Profile 克隆基础配置，让新 Profile 沿用你已经配好的模型和密钥。

但要注意：

```
--clone 不是完整复制。
```

通常你可以这样理解：

| 创建方式 | 大致效果 | 适合场景 |
| --- | --- | --- |
| `hermes profile create NAME` | 创建一个空白 Profile | 你想从零配置 |
| `hermes profile create NAME --clone` | 克隆当前 Profile 的基础配置，例如 config、.env、SOUL.md | 新建一个干净 Agent，但沿用模型/API Key |
| `hermes profile create NAME --clone-all` | 尽可能完整复制当前 Profile，包括更多状态数据 | 备份或完整 fork 一个已有 Agent |

日常新建 Agent，优先用：

```
hermes profile create ai-radar --clone
```

不要一上来就用 `--clone-all`，否则你可能把旧 Profile 的历史状态、旧任务、旧会话也一起带过去。

---

### 2\. 验证创建成功

查看所有 Profile：

```
hermes profile list
```

你应该能看到类似输出：

```
default      (active)
  ai-radar
```

再查看 `ai-radar` 的详细信息：

```
hermes profile show ai-radar
```

这会显示它的路径、配置、Gateway 状态等信息。

---

### 3\. 切换 Profile

有两种方式。

#### 方式一：设为默认 Profile

```
hermes profile use ai-radar
```

执行后，之后你运行 `hermes chat`、`hermes gateway` 等命令，默认都会使用 `ai-radar`。

切回默认 Profile：

```
hermes profile use default
```

#### 方式二：临时指定 Profile

更推荐新手先用这种方式，因为不容易忘记自己切到了哪里：

```
hermes -p ai-radar chat
```

这表示：只在这一次命令中使用 `ai-radar`。

如果你只是临时跑一次任务，建议用：

```
hermes -p ai-radar chat
```

而不是频繁 `profile use` 来回切换。

---

### 4\. 查看当前 Profile

查看当前默认 Profile：

```
hermes profile
```

它会显示当前 Profile 名称和对应路径。

---

## 13.4 Profile 里通常有什么

每个 Profile 都对应一个独立目录。

你可以把它理解成类似这样的结构：

```
～/.hermes/profiles/ai-radar/
├── config.yaml       # 模型、Provider、工具等配置
├── .env              # API Key、Bot Token 等敏感配置
├── SOUL.md           # Agent 的角色人格和行为协议
├── profile.yaml      # Profile 元信息
├── memories/         # 记忆文件，具体结构取决于你的 memory provider
├── sessions/         # 对话历史
├── skills/           # 已安装或可用的 Skills
├── cron/             # 定时任务相关状态
├── logs/             # 运行日志
├── state.db          # 会话和状态数据库
└── gateway.pid       # Gateway 进程状态
```

不同版本、不同安装方式下，目录结构可能略有差异。你不需要死记目录结构，只需要记住：

```
每个 Profile 都有自己的一套 Hermes 状态。
```

---

## 13.5 `--clone`、`--clone-all`、`--clone-from` 怎么选

### 最常用：从当前 Profile 克隆基础配置

```
hermes profile create radar-agent --clone
```

适合：

-   新建一个独立 Agent
    
-   沿用当前模型配置和 API Key
    
-   不想把旧会话、旧记忆完整复制过去
    

---

### 完整复制当前 Profile

```
hermes profile create backup-agent --clone-all
```

适合：

-   做完整备份
    
-   完整 fork 一个已有 Agent
    
-   希望尽量保留旧 Profile 的全部状态
    

不建议新手在日常创建 Agent 时默认使用 `--clone-all`，因为它容易把旧状态也带过去。

---

### 从指定 Profile 克隆

如果你当前在 `default`，但想从 `dev` 这个 Profile 克隆配置，应该这样写：

```
hermes profile create writing-agent --clone --clone-from dev
```

如果要完整复制 `dev`：

```
hermes profile create writing-agent --clone-all --clone-from dev
```

注意，不建议写成：

```
hermes profile create writing-agent --clone-from dev
```

因为 `--clone-from` 应该和 `--clone` 或 `--clone-all` 搭配使用。

---

## 13.6 推荐 Profile 设计

不要一上来建 10 个 Profile。先建 2～3 个，跑稳了再加。

建议从这几个开始：

| Profile | 用途 | 创建命令 |
| --- | --- | --- |
| `default` | 日常聊天、临时任务 | 默认已有，不需要创建 |
| `radar-agent` | AI 工具雷达、每日信息报告 | `hermes profile create radar-agent --clone` |
| `writing-agent` | 公众号写作、选题、大纲、素材整理 | `hermes profile create writing-agent --clone` |

等你熟悉之后，再加：

```
hermes profile create cleaner-agent --clone
hermes profile create commander --clone
```

更进一步，还可以按场景拆分：

```
hermes profile create dev-agent --clone          # 代码开发
hermes profile create research-agent --clone     # 论文/资料研究
hermes profile create ops-agent --clone          # 自动化运维
```

原则是：

```
任务边界越清晰，越适合拆成独立 Profile。
```

但不要为了"看起来很 Agent 团队"而拆太多。拆太多之后，你会增加维护成本。

---

## 13.7 常用命令速查

### 创建

```
# 创建空白 Profile
hermes profile create <名称>

# 从当前 Profile 克隆基础配置
hermes profile create <名称> --clone

# 从当前 Profile 完整克隆
hermes profile create <名称> --clone-all

# 从指定 Profile 克隆基础配置
hermes profile create <名称> --clone --clone-from <来源Profile>

# 从指定 Profile 完整克隆
hermes profile create <名称> --clone-all --clone-from <来源Profile>
```

### 切换

```
# 设为默认 Profile
hermes profile use <名称>

# 临时使用某个 Profile 进入聊天
hermes -p <名称> chat

# 切回 default
hermes profile use default
```

### 查看

```
# 查看当前默认 Profile
hermes profile

# 查看所有 Profile
hermes profile list

# 查看某个 Profile 的详细信息
hermes profile show <名称>
```

### 管理

```
# 删除 Profile，会有二次确认
hermes profile delete <名称>

# 重命名 Profile
hermes profile rename <旧名> <新名>

# 导出 Profile 为压缩包
hermes profile export <名称>
```

---

# 十四、24h Agent 团队：从单兵到协作系统

> 本章目标：基于 Profile、Gateway、Cron 和共享目录，搭一个"低成本、可维护、可验证"的个人 Agent 团队。
> 
> 注意：这里讲的是一个适合个人用户落地的 MVP，不是复杂的企业级多 Agent 调度平台。
> 
> **具体的动手实操步骤，请看第十七章"实战一：AI 工具雷达 Agent"。**

---

## 14.1 多 Agent 不是魔法

多 Agent 不是让几个 AI 自动变成一家公司。

更现实的理解是：

```
多个独立 Profile + 明确分工 + 共享目录 + 定时任务 + 人工确认
```

也就是把你之前已经学过的能力组合起来：

-   **Profile**：每个 Agent 一套独立配置、记忆、会话和技能
    
-   **Gateway**：让某个 Agent 通过 Telegram / 飞书等入口接收消息
    
-   **Cron**：让某个 Agent 在固定时间自动执行任务
    
-   **共享目录**：让不同 Agent 通过文件交换信息
    
-   **人工确认**：安装、删除、发布、改配置等高风险操作必须你确认
    

一个可落地的 24h Agent 团队，不是每个 Agent 24 小时烧 Token，而是：

```
该常驻的常驻，该定时的定时，该手动的手动。
```

比如：

-   Commander 常驻，用 Gateway 接收你的消息
    
-   Radar 每天定时跑一次
    
-   Cleaner 每周定时跑一次
    
-   Writing 只在你写文章时手动启动
    

---

## 14.2 先讲清楚：Commander 不是天然会"调用其他 Profile"

很多人第一次搭多 Agent，会误以为：

```
我在 Commander 里说"让 Radar 去查一下"，Radar 就会自动启动。
```

这不一定成立。

如果你没有额外封装跨 Profile 调用工具、脚本或调度器，那么 Commander 并不会天然拥有"远程调用另一个 Profile"的能力。

所以本章的 MVP 采用更稳的方式：

```
不同 Agent 通过共享目录协作。
```

也就是说：

-   Radar Agent 每天生成报告到共享目录
    
-   Cleaner Agent 每周生成清理建议到共享目录
    
-   Writing Agent 按需读取素材、输出文章草稿
    
-   Commander 主要读取共享目录里的结果，帮你摘要和汇总
    

这比"幻想 Agent 之间自动互相调度"更稳，也更适合读者跟做。

后续如果你想进一步自动化，可以再封装脚本，让 Commander 调用：

```
hermes -p radar-agent chat
```

但这已经是下一层工程化能力了，不建议在第一版里一上来就做。

---

## 14.3 一个最小团队的架构

从最小可用开始，4 个角色：

```
Commander（统一入口，通过飞书/Telegram 接收你的消息）
├── Radar Agent（每天自动搜集 AI 工具新动态）
├── Writing Agent（帮你做公众号选题和素材整理）
└── Cleaner Agent（每周检查系统，输出清理建议）
```

它们不是全天候都在运行：

| Agent | 是否需要常驻 | 运行方式 |
| --- | --- | --- |
| Commander | 是 | Gateway 常驻 |
| Radar Agent | 不一定 | Cron 定时触发，或手动运行 |
| Writing Agent | 否 | 需要写作时手动启动 |
| Cleaner Agent | 不一定 | Cron 每周触发 |

---

## 14.4 共享目录：Agent 之间怎么协作

多个 Agent 之间不要指望它们"互相知道对方在想什么"。

最稳的方式是共享一个目录：

```
～/hermes-lab/shared/
├── reports/           # Radar、Cleaner、Writing 输出的报告
├── decisions/         # 你人工确认过的决策
└── skill-candidates/  # 待沉淀成 Skill 的候选流程
```

约定：

| 目录 | 用途 | 写入者 | 读取者 |
| --- | --- | --- | --- |
| `reports/` | 每日/每周报告 | Radar、Cleaner、Writing | Commander、你本人 |
| `decisions/` | 人工确认过的决策 | 你，或 Commander 在你确认后代写 | 所有 Agent |
| `skill-candidates/` | 待沉淀成 Skill 的流程 | 任何 Agent | 你和 Writing/Cleaner |

比如：

```
Radar Agent 每天生成：
～/hermes-lab/shared/reports/radar-2026-05-25.md

Cleaner Agent 每周生成：
～/hermes-lab/shared/reports/cleanup-2026-05-25.md

Commander 读取这些报告后回答：
"今天 AI 雷达有什么发现？"
```

这就是最小可用的协作方式。

---

## 14.5 核心原则

1.  **Profile 负责隔离**不同 Agent 的配置、记忆、会话、技能尽量分开。
    
2.  **Gateway 负责入口**MVP 阶段只让 Commander 接 Gateway，避免多个 Bot Token 冲突。
    
3.  **Cron 负责定时**Radar 每天跑，Cleaner 每周跑，但要记得 Cron 通常依赖 Gateway / scheduler 长期运行。
    
4.  **共享目录负责协作**Agent 之间先通过文件交换信息，不要一上来追求复杂调度。
    
5.  **人工确认负责安全**安装、删除、发布、改配置、访问敏感信息，都必须人工确认。
    
6.  **先手动，再自动**先用 CLI 手动跑通，再接 Gateway，最后接 Cron。
    

真正可落地的 24h Agent 团队，不是让 AI 全天候乱跑，而是让不同 Agent 在合适的时间、合适的入口、用合适的权限完成自己的任务。

> 具体的动手实操步骤——创建 Profile、配置 SOUL.md、接 Gateway、接 Cron、用 tmux 启动——请看第十七章。

---

# 十五、一键满配与生态导航：可以看，但不要盲信

## 一键满配的价值

一键脚本适合：

-   新手快速体验
    
-   测试环境
    
-   看别人怎么组织配置
    
-   快速生成默认模板
    

但不适合：

-   生产环境
    
-   有隐私数据的环境
    
-   高权限工具环境
    
-   你还没理解每个模块作用的时候
    

> 一键满配可以作为参考，但真正长期使用的 Hermes，还是应该一层一层手动配置。否则你不知道自己到底打开了什么权限。

## 生态导航可以怎么用？

| 资源类型 | 用途 |
| --- | --- |
| 官方文档 | 查命令、配置、能力边界 |
| GitHub Repo | 看最新版本、issue、release |
| Skills Hub | 搜索 Skill |
| Awesome List | 找社区项目 |
| 生态导航站 | 快速发现工具 |
| 个人雷达 Agent | 定期筛选，而不是自己每天刷 |

最终还是回到第一个实战：

> 不要靠人肉刷工具，而是让 Hermes 帮你做工具雷达，再由你决定是否安装。

---

# 十六、我的推荐满配路线

如果你是第一次从裸装走向满配，我建议按 6 个阶段来。

## 阶段 0：跑通基础环境

```
hermes setup
hermes model
hermes doctor
hermes
```

目标：能正常聊天，Provider 稳定，能排查错误。

## 阶段 1：配置输入系统

准备 SOUL.md、AGENTS.md、一个实验目录、一个任务输入模板。

目标：Hermes 知道你是谁，知道当前目录是干什么的，输出风格更稳定。

## 阶段 2：配置 Memory 和 Skills

```
hermes memory status
hermes skills list
hermes skills browse
hermes skills inspect 
```

目标：先用原生 Memory，少量安装 Skill，写 1 个自己的 Skill。

## 阶段 3：配置 MCP 和工具边界

```
hermes tools list
hermes mcp list
# 编辑 ～/.hermes/config.yaml 添加 MCP Server
hermes mcp test 
hermes mcp configure 
```

目标：接入 1-2 个真正有用的外部工具，不全量打开，高危工具默认关闭。

## 阶段 4：配置 Gateway 和 Cron

```
hermes gateway setup
hermes gateway run
hermes cron create "every day 9am"
hermes cron list
```

目标：让 Hermes 主动给你报告，先只做只读任务，安装和删除必须人工确认。

## 阶段 5：配置 Profiles

```
hermes profile create ai-radar --clone
hermes profile create brainstorm-room --clone
```

目标：不同场景隔离，不同工具隔离。

## 阶段 6：尝试 24h Agent 团队

```
hermes profile create commander --clone
hermes profile create radar-agent --clone
hermes profile create cleaner-agent --clone
```

目标：Commander 做入口，Radar 做每日工具雷达，Cleaner 做每周清理，共享目录做协作，人工确认做安全边界。

---

# 十七、实战一：AI 工具雷达 Agent

前面的模块讲完了，下面用一个轻量但完整的例子把它们串起来。

本章只聚焦一个目标：**搭一个 AI 工具雷达 Agent，让它每天帮你搜集 AI 生态的新项目和更新。**

如果你想搭完整的多 Agent 团队（Commander + Radar + Writing + Cleaner），建议先完成本章，再回看第十四章的架构说明。

> 适用环境：macOS / Linux / WSL2。Windows 原生命令行用户需要把 `～`、``cat <、`date` 等 Shell 写法换成 PowerShell 对应写法。``

## `目标`

`做一个 AI 工具雷达 Agent：`

> `每天自动搜集 GitHub 上关于 Memory、Skill、MCP、Agent Tools、Hermes 等方向的新项目和重要更新；结合我当前安装的工具、Skills、SOUL.md、CLAUDE.md、长期记忆，判断哪些值得更新、哪些不建议更新；每天给我一个潜在更新列表；每周给我一个建议 disable / 合并 / 清理的列表。`

`这个例子不涉及公司隐私，只处理公开信息和本地配置元信息。`

## `用到了哪些 Hermes 特性？`

| 特性 | 用法 |
| --- | --- |
| Profile | 创建 `radar-agent` 独立环境 |
| Cron | 每天定时生成报告 |
| Gateway | 让 Cron scheduler 常驻运行；也可把报告推送到 Telegram / 飞书 |
| Skills | 固化"工具是否值得安装"的判断标准 |
| MCP / gh CLI | 搜集 GitHub 更新，可选 |
| Memory | 记住长期偏好 |
| Sessions / Reports | 保存历史分析结果 |

---

## `Step 1：创建 Profile 和共享目录`

`先创建报告的输出目录：`

```
mkdir -p ～/hermes-lab/shared/{reports,decisions,skill-candidates}
```

`目录用途：`

| 目录 | 用途 |
| --- | --- |
| `reports/` | Radar Agent 每天输出的报告 |
| `decisions/` | 你人工确认过的决策 |
| `skill-candidates/` | 待沉淀成 Skill 的流程或经验 |

`然后创建 Radar Agent 的 Profile：`

```
hermes profile create radar-agent --clone
```

``这里用 `--clone` 是为了复用当前 Profile 的模型、API Key 和基础 SOUL 配置。它不会复制旧会话和长期记忆，因此适合创建一个相对干净的新 Agent。``

`确认创建成功：`

```
hermes profile list
hermes profile show radar-agent
```

``应该能看到 `radar-agent`。``

> ``注意：如果你之前已经创建过 `radar-agent`，会报 `profile already exists`。新手建议先直接复用已有的，不要急着删除。`hermes profile delete radar-agent` 会删除这个 Profile 的配置和状态，执行前请确认里面没有重要内容。``

## `Step 2：配置 Radar Agent 的 SOUL.md`

`直接写入对应 Profile 目录，不需要先切换 Profile：`

```
cat > ～/.hermes/profiles/radar-agent/SOUL.md << 'EOF_SOUL'
你是 AI 工具雷达 Agent。

你的职责是每天搜集 AI Agent 生态的新动态，输出结构化报告。

## 工作范围
你只处理公开信息：
- GitHub 上 Agent / MCP / Skill / 工具相关的项目和 release
- 开源项目的 README 和 changelog
- 公开的技术博客和文档

## 安全边界
- 不读取任何公司文件或私有仓库
- 不上传密钥或凭证
- 不自动安装任何工具，只输出建议
- 所有安装、删除、修改配置类建议，必须经过人工确认后才执行

## 输出格式
每次执行后输出一份 Markdown 报告，包含：
1. 今日新增项目：名称、链接、一句话描述
2. 已关注项目的更新：版本号、主要变更
3. 建议分类：建议安装 / 仅观察 / 不建议安装
4. 需要人工确认的事项

## 文件保存规则
报告保存到：
～/hermes-lab/shared/reports/

文件名格式：
radar-YYYY-MM-DD.md

不要覆盖历史报告。
EOF_SOUL
```

``为什么不先 `hermes profile use radar-agent`？因为直接写入文件路径更不容易误操作。如果你后面忘记切回 default，后续所有命令都可能跑到 `radar-agent` 里，读者很容易搞混。``

---

## `Step 3：手动跑一次验证`

`这一步是**最小可用的终点**。跑通这一步，说明 Radar Agent 已经能工作了。`

`进入 Radar Agent：`

```
hermes -p radar-agent chat
```

`然后输入：`

```
执行今天的 AI 工具雷达任务：
搜集过去 24 小时 GitHub 上关于 MCP、Agent Tools、AI Agent 框架的新项目和重要更新。
输出一份 Markdown 报告，保存到 ～/hermes-lab/shared/reports/ 目录。
文件名使用 radar-YYYY-MM-DD.md，不要覆盖历史报告。
```

`检查是否生成报告：`

```
ls -lh ～/hermes-lab/shared/reports/
```

`查看最新报告：`

```
latest_report="$(ls -t ～/hermes-lab/shared/reports/radar-*.md 2>/dev/null | head -n 1)"
echo "$latest_report"
cat "$latest_report"
```

``如果能看到类似 `radar-2026-05-25.md` 的文件，并且内容包含项目列表和分类建议，说明第一个 Agent 已经跑通。``

`如果报告内容太简略或没生成，优先检查：`

```
hermes -p radar-agent status
hermes -p radar-agent dump
```

`重点看：`

-   `模型 Provider 是否配置正确`
    
-   `API Key 是否有效`
    
-   `` 当前 Profile 是否真的是 `radar-agent` ``
    
-   `是否启用了 Web/Search/Terminal/File 相关工具`
    
-   `当前网络是否能访问 GitHub`
    

> ``注意：让 Agent “搜集 GitHub 过去 24 小时更新”并不一定天然稳定。它依赖你当前 Hermes 是否启用了 Web/Search 工具，或者是否安装并登录了 `gh`。下面的 Step 5 会给一个更可控的 GitHub CLI 方案。``

---

> `到这里，如果你只想体验一下，可以停了。下面是进阶内容：让 Radar Agent 更智能（快照 + Skill）、自动定时跑、推送到飞书 / Telegram。`

---

## `Step 4：本地状态快照（进阶）`

`这个脚本帮 Radar Agent 知道你当前装了什么工具、什么 Skills、什么 MCP Server。这样它在输出建议时能对比“你已有的”和“新出现的”。`

`创建快照脚本：`

```
mkdir -p ～/hermes-lab/ai-radar/{inputs,snapshots}

cat > ～/hermes-lab/ai-radar/snapshots/collect_local_state.sh << 'EOF_SCRIPT'
#!/usr/bin/env bash
set -u

PROFILE="${1:-radar-agent}"
OUT_DIR="$HOME/hermes-lab/ai-radar/snapshots"
DATE="$(date +"%Y-%m-%d")"
OUT="$OUT_DIR/local_state_$DATE.md"

mkdir -p "$OUT_DIR"

{
echo"# Local AI Agent State Snapshot - $DATE"
echo
echo"## Target Profile"
echo"$PROFILE"
echo

echo"## Hermes Version"
  hermes --version || true
echo

echo"## Hermes Status"
  hermes -p "$PROFILE" status || true
echo

echo"## Hermes Dump"
echo"下面是 Hermes 的脱敏环境摘要，适合给 Agent 读取做判断："
  hermes -p "$PROFILE" dump || true
echo

echo"## Installed Skills"
  hermes -p "$PROFILE" skills list || true
echo

echo"## MCP Servers"
  hermes -p "$PROFILE" mcp list || true
echo

echo"## Enabled Tools Summary"
  hermes -p "$PROFILE" tools --summary || true
echo

echo"## Important Context Files"
for dir in"$HOME/.hermes""$HOME/.claude""$HOME/hermes-lab"; do
    if [ -d "$dir" ]; then
      echo
      echo"### $dir"
      find "$dir" -maxdepth 4 \( -name "CLAUDE.md" -o -name "AGENTS.md" -o -name "SOUL.md" \) 2>/dev/null | sort
    fi
done
} > "$OUT"

echo"Snapshot saved to $OUT"
EOF_SCRIPT

chmod +x ～/hermes-lab/ai-radar/snapshots/collect_local_state.sh
```

`手动跑一次看看效果：`

```
bash ～/hermes-lab/ai-radar/snapshots/collect_local_state.sh radar-agent
latest_snapshot="$(ls -t ～/hermes-lab/ai-radar/snapshots/local_state_*.md | head -n 1)"
cat "$latest_snapshot"
```

`你应该能看到当前 Hermes 的版本、Profile 状态、已安装 Skills、MCP Servers、工具启用摘要等信息。`

> `安全提醒：快照脚本会读取本机 Hermes 配置摘要和部分上下文文件路径。它不应该输出 API Key 明文，但你仍然应该先自己看一遍输出内容，再决定是否让 Agent 使用。`

`之后每次跑 Radar 之前先执行一次这个脚本，Radar Agent 就能读取快照来对比差异。`

---

## `Step 5：搜集 GitHub 更新（进阶）`

`如果你安装了 GitHub CLI，可以让 Radar Agent 读取更稳定的 GitHub 搜索结果。`

`安装 GitHub CLI：`

```
# macOS
brew install gh

# Ubuntu / Debian
sudo apt update
sudo apt install gh
```

`登录并检查状态：`

```
gh auth login
gh auth status
```

`创建搜集目录：`

```
mkdir -p ～/hermes-lab/ai-radar/inputs/github
```

`搜集最近 24 小时有更新的仓库。下面这段兼容 macOS 和 Linux：`

```
SINCE="$(date -u -d '1 day ago' +%Y-%m-%d 2>/dev/null || date -u -v-1d +%Y-%m-%d)"
OUT_DIR="$HOME/hermes-lab/ai-radar/inputs/github"
mkdir -p "$OUT_DIR"

gh search repos "agent memory pushed:>=$SINCE" --sort updated --order desc --limit 20 > "$OUT_DIR/memory_repos.txt"
gh search repos "mcp server pushed:>=$SINCE" --sort updated --order desc --limit 20 > "$OUT_DIR/mcp_repos.txt"
gh search repos "agent skills pushed:>=$SINCE" --sort updated --order desc --limit 20 > "$OUT_DIR/skill_repos.txt"
gh search repos "context engineering agent pushed:>=$SINCE" --sort updated --order desc --limit 20 > "$OUT_DIR/context_repos.txt"
```

`检查结果：`

```
ls -lh ～/hermes-lab/ai-radar/inputs/github/
head -20 ～/hermes-lab/ai-radar/inputs/github/mcp_repos.txt
```

``然后在 `radar-agent` 里让它基于这些输入生成报告：``

```
hermes -p radar-agent chat
```

`输入：`

```
请执行今天的 AI 工具雷达任务。

请读取以下目录：
1. ～/hermes-lab/ai-radar/inputs/github/
2. ～/hermes-lab/ai-radar/snapshots/ 最新的 local_state_*.md
3. ～/hermes-lab/shared/reports/ 最近一份 radar-*.md

请对比我的本地状态和 GitHub 搜索结果，输出：
- 新出现或近期活跃的项目
- 与我的 Memory / Skill / MCP / Agent Tools 系统相关的项目
- 建议安装 / 建议试用 / 仅观察 / 不建议安装
- 需要人工确认的命令

最后保存到 ～/hermes-lab/shared/reports/radar-YYYY-MM-DD.md，不要覆盖历史报告。
```

``如果不想安装 `gh`，也可以让 Hermes 通过 Web/Search 工具搜索。但这依赖你是否启用了相关 toolset，结果也更依赖模型和搜索工具质量。给读者跟做时，`gh` 方案更可控。``

---

## `Step 6：自定义 Skill（进阶）`

`这一步是第五章“从零写一个 Skill”的实战延伸。如果你还没读过第五章，建议先回去看一眼 Skill 的文件结构。`

`这个 Skill 帮 Radar Agent 用统一标准判断一个工具值不值得安装。`

``Hermes 的 Skill 不是直接把一个 `.md` 文件丢进 `skills/` 根目录，而是建议使用：``

```
skills/<分类>//SKILL.md
```

`所以这里这样创建：`

```
mkdir -p ～/.hermes/profiles/radar-agent/skills/custom/ai-tool-update-advisor

cat > ～/.hermes/profiles/radar-agent/skills/custom/ai-tool-update-advisor/SKILL.md << 'EOF_SKILL'
---
name: ai-tool-update-advisor
description: 分析 AI 工具、GitHub 项目、MCP Server、Agent Skill 是否值得纳入我的个人 Agent 系统。
version: 1.0.0
metadata:
  hermes:
    category: custom
    tags: [ai-agent, mcp, skills, memory, tool-evaluation]
---

# AI Tool Update Advisor

## When to Use

当用户提供 GitHub 项目、工具更新列表、Skill Hub 更新、MCP Server 更新，或者要求判断某个 AI 工具是否值得安装时，使用本技能。

## 判断标准

1. 是否增强 Memory / Skill / Eval / Tool / Sandbox 能力
2. 是否减少重复劳动
3. 是否能沉淀为长期个人能力
4. 是否与现有系统重复
5. 是否需要过高权限
6. 是否有供应链风险
7. 是否只是短期噱头
8. 是否会让系统更复杂但收益不明显

## Procedure

1. 先识别项目类型：MCP Server / Skill / CLI 工具 / Agent Framework / Memory 工具 / 其他
2. 评估它和用户当前系统的相关度
3. 对比用户本地已有工具和历史报告，避免重复推荐
4. 输出建议动作
5. 对涉及安装、删除、修改配置的操作，必须标记为“需要人工确认”

## 输出格式

| 项目 | 类型 | 相关度 | 建议动作 | 更新价值 | 风险 | 原因 |
|---|---|---|---|---|---|---|

建议动作只能从以下选项中选择：

- 立即安装
- 建议试用
- 仅观察
- 只吸收思想
- 不建议安装
- 替换已有工具
- 合并到已有 Skill
- 建议禁用旧工具

## 最终输出

### 推荐更新
### 暂不建议
### 只吸收思想
### 需要人工确认的命令
### 本周建议清理

## Verification

输出中必须包含：
- 至少一个“不建议安装”的理由
- 所有高风险操作必须进入“需要人工确认的命令”
- 不得自动执行安装、删除、修改配置命令
EOF_SKILL
```

`重新扫描 Skills：`

```
hermes -p radar-agent chat
```

`进入会话后输入：`

```
/reload-skills
```

`也可以用一次性命令检查：`

```
hermes -p radar-agent chat --toolsets skills -q "请列出当前可用的 Skill 名称，确认是否能看到 ai-tool-update-advisor。"
```

``如果能看到 `ai-tool-update-advisor`，说明 Skill 已经生效。``

---

## `Step 7：接 Cron 让它每天自动跑`

`前面的 Step 3 是手动跑。如果你希望它每天早上 9 点自动执行，可以接 Cron。`

> `Cron 的概念和配置细节在第九章已讲过。这里只给具体命令。`

`创建每日任务：`

```
hermes cron create "0 9 * * *" '执行今天的 AI 工具雷达任务：
1. 先运行或读取 ～/hermes-lab/ai-radar/snapshots/ 下最新的 local_state_*.md；
2. 读取 ～/hermes-lab/ai-radar/inputs/github/ 下已有的 GitHub 搜索结果；
3. 如果具备 Web/Search 工具，可以补充搜索过去 24 小时 GitHub 上关于 MCP、Agent Tools、AI Agent 框架的新项目和重要更新；
4. 对比 ～/hermes-lab/shared/reports/ 下最近一次的 radar 报告；
5. 输出新增项目、已更新项目、建议分类；
6. 根据当前日期保存到 ～/hermes-lab/shared/reports/radar-YYYY-MM-DD.md；
7. 不要覆盖历史报告；
8. 不要自动安装任何工具，只输出建议。' \
  --profile radar-agent \
  --name "daily-ai-radar"
```

``注意：prompt 用单引号包起来。不要在里面写 `$(date +%Y-%m-%d)`，因为 shell 会在创建任务时提前展开日期，导致以后每天都写同一个固定文件名。``

`查看任务：`

```
hermes cron list
hermes cron status
```

### `手动测试 Cron`

`` `hermes cron run daily-ai-radar` 的含义是“让这个任务在下一次 scheduler tick 执行”，不是普通 shell 里的“立刻同步执行完”。所以建议先开一个终端运行 Gateway： ``

```
hermes -p radar-agent gateway run
```

`然后另开一个终端执行：`

```
hermes cron run daily-ai-radar
```

`等待 scheduler tick 后，再检查报告：`

```
ls -lh ～/hermes-lab/shared/reports/
```

`如果你只是想让 scheduler 跑一次，也可以执行：`

```
hermes cron run daily-ai-radar
hermes cron tick
```

### `Cron 自动触发的前提`

`Hermes 的 Cron 执行由 Gateway daemon 里的 scheduler 负责。创建 Cron 任务不等于它一定会自动运行。要想每天自动触发，需要让对应环境的 Gateway / scheduler 长期运行。`

`新手建议先用前台方式测试：`

```
hermes -p radar-agent gateway run
```

`稳定后再考虑后台服务：`

```
hermes -p radar-agent gateway install
hermes -p radar-agent gateway start
hermes -p radar-agent gateway status
```

``如果你在 WSL / Docker / Termux 里使用，优先用 `gateway run` 配合 tmux，不要一开始就用 `gateway start`。``

---

## `Step 8：接 Gateway 推送结果（可选）`

``如果你想让 Radar 的报告自动推送到飞书 / Telegram，可以给 `radar-agent` 接 Gateway。``

> `Gateway 的概念和配置细节在第八章已讲过。这里只给具体命令。`

```
hermes -p radar-agent gateway setup    # 选择平台、填写 Token
hermes -p radar-agent gateway run      # 前台运行，推荐先用这个测试
```

``注意：如果你已经在 Commander 或 default Profile 上接了 Gateway，不要让 `radar-agent` 用同一个 Bot Token，否则会冲突。MVP 阶段建议只让一个 Profile 接 Gateway；更稳的方式是只让 Commander 接 Gateway，Radar 只负责生成报告。``

`也可以不接 Gateway，直接在 CLI 里看报告：`

```
latest_report="$(ls -t ～/hermes-lab/shared/reports/radar-*.md 2>/dev/null | head -n 1)"
cat "$latest_report"
```

---

## `扩展建议`

`如果你已经跑通了单个 Radar Agent，想搭完整的多 Agent 团队（Commander + Radar + Writing + Cleaner），可以参考第十四章的架构设计。核心步骤：`

1.  `创建 Commander、Writing Agent、Cleaner Agent 的 Profile`
    
2.  `给每个 Profile 配 SOUL.md`
    
3.  `只让 Commander 接 Gateway，作为统一入口`
    
4.  `Radar 每天 Cron 跑一次`
    
5.  `Cleaner 每周 Cron 跑一次`
    
6.  ``所有 Agent 通过 `～/hermes-lab/shared/` 目录协作``
    

`多 Agent 进程管理可以用 tmux：`

```
tmux new -s hermes-team
# 窗口 1
hermes -p commander gateway run
# Ctrl+B C 创建新窗口
# 窗口 2
hermes -p radar-agent chat
```

`也可以使用一键 tmux 启动命令：`

```
tmux new-session -d -s hermes-team -n commander 'hermes -p commander gateway run'
tmux new-window -t hermes-team -n radar 'hermes -p radar-agent chat'
tmux attach -t hermes-team
```

---

## `这个例子把什么串起来了？`

-   `Profile 隔离 Radar Agent 的配置和记忆`
    
-   `Cron 主动触发每日任务`
    
-   `GitHub / Search 负责外部输入`
    
-   `Memory 理解你的长期偏好`
    
-   `Skill 固化判断流程`
    
-   `Gateway 推送结果（可选）`
    
-   `共享目录作为与其他 Agent 协作的桥梁`
    

`一句话：`

> `以前我看到新工具就想装，现在我让 Hermes 先帮我判断：它到底是新能力，还是新噪音。`

---

# `十八、实战二：头脑风暴聊天室 Agent`

## `目标`

`输入一篇文章、一个观点、或一个 md 文件，让 Hermes 调用多个“思维角色”进行讨论。`

`这里不是模拟真实人物，而是抽象成不同思维方式：`

-   `系统架构师：关注怎么变成可运行的系统`
    
-   `产品怀疑者：检查是否有真实用户价值`
    
-   `投资人视角：看长期壁垒和商业化空间`
    
-   `哲学观察者：看底层假设和认知盲点`
    
-   `写作编辑：看表达和信息密度`
    
-   `反方辩手：找最致命的反例`
    

`目标是：`

> `让一个观点经过多轮讨论后，变成你真正理解的知识卡片、文章选题或 Skill 候选。`

## `用到了哪些 Hermes 特性？`

| 特性 | 用法 |
| --- | --- |
| Profile | `brainstorm-room`独立上下文 |
| SOUL.md | 定义 Agent 的讨论规则和输出格式 |
| Memory | 只保存稳定结论 |
| Sessions | 保存完整讨论记录 |
| 文件系统 | 输入 md 文件、输出知识卡片 |

---

## `Step 1：创建 Profile 和目录`

```
hermes profile create brainstorm-room --clone

mkdir -p ～/hermes-lab/brainstorm-room/{inputs,personas,outputs,memory-cards}
```

`确认：`

```
hermes profile list
hermes profile show brainstorm-room
```

``应该能看到 `brainstorm-room`。``

---

## `Step 2：配置 SOUL.md`

```
cat > ～/.hermes/profiles/brainstorm-room/SOUL.md << 'EOF_SOUL'
你是头脑风暴聊天室的主持人。

你的职责是组织多个“思维角色”对一个观点或文章进行结构化讨论。

## 讨论规则
1. 每个角色用 150 字以内发言
2. 第一轮：每个角色说出自己看到的核心价值
3. 第二轮：每个角色指出一个最大问题
4. 第三轮：角色之间互相回应
5. 第四轮：你作为主持人总结

## 主持人总结必须包含
1. 这个观点真正有价值的地方
2. 最大的不确定性
3. 对我的具体启发（不是泛泛而谈）
4. 是否值得写成公众号文章（附理由）
5. 是否值得沉淀成 Skill / Memory / Eval（附理由）

## 文件保存规则
- 完整讨论总结保存到 ～/hermes-lab/brainstorm-room/outputs/
- 知识卡片保存到 ～/hermes-lab/brainstorm-room/memory-cards/
- 文件名使用 topic-YYYY-MM-DD.md 或 card-YYYY-MM-DD.md
- 不要覆盖历史文件

## 记忆规则
- 不要把整场讨论写进 Memory
- 只把稳定、长期、与我的目标相关的 1-3 条结论作为 Memory 候选
- 讨论过程保存在 session 或 outputs 里即可

## 安全边界
- 不访问需要登录的内容
- 不伪造角色观点
- 不把未经讨论的信息写成结论
EOF_SOUL
```

---

## `Step 3：创建角色文件`

``角色文件放在 `～/hermes-lab/brainstorm-room/personas/` 目录下。每个角色一个 `.md` 文件。``

`Hermes 在讨论时会读取这些文件来理解每个角色的视角。`

### `系统架构师`

```
cat > ～/hermes-lab/brainstorm-room/personas/system-architect.md << 'EOF_PERSONA'
# System Architect

你关注一个观点如何变成可运行的系统。

你重点分析：
1. 输入是什么
2. 输出是什么
3. 中间状态如何表示
4. 哪些模块必须有
5. 哪些模块可以后置
6. 最小可验证版本是什么
7. 系统边界在哪里

发言风格：
- 用结构化方式表达
- 画出数据流向（用文字描述）
- 每次发言至少指出一个架构风险
EOF_PERSONA
```

### `产品怀疑者`

```
cat > ～/hermes-lab/brainstorm-room/personas/product-skeptic.md << 'EOF_PERSONA'
# Product Skeptic

你的职责不是否定一切，而是检查一个观点是否真的有用户价值。

你重点关注：
1. 这个问题是否真实存在
2. 用户是否愿意付出时间、金钱或迁移成本
3. 它是否只是技术自嗨
4. 是否存在更简单的替代方案
5. 这个观点落地时最可能死在哪里

发言风格：
- 直接
- 不说空话
- 每次最多提出 3 个关键质疑
- 质疑后必须给出一个改进方向
EOF_PERSONA
```

### `投资人视角`

```
cat > ～/hermes-lab/brainstorm-room/personas/investor-view.md << 'EOF_PERSONA'
# Investor View

你从投资回报和长期壁垒的角度看问题。

你重点关注：
1. 这个方向的天花板在哪里
2. 竞争对手会怎么做
3. 护城河是什么（技术壁垒、数据壁垒、网络效应）
4. 变现路径是否清晰
5. 时机是否对：是太早还是太晚

发言风格：
- 用数字和类比说话
- 每次发言至少提一个对标案例
- 不回避“这个不值得做”的结论
EOF_PERSONA
```

### `哲学观察者`

```
cat > ～/hermes-lab/brainstorm-room/personas/philosophy-observer.md << 'EOF_PERSONA'
# Philosophy Observer

你关注一个观点背后的底层假设和认知盲点。

你重点关注：
1. 这个观点的隐性假设是什么
2. 它用了什么类比，这个类比是否合适
3. 是否存在确认偏误：只看到支持这个观点的证据
4. 反过来想：如果这个观点完全错了，最可能的原因是什么
5. 这个观点在 5 年后是否还有意义

发言风格：
- 善于追问“为什么”
- 用提问而不是判断
- 每次发言至少提出一个被忽略的角度
EOF_PERSONA
```

### `写作编辑`

```
cat > ～/hermes-lab/brainstorm-room/personas/writing-editor.md << 'EOF_PERSONA'
# Writing Editor

你从信息表达和读者接收的角度看问题。

你重点关注：
1. 这个观点能一句话说清楚吗
2. 目标读者是谁，他们能看懂吗
3. 信息密度够不够：有没有废话
4. 标题是否吸引人且不标题党
5. 如果写成文章，最好的切入点是什么

发言风格：
- 用读者视角说话
- 每次发言给出一个具体的表达建议
- 区分“信息增量”和“已知信息”
EOF_PERSONA
```

### `反方辩手`

```
cat > ～/hermes-lab/brainstorm-room/personas/devils-advocate.md << 'EOF_PERSONA'
# Devil's Advocate

你的职责是找出这个观点最致命的弱点。

你重点关注：
1. 最好的反例是什么
2. 这个观点在什么条件下会完全失效
3. 支持者最容易忽略的风险是什么
4. 如果执行失败，最可能的失败模式是什么
5. 有没有更简单的替代解释

发言风格：
- 直接挑战
- 每次发言必须给出一个具体的反例或反驳
- 不做“既…又…”的骑墙发言
- 反驳完必须说明“如果我的反驳成立，这意味着什么”
EOF_PERSONA
```

`确认角色文件创建完成：`

```
ls ～/hermes-lab/brainstorm-room/personas/
```

``应该能看到 6 个 `.md` 文件。``

---

## `Step 4：准备一个示例输入`

`先创建一个示例观点文件，用来测试：`

```
cat > ～/hermes-lab/brainstorm-room/inputs/sample-ai-memory.md << 'EOF_INPUT'
# 观点：Agent 应该有“遗忘机制”

当前所有 Agent Memory 系统都在追求“记住更多”，但人类记忆的优势恰恰在于遗忘。

具体想法：
1. Agent 的 Memory 应该有 TTL（过期时间），不是所有信息都永久保存
2. 长期不用的记忆应该被降权，不是删除，而是降低召回优先级
3. 每次写入 Memory 前应该有一个“值得记住吗”的判断环节
4. 可以参考人类睡眠中的记忆巩固机制：白天写入短期记忆，晚上做一次“蒸馏 + 淘汰”

我觉得这个方向值得做，因为当前 Agent 的 Memory 越来越臃肿，导致召回质量下降、Token 浪费、上下文污染。
EOF_INPUT
```

---

## `Step 5：手动跑一次验证`

``进入 `brainstorm-room`：``

```
hermes -p brainstorm-room chat
```

`输入启动 Prompt：`

```
请启动“头脑风暴聊天室”。

输入材料：
- 文件：～/hermes-lab/brainstorm-room/inputs/sample-ai-memory.md

请先读取 ～/hermes-lab/brainstorm-room/personas/ 目录下的所有角色文件，然后调用以下角色参与讨论：

1. 系统架构师
2. 产品怀疑者
3. 投资人视角
4. 哲学观察者
5. 写作编辑
6. 反方辩手

讨论规则：
- 第一轮：每个角色用 150 字以内说出自己看到的核心价值
- 第二轮：每个角色指出一个最大问题
- 第三轮：角色之间互相回应
- 第四轮：你作为主持人总结

请按 SOUL.md 的要求输出主持人总结。
最后把完整总结保存到 ～/hermes-lab/brainstorm-room/outputs/，文件名使用 topic-YYYY-MM-DD.md，不要覆盖历史文件。
```

`正常情况下你会看到多轮输出，最后是主持人的结构化总结。`

`如果讨论正常完成，说明头脑风暴聊天室已经跑通。`

`如果 Agent 没有读取角色文件，可能是当前会话没有文件读取工具，或者工具权限没有开启。可以改为直接把所有角色的核心观点写在 Prompt 里：`

```
请启动“头脑风暴聊天室”，讨论材料在 ～/hermes-lab/brainstorm-room/inputs/sample-ai-memory.md。

角色定义在 ～/hermes-lab/brainstorm-room/personas/ 目录下，请先读取所有 .md 文件。

如果无法读取目录，则使用以下简化角色：
- 系统架构师：关注输入/输出/模块/边界
- 产品怀疑者：检查是否有真实用户价值
- 投资人视角：看长期壁垒和变现路径
- 哲学观察者：找底层假设和认知盲点
- 写作编辑：看信息密度和读者视角
- 反方辩手：找最致命的反例

讨论规则：
- 第一轮：每个角色 150 字说核心价值
- 第二轮：每个角色指出一个最大问题
- 第三轮：互相回应
- 第四轮：主持人总结

总结必须包含：真正价值、最大不确定性、对我的启发、是否值得写文章、是否沉淀进 Memory/Skill/Eval。
最后保存到 ～/hermes-lab/brainstorm-room/outputs/topic-YYYY-MM-DD.md，不要覆盖历史文件。
```

`验证输出文件：`

```
ls -lh ～/hermes-lab/brainstorm-room/outputs/
latest_output="$(ls -t ～/hermes-lab/brainstorm-room/outputs/topic-*.md 2>/dev/null | head -n 1)"
cat "$latest_output"
```

---

## `Step 6：记忆策略`

`不要把整场聊天室写进 Memory。`

| 内容 | 保存位置 | 保存方式 |
| --- | --- | --- |
| 完整讨论记录 | sessions/（自动保存）或 `outputs/` | 不需要写进长期记忆 |
| 结构化总结 | `outputs/` | 让 Agent 保存 |
| 知识卡片 | `memory-cards/` | 让 Agent 生成 |
| 稳定长期结论 | Memory | 只写 1-3 条，用自然语言让 Agent 保存 |

`长期记忆只写真正稳定、长期、与你的目标相关的结论。`

``不要写成 `/memory add ...`，因为这不是所有版本都支持的通用 Slash Command。更稳的方式是在 Hermes 对话里直接说：``

```
请把下面这条作为长期记忆候选保存。如果你认为它不够稳定，请先指出原因，不要强行写入。

用户关注“Agent 遗忘机制”方向。他认为关键不在于收藏更多信息，而在于让信息经过评估、蒸馏、淘汰，只保留高价值记忆。这与他之前在 Memory 架构文章中提出的“四层记忆模型”一脉相承。
```

`不写的例子：`

-   `某个角色说了什么具体观点（这只是讨论过程）`
    
-   `还没确定的结论（需要多几轮讨论才能沉淀）`
    
-   `与你的目标无关的理论推导`
    

---

## `这个例子的价值`

`它体现的是“更好输入”和“更好吸收”。不是让 AI 总结文章，而是让文章进入一个多视角讨论场。`

`一句话：`

> `理解不是一次摘要完成的，而是在多轮冲突、追问和重组中发生的。`

---

# `十九、最终满配清单`

## `必配`

| 模块 | 推荐动作 |
| --- | --- |
| Hermes 本体 | 安装并通过 doctor |
| Model Provider | 先选一个稳定 Provider |
| SOUL.md | 写长期工作协议 |
| AGENTS.md / CLAUDE.md | 写项目上下文 |
| Memory | 先用原生 Memory |
| Skills | 至少写 1 个自己的 Skill |
| Tools | 只开基础、安全工具 |
| Sessions | 学会 list / browse / export |
| Compression | 开启上下文压缩 |

## `强烈推荐`

| 模块 | 推荐动作 |
| --- | --- |
| MCP | 先接 GitHub 或 Filesystem |
| Cron | 每日/每周只读任务 |
| Gateway | Telegram 或飞书 |
| Profiles | 至少 3 个场景隔离 |
| Markdown 报告 | 每日/每周输出 |
| Skill 清理 | 每周一次 |

## `进阶再上`

| 模块 | 适合时机 |
| --- | --- |
| Langfuse / Tokscale | Token 和成本需要精细监控 |
| Voice Mode | 想移动端语音交互 |
| 多 Agent 团队 | 单 Profile 流程稳定后 |
| 社区技能包 | 知道自己需要什么后 |
| 上下文第三方引擎 | 内置压缩不够后 |
| Browser / Database MCP | 明确权限边界后 |

## `暂时不建议新手碰`

| 模块 | 原因 |
| --- | --- |
| 全量 MCP | Token 和权限风险都高 |
| 大量社区 Skill | 质量参差、容易冲突 |
| 自动安装/删除工具 | 风险高 |
| 自动提交代码/部署 | 需要严格审批 |
| 把公司数据接入 Gateway | 隐私和合规风险 |
| 不明来源一键脚本 | 不知道打开了什么权限 |

---

# `二十、总结：满配之后，Hermes 变成了什么？`

`如果只把 Hermes 当成一个终端聊天工具，它的价值其实有限。`

`但如果把它按照本文的方式配置，它会逐渐变成几个东西。`

## `1. 一个 AI 工具雷达`

`它每天帮你观察外部世界：新工具、新 MCP、新 Skill、新记忆系统、新 Agent 框架。但它不会看到就装，而是先判断是否和长期目标相关、是否和已有工具重复、是否值得安装、是否只值得吸收思想、是否应该提醒清理旧工具。`

## `2. 一个思想聊天室`

`你可以把文章、观点、论文、灵感丢进去，让不同思维角色讨论。它帮你理解观点、发现漏洞、找到写作角度、生成知识卡片、判断是否沉淀为 Skill / Memory / Eval。`

## `3. 一个 24h 轻量 Agent 团队`

`它不是全天烧 Token，而是在合适时间主动出现：Commander 接收入口，Radar 每天搜集信息，Cleaner 每周清理系统，Brainstorm 处理文章，Writing 辅助文章生产。`

`这才是我理解的 Hermes 满配。`

`不是"我装了 100 个插件"。`

`而是：`

> `**我让 Hermes 有了更好的输入、更稳的记忆、更清晰的技能、更受控的工具、更主动的自动化、更低的 Token 成本，以及更可观察的运行状态。**`

`最终，它不只是一个 Agent，而是一个逐渐"装下你自己"的系统。`

---

# `附录：本文命令使用说明`

`本文命令基于当前 Hermes 官方文档和作者本地实践整理。由于 Hermes 仍在快速迭代，不同版本的 CLI、配置字段、插件命令可能有差异。遇到报错时，请优先执行：`

```
hermes --help
hermes doctor
hermes config show
hermes  --help
```

`第三方插件、社区技能包、一键脚本和生态工具请以对应项目最新 README 为准，不建议在含有隐私数据或生产权限的环境中直接运行。`

`这篇文章内容比较多，建议先收藏，跟着自己的节奏慢慢上手。水平一般，能力有限。细节上如果有纰漏，希望谅解——遇到不确定的地方，可以结合 Hermes 官方文档和 AI 工具搞清楚，也欢迎私信沟通。`

  

---

内容效果不满意？[点此反馈](https://feedback.notebooksyncer.com/feedback/9b72b013_1781169511127?u=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzU0NDE2NDU1NA%3D%3D%26mid%3D2247483830%26idx%3D1%26sn%3Dbb97e6813ef592ae0ba8e46ebedde4e5%26chksm%3Dfaffae2b170958bda694a35d4c977889d6b52eb9be0c03e8401637ab9b80eb22448f939e3c1a%26mpshare%3D1%26scene%3D1%26srcid%3D0611Z0I0NXVlCJTOXhAXVVeY%26sharer_shareinfo%3Da225c3d9442d332bacf64c57b7a1305a%26sharer_shareinfo_first%3Da225c3d9442d332bacf64c57b7a1305a%23rd&s=obsidian)