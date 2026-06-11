---
tags:
  - 参考资料
  - 工具与框架
  - Claude-Code
  - 架构
aliases:
  - ECC 架构设计与 Token 优化
---

# ECC 架构设计与 Token 优化

> ECC 的参考架构、Token 优化策略、跨平台适配模型、自改进评估原型、可观测性规范。

## 参考架构（ECC 2.0）

ECC 2.0 定义了一个 Harness OS 的参考架构——以 Adapter 层为核心，支持多种智能体运行环境（Claude Code、Codex、OpenCode、Cursor 等）。

### 核心架构层次

```
┌──────────────────────────────┐
│    Operator Layer            │  ← 人类开发者
│  (CLAUDE.md, rules, memory)  │
├──────────────────────────────┤
│    Harness OS                │  ← 智能体操作系统
│  (Adapter, Worktree, Loop)   │
├──────────────────────────────┤
│    Agent Layer               │  ← 60+ 专业智能体
│  (Skills, Hooks, Commands)   │
├──────────────────────────────┤
│    Tool Layer                │  ← MCP Servers
│  (External services)         │
└──────────────────────────────┘
```

### 自改进循环

ECC 2.0 的核心设计是自改进循环：

1. **Observe** — Hook 观察工具使用模式
2. **Score** — Instinct 评分判断模式质量
3. **Extract** — 持续学习提取可复用模式
4. **Validate** — Validator 验证新 Skill
5. **Promote** — 合格的从用户级提升到项目级

---

## Token 优化指南

### 推荐设置

```json
{
  "model": "sonnet",
  "env": {
    "MAX_THINKING_TOKENS": "10000",
    "CLAUDE_CODE_SUBAGENT_MODEL": "haiku"
  }
}
```

| 设置 | 默认值 | 推荐值 | 效果 |
|------|--------|--------|------|
| `model` | opus | **sonnet** | Sonnet 处理 ~80% 编码任务，~60% 成本削减 |
| `MAX_THINKING_TOKENS` | 31,999 | **10,000** | 减少隐藏推理成本 ~70% |
| `CLAUDE_CODE_SUBAGENT_MODEL` | 继承主模型 | **haiku** | Subagent 用 Haiku，~80% 更便宜 |

### 模型选择决策树

```
任务类型？
├── 简单查找/探索 → Haiku（最便宜）
├── 日常编码/审查 → Sonnet（最佳平衡）
├── 复杂架构/多步推理 → Opus（最强推理）
└── 安全分析 → Opus（不能漏漏洞）
```

**切换方式：** `/model sonnet` / `/model opus` / `/model haiku`

### 上下文管理命令

| 命令 | 使用时机 |
|------|---------|
| `/clear` | 不相关任务之间——陈旧上下文浪费后续每条消息的 Token |
| `/compact` | 逻辑任务断点（规划后、调试后、切换焦点前） |
| `/cost` | 检查当前会话 Token 支出 |

### 战略压缩时机

**何时压缩：**
- 探索完成后，实现开始前
- 完成一个里程碑后
- 调试完成后，继续新工作前
- 主要上下文转换前

**何时不压缩：**
- 相关变更的中期实现
- 活跃问题调试中
- 多文件重构中

### Subagent 保护上下文

用 Subagent（Task 工具）做探索而非在主会话中读很多文件。Subagent 读 20 个文件但只返回摘要——主上下文保持干净。

---

## MCP 服务器管理

每个启用的 MCP 服务器向上下文窗口添加工具定义。

**核心原则：保持每个项目 10 个以下启用。**

- `/mcp` 查看活跃服务器及其上下文成本
- 有 CLI 替代时优先用 CLI（`gh` 替 GitHub MCP）
- `memory` MCP 默认配置但不被任何 Skill/Agent/Hook 使用——考虑禁用

---

## 跨平台适配模型（Cross-Harness）

### 可移植层

| 内容类型 | 跨平台可移植性 | 说明 |
|----------|--------------|------|
| `SKILL.md` | 完全可移植 | 所有平台通用 |
| Rules | 完全可移植 | Markdown 格式通用 |
| Agents | 部分适配 | frontmatter 字段可能需调整 |
| Hooks | 需适配 | 各平台 Hook 格式不同 |
| MCP 配置 | 需适配 | 各平台 MCP 注册方式不同 |

### 适配策略

```
SKILL.md（通用）→ Adapter 层 → 各平台特定格式
                          ├→ Claude Code（.claude/ 目录）
                          ├→ Codex（codex.json）
                          ├→ Cursor（.cursor/ 目录）
                          └→ OpenCode（配置文件）
```

---

## 评估原型（Evaluator RAG）

### 自改进只读评估循环

ECC 定义了一个只读评估器循环——不修改代码，只评估和反馈。

1. **Scenario Specs** — 定义评估场景
2. **Traces** — 记录执行轨迹
3. **Reports** — 生成评估报告
4. **Promotion Rules** — 合格的提升

### 评估指标

```
pass@k: 至少 ONE of k 次尝试成功
  k=1: 70%  k=3: 91%  k=5: 97%

pass^k: ALL k 次尝试必须成功
  k=1: 70%  k=3: 34%  k=5: 17%
```

用 **pass@k** 当你只需要它工作。用 **pass^k** 当一致性至关重要。

---

## 可观测性规范

### 本地文件后端的可观测性门控

在 ECC 变自主之前，必须通过本地文件后端的可观测性门控：

| 门控 | 内容 |
|------|------|
| **信号模型** | 定义哪些信号需要观察 |
| **会话轨迹** | 记录每次会话的工具调用轨迹 |
| **风险账本** | 记录风险事件和决策 |
| **操作员工作流** | 操作员的审批流程 |

### 结构化日志格式

```json
{
  "timestamp": "2026-03-15T06:40:00Z",
  "session_id": "abc123",
  "tool": "Bash",
  "command": "...",
  "approval": "blocked",
  "risk_score": 0.94
}
```

---

## 持续学习 v2 规范

ECC v2 持续学习架构基于 Hook 的观察：

1. **Hook-Based Observation** — Stop Hook 观察会话模式
2. **Instinct Scoring** — 判断模式是否值得提取
3. **Pattern Extraction** — 提取可复用模式为新 Skill
4. **Validation** — Validator 验证新 Skill 质量
5. **Promotion** — 合格的从 `~/.claude/skills/` 提升到项目级

---

## Agent Teams 成本警告

Agent Teams（实验性）生成多个独立上下文窗口。每个队友独立消耗 Token。

- 仅在并行性有明显价值时使用（多模块工作、并行审查）
- 简单顺序任务用 Subagent（Task 工具）更省 Token

---

## 快速参考

```bash
# 每日工作流
/model sonnet              # 从这里开始
/model opus                # 仅复杂推理
/clear                     # 不相关任务间
/compact                   # 逻辑断点处
/cost                      # 检查支出

# 环境变量（加到 ~/.claude/settings.json "env" 段）
MAX_THINKING_TOKENS=10000
CLAUDE_CODE_SUBAGENT_MODEL=haiku
```

---

## 关联知识

- [[上下文窗口管理]] — 长对话处理策略
- [[上下文压缩]] — 上下文压缩方法
- [[记忆系统]] — 短期与长期记忆
- [[MCP]] — MCP 协议定义
- [[多智能体协作]] — 多智能体协作架构
- [[ECC使用指南]] — Skills、Hooks 配置方法

## 参考资料

- [[ECC智能体设计模式]] — 智能体设计模式详解