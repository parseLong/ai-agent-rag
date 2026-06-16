---
title: Skills 技能
tags:
  - 概念
  - 方法
  - Agent交互
---

# Skills 技能

> 来源：罗福莉（小米）访谈

## 定义

Skills是Agent框架中定义的一套执行规范，是人类经验沉淀为可执行指令的重要方式。

> "Skills它定义了一套执行规范。这套执行规范很难在预训练的数据里边具备，因为预训练数据里边没有这种internal（内部的）信息。这些信息通常是大量企业内部自己去沉淀和积累的，由人和人之间产生的一些组织上遗留下来的规范。"
> — 罗福莉

## 核心价值

### 1. 信息补充

> "预训练大部分依赖的知识，还是你在互联网上可访问到的知识。但很多智能，我们是在互联网上访问不到的。"
> — 罗福莉

- 预训练数据中缺乏的internal信息（如企业内部规范）
- 来自真实业务场景的know-how

### 2. 交互方式创新

> "它其实提供了一种交互的方式，让人去主动贡献数据，贡献让模型执行任务成功率更高的方式。"
> — 罗福莉

### 3. 阿尔法来源

> "Skills算一种。它确实提供了一种另类信息，是另类信息——跟当前的Agent共创的话，那么Agent或者说最顶尖模型的能力也很难发挥出来。"
> — 罗福莉

- Skills是一种另类信息，是企业独特竞争力的数字化

### 4. 自进化能力

> "大量Skills其实是Agent自己写的。"
> — 罗福莉

- Skills可以由Agent自己编写，形成自迭代

## 与预训练的关系

| 数据来源 | 类型 | 说明 |
|----------|------|------|
| 预训练 | 互联网知识 | 公开可访问 |
| Skills | 内部知识 | 企业独特竞争力 |

> "预训练大部分依赖的知识，还是你在互联网上可访问到的知识。但很多智能，我们是在互联网上访问不到的。 
> 
> 这个时候就以另外一种形态出现，Skills算一种。"
> — 罗福莉

## 实际应用

### 在OpenClaw框架中

> "我跟他说了过后，它最后能形成一套非常体系化的东西，并且变成一套Skills。它现在至少在这个事情上，变成了我的数字分身。"
> — 罗福莉

- Skills是由Agent自己写的
- 形成了一套可以不断迭代的规范体系
- 可以变成个人的"数字分身"

### 执行规范的例子

- 业务逻辑是企业内部沉淀的组织规范
- 人的经验通过多轮交互教给Agent
- Agent学会后形成可复用的执行标准

## Workflow 范式演变中的 Skills 角色

> 来源：[[Agent核心技术概念与范式发生了哪些演变以及背后的思考]]

随着模型能力的跃升和 Skills 体系的出现，Workflow 的形态正在发生深刻重构——从"刚性的流程编排"转向"动态的 Skill 封装与混合架构"。Skills 在这一转变中承担了两个核心角色：

### 逻辑内聚化

原本分散在 Workflow 引擎中的步骤定义、约束条件、核心判断逻辑，现在可以直接写入 SKILL.md 的 Markdown 描述文件中。模型通过阅读 Skills 的文档，即可理解任务的完整链路——==Workflow 的逻辑从外部引擎"内聚"到 Skill 内部==。

### 执行脚本化

对于需要精确控制的环节，不再依赖外部工作流引擎的状态跳转，而是通过 Skill 关联的 Script 脚本进行代码级的编排和控制。这意味着，==一个复杂的业务流程可以被打包成一个独立的、可复用的 Skill==。

> [!important] 混合架构最佳实践：Skill为主，Workflow为辅/兜底
> 纯 Skill 驱动赋予 Agent 更高自主性，但在极端复杂或容错率极低的场景下仍可能出现理解偏差或执行跳跃。因此当前最佳实践是==Skill 为主，Workflow 为辅/兜底==：
> - **成熟标准化子任务** → 封装为 Skills（灵活+易用）
> - **关键稳定性高的主干流程** → 保留 Workflow 或将 Workflow 封装为特殊 Tool
> - ==代码可读性==是保留部分 Workflow 的一个实际考量——Workflow 里的固定运行部分可以转换为 Script，但代码可读性很多时候没有 Workflow 方便

## Script 作为 Resources 形态

> 来源：[[Agent技术范式演变（2023-2026）]]

在 Agent Skills 体系中，Script 脚本逐渐成为工具承载的主流模式，与 CLI 命令共同构成"利用模型原生能力"的工具生态：

