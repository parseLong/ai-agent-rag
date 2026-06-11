# Using Superpowers

> 任何对话开始时使用 — 建立如何查找和使用技能，在 ANY 响应之前调用 Skill 工具。

---

## 指令优先级

Superpowers 技能覆盖默认系统提示行为，但 **用户指令始终优先：**

1. **用户显式指令** (CLAUDE.md, GEMINI.md, AGENTS.md, 直接请求) — 最高优先级
2. **Superpowers 技能** — 覆盖冲突的默认系统行为
3. **默认系统提示** — 最低优先级

如果 CLAUDE.md 说"不要使用 TDD"，而技能说"始终使用 TDD"，遵循用户指令。

## 如何访问技能

| 平台 | 方式 |
|------|------|
| **Claude Code** | 使用 `Skill` 工具 |
| **Copilot CLI** | 使用 `skill` 工具 |
| **Gemini CLI** | 通过 `activate_skill` 工具 |
| **其他环境** | 查看平台文档 |

## 规则

**在 ANY 响应或行动之前调用相关或请求的技能。**

即使 1% 的可能性技能适用，也应该调用它检查。如果调用的技能不适用于情况，不需要使用它。

## 技能优先级

当多个技能可能适用时，使用此顺序：

1. **流程技能优先** (brainstorming, debugging) — 这些决定 HOW 接近任务
2. **实现技能其次** (frontend-design, mcp-builder) — 这些指导执行

"Let's build X" → brainstorming first, then implementation skills.
"Fix this bug" → debugging first, then domain-specific skills.

## 技能类型

| 类型 | 特征 | 示例 |
|------|------|------|
| **Rigid（严格）** | 严格遵循。不要偏离纪律。 | TDD, debugging |
| **Flexible（灵活）** | 根据上下文调整原则。 | patterns |

技能本身会告诉你属于哪种。

## 红旗 — 你在理性化

| 想法 | 真相 |
|------|------|
| "这只是简单问题" | 问题是任务。检查技能。 |
| "我需要更多上下文" | 技能检查先于澄清问题。 |
| "让我先探索代码库" | 技能告诉你 HOW 探索。先检查。 |
| "我可以快速检查文件" | 文件缺乏对话上下文。检查技能。 |
| "这不需要正式技能" | 如果技能存在，使用它。 |
| "我记得这个技能" | 技能会进化。阅读当前版本。 |
| "这不算任务" | 行动 = 任务。检查技能。 |
| "技能大材小用" | 简单的事情会变得复杂。使用它。 |
| "我感觉有生产力" | 无纪律的行动浪费时间。技能防止这个。 |

## 链接

- [[Superpowers MOC]]
