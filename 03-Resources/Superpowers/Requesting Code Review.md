# Requesting Code Review

> 完成任务、实现主要功能或合并前使用，验证工作是否符合要求。

---

## 核心原则

**早审查，常审查。**

## 何时请求审查

**强制：**
- 子代理驱动开发中每个任务后
- 完成主要功能后
- 合并到主分支前

**可选但有价值的：**
- 卡住时（新视角）
- 重构前（基线检查）
- 修复复杂 Bug 后

## 如何请求

### 1. 获取 git SHA

```bash
BASE_SHA=$(git rev-parse HEAD~1)  # 或 origin/main
HEAD_SHA=$(git rev-parse HEAD)
```

### 2. 派遣代码审查者子代理

使用 Task 工具，`general-purpose` 类型，填充 `code-reviewer.md` 模板。

**占位符：**
- `{DESCRIPTION}` — 构建内容的简要总结
- `{PLAN_OR_REQUIREMENTS}` — 它应该做什么
- `{BASE_SHA}` — 起始提交
- `{HEAD_SHA}` — 结束提交

### 3. 根据反馈行动

- **严重问题**：立即修复
- **重要问题**：继续前修复
- **次要问题**：记录，稍后处理
- **审查者错误**：用推理反驳

## 与子代理驱动开发的集成

**每个任务后审查：**
- 在问题级联前捕获
- 修复后再继续下一个任务

## 红线

**永远不要：**
- 因为"简单"而跳过审查
- 忽略严重问题
- 继续执行未修复的重要问题
- 与有效的技术反馈争论

## 链接

- [[Superpowers MOC]]
- [[Receiving Code Review]] — 如何回应审查反馈
- [[Subagent-Driven Development]] — 任务间自动审查