### 核心特征

| 特征 | 说明 |
|------|------|
| **本地与远程统一** | Script 既可直接执行本地命令（文件操作、环境配置），也可内部封装对远程 API 的调用 |
| **协议黑盒化** | 复杂的 API 鉴权、参数拼接等细节被隐藏在脚本内部，Agent 只需关注"调用哪个脚本"+"传入什么核心参数" |
| **Skill 内置工具** | 安装一个 Skill 不仅包含 Prompt 指引，更内置了可执行的工具脚本——所以安装 Skill 就能赋予 Agent 处理复杂任务的能力 |

### Script 与 CLI 的互补关系

| 方面 | CLI 命令 | Script 脚本 |
|------|---------|-------------|
| **知识来源** | 模型预训练的"先天知识"（零样本） | Skill 描述文件中的"后天知识" |
| **执行方式** | 直接在终端执行 | 作为 Skill 的 Resources 按需执行 |
| **适用场景** | 标准文件操作、系统管理 | 复杂业务逻辑、API 集成 |
| **灵活性** | 高（任何标准命令） | 更高（可封装任意逻辑） |

### Skill 的完整构成

一个 Skill 不仅仅是 Prompt 指引，而是==Prompt + Script + References==的完整能力包：

```
Skill = SKILL.md（操作手册 + 触发条件 + 规则）
      + scripts/（可执行脚本 — 工具能力）
      + references/（参考资料 — 按需加载）
```

> [!tip] 核心洞察
> Skill 的本质是"人类经验的可执行封装"——不只是告诉 Agent 怎么做，更给了 Agent 能做的工具。

## 工程实践视角

### SKILL.md 标准

每个 Skill 是一个独立目录，核心是 `SKILL.md` 文件：

```
skill/
└── log-diagnosis/
    ├── SKILL.md              # YAML frontmatter(元数据) + Markdown body(操作手册)
    ├── references/           # 参考资料（按需加载）
    │   └── error-patterns.md
    ├── scripts/              # 可执行脚本
    │   └── parse_trace.py
    └── assets/               # 模板、配置等静态资源
```

SKILL.md = SOP + 工具 + 资源。LLM不是在"猜"怎么用工具，而是在"按手册操作"。

> [!example] 为什么选择 YAML frontmatter + Markdown body？
> SKILL.md 的格式选择并非偶然——它要同时服务**两个"读者"**：
> - **人**（开发者/运维）：阅读 Markdown body 理解操作规范和执行流程
> - **机器**（LLM/框架）：解析 YAML frontmatter 获取元数据（触发条件、依赖工具、MCP声明等）
> 
> 一个文件解决两个问题，避免了 JSON 配置文件与 Markdown 文档分离带来的维护负担和一致性风险。

> [!example] Skill 类的运行时表示
> Skill 在框架运行时被解析为以下核心属性：
> - `name`：技能标识符
> - `description`：技能描述（用于 Advertise 注入 system prompt）
> - `metadata`：YAML frontmatter 解析出的元数据字典
> - `instructions`：Markdown body 内容（Load 阶段返回的完整操作手册）
> - `mcp_servers`：技能声明的专属 MCP 服务配置
> - `max_rounds`：技能执行时允许的最大对话轮次

> [!tip] Frontmatter 解析实现
> SKILL.md 的解析按 `"---"` 分割为三段：
> 1. 第一段：`---` 标记开始
> 2. 第二段：YAML frontmatter 内容 → 解析为 `metadata`
> 3. 第三段：Markdown body 内容 → 作为 `instructions`
> 
> 分离逻辑确保元数据和操作手册各司其职，互不干扰。

### 渐进式加载（Progressive Disclosure）

核心灵感来自操作系统的按需加载——不把所有代码都加载到内存，需要时才加载：

| 阶段 | 动作 | Token开销 |
|------|------|----------|
| **Advertise** | 启动时扫描skill目录，注入system prompt摘要 | ~100 tokens/skill |
| **Load** | 按需返回完整SKILL.md body + 触发MCP工具注入 | +1,000-3,000 tokens |
| **Read** | 按需读取references/参考文档 | 按需 |
| **Run** | 按需执行scripts/脚本（30s超时保护） | 按需 |

> [!important] Token效率对比
> - 传统全量注入50个工具：~50,000 tokens
> - Skill Advertise：~5,000 tokens（**10x效率**）
> - Skill Load后：+1,000~3,000 tokens（仅加载用到的1-2个skill）

