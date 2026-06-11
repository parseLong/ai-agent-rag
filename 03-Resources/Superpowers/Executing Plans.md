# Executing Plans

> 当你有书面实施计划要在单独会话中执行时使用，带有审查检查点。

---

## 核心原则

加载计划，批判性审查，执行所有任务，完成后报告。

**注意：** 告诉人类伙伴，Superpowers 在有子代理访问时效果更好。如果有子代理可用，使用 `subagent-driven-development` 代替此技能。

## 流程

### 步骤 1: 加载和审查计划

1. 读取计划文件
2. 批判性审查 — 识别关于计划的任何问题或担忧
3. 如果有担忧：在开始前向人类伙伴提出
4. 如果没有担忧：创建 TodoWrite 并继续

### 步骤 2: 执行任务

对于每个任务：
1. 标记为 in_progress
2. 严格遵循每个步骤（计划有细粒度步骤）
3. 按指定运行验证
4. 标记为 completed

### 步骤 3: 完成开发

所有任务完成并验证后：
- 宣布："I'm using the finishing-a-development-branch skill to complete this work."
- **必需子技能：** 使用 `superpowers:finishing-a-development-branch`
- 遵循该技能验证测试、呈现选项、执行选择

## 何时停止并寻求帮助

**立即停止执行：**
- 遇到阻碍（缺失依赖、测试失败、指令不清楚）
- 计划有严重缺口阻止开始
- 不理解某个指令
- 验证反复失败

**询问澄清，不要猜测。**

## 记忆

- 先批判性审查计划
- 严格遵循计划步骤
- 不要跳过验证
- 计划要求时引用技能
- 遇到阻碍时停止，不要猜测
- 不要在未经明确同意的情况下在主分支上开始实现

## 链接

- [[Superpowers MOC]]
- [[Subagent-Driven Development]] — 如果有子代理，推荐代替此技能
- [[Writing Plans]] — 创建此技能执行的计划
- [[Finishing a Development Branch]] — 完成所有任务后
