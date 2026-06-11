# Using Git Worktrees

> 在开始需要隔离的功能工作时使用，确保隔离工作空间存在。

---

## 核心原则

**先检测现有隔离。然后使用原生工具。然后回退到 git。永远不要与工具对抗。**

## 步骤 0: 检测现有隔离

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
BRANCH=$(git branch --show-current)
```

**子模块守卫：** 如果 `GIT_DIR != GIT_COMMON`，也可能是 git 子模块内部。先验证：

```bash
git rev-parse --show-superproject-working-tree 2>/dev/null
# 如果返回路径，你在子模块中，不是工作区
```

**如果 `GIT_DIR != GIT_COMMON`（且不是子模块）：** 已经在隔离工作区。跳到步骤 3。

**如果 `GIT_DIR == GIT_COMMON`：** 在正常仓库中。询问用户是否设置工作区。

## 步骤 1: 创建隔离工作空间

### 1a. 原生工作区工具（优先）

如果平台提供原生工作区工具（如 `EnterWorktree`、`WorktreeCreate`），使用它。

**注意：** 在有原生工具时使用 `git worktree add` 会创建工具无法管理的幻影状态。

### 1b. Git 工作区回退

```bash
project=$(basename "$(git rev-parse --show-toplevel)")
git worktree add "$path" -b "$BRANCH_NAME"
cd "$path"
```

## 步骤 3: 项目设置

自动检测并运行适当的设置：

```bash
# Node.js
if [ -f package.json ]; then npm install; fi

# Rust
if [ -f Cargo.toml ]; then cargo build; fi

# Python
if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
if [ -f pyproject.toml ]; then poetry install; fi

# Go
if [ -f go.mod ]; then go mod download; fi
```

## 步骤 4: 验证干净基线

```bash
npm test / cargo test / pytest / go test ./...
```

- **测试失败：** 报告失败，询问是否继续或调查
- **测试通过：** 报告就绪

## 快速参考

| 情况 | 操作 |
|------|------|
| 已在链接工作区 | 跳过创建（步骤 0） |
| 在子模块中 | 视为正常仓库（步骤 0 守卫） |
| 有原生工作区工具 | 使用它（步骤 1a） |
| 无原生工具 | Git 工作区回退（步骤 1b） |
| `.worktrees/` 存在 | 使用它（验证已忽略） |
| `worktrees/` 存在 | 使用它（验证已忽略） |
| 目录未忽略 | 添加到 .gitignore + 提交 |
| 创建时权限错误 | 沙箱回退，在原位工作 |
| 基线测试失败 | 报告失败 + 询问 |

## 常见错误

| 错误 | 修复 |
|------|------|
| **对抗工具** | 步骤 0 检测现有隔离，步骤 1a 优先使用原生工具 |
| **跳过检测** | 在现有工作区内嵌套创建工作区 |
| **跳过忽略验证** | 使用 `git check-ignore` 验证 |
| **假设目录位置** | 遵循优先级：现有 > 全局遗留 > 指令文件 > 默认 |
| **继续执行失败的测试** | 报告失败，获得明确许可再继续 |

## 链接

- [[Superpowers MOC]]
- [[Finishing a Development Branch]] — 完成工作后清理
