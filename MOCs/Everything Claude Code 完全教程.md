# Everything Claude Code (ECC) 完全教程

> **版本：** 2.0.0-rc.1 | **作者：** Affaan Mustafa | **许可证：** MIT
>
> Anthropic 黑客松获奖作品 — AI Agent 编码助手的性能优化系统

---

## 目录

- [第一章：项目概览](#第一章项目概览)
- [第二章：核心设计理念](#第二章核心设计理念)
- [第三章：项目结构详解](#第三章项目结构详解)
- [第四章：Agent 系统](#第四章agent-系统)
- [第五章：Skills 技能体系](#第五章skills-技能体系)
- [第六章：Commands 命令系统](#第六章commands-命令系统)
- [第七章：Hooks 自动化钩子](#第七章hooks-自动化钩子)
- [第八章：Rules 规则引擎](#第八章rules-规则引擎)
- [第九章：MCP 服务器集成](#第九章mcp-服务器集成)
- [第十章：安装与配置](#第十章安装与配置)
- [第十一章：开发工作流](#第十一章开发工作流)
- [第十二章：安全体系](#第十二章安全体系)
- [第十三章：持续学习与进化](#第十三章持续学习与进化)
- [第十四章：跨平台与多 Harness 支持](#第十四章跨平台与多-harness-支持)
- [第十五章：ECC 2.0 控制面与 Dashboard](#第十五章ecc-20-控制面与-dashboard)
- [第十六章：贡献指南](#第十六章贡献指南)
- [附录：速查表与资源](#附录速查表与资源)

---

## 第一章：项目概览

### 1.1 ECC 是什么

Everything Claude Code (ECC) 是一个为 AI 编码助手量身打造的**性能优化系统**，而不仅仅是一个配置包。它提供了完整的 Agent 编排、技能知识、自动化钩子、安全防护和持续学习能力。

用一句话概括：

> **不只是配置文件，而是一套完整的系统：技能、本能、内存优化、持续学习、安全扫描和研究优先开发。**

### 核心文档索引

| 文档 | 说明 |
|------|------|
| [[ECC 核心配置（CLAUDE.md）]] | 项目核心指导文件 |
| [[核心 Agent 系统]] | 60+ 专用子 Agent 详解 |
| [[关键 Skills 精选]] | 核心 Skill 汇总（编码标准、TDD、安全） |
| [[Hooks 自动化钩子系统]] | 自动化钩子详解 |
| [[核心规则体系（Rules）]] | 编码规范和安全规则 |
| [[MCP 服务器集成]] | MCP 配置和使用指南 |

### 1.2 核心数据

| 指标 | 数值 |
|------|------|
| 专用 Agent | 60 个 |
| 工作流 Skills | 229 个 |
| 斜杠命令 | 75 个 |
| 规则文件 | 110+ 个 |
| 语言生态 | 12+ |
| 测试用例 | 997+ |
| GitHub Stars | 140K+ |
| 社区贡献者 | 170+ |

### 1.3 支持的 AI 编码平台

ECC 不仅仅服务于 Claude Code，它支持多个主流 AI 编码平台：

- **Claude Code** — Anthropic 官方 CLI
- **Codex** — OpenAI 编码助手
- **Cursor** — AI 代码编辑器
- **OpenCode** — 开源编码平台
- **Gemini** — Google AI 编码工具
- **GitHub Copilot** — GitHub AI 助手

### 1.4 版本演进

| 版本 | 时间 | 核心特性 |
|------|------|---------|
| v1.2.0 | 2026.02 | Python/Django 支持，Session 管理，持续学习 v2 |
| v1.3.0 | 2026.02 | OpenCode 插件支持 |
| v1.4.0 | 2026.02 | 交互式安装向导，PM2 编排，多语言规则 |
| v1.6.0 | 2026.02 | Codex CLI，AgentShield 安全审计，GitHub Marketplace |
| v1.7.0 | 2026.02 | Codex app 支持，演示文稿构建器 |
| v1.8.0 | 2026.03 | Harness 性能系统，Hook 可靠性重构 |
| v1.9.0 | 2026.03 | 选择性安装，6 个新 Agent，10+ 语言生态 |
| v2.0.0-rc.1 | 2026.04 | Dashboard GUI，Hermes 运维工作流，ECC 2.0 Alpha |

---

## 第二章：核心设计理念

ECC 的设计围绕五大核心原则，贯穿所有模块：

### 2.1 Agent-First（Agent 优先）

> 遇到领域任务，优先委派给专用 Agent，而非通用处理。

ECC 拥有 60 个专用 Agent，每个都专注于特定领域（代码审查、安全检测、构建修复等）。这就像一家公司有不同职能的专家，而非让一个人做所有事。

### 2.2 Test-Driven（测试驱动）

> 先写测试，再写实现，最低覆盖率 80%。

TDD 工作流三步法：
1. **RED** — 写一个失败的测试
2. **GREEN** — 写最少的代码让测试通过
3. **IMPROVE** — 重构并确认覆盖率 ≥ 80%

### 2.3 Security-First（安全优先）

> 永远不在安全上妥协，验证所有输入。

任何提交前必须通过安全检查：无硬编码密钥、SQL 注入防护、XSS 防护、CSRF 保护等。

### 2.4 Immutability（不可变性）

> 始终创建新对象，永远不修改已有对象。

这是 ECC 最严格的编码规范。所有更新操作返回新副本，而非原地修改。

### 2.5 Plan Before Execute（先计划后执行）

> 复杂功能先规划，再动手实现。

复杂功能 → 使用 `planner` Agent → 识别依赖和风险 → 分阶段执行。

---

## 第三章：项目结构详解

```
everything-claude-code/
│
├── agents/              # 60 个专用子 Agent
├── skills/              # 229 个工作流技能和领域知识
├── commands/            # 75 个斜杠命令（兼容层，优先使用 skills）
├── hooks/               # 触发式自动化钩子
│   ├── hooks.json           # 所有钩子配置
│   └── memory-persistence/  # 会话生命周期钩子
│
├── rules/               # 必须遵循的规则（通用 + 语言专属）
│   ├── common/              # 语言无关原则
│   ├── typescript/          # TypeScript 专属
│   ├── python/              # Python 专属
│   ├── golang/              # Go 专属
│   ├── java/                # Java 专属
│   ├── kotlin/              # Kotlin 专属
│   ├── rust/                # Rust 专属
│   ├── cpp/                 # C++ 专属
│   ├── swift/               # Swift 专属
│   ├── php/                 # PHP 专属
│   ├── arkts/               # HarmonyOS/ArkTS 专属
│   └── zh/                  # 中文翻译
│
├── scripts/             # 跨平台 Node.js 工具脚本
│   ├── lib/                 # 共享工具库
│   └── hooks/               # 钩子实现
│
├── mcp-configs/         # MCP 服务器配置（14+ 服务）
├── tests/               # 测试套件
├── contexts/            # 动态系统提示注入上下文
├── examples/            # 示例配置和会话
├── schemas/             # JSON Schema 验证
├── docs/                # 项目文档
├── ecc2/                # ECC 2.0 Rust 控制面（Alpha）
├── ecc_dashboard.py     # 桌面 GUI 仪表盘
└── .claude-plugin/      # 插件和 Marketplace 清单
```

### 关键目录说明

| 目录 | 作用 | 格式 |
|------|------|------|
| `agents/` | 专用子 Agent 定义 | Markdown + YAML frontmatter |
| `skills/` | 工作流技能和领域知识 | `skills/<name>/SKILL.md` |
| `commands/` | 斜杠命令入口 | Markdown + frontmatter |
| `hooks/` | 自动化钩子配置 | JSON（matcher + hooks 数组） |
| `rules/` | 必须遵循的编码规则 | Markdown |
| `scripts/` | 跨平台 Node.js 工具 | JavaScript |

---

## 第四章：Agent 系统

### 4.1 什么是 Agent

Agent 是 ECC 中专注特定领域的专家。每个 Agent 用 Markdown 文件定义，包含 YAML frontmatter 描述名称、用途、可用工具和推荐模型。

### 4.2 Agent 分类

#### 核心工作流 Agent

| Agent | 用途 | 何时使用 |
|-------|------|---------|
| `planner` | 实现规划 | 复杂功能、重构 |
| `architect` | 系统设计 | 架构决策 |
| `tdd-guide` | 测试驱动开发 | 新功能、Bug 修复 |
| `code-reviewer` | 代码质量审查 | 写完/修改代码后 |
| `security-reviewer` | 漏洞检测 | 提交前、敏感代码 |
| `build-error-resolver` | 构建/类型错误修复 | 构建失败时 |
| `e2e-runner` | Playwright E2E 测试 | 关键用户流程 |
| `refactor-cleaner` | 死代码清理 | 代码维护 |
| `doc-updater` | 文档更新 | 更新文档时 |
| `loop-operator` | 自主循环执行 | 安全运行循环、监控停滞 |
| `harness-optimizer` | Harness 配置调优 | 可靠性、成本、吞吐量 |

#### 语言专属 Agent

| 语言 | 代码审查 | 构建修复 |
|------|---------|---------|
| TypeScript/JS | `typescript-reviewer` | `build-error-resolver` |
| Python | `python-reviewer` | — |
| Go | `go-reviewer` | `go-build-resolver` |
| Java/Spring | `java-reviewer` | `java-build-resolver` |
| Kotlin/Android | `kotlin-reviewer` | `kotlin-build-resolver` |
| Rust | `rust-reviewer` | `rust-build-resolver` |
| C/C++ | `cpp-reviewer` | `cpp-build-resolver` |
| Django | `django-reviewer` | `django-build-resolver` |
| F# | `fsharp-reviewer` | — |
| PyTorch | — | `pytorch-build-resolver` |

#### 基础设施与安全 Agent

| Agent | 用途 |
|-------|------|
| `database-reviewer` | PostgreSQL/Supabase 专家 |
| `network-architect` | 网络架构设计 |
| `network-troubleshooter` | 网络故障排查 |
| `mle-reviewer` | 生产 ML 流水线审查 |
| `healthcare-reviewer` | 医疗 HIPAA 合规 |
| `homelab-architect` | 家庭实验室架构 |
| `performance-optimizer` | 性能优化 |

### 4.3 Agent 编排策略

ECC 的 Agent 可以**主动触发**，无需用户提示：

```
复杂功能请求 → planner
代码刚写完/修改 → code-reviewer
Bug 修复或新功能 → tdd-guide
架构决策 → architect
安全敏感代码 → security-reviewer
自主循环/循环监控 → loop-operator
```

**并行执行**：独立操作可同时启动多个 Agent。

### 4.4 Agent 文件格式

```markdown
---
name: planner
description: Implementation planning for complex features
tools: Read, Write, Edit, Bash, Glob, Grep
model: sonnet
---

# Planner Agent

## When to Use
Complex feature requests that need structured planning...

## How It Works
1. Analyze the request
2. Identify dependencies
3. Break into phases
...
```

---

## 第五章：Skills 技能体系

### 5.1 什么是 Skill

Skill 是 ECC 的核心工作流表面（canonical workflow surface）。每个 Skill 是一个知识模块，包含特定领域的工作流定义、最佳实践和实用指南。

### 5.2 Skill 的分类

ECC 的 229 个 Skills 覆盖以下领域：

#### 编码与架构

| Skill | 说明 |
|-------|------|
| `coding-standards` | 语言编码最佳实践 |
| `backend-patterns` | API、数据库、缓存模式 |
| `frontend-patterns` | React、Next.js 模式 |
| `api-design` | REST API 设计、分页、错误响应 |
| `tdd-workflow` | TDD 方法论 |
| `error-handling` | 错误处理模式 |

#### 语言专属模式

| 语言 | Skills |
|------|--------|
| Go | `golang-patterns`, `golang-testing` |
| Python | `python-patterns`, `python-testing` |
| Java | `springboot-*`, `quarkus-*`, `java-coding-standards` |
| Kotlin | `kotlin-patterns`, `kotlin-testing`, `kotlin-coroutines-flows` |
| Rust | `rust-patterns`, `rust-testing` |
| C++ | `cpp-coding-standards`, `cpp-testing` |
| Swift | `swiftui-patterns`, `swift-actor-persistence` |
| Django | `django-patterns`, `django-security`, `django-tdd` |
| Laravel | `laravel-patterns`, `laravel-security`, `laravel-tdd` |

#### DevOps 与部署

| Skill | 说明 |
|-------|------|
| `deployment-patterns` | CI/CD、Docker、健康检查、回滚 |
| `docker-patterns` | Docker Compose、网络、卷、容器安全 |
| `database-migrations` | Prisma、Drizzle、Django、Go 迁移模式 |
| `e2e-testing` | Playwright E2E 和 Page Object Model |

#### AI 与 ML

| Skill | 说明 |
|-------|------|
| `mle-workflow` | 生产 ML 数据契约、评估、部署、监控 |
| `pytorch-patterns` | PyTorch 深度学习工作流 |
| `cost-aware-llm-pipeline` | LLM 成本优化、模型路由、预算追踪 |
| `continuous-learning` | 会话模式提取 |
| `continuous-learning-v2` | 基于本能的学习系统 |

#### 商业与运营

| Skill | 说明 |
|-------|------|
| `market-research` | 市场与竞品研究 |
| `investor-materials` | 融资材料 |
| `article-writing` | 长文写作 |
| `content-engine` | 多平台内容运营 |
| `brand-voice` | 品牌写作风格系统 |

#### 网络与安全

| Skill | 说明 |
|-------|------|
| `security-review` | 安全检查清单 |
| `security-scan` | AgentShield 安全审计集成 |
| `cisco-ios-patterns` | Cisco IOS 配置模式 |
| `network-bgp-diagnostics` | BGP 网络诊断 |

### 5.3 Skill 文件结构

```
skills/
└── your-skill-name/
    └── SKILL.md
```

SKILL.md 模板：

```markdown
---
name: your-skill-name
description: 简短描述，用于技能列表和自动激活
origin: ECC    # ECC 为官方，community 为社区
---

# Skill 标题

## When to Use
何时使用此技能...

## How It Works
工作原理...

## Examples
示例...
```

### 5.4 Skill vs Commands

> **`skills/` 是首要工作流表面。`commands/` 是兼容层，仅在有迁移需求时才更新。**

新贡献应首先落地在 `skills/` 中。`commands/` 是旧版斜杠入口兼容表面，长期方向是 Skills 优先。

---

## 第六章：Commands 命令系统

### 6.1 命令概览

ECC 提供 75 个斜杠命令，作为快捷入口调用底层技能和工作流。

### 6.2 核心命令速查

| 命令 | 用途 | 对应 Skill |
|------|------|-----------|
| `/plan` | 实现规划 | — |
| `/code-review` | 代码质量审查 | — |
| `/build-fix` | 修复构建错误 | — |
| `/refactor-clean` | 死代码清理 | — |
| `/quality-gate` | 验证关卡 | — |
| `/learn` | 提取会话模式 | `continuous-learning` |
| `/learn-eval` | 提取、评估并保存模式 | `continuous-learning-v2` |
| `/skill-create` | 从 Git 历史生成技能 | — |
| `/checkpoint` | 保存验证状态 | — |
| `/setup-pm` | 配置包管理器 | — |
| `/security-scan` | 安全扫描 | `security-scan` |

### 6.3 语言专属命令

| 命令 | 语言 |
|------|------|
| `/go-review`, `/go-test`, `/go-build` | Go |
| `/python-review` | Python |
| `/kotlin-review`, `/kotlin-test`, `/kotlin-build` | Kotlin |
| `/rust-review`, `/rust-test`, `/rust-build` | Rust |
| `/cpp-review`, `/cpp-test`, `/cpp-build` | C++ |
| `/fastapi-review` | FastAPI |

### 6.4 多 Agent 编排命令

| 命令 | 用途 |
|------|------|
| `/multi-plan` | 多 Agent 任务分解 |
| `/multi-execute` | 多 Agent 协作执行 |
| `/multi-backend` | 后端多服务编排 |
| `/multi-frontend` | 前端多服务编排 |
| `/multi-workflow` | 通用多服务工作流 |

> ⚠️ 多 Agent 命令需要额外安装 `ccg-workflow` 运行时。

### 6.5 会话与学习命令

| 命令 | 用途 |
|------|------|
| `/sessions` | 会话历史管理 |
| `/save-session` | 保存会话 |
| `/resume-session` | 恢复会话 |
| `/instinct-status` | 查看学习本能 |
| `/instinct-import` | 导入本能 |
| `/instinct-export` | 导出本能 |
| `/evolve` | 聚类本能为技能 |
| `/prune` | 删除过期本能 |

### 6.6 Harness 命令

| 命令 | 用途 |
|------|------|
| `/harness-audit` | 审计 Harness 配置 |
| `/loop-start` | 启动自主循环 |
| `/loop-status` | 循环状态 |
| `/model-route` | 模型路由 |
| `/pm2` | PM2 服务管理 |

---

## 第七章：Hooks 自动化钩子

### 7.1 钩子系统架构

ECC 的 Hooks 是基于匹配器（matcher）的触发式自动化，在 Claude Code 的工具生命周期中自动执行。所有钩子用 Node.js 编写，确保跨平台兼容。

### 7.2 钩子生命周期阶段

```
SessionStart → PreToolUse → [工具执行] → PostToolUse / PostToolUseFailure → Stop → SessionEnd
              ↑ PreCompact（压缩前保存状态）
```

### 7.3 预置钩子详解

#### SessionStart 阶段

| 钩子 ID | 功能 |
|----------|------|
| `session:start` | 加载上次上下文，检测包管理器 |

#### PreToolUse 阶段

| 钩子 ID | 匹配器 | 功能 |
|----------|--------|------|
| `pre:bash:dispatcher` | Bash | Bash 预检：质量检查、tmux、push、GateGuard |
| `pre:write:doc-file-warning` | Write | 警告非标准文档文件 |
| `pre:edit-write:suggest-compact` | Edit\|Write | 建议在逻辑间隔手动压缩 |
| `pre:observe:continuous-learning` | * | 捕获工具使用观察（持续学习） |
| `pre:governance-capture` | Bash\|Write\|Edit\|MultiEdit | 捕获治理事件（密钥、违规） |
| `pre:config-protection` | Write\|Edit\|MultiEdit | 阻止修改 linter/formatter 配置 |
| `pre:mcp-health-check` | * | 检查 MCP 服务器健康状态 |
| `pre:edit-write:gateguard-fact-force` | Edit\|Write\|MultiEdit | 事实强制门：首次编辑前强制调查 |

#### PostToolUse 阶段

| 钩子 ID | 匹配器 | 功能 |
|----------|--------|------|
| `post:bash:dispatcher` | Bash | Bash 后处理：日志、PR、构建通知 |
| `post:quality-gate` | Edit\|Write\|MultiEdit | 编辑后质量门检查 |
| `post:edit:design-quality-check` | Edit\|Write\|MultiEdit | 前端设计质量检查 |
| `post:edit:accumulator` | Edit\|Write\|MultiEdit | 记录编辑的文件路径（批量格式化） |
| `post:edit:console-warn` | Edit | 警告 console.log 语句 |
| `post:governance-capture` | Bash\|Write\|Edit\|MultiEdit | 捕获工具输出中的治理事件 |
| `post:session-activity-tracker` | * | 追踪每会话工具调用和文件活动 |
| `post:observe:continuous-learning` | * | 捕获工具使用结果（持续学习） |
| `post:ecc-metrics-bridge` | * | 维护会话指标聚合 |
| `post:ecc-context-monitor` | * | 上下文耗尽、高成本、范围蔓延警告 |

#### Stop 阶段

| 钩子 ID | 功能 |
|----------|------|
| `stop:format-typecheck` | 批量格式化（Biome/Prettier）+ 类型检查（tsc） |
| `stop:check-console-log` | 检查修改文件中的 console.log |
| `stop:session-end` | 持久化会话状态 |
| `stop:evaluate-session` | 评估会话以提取模式 |
| `stop:cost-tracker` | 追踪每会话 token 和成本 |
| `stop:desktop-notify` | 桌面通知（macOS/WSL） |

### 7.4 钩子运行时控制

```bash
# 钩子严格度配置（默认 standard）
export ECC_HOOK_PROFILE=minimal|standard|strict

# 禁用特定钩子
export ECC_DISABLED_HOOKS="pre:bash:tmux-reminder,post:edit:typecheck"

# 限制 SessionStart 上下文大小
export ECC_SESSION_START_MAX_CHARS=4000

# 完全关闭 SessionStart 附加上下文
export ECC_SESSION_START_CONTEXT=off
```

### 7.5 钩子文件格式

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "node scripts/hooks/my-hook.js",
            "timeout": 10
          }
        ],
        "description": "描述此钩子的功能",
        "id": "unique-hook-id"
      }
    ]
  }
}
```

---

## 第八章：Rules 规则引擎

### 8.1 规则体系结构

ECC 的规则采用**分层架构**：

```
rules/
├── common/          # 通用规则（所有项目适用）
│   ├── coding-style.md    # 编码风格：不可变性、文件组织
│   ├── git-workflow.md    # Git 工作流：提交格式、PR 流程
│   ├── testing.md         # 测试要求：TDD、80% 覆盖率
│   ├── performance.md     # 性能优化：模型选择、上下文管理
│   ├── patterns.md        # 设计模式、骨架项目
│   ├── hooks.md           # 钩子架构
│   ├── agents.md          # 何时委派给子 Agent
│   └── security.md        # 安全检查
│
├── typescript/      # TypeScript/JavaScript 专属
├── python/          # Python 专属
├── golang/          # Go 专属
├── java/            # Java 专属
├── kotlin/          # Kotlin/Android/KMP 专属
├── rust/            # Rust 专属
├── cpp/             # C++ 专属
├── swift/           # Swift 专属
├── php/             # PHP 专属
├── csharp/          # C# 专属
├── dart/            # Dart/Flutter 专属
├── fsharp/          # F# 专属
├── angular/         # Angular 专属
├── arkts/           # HarmonyOS/ArkTS 专属
├── ruby/            # Ruby 专属
├── web/             # Web 前端通用
└── zh/              # 中文翻译
```

### 8.2 核心规则摘要

#### 必须始终遵守

- 委派专用 Agent 处理领域任务
- 实现前先写测试，验证关键路径
- 验证输入，保持安全检查
- 优先不可变更新，而非修改共享状态
- 遵循已有仓库模式，不要凭空发明
- 保持贡献聚焦、可审查、描述清晰

#### 绝对禁止

- 在输出中包含敏感数据（API Key、Token、Secrets、绝对路径）
- 提交未测试的更改
- 绕过安全检查或验证钩子
- 没有明确理由地重复已有功能
- 不检查相关测试套件就发布代码

### 8.3 规则安装

> **重要**：Claude Code 插件无法自动分发 `rules/`。需要手动复制。

```bash
# 只复制你需要的规则
mkdir -p ~/.claude/rules/ecc
cp -R rules/common ~/.claude/rules/ecc/
cp -R rules/typescript ~/.claude/rules/ecc/   # 按需选择语言
```

---

## 第九章：MCP 服务器集成

### 9.1 什么是 MCP

MCP（Model Context Protocol）是一种标准协议，允许 AI 编码助手与外部工具和服务交互。ECC 预置了 20+ 个 MCP 服务器配置。

### 9.2 预置 MCP 服务器

#### 项目管理

| 服务器 | 功能 | 启动方式 |
|--------|------|---------|
| `jira` | Jira 问题追踪 | `uvx mcp-atlassian` |
| `github` | GitHub 操作（PR、Issue） | `npx @modelcontextprotocol/server-github` |
| `confluence` | Confluence 内容集成 | `npx confluence-mcp-server` |

#### 数据库与存储

| 服务器 | 功能 |
|--------|------|
| `supabase` | Supabase 数据库操作 |
| `clickhouse` | ClickHouse 分析查询 |
| `memory` | 跨会话持久化记忆 |
| `omega-memory` | 语义搜索 + 知识图谱记忆 |

#### 开发与部署

| 服务器 | 功能 |
|--------|------|
| `vercel` | Vercel 部署和项目 |
| `railway` | Railway 部署 |
| `filesystem` | 文件系统操作 |

#### AI 与研究

| 服务器 | 功能 |
|--------|------|
| `context7` | 实时文档查找 |
| `exa-web-search` | Web 搜索和研究 |
| `sequential-thinking` | 链式推理 |
| `fal-ai` | AI 图像/视频/音频生成 |

#### 浏览器自动化

| 服务器 | 功能 |
|--------|------|
| `playwright` | Playwright 浏览器自动化和测试 |
| `browserbase` | 云端浏览器会话 |
| `browser-use` | AI 浏览器 Agent |

#### 其他

| 服务器 | 功能 |
|--------|------|
| `firecrawl` | Web 抓取和爬虫 |
| `devfleet` | 多 Agent 编排（并行 Claude Code Agent） |
| `token-optimizer` | 95%+ 上下文压缩 |
| `evalview` | AI Agent 回归测试 |
| `magic` | Magic UI 组件 |
| `longhand` | 无损会话历史（SQLite + ChromaDB） |
| `laraplugins` | Laravel 插件发现 |

### 9.3 MCP 配置方式

将需要的 MCP 服务器配置复制到 `~/.claude.json` 的 `mcpServers` 部分：

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "你的Token"
      }
    }
  }
}
```

> ⚠️ 建议同时启用的 MCP 不超过 10 个，以保留上下文窗口空间。

---

## 第十章：安装与配置

### 10.1 前置要求

- **Claude Code CLI** v2.1.0+
- **Node.js** v18+
- 支持平台：Windows、macOS、Linux

### 10.2 三种安装方式（只选一种）

> ⚠️ **不要叠加安装方法！** 最常见的故障就是先 `/plugin install`，然后再 `install.sh --profile full`。

#### 方式一：插件安装（推荐）

```bash
# 添加 Marketplace
/plugin marketplace add https://github.com/affaan-m/everything-claude-code

# 安装插件
/plugin install ecc@ecc
```

然后手动复制规则：
```bash
mkdir -p ~/.claude/rules/ecc
cp -R rules/common ~/.claude/rules/ecc/
cp -R rules/typescript ~/.claude/rules/ecc/
```

#### 方式二：完整手动安装

```bash
git clone https://github.com/affaan-m/everything-claude-code.git
cd everything-claude-code
npm install

# Linux/macOS
./install.sh --profile full

# Windows PowerShell
.\install.ps1 --profile full

# 或用 npx
npx ecc-install --profile full
```

#### 方式三：最小化安装（无钩子）

适合只需要规则、Agent、命令和核心技能的用户：

```bash
./install.sh --profile minimal --target claude
```

### 10.3 选择性安装

按需选择组件安装：

```bash
# 查询组件
npx ecc consult "security reviews" --target claude

# 安装特定能力
npx ecc install --profile minimal --target claude --with capability:machine-learning
```

### 10.4 包管理器检测

ECC 自动检测包管理器，优先级：

1. 环境变量 `CLAUDE_PACKAGE_MANAGER`
2. 项目配置 `.claude/package-manager.json`
3. `package.json` 的 `packageManager` 字段
4. 锁文件检测（package-lock.json, yarn.lock, pnpm-lock.yaml, bun.lockb）
5. 全局配置 `~/.claude/package-manager.json`
6. 回退到第一个可用的包管理器

手动设置：
```bash
export CLAUDE_PACKAGE_MANAGER=pnpm
# 或
node scripts/setup-package-manager.js --global pnpm
# 或在 Claude Code 中使用
/setup-pm
```

### 10.5 卸载与重置

```bash
# 预览卸载
node scripts/uninstall.js --dry-run

# 执行卸载
node scripts/uninstall.js

# 使用生命周期管理器
node scripts/ecc.js list-installed
node scripts/ecc.js doctor
node scripts/ecc.js repair
node scripts/ecc.js uninstall --dry-run
```

如果叠加安装了，按以下顺序清理：
1. 移除 Claude Code 插件安装
2. 从仓库根目录运行 ECC 卸载命令
3. 删除手动复制的规则文件夹
4. 用单一方法重新安装一次

---

## 第十一章：开发工作流

### 11.1 标准 ECC 开发流程

```
1. Plan（规划）
   └── 使用 planner Agent
   └── 识别依赖和风险
   └── 分阶段执行

2. TDD（测试驱动开发）
   └── 使用 tdd-guide Agent
   └── 先写测试
   └── 实现
   └── 重构

3. Review（审查）
   └── 使用 code-reviewer Agent
   └── 立即处理 CRITICAL/HIGH 级别问题

4. Capture（捕获知识）
   └── 个人调试笔记 → 自动记忆
   └── 团队/项目知识 → 项目文档

5. Commit（提交）
   └── 常规提交格式
   └── 全面的 PR 摘要
```

### 11.2 常用工作流命令

```bash
# 规划新功能
/plan "添加用户认证功能"

# 代码审查
/code-review

# 修复构建错误
/build-fix

# 创建技能
/skill-create

# 安全扫描
/security-scan

# 会话管理
/save-session
/resume-session
```

### 11.3 Git 工作流

**提交格式**：`<type>: <description>`

类型：`feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`, `ci`

**PR 工作流**：
1. 分析完整提交历史
2. 起草全面摘要
3. 包含测试计划
4. 使用 `-u` 标志推送

### 11.4 上下文管理

ECC 建议：避免在上下文窗口的最后 20% 执行大型重构和多文件功能。

- 低敏感度任务（单次编辑、文档、简单修复）可容忍更高上下文利用率
- 使用 `strategic-compact` Skill 在逻辑间隔建议手动压缩
- 钩子 `post:ecc-context-monitor` 会自动警告上下文耗尽、高成本和范围蔓延

---

## 第十二章：安全体系

### 12.1 安全原则

ECC 的安全体系遵循**安全优先**原则，任何提交前必须通过安全检查。

### 12.2 提交前安全清单

- [ ] 无硬编码密钥（API Key、密码、Token）
- [ ] 所有用户输入已验证
- [ ] SQL 注入防护（参数化查询）
- [ ] XSS 防护（HTML 消毒）
- [ ] CSRF 保护已启用
- [ ] 认证/授权已验证
- [ ] 所有端点已限速
- [ ] 错误消息不泄露敏感数据

### 12.3 密钥管理

- **绝不硬编码密钥**，使用环境变量或密钥管理器
- 启动时验证所需密钥
- 暴露的密钥立即轮换

### 12.4 安全问题处理流程

```
发现安全问题 → 停止 → 使用 security-reviewer Agent → 修复 CRITICAL 问题 → 轮换暴露的密钥 → 审查整个代码库
```

### 12.5 AgentShield 安全审计

ECC 集成了 AgentShield 安全审计工具（1282 测试，98% 覆盖率，102 条静态分析规则）：

```bash
# 快速扫描
npx ecc-agentshield scan

# 自动修复安全项
npx ecc-agentshield scan --fix

# 深度分析（三个 Opus Agent）
npx ecc-agentshield scan --opus --stream

# 从零生成安全配置
npx ecc-agentshield init
```

**扫描范围**：
- CLAUDE.md、settings.json、MCP 配置
- 钩子、Agent 定义、技能
- 5 大类别：密钥检测（14 种模式）、权限审计、钩子注入分析、MCP 服务器风险、Agent 配置审查

**输出格式**：终端（A-F 评级）、JSON（CI 管道）、Markdown、HTML

### 12.6 钩子级安全防护

ECC 的钩子系统内置了多层安全防护：

| 钩子 | 安全功能 |
|------|---------|
| `pre:bash:dispatcher` | Bash 预检，防止危险命令 |
| `pre:config-protection` | 阻止修改 linter/formatter 配置（防止降低安全标准） |
| `pre:edit-write:gateguard-fact-force` | 首次编辑前强制调查，防止盲目修改 |
| `pre:governance-capture` | 捕获治理事件（密钥泄露、策略违规） |

---

## 第十三章：持续学习与进化

### 13.1 持续学习 v1

传统的 Stop-hook 模式提取系统。每次会话结束时自动从会话中提取可复用模式。

```bash
/learn        # 会话中提取模式
```

### 13.2 持续学习 v2（推荐）

基于**本能（Instinct）**的学习系统，带有置信度评分：

```bash
/instinct-status        # 查看已学习本能及置信度
/instinct-import <file> # 导入他人的本能
/instinct-export        # 导出你的本能（共享）
/evolve                 # 将相关本能聚类为技能
/prune                  # 删除过期的待定本能
```

### 13.3 技能创建

两种方式从仓库生成 Claude Code 技能：

#### 方式 A：本地分析（内置）

```bash
/skill-create                    # 分析当前仓库
/skill-create --instincts        # 同时生成本能
```

#### 方式 B：GitHub App（高级）

适用于 10K+ 提交、自动 PR、团队共享的场景。

### 13.4 观察者系统

钩子 `pre:observe:continuous-learning` 和 `post:observe:continuous-learning` 自动捕获工具使用行为，作为持续学习的输入。

### 13.5 会话评估

`stop:evaluate-session` 钩子在每个会话结束时评估可提取的模式，自动积累经验。

---

## 第十四章：跨平台与多 Harness 支持

### 14.1 跨平台支持

ECC 完全支持 **Windows、macOS、Linux**，所有钩子和脚本均已用 Node.js 重写以确保最大兼容性。

### 14.2 多 Harness 适配

ECC 可同时在以下平台使用，共享同一套核心：

| Harness | 适配方式 |
|---------|---------|
| Claude Code | 原生插件支持 |
| Codex | AGENTS.md 适配 + Codex CLI |
| Cursor | 配置文件适配 |
| OpenCode | 插件系统（12 Agent、24 命令、16 技能） |
| Gemini | 配置适配 |
| GitHub Copilot | Marketplace 集成 |

### 14.3 跨 Harness 架构

```
ECC 核心层（Skills + Agents + Rules + Hooks）
         │
    ┌────┼────┐────────┐──────────┐
    ▼    ▼    ▼        ▼          ▼
Claude  Codex  Cursor  OpenCode  Gemini
Code    CLI    IDE     Plugin    AI
```

---

## 第十五章：ECC 2.0 控制面与 Dashboard

### 15.1 Dashboard GUI

ECC 提供了一个基于 Tkinter 的桌面仪表盘：

```bash
npm run dashboard
# 或
python3 ./ecc_dashboard.py
```

**功能**：
- 标签页界面：Agents、Skills、Commands、Rules、Settings
- 深色/浅色主题切换
- 字体自定义（字体族和大小）
- 项目 Logo 显示在标题栏和任务栏
- 跨组件搜索和过滤

### 15.2 ECC 2.0 Alpha

`ecc2/` 目录包含 Rust 编写的控制面原型（Alpha 版本），目前支持以下命令：

```bash
cargo build --manifest-path ecc2/Cargo.toml

ecc-tui dashboard    # 仪表盘
ecc-tui start        # 启动
ecc-tui sessions     # 会话列表
ecc-tui status       # 状态
ecc-tui stop         # 停止
ecc-tui resume       # 恢复
ecc-tui daemon       # 守护进程
```

> 注意：ECC 2.0 目前是 Alpha 版本，可用于本地实验，但不应视为正式发布。

### 15.3 状态快照

```bash
# 导出 Markdown 格式状态
ecc status --markdown --write status.md

# 同步 GitHub 工作项
ecc work-items sync-github --repo owner/repo

# 自动化退出码
ecc status --exit-code    # 需要关注时返回非零
```

---

## 第十六章：贡献指南

### 16.1 贡献类型

ECC 欢迎以下类型的贡献：

- **Agents** — 专用 Agent（语言审查员、框架专家、DevOps 专家等）
- **Skills** — 工作流定义和领域知识
- **Hooks** — 自动化钩子
- **Commands** — 斜杠命令

### 16.2 文件命名规范

- 全小写 + 连字符（如 `python-reviewer.md`、`tdd-workflow.md`）
- Agent 文件名必须与 Agent 名称一致

### 16.3 贡献流程

```bash
# 1. Fork 并克隆
gh repo fork affaan-m/everything-claude-code --clone
cd everything-claude-code

# 2. 创建分支
git checkout -b feat/my-contribution

# 3. 添加贡献

# 4. 本地测试
cp -r skills/my-skill ~/.claude/skills/  # 测试技能

# 5. 提交 PR
git add . && git commit -m "feat: add my-skill" && git push -u origin feat/my-contribution
```

### 16.4 各类型格式要求

#### Agent 格式
```markdown
---
name: agent-name
description: 简短描述
tools: Read, Write, Edit, Bash
model: sonnet
---
# Agent 内容...
```

#### Skill 格式
```markdown
---
name: skill-name
description: 简短描述
origin: ECC|community
---
# Skill 内容，包含 When to Use / How It Works / Examples
```

#### Hook 格式
```json
{
  "matcher": "Bash|Write|Edit",
  "hooks": [{
    "type": "command",
    "command": "node scripts/hooks/my-hook.js"
  }],
  "description": "描述",
  "id": "unique-id"
}
```

### 16.5 提交格式

使用 Conventional Commits：

```
feat(skills): add pytorch-patterns skill
fix(hooks): resolve session-start root fallback
docs: update installation guide
refactor(agents): simplify build-error-resolver
```

---

## 附录：速查表与资源

### A. 核心命令速查

| 场景 | 命令/Skill |
|------|-----------|
| 规划功能 | `/plan` 或 `planner` Agent |
| 代码审查 | `/code-review` |
| 修复构建 | `/build-fix` |
| TDD 开发 | `tdd-workflow` Skill |
| 安全扫描 | `/security-scan` |
| 创建技能 | `/skill-create` |
| 保存会话 | `/save-session` |
| 查看本能 | `/instinct-status` |
| 导出本能 | `/instinct-export` |
| 查看会话 | `/sessions` |
| 设置包管理 | `/setup-pm` |

### B. 环境变量速查

| 变量 | 用途 | 默认值 |
|------|------|--------|
| `CLAUDE_PACKAGE_MANAGER` | 指定包管理器 | 自动检测 |
| `ECC_HOOK_PROFILE` | 钩子严格度 | `standard` |
| `ECC_DISABLED_HOOKS` | 禁用特定钩子 | 无 |
| `ECC_SESSION_START_MAX_CHARS` | SessionStart 上下文上限 | `8000` |
| `ECC_SESSION_START_CONTEXT` | 关闭 SessionStart 上下文 | — |
| `ECC_GOVERNANCE_CAPTURE` | 启用治理捕获 | `0` |

### C. 重要链接

| 资源 | 链接 |
|------|------|
| GitHub 仓库 | https://github.com/affaan-m/everything-claude-code |
| npm 包 | https://www.npmjs.com/package/ecc-universal |
| AgentShield | https://github.com/affaan-m/agentshield |
| GitHub Marketplace | https://github.com/marketplace/ecc-tools |
| 官网 | https://ecc.tools |
| 作者 Twitter | https://x.com/affaanmustafa |

### D. 测试与验证

```bash
# 运行所有测试
node tests/run-all.js

# 运行单个测试
node tests/lib/utils.test.js
node tests/lib/package-manager.test.js
node tests/hooks/hooks.test.js

# 检查覆盖率
npm run coverage

# CI 验证
npm test
```

### E. 三个公共标识符

| 标识符 | 用途 | 不要混淆 |
|--------|------|---------|
| `affaan-m/everything-claude-code` | GitHub 源仓库 | — |
| `ecc@ecc` | Claude Marketplace/插件标识 | 保持命名空间简短 |
| `ecc-universal` | npm 包名 | 与 Marketplace 不同 |

---

> **教程版本**：基于 ECC v2.0.0-rc.1 生成
>
> **最后更新**：2026-05-18

## 知识库文档索引

| 文档 | 说明 |
|------|------|
| [[ECC 核心配置（CLAUDE.md）]] | 项目核心指导文件 |
| [[核心 Agent 系统]] | 60+ 专用子 Agent 详解 |
| [[关键 Skills 精选]] | 核心 Skill 汇总（编码标准、TDD、安全） |
| [[Hooks 自动化钩子系统]] | 自动化钩子详解 |
| [[核心规则体系（Rules）]] | 编码规范和安全规则 |
| [[MCP 服务器集成]] | MCP 配置和使用指南 |
| [[Commands 命令系统]] | 75+ 斜杠命令详解 |
| [[持续学习与进化系统]] | 本能学习系统详解 |
| [[Skill 开发指南]] | 创建有效 Skill 的完整指南 |
| [[故障排除指南]] | 常见问题和解决方案 |
| [[智能体（Agent）]] | 智能体核心概念 |
| [[PEAS 模型]] | 智能体 PEAS 模型 |
| [[ReAct 范式]] | ReAct 推理-行动范式 |
| [[智能体核心概念图谱]] | 智能体概念总览 |
