# Superpowers 概述

> 一个完整的 AI Agent 软件开发方法论，确保编码助手在正确的时间使用正确的工作流。

---

## 起源

Superpowers 由 [Jesse Vincent](https://blog.fsck.com) 创建，是一个为 AI 编码助手设计的**技能插件系统**。其核心洞察是：**大多数 AI 编码失败不是因为模型能力不足，而是因为缺乏正确的工作流**。

---

## 核心设计哲学

### 1. 技能是代码，不是散文

技能文档不是参考资料，而是**塑造 Agent 行为的代码**。每个技能都经过：
- 压力测试
- 对抗性评估
- 真实场景验证

### 2. 自动触发

技能不是手动调用的，而是在正确的条件下自动激活。例如：
- 用户说"我们来构建一个 React Todo 列表" → 自动触发 `brainstorming`
- 发现 Bug → 自动触发 `systematic-debugging`
- 开始实现功能 → 自动触发 `test-driven-development`

### 3. 强制工作流

技能不是建议，而是**强制执行的规则**。Agent 不能理性化地跳过它们。

### 4. 人类伙伴语言

Superpowers 刻意使用"人类伙伴"（human partner）而非"用户"，以建立正确的协作关系。

---

## 安装方式

支持多种 AI 编码工具：

| 工具 | 安装命令 |
|------|---------|
| Claude Code | `/plugin install superpowers@claude-plugins-official` |
| Codex CLI | `/plugins` → 搜索 Superpowers |
| Cursor | `/add-plugin superpowers` |
| GitHub Copilot CLI | `copilot plugin install superpowers@superpowers-marketplace` |
| Gemini CLI | `gemini extensions install https://github.com/obra/superpowers` |
| OpenCode | 从 URL 获取安装说明 |

---

## 基本工作流

```
用户输入 → 触发技能 → 执行工作流 → 交付成果
```

1. **Brainstorming** — 在写代码前激活，通过提问精炼想法，探索替代方案
2. **Using Git Worktrees** — 创建隔离工作空间
3. **Writing Plans** — 将工作分解为 2-5 分钟的小任务
4. **Subagent-Driven Development** — 为每个任务派遣子代理，进行两阶段审查
5. **Test-Driven Development** — 强制执行红-绿-重构循环
6. **Requesting Code Review** — 任务间审查
7. **Finishing a Development Branch** — 完成工作并清理

---

## 核心原则

| 原则 | 描述 |
|------|------|
| **Test-Driven Development** | 先写测试，永远 |
| **Systematic over ad-hoc** | 流程优于猜测 |
| **Complexity reduction** | 简单是首要目标 |
| **Evidence over claims** | 验证后再宣称成功 |

---

## 为什么有效

传统 AI 编码的问题：
- ❌ 立即开始写代码，不考虑设计
- ❌ 跳过测试，直接实现
- ❌ 遇到 Bug 时随机尝试修复
- ❌ 没有审查就提交代码

Superpowers 的解决方案：
- ✅ 在写代码前先设计和规划
- ✅ 强制 TDD 红-绿-重构循环
- ✅ 系统化的四阶段调试流程
- ✅ 自动代码审查和质量检查

---

## 链接

- [[Superpowers MOC]]
