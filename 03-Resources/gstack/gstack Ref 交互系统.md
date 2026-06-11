---
title: gstack Ref 交互系统
description: AI 代理如何通过 @e/@c 引用寻址网页元素 — 基于 ARIA 树 + Playwright Locators，而非 DOM 注入
date: 2026-05-19
tags: [gstack, Ref系统, 浏览器交互, ARIA, Playwright, 参考笔记]
---

# gstack Ref 交互系统

## 📖 定义

Refs（`@e1`, `@e2`, `@c1`）是 AI 代理寻址网页元素的方式——**无需编写 CSS 选择器或 XPath**。代理说 `$B click @e3`，服务器自动将 @e3 映射到对应的 Playwright Locator 并执行点击。

## 🔁 工作流程

```
1. 代理运行: $B snapshot -i
2. 服务器调用 Playwright 的 page.accessibility.snapshot()
3. 解析器遍历 ARIA 树，分配顺序引用: @e1, @e2, @e3...
4. 为每个 ref 构建 Playwright Locator: getByRole(role, { name }).nth(index)
5. 存储 Map<string, RefEntry>（role + name + Locator）
6. 返回标注后的树作为纯文本

7. 代理运行: $B click @e3
8. 服务器解析 @e3 → Locator → locator.click()
```

## 🧠 为什么用 Locators 而非 DOM 注入

直觉方法是在 DOM 中注入 `data-ref="@e1"` 属性。但这在三种场景下会失败：

| 场景 | DOM 注入问题 | Locator 方案 |
|------|-------------|-------------|
| **CSP（内容安全策略）** | 生产站点阻止 DOM 修改 | 外部于 DOM，无修改 |
| **React/Vue/Svelte 水合** | 框架对账会剥离注入属性 | 使用 ARIA 树，不触碰 DOM |
| **Shadow DOM** | 无法从外部触及 shadow root | getByRole() 跨 Shadow DOM 工作 |

**核心洞察：** Playwright Locators 是 DOM 外部的。它们使用 Chromium 内部维护的**无障碍树（Accessibility Tree）**和 `getByRole()` 查询。无需 DOM 修改、无 CSP 问题、无框架冲突。

## 🔄 Ref 生命周期

Refs 在**导航时清空**（`framenavigated` 事件）。这是正确的——导航后所有 Locator 都过期了。代理必须重新运行 `snapshot` 获取新 refs。

> 过期 refs 应**快速失败**，而非点击错误元素。

### SPA 场景下的过期检测

SPA 可在不触发 `framenavigated` 的情况下修改 DOM（React Router、tab 切换、modal 打开）。`resolveRef()` 在使用任何 ref 前执行异步 `count()` 检查：

```
resolveRef(@e3) → entry = refMap.get("e3")
                → count = await entry.locator.count()
                → if count === 0: "Ref @e3 is stale — element no longer exists. Run 'snapshot' to get fresh refs."
                → if count > 0: return { locator }
```

**~5ms 开销**，比让 Playwright 30 秒 action timeout 在缺失元素上到期好得多。`RefEntry` 存储 `role` 和 `name` 元数据，错误信息可告诉代理该元素**曾经是什么**。

## 📌 两类 Ref

| 类型 | 前缀 | 发现方式 | 用途 |
|------|------|---------|------|
| **ARIA Refs** | `@e1`, `@e2`... | ARIA 树标准元素 | 按钮、链接、输入框 |
| **Cursor-interactive Refs** | `@c1`, `@c2`... | `-C` flag：`cursor: pointer` / `onclick` / `tabindex` | 自定义组件渲染为 `<div>` 实际是按钮 |

`@c` refs 捕获框架渲染的**自定义组件**——它们在 ARIA 树中不可见，但确实可点击。

## 🔗 关联知识

- [[gstack 守护进程架构]] — 服务器如何处理命令
- [[gstack 项目概览]] — 整体架构
- [[智能体（Agent）]] — 代理感知-决策-行动循环
- [[ReAct 范式]] — 代理的观察-思考-行动范式

## 🏷️ 标签

#gstack #Ref系统 #浏览器交互 #ARIA #Playwright #参考笔记