---
title: gstack 安全架构
description: gstack 的三层安全设计 — 双监听器隧道隔离、L1-L6 分层提示注入防御、Unicode 清理管道
date: 2026-05-19
tags: [gstack, 安全, 提示注入, 隧道, Unicode, 参考笔记]
---

# gstack 安全架构

## 📖 定义

gstack 的安全设计包含三个独立但互补的系统：(1) 双监听器隧道架构隔离本地与远程访问；(2) L1-L6 分层提示注入防御保护侧边栏代理；(3) Unicode 清理管道防止孤立代理对触发 API 400 错误。

## 🏗️ 一、双监听器隧道架构

当 `pair-agent` 启动 ngrok 隧道让远程代理驱动浏览器时，暴露整个守护进程到互联网是不安全的。解决方案是**两个 HTTP 监听器**：

| 监听器 | 绑定 | 可达路径 | 安全属性 |
|--------|------|---------|---------|
| **Local** | `127.0.0.1:LOCAL_PORT` | 全部（/health, /command, /cookie-picker, /inspector...） | 从不转发 |
| **Tunnel** | `127.0.0.1:TUNNEL_PORT` | 仅 /connect + /command(scoped) + /sidebar-chat | ngrok 仅转发此端口 |

**安全属性来自物理端口分离**——隧道调用者无法到达 `/health` 或 `/cookie-picker`，因为这些路径在隧道 TCP socket 上不存在。Header 推断（x-forwarded-for）不可靠，socket 分离则不然。

| 端点 | Local | Tunnel |
|------|-------|--------|
| `GET /health` | public（token bootstrap） | **404** |
| `POST /command` | auth(root OR scoped) | auth(**scoped only**, allowlisted commands) |
| `POST /cookie-picker` | public UI | **404** |
| Root token on tunnel | — | **403** |

**模块边界约束：** `sse-session-cookie.ts` **不得导入** `token-registry.ts`——这是 scope isolation 的承重墙。

### SSE 会话 Cookie

EventSource 无法发送 Authorization 头。侧边栏 POST `/sse-session`（Bearer root）获取 30 分钟 HttpOnly cookie（`gstack_sse`, SameSite=Strict）。此 cookie **仅**对 `/activity/stream` 和 `/inspector/events` 有效——**不是** scoped token，不能用于 `/command`。

## 🏗️ 二、L1-L6 提示注入防御

侧边栏代理有工具（Bash, Read, WebFetch）且读取敌意网页，是 gstack 最易受提示注入的部分。防御是**分层的**：

| 层 | 模块 | 位置 | 机制 |
|----|------|------|------|
| **L1-L3** | `content-security.ts` | server + agent | 数据标记 + 隐藏元素剥离 + ARIA 正则 + URL 黑名单 + 信封包装 |
| **L4** | `security-classifier.ts` | **仅侧边栏代理** | TestSavantAI ONNX (22MB BERT-small)，本地扫描 |
| **L4b** | Haiku transcript classifier | **仅侧边栏代理** | 看完整对话形状， gated by LOG_ONLY: 0.40 |
| **L5** | `security.ts` (canary) | both | 随机 token 注入 system prompt，检测是否泄漏到输出 |
| **L6** | `security.ts` (combineVerdict) | both | 组合所有层的结果 |

### 关键阈值

| 阈值 | 值 | 含义 |
|------|---|------|
| `BLOCK` | 0.85 | 单层高分（需交叉确认才 BLOCK） |
| `WARN` | 0.75 | 交叉确认阈值：L4 AND L4b >= 0.75 → BLOCK |
| `LOG_ONLY` | 0.40 | 跳过 Haiku 调用（干净流量直接通过） |
| `SOLO_CONTENT_BLOCK` | 0.92 | 无标签分类器的单层阈值（故意更高） |

**集成规则：** BLOCK 需要 ML 内容分类器 AND 转录分类器**都** >= WARN。单层高分降级为 WARN——这是 Stack Overflow 指令式写作的 FP 缓解。Canary 泄漏**总是** BLOCK（确定性）。

### 关键约束

`security-classifier.ts` **不能**从编译 browse 二进制导入。`@huggingface/transformers` v4 需要 `onnxruntime-node`，而 Bun compile 的临时解压目录中 `dlopen` 会失败。只有纯字符串操作（canary, verdict, attack log）才能安全用于 `server.ts`。

### 环境开关

- `GSTACK_SECURITY_OFF=1` — 紧急关闭（跳过 ML扫描，canary 仍注入）
- `GSTACK_SECURITY_ENSEMBLE=deberta` — 启用 DeBERTa-v3 三分类器集成（721MB 首次下载）

## 🏗️ 三、Unicode 清理管道

页面 DOM 文本可能包含**孤立 UTF-16 代理对**（`\uD800–\uDFFF`）。Anthropic API 拒绝这些字符返回 400。

**核心洞察：** `JSON.stringify` 将 `\uD800` 转换为 `"\\ud800"` 逃逸序列。因此**正则表达式在 stringify 后是无效的**——清理必须在 stringify **过程中**处理。

```
解决方案：JSON.stringify(payload, sanitizeReplacer)
```

`sanitizeReplacer` 是一个 `JSON.stringify` replacer 函数，在编码过程中清理每个字符串值。

| 出口路径 | 清理点 |
|----------|--------|
| `POST /command` (HTTP) | `handleCommandInternal` 包装器 |
| `POST /command/batch` | 同上 |
| `GET /activity/stream` (SSE) | `sanitizeReplacer` → `JSON.stringify` |
| `GET /inspector/events` (SSE) | 同上 |

**架构不变量：** 每个新出口必须经过 `sanitizeReplacer` 或 `sanitizeLoneSurrogates`。绕过两者将破坏系统。

## 🔗 关联知识

- [[gstack 守护进程架构]] — 双监听器是守护进程的一部分
- [[gstack 项目概览]] — gstack 整体架构
- [[分层安全防御]] — L1-L6 模式的通用化
- [[MCP 服务器集成]] — MCP vs HTTP 的安全考量对比

## 🏷️ 标签

#gstack #安全 #提示注入 #隧道 #Unicode #参考笔记