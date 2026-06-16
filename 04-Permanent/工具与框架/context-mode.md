---
title: context-mode
description: MCP 工具输出沙箱化 + 跨compact会话连续性——四层能力（沙箱/连续性/脚本代替读文件/不干预输出），工具输出压缩98%
tags: [工具, Token成本, 上下文压缩, 智能体, MCP, 工程实践]
aliases:
  - ctx-doctor
  - ctx-stats
date: 2026-06-15
source: "[[一篇搞懂 AI Coding Agent 的 Token 成本控制]]"
github: https://github.com/mksglu/context-mode
---

# context-mode

> MCP 工具调用的返回值太大，40% 上下文 30 分钟就没了——context-mode 四层能力一起上，工具输出压缩 98%。

## 🧠 核心痛点

一次 Playwright 快照 56KB，20 条 GitHub issue 59KB，一份访问日志 45KB。用 30 分钟 40% 上下文就没了。然后 `/compact` 一压缩，Agent 忘了在改哪些文件、任务进行到哪了。

## 🔑 四层能力

### 能力一：Sandbox 工具输出（98% 压缩）

工具调用结果先沙箱化，不直接进上下文。315KB → 5.4KB。LLM 按需取回原始数据，不是真的删掉。

### 能力二：跨 compact 会话连续性

所有文件编辑、git 操作、任务进度、错误信息记录到本地 SQLite。`/compact` 后不丢失现场——用 FTS5 全文索引按相关性取回，而不是把所有历史重新塞回去。

### 能力三：用代码代替读文件

`ctx_execute()` 工具让 Agent 写脚本处理数据，而不是读 50 个文件进上下文：

```javascript
// Before: 47 次 Read() = 700 KB
// After: 1 次 ctx_execute() = 3.6 KB
ctx_execute("javascript", `
  const files = fs.readdirSync('src').filter(f => f.endsWith('.ts'));
  files.forEach(f => {
    const lines = fs.readFileSync('src/' + f, 'utf8').split('\\n').length;
    console.log(f + ': ' + lines + ' lines');
  });
`);
```

Agent 生成并运行脚本，只把结果放进上下文，而不是把 47 个文件的内容全部读进来。

### 能力四：不干预输出格式

context-mode 只管数据往哪走，不管 Agent 怎么回答。与 [[Caveman]] 各管各的，不冲突。

## 🛠️ 安装与使用

```bash
npm install -g context-mode
```

> [!note] CodeBuddy 支持
> 使用 [studyzy Fork 版](https://github.com/studyzy/context-mode)（已增加 CodeBuddy 支持）。

重启 CodeBuddy 后生效。

### 验证

```
/context-mode:ctx-doctor
```

所有项显示 `[x]` 即正常。

### 常用命令

```
/context-mode:ctx-stats    # 查看节省统计，按工具分类
/context-mode:ctx-insight  # 打开本地分析看板（90 个指标）
```

## 🔗 关联知识

- [[AI Coding Agent Token 成本控制]] — 五层优化框架中第三层的具体工具
- [[RTK]] — 压缩终端命令输出
- [[Caveman]] — 压缩输出端，与 context-mode 不冲突各管各的
- [[headroom]] — 压缩所有进上下文的内容
- [[上下文压缩]] — 第三层优化的理论基础
- [[MCP]] — context-mode 的治理对象
- [[上下文重置与交接]] — 跨 compact 的连续性问题

## 🏷️ 标签

#工具 #Token成本 #上下文压缩 #智能体 #MCP #工程实践
