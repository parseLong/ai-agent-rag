---
tags:
  - 参考资料
  - 工具与框架
  - Claude-Code
  - Skills
aliases:
  - Claude Code Skills 开发指南
  - Skill Development Guide
---

# Claude Code Skills 开发指南

> 如何为 Claude Code 创建有效的 Skills——从架构到分类、写作技巧到测试验证的完整指南。

## Skills 是什么？

Skills 是 Claude Code 基于上下文加载的**知识模块**，提供：

- **领域专业知识** — 框架模式、语言惯用法、最佳实践
- **工作流定义** — 常见任务的分步流程
- **参考资料** — 代码片段、检查清单、决策树
- **上下文注入** — 特定条件激活

### Skill vs Agent vs Command vs Hook vs Rule

| 组件 | 目的 | 激活方式 |
|------|------|---------|
| **Skill** | 知识仓库 | 基于上下文（自动） |
| **Agent** | 任务执行器 | 显式委派 |
| **Command** | 用户动作 | 用户触发（`/command`） |
| **Hook** | 自动化 | 事件触发 |
| **Rule** | 始终遵循的指南 | 永久激活 |

---

## Skill 架构

### 文件结构

```
skills/
└── your-skill-name/
    ├── SKILL.md           # 必需：主要 Skill 定义
    ├── examples/          # 可选：代码示例
    │   ├── basic.ts
    │   └── advanced.ts
    └── references/        # 可选：外部参考
        └── links.md
```

### SKILL.md 格式

```markdown
---
name: skill-name
description: 简短描述，用于 Skill 列表和自动激活
origin: ECC
---

# Skill 标题

简短概述。

## When to Activate

描述 Claude 应使用此 Skill 的场景。

## Core Concepts

主要模式和指南。

## Code Examples

实用的、可测试的示例。

## Anti-Patterns

展示不该做什么。

## Best Practices

- 可操作的指南
- 应做和不应做的事

## Related Skills

链接到互补的 Skills。
```

### YAML Frontmatter 字段

| 字段 | 必需 | 说明 |
|------|------|------|
| `name` | 是 | 小写、连字符分隔标识符（如 `react-patterns`） |
| `description` | 是 | 一行描述，用于 Skill 列表和自动激活 |
| `origin` | 否 | 来源标识（如 `ECC`、`community`） |
| `tags` | 否 | 分类标签数组 |
| `version` | 否 | Skill 版本号 |

---

## Skill 分类

### 语言标准

专注于惯用代码、命名约定和语言特定模式。

**示例：** `python-patterns`、`golang-patterns`、`typescript-standards`

```markdown
---
name: python-patterns
description: Python 惯用法、最佳实践和模式。
---

## When to Activate

- 编写 Python 代码
- 重构 Python 模块
- Python 代码审查

## Core Concepts

### Context Managers

\```python
# 始终使用 Context Manager 处理资源
with open('file.txt') as f:
    content = f.read()
\```
```

### 框架模式

框架特定的约定、常见模式和反模式。

**示例：** `django-patterns`、`nextjs-patterns`

### 工作流 Skills

定义常见开发任务的分步流程。

**示例：** `tdd-workflow`、`code-review-workflow`、`deployment-checklist`

### 领域知识

特定领域的专业知识（安全、性能等）。

**示例：** `security-review`、`performance-optimization`

---

## 写作有效 Skill 内容

### 1. 从"When to Activate"开始

此节对自动激活至关重要，要具体：

```markdown
## When to Activate

- 创建新的 React 组件
- 重构现有组件
- 调试 React 状态问题
```

### 2. 展示而非讲述（Show, Don't Tell）

**差：** "总是在 async 函数中正确处理错误。"（笼统）

**好：** 提供完整的代码示例 + 关键要点注释。

### 3. 包含反模式

```markdown
## Anti-Patterns

### FAIL: 直接状态修改
\```typescript
user.name = 'New Name'  // 绝不要这样做
\```

### PASS: 不可变更新
\```typescript
const updatedUser = { ...user, name: 'New Name' }  // 总是这样做
\```
```

### 4. 提供检查清单

检查清单是可操作的、易于遵循的：

```markdown
## Pre-Deployment Checklist

- [ ] 所有测试通过
- [ ] 无 console.log 在生产代码中
- [ ] 密钥未硬编码
- [ ] 错误处理完整
```

### 5. 使用决策树

```
需要获取数据？
├── 单次请求 → 直接使用 fetch
├── 多个独立请求 → Promise.all()
├── 多个依赖请求 → await 顺序执行
└── 需要缓存 → 使用 SWR 或 React Query
```

---

## 最佳实践

| 做什么 | 为什么好 |
|--------|---------|
| **具体** | "用 `useCallback` 给子组件的事件处理器"——可操作 |
| **展示示例** | 可复制粘贴的代码 |
| **解释 WHY** | "不可变性防止 React 状态的意外副作用" |
| **链接相关 Skills** | "参见：`react-performance`" |
| **保持聚焦** | 一个 Skill = 一个领域/概念 |
| **使用段落标题** | 清晰标题便于扫描 |

| 不做什么 | 为什么差 |
|---------|---------|
| **笼统** | "写好代码"——不可操作 |
| **长散文** | 难解析，代码更好 |
| **覆盖太多** | "Python、Django、Flask 模式"——太宽 |
| **跳过示例** | 理论无实践价值低 |

### 内容指南

- **长度：** 200-500 行典型，800 行最大
- **代码块：** 包含语言标识符
- **标题：** 使用 `##` 和 `###` 层级
- **表格：** 用于比较和参考

---

## Skill 放置策略

### 精选内容 vs 导入/生成内容

| 类型 | 放置位置 | 说明 |
|------|---------|------|
| **精选/策划** | `skills/`（项目级） | 经人工审查的高质量内容 |
| **生成/导入** | `~/.claude/skills/`（用户级） | 从 Git 历史生成、社区导入 |
| **演进** | `~/.claude/skills/`（用户级） | 从会话中持续学习产生 |

### 来源元数据

每个 Skill 应记录：
- `origin` — 来源标识（ECC、community、生成）
- `provenance` — 来源 URL 或生成方法
- 验证状态 — 是否经过测试和审查

---

## 测试与验证

### 验证检查清单

- [ ] YAML frontmatter 有效
- [ ] Name 遵循小写-连字符惯例
- [ ] Description 清晰
- [ ] 示例代码可编译运行
- [ ] 链接有效
- [ ] 无敏感数据

---

## 关联知识

- [[MCP]] — MCP 协议与外部工具连接
- [[智能体通信协议]] — MCP/A2A/Function Calling 对比
- [[多智能体协作]] — 智能体分工与协作
- [[上下文注入]] — 将检索结果注入 Prompt 的策略

## 参考资料

- [[ECC使用指南]] — Skills 配置与使用方法
- [ECC GitHub](https://github.com/affoon-m/everything-claude-code)