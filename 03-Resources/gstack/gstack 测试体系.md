---
title: gstack 测试体系
description: gstack 的三层测试体系 — 静态验证(T1)、E2E via claude -p(T2)、LLM-as-judge(T3)，加上 diff-based 选择和 gate/periodic 分类
date: 2026-05-19
tags: [gstack, 测试, E2E, LLM评测, 参考笔记]
---

# gstack 测试体系

## 📖 定义

gstack 采用三层测试体系，核心思想：**用免费测试抓住 95% 的问题，仅用 LLM 处理判断性任务**。同时采用 diff-based 自动选择和 gate/periodic 两级分类。

## 📊 三层体系

| 层 | 内容 | 成本 | 速度 | 运行条件 |
|----|------|------|------|---------|
| **T1 — 静态验证** | 解析 `$B` 命令→验证注册表、快照标志、SKILL.md 正确性、可观测性单元测试 | 免费 | <5s | 每次 `bun test` |
| **T2 — E2E** | 通过 `claude -p` 子进程启动真实 Claude 会话，运行每个技能，扫描错误 | ~$3.85/运行 | ~20min | `EVALS=1` |
| **T3 — LLM-as-judge** | Claude Sonnet 对生成文档评分：清晰度/完整性/可操作性 (1-5) | ~$0.15/运行 | ~30s | `EVALS=1` |

## 🔑 关键设计

### Diff-based 测试选择

`test:evals` 和 `test:e2e` 基于 `git diff` **自动选择**要运行的测试。每个测试在 `test/helpers/touchfiles.ts` 中声明文件依赖：

- 改了全局 touchfile（session-runner, eval-store, touchfiles.ts 本身）→ **触发所有测试**
- `EVALS_ALL=1` 或 `:all` 脚本变体 → 强制所有测试
- `eval:select` → 预览哪些测试会运行

### 两级分类

| 级别 | 含义 | CI 运行 | 频率 |
|------|------|---------|------|
| **gate** | 安全护栏或确定性功能测试 | ✅ 每次 PR | CI 默认 |
| **periodic** | 质量基准、Opus 模型测试、非确定性测试 | ❌ | 每周 cron / 手动 |

**新 E2E 测试分类规则：**
1. 安全护栏 / 确定性功能 → `gate`
2. 质量基准 / Opus / 非确定性 → `periodic`
3. 需要外部服务（Codex, Gemini）→ `periodic`

### E2E 会话运行器

`test/helpers/session-runner.ts` 通过 `claude -p` 独立子进程运行：

```
1. 写 prompt 到临时文件（避免 shell 转义问题）
2. sh -c 'cat prompt | claude -p --output-format stream-json --verbose'
3. 流式读取 NDJSON
4. 与可配置超时竞争
5. 解析完整 NDJSON transcript 为结构化结果
```

**parseNDJSON() 是纯函数**——无 I/O、无副作用——可独立测试。

### 可观测性数据流

```
skill-e2e-*.test.ts
      │
  ┌───┼────────────────┐
  │   │                │
  │  runSkillTest()   evalCollector
  │  (session-runner)  (eval-store)
  │   │                │
  │   ▼                ▼
  │ [心跳] [进度]    partial-e2e.json
  │                (原子覆写)
  │
  │  失败时:
  │  {name}-failure.json
  │
  │  ALL in ~/.gstack-dev/
  └─────────────────────

         eval-watch.ts
              │
        读取心跳 + partial
              ▼
        渲染仪表板
```

**分离所有权：** session-runner 拥有心跳（当前状态），eval-store 拥有 partial（已完成状态）。watcher 读取两者。两者互不知晓——仅通过文件系统共享数据。

## ⚠️ E2E 测试 Fixture 规则

**永远不要将完整 SKILL.md 复制到 E2E fixture。** SKILL.md 1500-2000 行，`claude -p` 读取如此大的文件会导致上下文膨胀→超时→5-10x 延长。

正确做法：**仅提取测试需要的段落**：

```typescript
// 好做法 — 代理读 ~60 行，38s 完成
const full = fs.readFileSync(path.join(ROOT, 'ship', 'SKILL.md'), 'utf-8');
const start = full.indexOf('## Review Readiness Dashboard');
const end = full.indexOf('\n---\n', start);
fs.writeFileSync(path.join(dir, 'ship-SKILL.md'), full.slice(start, end));
```

## 🔗 关联知识

- [[gstack 项目概览]] — gstack 整体架构
- [[gstack Skill 模板系统]] — 模板验证是 T1 的核心

## 🏷️ 标签

#gstack #测试 #E2E #LLM评测 #参考笔记