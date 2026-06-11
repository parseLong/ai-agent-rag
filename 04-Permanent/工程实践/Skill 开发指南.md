---
title: Skill 开发指南
description: 为 Everything Claude Code (ECC) 创建有效 Skill 的完整指南
date: 2026-05-18
tags: [ECC, Skills, 开发指南, 最佳实践]
---

# Skill 开发指南

> 本文档提供为 Everything Claude Code (ECC) 创建有效 Skill 的完整指南。

## 什么是 Skills

Skills 是 Claude Code 根据上下文加载的**知识模块**。它们提供：

- **领域专业知识**：框架模式、语言惯用法、最佳实践
- **工作流定义**：常见任务的逐步流程
- **参考材料**：代码片段、检查清单、决策树
- **上下文注入**：满足特定条件时激活

### Skill vs Agent vs Command

| 组件 | 用途 | 激活方式 |
|------|------|---------|
| **Skill** | 知识库 | 基于上下文（自动） |
| **Agent** | 任务执行器 | 显式委派 |
| **Command** | 用户操作 | 用户调用（`/command`） |
| **Hook** | 自动化 | 事件触发 |
| **Rule** | 始终开启的指南 | 始终激活 |

### Skill 何时激活

Skills 在以下情况激活：
- 用户任务匹配 Skill 的领域
- Claude Code 检测到相关上下文
- 命令引用某个 Skill
- Agent 需要领域知识

---

## Skill 架构

### 文件结构

```
skills/
└── your-skill-name/
    ├── SKILL.md           # 必需：主 Skill 定义
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
description: 在技能列表中显示的简短描述，用于自动激活
origin: ECC    # ECC 为官方，community 为社区
---

# Skill 标题

## When to Activate
何时使用此技能...

## How It Works
工作原理...

## Examples
示例...
```

---

## 创建你的第一个 Skill

### 1. 选择名称

- 小写，连字符分隔
- 描述性但简洁
- 示例：`react-patterns`, `api-design`, `testing-strategies`

### 2. 编写描述

描述应回答：
- 此 Skill 解决什么问题？
- 何时应激活？
- 用户能获得什么价值？

**好的描述**：
```markdown
Use this skill when building REST APIs with Node.js. Covers route design, 
validation patterns, error handling, and testing strategies.
```

**不好的描述**：
```markdown
This skill helps with APIs.
```

### 3. 编写 When to Activate

明确列出激活条件：

```markdown
## When to Activate

- Creating new API endpoints
- Designing route structures
- Adding validation to user input
- Handling errors in API responses
- Writing tests for API routes
```

### 4. 编写 How It Works

提供可操作的指导：

```markdown
## How It Works

### Route Design Pattern

```typescript
// Use resource-based routes
app.get('/api/users', getAllUsers)
app.get('/api/users/:id', getUserById)
app.post('/api/users', createUser)
app.put('/api/users/:id', updateUser)
app.delete('/api/users/:id', deleteUser)
```

### Validation

```typescript
import { z } from 'zod'

const UserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100)
})
```
```

### 5. 添加 Examples

展示实际使用模式：

```markdown
## Examples

### Basic CRUD Endpoint

```typescript
app.get('/api/users/:id', async (req, res) => {
  const { id } = req.params
  const user = await db.users.findById(id)
  
  if (!user) {
    return res.status(404).json({ error: 'User not found' })
  }
  
  res.json(user)
})
```
```

---

## 有效的 Skill 内容

### 提供上下文

**差的 Skill**：
```markdown
Use TypeScript types.
```

**好的 Skill**：
```markdown
## Type Patterns

### Discriminated Unions

Use for state machines and variant types:

```typescript
type Result<T> = 
  | { status: 'success'; data: T }
  | { status: 'error'; error: string }

function handleResult<T>(result: Result<T>) {
  switch (result.status) {
    case 'success': return processData(result.data)
    case 'error': return logError(result.error)
  }
}
```
```

### 使用具体示例

**差的 Skill**：
```markdown
Handle errors properly.
```