> [!example] Advertise 阶段实现细节
> `_discover_skills()` 扫描所有 skill 目录 → 解析每个 SKILL.md 的 frontmatter → 将 `name + description` 注入 system prompt（每个 skill 约100 tokens）。
> 
> 关键设计：`disable-model-invocation` 的 skill 只能由用户主动触发，LLM 不会自动 load——防止模型在不确定意图时随意加载技能浪费 tokens。

> [!example] Load 阶段实现细节
> Load 不仅返回完整的 SKILL.md body（操作手册），还同时触发 `inject_skill_mcp_tools`——将 skill 声明的 MCP 工具动态注入到 LLM 的 tools 列表中，确保技能加载后"即插即用"。

> [!example] Read 阶段实现细节
> 路径穿越安全检查：使用 `os.path.realpath()` 将请求路径与 skill 目录的真实路径比较，防止 LLM 被注入后通过 `../../../etc/passwd` 读取系统文件。
> 
> 贴心设计：当请求的资源不存在时，不返回冰冷报错，而是列出该 skill 下所有可用资源，引导 LLM 正确选择。

> [!example] Run 阶段实现细节
> 通过 `subprocess` 执行脚本，关键参数：
> - `timeout=30s`：防止脚本无限执行导致 Agent 卡死
> - `cwd=skill.directory`：工作目录设为 skill 自身目录，脚本中的相对路径（如读取同目录配置文件）可正常工作

### Skill与MCP联动

Skill加载时自动注入声明需要的MCP工具，实现按需加载——不是所有工具都出现在LLM的tools列表中。

> [!example] MCP 工具按需注入的实现
> `load_skill` 触发时，`inject_skill_mcp_tools` 的执行流程：
> 1. 从 skill 的 SKILL.md `tools:` 字段获取工具名列表
> 2. 从 `McpService` 获取每个工具的完整定义（schema + description）
> 3. 去重后 append 到 LLM 的 tools 列表
> 
> 这样确保只有被激活 skill 声明的 MCP 工具才会出现在工具列表中，而非全局全量暴露。

### 条件注册机制

> [!important] 工具条件注册与 Token 经济学
> 三个 scoped 原生工具的注册条件各不相同：
> 
> | 工具 | 注册条件 | 原因 |
> |------|----------|------|
> | `run_skill_script` | 至少有一个 skill 包含 `scripts/` 子目录 | 否则工具定义就是纯浪费 |
> | `read_skill_resource` | 至少有一个 skill 包含 `resources/` 目录 | 否则工具定义就是纯浪费 |
> | `load_skill` | 始终注册（只要有 skill） | 加载能力是基础功能，必须可用 |
> 
> 为什么条件注册：每个注册的工具出现在 LLM 的 tools 列表中约占 **200 tokens**。没有对应 skill 时，这些工具定义既不会被调用，又白白消耗 token 额度——是纯粹的噪声。

### 工具分发路由优先级

> [!important] 三级路由：scoped > skill专属MCP > 全局MCP
> 当 LLM 调用一个工具名时，路由按以下优先级匹配：
> 
> 1. **scoped 原生工具**（`run_script` / `read_resource`）：优先级最高
> 2. **skill 专属 MCP 工具**（SKILL.md `mcp_servers:` 声明）：次高
> 3. **全局 MCP 工具**（兜底）
> 
> 为什么 scoped 最高：skill 自带的脚本和资源不依赖外部服务，执行最快最可靠。全局 MCP 可能存在同名工具，但 scoped 工具是 skill 作者为特定场景精心定制的，语义更精准。

### 四源工具组装（get_all_tools_for_skill）

> [!example] 工具的四个来源
> `get_all_tools_for_skill` 为已加载的 skill 组装完整工具列表，来自四个独立来源：
> 
> | 来源 | 工具 | 条件 | 说明 |
> |------|------|------|------|
> | 来源1 | `run_script` | skill 有 `scripts/` 子目录时提供 | description 中写入可用脚本列表，引导 LLM 选择正确脚本 |
> | 来源2 | `read_resource` | skill 有 `resources/` 目录时提供 | 让 LLM 按需读取 skill 内参考资料 |
> | 来源3 | 全局 MCP 工具 | SKILL.md `tools:` 字段引用 | skill 声明需要但自身不提供的通用工具 |
> | 来源4 | skill 专属 MCP 工具 | SKILL.md `mcp_servers:` 字段 | skill 自带的外部服务连接，独立于全局 MCP 配置 |
> 
> 四源组装确保 skill 的工具生态完整：自有能力（来源1-2）+ 外部依赖（来源3-4），按条件裁剪避免冗余。

