---
title: gstack Skill 模板系统
description: 代码驱动文档生成管道 — 从 .tmpl 模板到 10 个宿主的 SKILL.md，以及序言组合根的分层架构
date: 2026-05-19
tags: [gstack, Skill系统, 模板, 文档生成, 多宿主, 参考笔记]
---

# gstack Skill 模板系统

## 📖 定义

gstack 的 SKILL.md 文件是**从 .tmpl 模板生成的**，不是手写的。一套模板 → 10 个宿主配置 → 10 组不同的 SKILL.md。这个管道解决了"手维护文档永远与代码漂移"的问题——如果代码中有命令，它就出现在文档中；如果代码中没有，它就不可能出现。

## 🔄 生成管道

```
SKILL.md.tmpl          （人类编写的流程 + 占位符）
       ↓
gen-skill-docs.ts      （读取源代码元数据 + 宿主配置）
       ↓
SKILL.md               （提交的、自动生成段落）
```

### 关键占位符

| 占位符 | 来源 | 生成内容 |
|--------|------|---------|
| `{{COMMAND_REFERENCE}}` | `commands.ts` | 分类命令表 |
| `{{SNAPSHOT_FLAGS}}` | `snapshot.ts` | 标志参考 + 示例 |
| `{{PREAMBLE}}` | 序言组合根 | 启动块：更新检查、会话跟踪、AskUserQuestion 格式 |
| `{{BROWSE_SETUP}}` | `gen-skill-docs.ts` | 二进制发现 + 设置指令 |
| `{{BASE_BRANCH_DETECT}}` | `gen-skill-docs.ts` | 动态基础分支检测 |
| `{{GBRAIN_CONTEXT_LOAD}}` | `resolvers/gbrain.ts` | 大脑优先上下文搜索 |
| `{{UX_PRINCIPLES}}` | `resolvers/design.ts` | 用户行为基础理论 |

## 🏗️ 序言组合根（Preamble Composition Root）

每个技能在执行前先运行 `{{PREAMBLE}}` 块，它是一个**分层组合**系统：

| 层级 | 包含内容 | 适用技能 |
|------|---------|---------|
| **T1** | 核心 + 升级检查 + 遥测 + 精简语音 | 简单工具技能 |
| **T2** | T1 + 完整语音 + 提问格式 + 完整性 + 困惑协议 | 评审/规划技能 |
| **T3** | T2 + 仓库模式 + 构建前搜索 | 高级分析技能 |
| **T4** | 同 T3 | 最高层技能 |

**设计亮点：** 没有内联逻辑，仅通过 tier gating 进行**声明式组合**。修改某个层（如写作风格迁移）只需编辑对应的 `./preamble/*.ts` 文件，不触碰核心组合逻辑。

### 序言处理的功能

1. **更新检查** — `gstack-update-check`，报告是否有升级可用
2. **会话跟踪** — 当 3+ 会话同时运行时，所有技能进入"ELI16 模式"
3. **运营自我改进** — 技能结束后反思失败，记录到 `learnings.jsonl`
4. **AskUserQuestion 格式** — 统一格式：上下文 + 问题 + 推荐 + 字母选项
5. **构建前搜索** — 三层知识：验证标准(L1) / 流行实践(L2) / 第一性原理(L3)

## 🌐 多宿主生成

10 个宿主配置（`hosts/*.ts`）控制生成差异：

| 宿主 | 输出目录 | 特殊处理 |
|------|---------|---------|
| Claude（主） | `{skill}/SKILL.md` | 完整 frontmatter |
| Codex | `.agents/skills/gstack-{skill}/SKILL.md` | 最简 frontmatter |
| Cursor | `~/.cursor/skills/gstack-*/` | IDE 风格适配 |
| OpenClaw | OpenClaw skill 目录 | 工具名映射 |

添加新宿主：创建 `hosts/myhost.ts` → 加入 `hosts/index.ts` → 运行 `gen:skill-docs --host all`。**零代码改动**。

## ⚠️ 关键约束

- **SKILL.md 是生成的**——永远编辑 `.tmpl`，不要直接改 `.md`
- **合并冲突**——NEVER 直接接受生成的 SKILL.md 任意一方。先解决 `.tmpl` 冲突，再重新生成
- **CI 验证**——`gen:skill-docs --dry-run` + `git diff --exit-code` 捕获过期文档
- **Git blame 可用**——可查看命令何时添加、在哪个提交

## 🔗 关联知识

- [[gstack 项目概览]] — gstack 整体架构
- [[代码-文档同步管道]] — 此模式的通用化提取
- [[序言组合根模式]] — 分层序言的通用化提取
- [[Skill 开发指南]] — ECC 的 Skill 开发指南（对比参考）

## 🏷️ 标签

#gstack #Skill系统 #模板 #文档生成 #多宿主 #参考笔记