**好的 Skill**：
```markdown
### Error Handling Pattern

```typescript
try {
  const data = await fetchUser(userId)
  return { success: true, data }
} catch (error) {
  if (error instanceof ValidationError) {
    return { success: false, error: 'Invalid input', code: 400 }
  }
  if (error instanceof NotFoundError) {
    return { success: false, error: 'User not found', code: 404 }
  }
  // Unknown error - don't leak details
  return { success: false, error: 'Internal error', code: 500 }
}
```
```

### 包含决策树

```markdown
## When to Use Each Pattern

```
Need caching?
├── Yes → Redis or in-memory?
│   ├── Redis → Use cacheService with TTL
│   └── In-memory → Use LRU cache with size limit
└── No → Skip caching layer
```
```

---

## 最佳实践

### 1. 聚焦单一领域

每个 Skill 应覆盖一个明确的主题。不要试图覆盖所有内容。

| 好的 Skill | 不好的 Skill |
|-----------|-------------|
| `react-hooks` | `frontend-development` |
| `api-pagination` | `backend-patterns` |
| `error-handling` | `coding-best-practices` |

### 2. 使用 Markdown 结构

```markdown
# Title

## When to Activate
Clear activation criteria

## How It Works
### Subtopic 1
Details...

### Subtopic 2
Details...

## Examples
### Example 1: Common case
### Example 2: Edge case

## References
- [Link](url)
```

### 3. 保持更新

Skills 应随最佳实践演进：
- 当框架发布新功能时更新
- 当模式变得过时时弃用
- 当发现更好的方法时改进

### 4. 测试你的 Skill

在提交前验证：
- 描述是否清晰且具体？
- 示例是否可运行？
- 模式是否是最新的？
- 是否涵盖了边缘情况？

---

## 常见模式

### 框架特定模式

```markdown
## React Hooks Pattern

### Custom Hook Template

```typescript
export function useAsync<T>(asyncFn: () => Promise<T>) {
  const [data, setData] = useState<T | null>(null)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState<Error | null>(null)

  useEffect(() => {
    setLoading(true)
    asyncFn()
      .then(setData)
      .catch(setError)
      .finally(() => setLoading(false))
  }, [asyncFn])

  return { data, loading, error }
}
```
```

### 工作流模式

```markdown
## Code Review Checklist

### Before Starting
- [ ] Pull latest changes
- [ ] Run tests locally
- [ ] Check for merge conflicts

### During Review
- [ ] Check for security issues
- [ ] Verify error handling
- [ ] Review naming and comments
- [ ] Check test coverage

### After Review
- [ ] Address feedback
- [ ] Re-run tests
- [ ] Update documentation
```

---

## 提交你的 Skill

### 提交前检查清单

- [ ] Skill 名称描述性强且唯一
- [ ] 描述清楚说明用途和激活条件
- [ ] 包含实用的代码示例
- [ ] 涵盖边缘情况和错误场景
- [ ] 格式正确（Markdown + YAML frontmatter）
- [ ] 遵循项目的编码标准

### 提交格式

使用 Conventional Commits：

```
feat(skills): add react-patterns skill
```

### 放置位置

- **ECC 官方 Skill**：`skills/` 目录
- **个人/项目 Skill**：`~/.claude/skills/`
- **已学习 Skill**：`~/.claude/skills/learned/`

---

## 示例库

### 优秀的 Skill 示例

| Skill | 为什么优秀 |
|-------|-----------|
| `tdd-workflow` | 清晰的步骤、具体的命令、覆盖所有测试类型 |
| `security-review` | 具体的安全检查清单、代码示例、严重程度分级 |
| `api-design` | 决策树、REST 约定、分页模式 |
| `error-handling` | 分类明确、具体模式、错误恢复策略 |

---

## 相关链接

- [[ECC 核心配置（CLAUDE.md）]] — 项目核心指导文件
- [[关键 Skills 精选]] — 核心 Skill 汇总
- [[Everything Claude Code 完全教程]] — 完整教程
