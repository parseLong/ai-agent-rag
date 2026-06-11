# Writing Skills

> 创建新技能、编辑现有技能或验证技能在部署前工作时使用。

---

## 核心原则

**编写技能是将 TDD 应用于流程文档。**

**铁律：**
```
NO SKILL WITHOUT A FAILING TEST FIRST
```

## TDD 映射

| TDD 概念 | 技能创建 |
|---------|---------|
| **测试用例** | 与子代理的压力场景 |
| **生产代码** | 技能文档 (SKILL.md) |
| **测试失败（RED）** | Agent 违反规则（基线） |
| **测试通过（GREEN）** | Agent 遵守技能 |
| **重构** | 关闭漏洞同时保持合规 |

## 何时创建技能

**创建时机：**
- 技术不是直观明显的
- 你会在多个项目中引用它
- 模式广泛适用（不是项目特定的）
- 其他人会受益

**不要为以下创建：**
- 一次性解决方案
- 其他地方有良好文档的标准实践
- 项目特定约定（放在 CLAUDE.md 中）

## SKILL.md 结构

```markdown
---
name: Skill-Name-With-Hyphens
description: Use when [特定触发条件和症状]
---

# Skill Name

## Overview
这是什么？1-2 句话的核心原则。

## When to Use
小流程图（如果决策不明显）

## Core Pattern
前后代码对比

## Quick Reference
表格或要点用于快速扫描

## Implementation
内联代码或链接到单独文件

## Common Mistakes
出错内容 + 修复

## Real-World Impact (可选)
具体结果
```

## 关键设计原则

### 1. 描述字段优化 (CSO)

**关键：描述 = 何时使用，不是技能做什么**

描述应仅描述触发条件。不要总结技能的工作流程。

```yaml
# ❌ 错误: 总结工作流程 — Agent 可能遵循此而非阅读完整技能
description: Use when executing plans - dispatches subagent per task with code review between tasks

# ✅ 正确: 仅触发条件
description: Use when executing implementation plans with independent tasks in the current session
```

### 2. 关键词覆盖

使用 Claude 会搜索的词：
- 错误消息："Hook timed out", "ENOTEMPTY", "race condition"
- 症状："flaky", "hanging", "zombie", "pollution"
- 同义词："timeout/hang/freeze", "cleanup/teardown/afterEach"

### 3. 命名

使用主动语态，动词优先：
- ✅ `creating-skills` 不是 `skill-creation`
- ✅ `condition-based-waiting` 不是 `async-test-helpers`

### 4. 令牌效率

**目标字数：**
- 入门工作流：<150 词
- 频繁加载的技能：<200 词
- 其他技能：<500 词

**技术：**
- 将详情移到工具帮助中
- 使用交叉引用
- 压缩示例
- 消除冗余

### 5. 交叉引用

```markdown
# ✅ 好: 使用技能名称 + 明确要求标记
**必需子技能:** Use superpowers:test-driven-development
**必需背景:** You MUST understand superpowers:systematic-debugging

# ❌ 坏: 不清楚是否需要
See skills/testing/test-driven-development
```

**为什么不用 @ 链接：** `@` 语法立即强制加载文件，消耗 200k+ 上下文。

## 流程图使用

**仅在以下情况使用流程图：**
- 不明显的决策点
- 你可能过早停止的流程循环
- "何时使用 A 而非 B" 的决策

**永远不要用于：**
- 参考材料 → 表格、列表
- 代码示例 → Markdown 块
- 线性指令 → 编号列表

## 代码示例

**一个优秀的示例胜过多个平庸的示例。**

选择最相关的语言：
- 测试技术 → TypeScript/JavaScript
- 系统调试 → Shell/Python
- 数据处理 → Python

**好的示例：**
- 完整且可运行
- 注释解释 WHY
- 来自真实场景
- 清晰展示模式
- 准备好适应（非通用模板）

**不要：**
- 用 5+ 种语言实现
- 创建填空模板
- 编写牵强示例

## 技能类型

| 类型 | 例子 | 测试方式 |
|------|------|---------|
| **纪律强化** | TDD、verification-before-completion | 学术问题、压力场景、多重压力组合 |
| **技术** | condition-based-waiting、root-cause-tracing | 应用场景、变体场景、缺失信息测试 |
| **模式** | reducing-complexity、information-hiding | 识别场景、应用场景、反例 |
| **参考** | API 文档、命令参考 | 检索场景、应用场景、缺口测试 |

## 防理性化

**关闭每个漏洞：**

```markdown
# 坏
Write code before test? Delete it.

# 好
Write code before test? Delete it. Start over.

**No exceptions:**
- Don't keep it as "reference"
- Don't "adapt" it while writing tests
- Don't look at it
- Delete means delete
```

**构建理性化表格：**

```markdown
| Excuse | Reality |
|--------|---------|
| "Too simple to test" | Simple code breaks. Test takes 30 seconds. |
| "I'll test after" | Tests passing immediately prove nothing. |
| "Tests after achieve same goals" | Tests-after = "what does this do?" Tests-first = "what should this do?" |
```

## RED-GREEN-REFACTOR 循环

### RED: 编写失败测试（基线）

运行无技能的压力场景。记录精确行为：
- 他们做了什么选择？
- 他们用了什么理性化（逐字）？
- 哪些压力触发了违规？

### GREEN: 编写最小技能

编写解决这些特定理性化的技能。不要添加假设案例的额外内容。

### REFACTOR: 关闭漏洞

Agent 找到了新的理性化？添加明确的对策。重新测试直到防弹。

## 部署前检查清单

**RED 阶段 — 编写失败测试：**
- [ ] 创建压力场景（3+ 组合压力）
- [ ] 无技能运行场景 — 记录基线行为
- [ ] 识别理性化/失败模式

**GREEN 阶段 — 编写最小技能：**
- [ ] 名称仅使用字母、数字、连字符
- [ ] YAML 前言带 `name` 和 `description`
- [ ] 描述以 "Use when..." 开头
- [ ] 关键词贯穿全文
- [ ] 清晰概述带核心原则
- [ ] 内联代码或链接
- [ ] 一个优秀示例
- [ ] 运行场景带技能 — 验证 Agent 遵守

**REFACTOR 阶段 — 关闭漏洞：**
- [ ] 识别新理性化
- [ ] 添加明确对策
- [ ] 构建理性化表格
- [ ] 创建红旗列表
- [ ] 重新测试直到防弹

## 链接

- [[Superpowers MOC]]
- [[Test-Driven Development]] — 技能写作遵循 TDD
