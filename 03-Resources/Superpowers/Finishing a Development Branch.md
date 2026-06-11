# Finishing a Development Branch

> 实现完成、所有测试通过后使用，决定如何集成工作。

---

## 核心原则

**验证测试 → 检测环境 → 呈现选项 → 执行选择 → 清理。**

## 流程

### 步骤 1: 验证测试

```bash
npm test / cargo test / pytest / go test ./...
```

- **测试失败：** 显示失败，停止。不继续步骤 2。
- **测试通过：** 继续步骤 2。

### 步骤 2: 检测环境

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
```

| 状态 | 菜单 | 清理 |
|------|------|------|
| `GIT_DIR == GIT_COMMON`（正常仓库） | 4 个选项 | 无工作区清理 |
| `GIT_DIR != GIT_COMMON`，命名分支 | 4 个选项 | 基于来源（见步骤 6） |
| `GIT_DIR != GIT_COMMON`，分离 HEAD | 3 个选项（无合并） | 无清理（外部管理） |

### 步骤 3: 确定基分支

```bash
git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null
```

### 步骤 4: 呈现选项

**正常仓库和命名分支工作区 — 恰好这 4 个选项：**

```
实现完成。你想做什么？

1. 合并回 <base-branch> 本地
2. 推送并创建 Pull Request
3. 保持分支原样（稍后处理）
4. 丢弃此工作

选择哪个选项？
```

**分离 HEAD — 恰好这 3 个选项：**

```
实现完成。你在分离 HEAD（外部管理工作区）。

1. 作为新分支推送并创建 Pull Request
2. 保持原样（稍后处理）
3. 丢弃此工作
```

### 步骤 5: 执行选择

#### 选项 1: 本地合并

```bash
cd "$MAIN_ROOT"
git checkout <base-branch>
git pull
git merge <feature-branch>
<test command>
```

然后清理工作区（步骤 6），删除分支。

#### 选项 2: 推送并创建 PR

```bash
git push -u origin <feature-branch>
gh pr create --title "<title>" --body "..."
```

**不要清理工作区** — 用户需要它来迭代 PR 反馈。

#### 选项 3: 保持原样

报告："保持分支 <name>。工作区保留于 <path>。"

#### 选项 4: 丢弃

**先确认：**
```
这将永久删除：
- 分支 <name>
- 所有提交: <commit-list>
- 工作区于 <path>

输入 'discard' 确认。
```

### 步骤 6: 清理工作区

**仅对选项 1 和 4 运行。** 选项 2 和 3 始终保留工作区。

**如果工作区路径在 `.worktrees/`、`worktrees/` 或 `~/.config/superpowers/worktrees/` 下：** Superpowers 创建此工作区 — 我们拥有清理权。

```bash
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"
git worktree remove "$WORKTREE_PATH"
git worktree prune
```

**否则：** 宿主环境（工具）拥有此工作区。不要移除它。

## 快速参考

| 选项 | 合并 | 推送 | 保持工作区 | 清理分支 |
|------|------|------|-----------|---------|
| 1. 本地合并 | 是 | - | - | 是 |
| 2. 创建 PR | - | 是 | 是 | - |
| 3. 保持原样 | - | - | 是 | - |
| 4. 丢弃 | - | - | - | 是（强制） |

## 链接

- [[Superpowers MOC]]
- [[Using Git Worktrees]] — 清理相关
