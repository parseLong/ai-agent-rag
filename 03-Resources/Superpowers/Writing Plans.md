# Writing Plans

> 当你有规格说明或需求时，在触碰代码之前使用。编写全面的实施计划，假设工程师对代码库一无所知且品味 questionable。

---

## 核心原则

**每个步骤是一个 2-5 分钟的动作：**
- "编写失败的测试" — 步骤
- "运行它以确认失败" — 步骤
- "实现最小代码使测试通过" — 步骤
- "运行测试确保通过" — 步骤
- "提交" — 步骤

## 计划文档头部

每个计划必须以此头部开始：

```markdown
# [功能名称] 实施计划

> **对于代理工作者：** 必需子技能：使用 superpowers:subagent-driven-development（推荐）或 superpowers:executing-plans 按任务实现此计划。步骤使用复选框 (`- [ ]`) 语法进行跟踪。

**目标：** [一句话描述构建什么]

**架构：** [2-3 句话描述方法]

**技术栈：** [关键技术/库]

---
```

## 任务结构

```markdown
### 任务 N: [组件名称]

**文件：**
- 创建: `exact/path/to/file.py`
- 修改: `exact/path/to/existing.py:123-145`
- 测试: `tests/exact/path/to/test.py`

- [ ] **步骤 1: 编写失败的测试**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **步骤 2: 运行测试确认失败**

运行: `pytest tests/path/test.py::test_name -v`
预期: FAIL with "function not defined"

- [ ] **步骤 3: 编写最小实现**

```python
def function(input):
    return expected
```

- [ ] **步骤 4: 运行测试确认通过**

运行: `pytest tests/path/test.py::test_name -v`
预期: PASS

- [ ] **步骤 5: 提交**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
```

## 禁止占位符

以下内容是计划失败，永远不要写：
- "TBD", "TODO", "稍后实现", "填写细节"
- "添加适当的错误处理" / "添加验证" / "处理边界情况"
- "为上述编写测试" (没有实际测试代码)
- "类似于任务 N" (重复代码)
- 只有描述没有代码的步骤
- 引用未在任何任务中定义的类型、函数或方法

## 自审清单

写完计划后，进行自查：

1. **规格覆盖**：每个规格要求都有对应的任务吗？列出任何缺口。
2. **占位符扫描**：搜索计划中的红旗模式。
3. **类型一致性**：后续任务中的类型、方法签名与前面任务定义的是否一致？

## 执行交接

保存计划后，提供执行选择：

> "计划完成并保存到 `docs/superpowers/plans/<filename>.md`。两种执行选项：
>
> 1. **子代理驱动（推荐）** — 我为每个任务派遣子代理，任务间审查，快速迭代
>
> 2. **内联执行** — 在此会话中使用 executing-plans 执行，批量执行并设置检查点
>
> 选择哪种方法？"

## 链接

- [[Superpowers MOC]]
- [[Subagent-Driven Development]] — 推荐执行方式
- [[Executing Plans]] — 替代执行方式