### 安全设计

- 路径穿越检查：`os.path.realpath` 比较防止LLM被注入后通过`../../../etc/passwd`读取系统文件
- 条件注册：只有当存在含脚本/资源的skill时才注册对应工具，避免无用选项占用tokens（详见[[#条件注册机制]]）

> [!tip] 安全设计的双重收益
> 路径穿越检查和条件注册都服务于同一个目标：**减少 LLM 的决策噪声**。前者防止恶意路径注入导致误操作，后者防止无关工具定义干扰模型判断。两者都是"最小暴露面"原则的工程实践。

## 2026年行业标准化进展

> 来源：[[2026过半：一万字，把这半年AI发生的事讲明白]]

整个上半年最被低估、又最影响一线工作流的事，是 Agent Skills 从一家厂的功能到行业标准的跨越。

| 时间 | 事件 |
|------|------|
| 2025年10月 | Anthropic 推出 Agent Skills |
| 2025年12月 | Anthropic 把 Skills 做成开放标准 |
| 2026年上半年 | OpenAI、谷歌以及国内 AI 厂商全跟上 |

### Skills 解决的真正问题

不只是 Prompt 长短，而是**个人知识的资产化**：

- 一个公司里最值钱的从来不是 SOP 文档，而是只有几个老员工才知道的"这个表必须按这个口径填"
- 过去这种东西要么靠人传人，要么写成员工手册然后没人看
- Skills 第一次让"个人或团队的方法论"具备了被分发、被复用、被版本化管理的形态

### Skills 是"教 AI"的最干净载体

时间走到2026年6月，再说"学会怎么问 AI"已经过时了。该学的是**怎么教 AI**，而 Skills 是这件事最干净的载体——"加它不亏，用它管用"的体验，是 Prompt 工程时代不可能有的。

### Skills 标准化对 Desktop Agent 的意义

Desktop Agent 跟 Coding Agent 的最大区别是面对的工具种类多到爆炸。每一个应用都是一种隐性 SOP，没法写死在模型里。Skills 给了一个把隐性知识沉淀下来的载体。Anthropic 把 Agent Skills 做成开放标准之后，Desktop Agent 这件事的拼图就齐了。

详见 → [[Desktop Agent（AI入住电脑）]]

## 关联概念

- [[OpenClaw框架]]
- [[智能体]]
- [[知识图谱]]
- [[大语言模型]]
- [[Post-train后训练]]
- [[记忆到技能转化]] — 记忆→规则→技能的三级递进学习框架
- [[知识沉淀护城河]] — Skills是"另类信息"，知识沉淀护城河是其工程化实践
- [[知识分层架构]] — Skills对应知识类型中的guideline/pitfall，可执行封装是L3(形成技能)
- [[团队知识库共建]] — Skills的团队共享与知识库共建是同一命题的两面
- [[多智能体协作模式]] — Skill advertise本质是一种动态路由模式
- [[Agent技术范式演变（2023-2026）]] — Skills在Workflow和Tools演进中的角色
- [[Desktop Agent（AI入住电脑）]] — Skills标准化是Desktop Agent拼图齐了的关键
- [[CLI回归（AI时代统一接口）]] — Skills在CLI中的加载与执行
- [[Coding Plan（AI编程订阅制）]] — Skills开发降低了编程成本焦虑

## Skill 的成本维度

> 来源：[[一篇搞懂 AI Coding Agent 的 Token 成本控制]]

每个 Skill 都带着 Description、Instructions、Examples、触发逻辑。这些信息如果常驻，就会进上下文。问题不是"装不装"，而是**哪些常驻，哪些按需加载**：

| 类型 | 策略 | 说明 |
|------|------|------|
| 高频、稳定、通用 | **常驻** | 类似后端服务的常驻能力，要尽量少而精 |
| 低频、长说明、低复用 | **按需触发** | 需要时才加载，不每轮都占上下文空间 |

> [!tip] Skill 绑定模型
> 以 CodeBuddy 为例，SKILL.md 头部可以声明 `model: deepseek-v4-pro` + `context: fork`——让低复杂度 Skill 自动走便宜模型，也不必把主会话的全部历史都带进去。详见 [[模型路由]]。

相关笔记：[[AI Coding Agent Token 成本控制]]、[[工具调用回路]]、[[模型路由]]
