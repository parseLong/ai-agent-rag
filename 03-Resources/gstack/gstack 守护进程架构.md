---
title: gstack 守护进程架构
description: gstack 的持久 Chromium 守护进程设计 — 子秒延迟、持久状态、自动生命周期管理
date: 2026-05-19
tags: [gstack, 守护进程, 架构, 浏览器, 参考笔记]
---

# gstack 守护进程架构

## 📖 定义

gstack 运行一个**长寿命 Chromium 守护进程**，CLI 通过 localhost HTTP 与其通信。核心洞察：AI 代理与浏览器交互需要**子秒级延迟**和**持久状态**。如果每次命令冷启动浏览器（3-5s），QA 会话 20+ 命令就是 40+ 秒纯启动开销。

## 🏗️ 架构图

```
Claude Code                     gstack
─────────                      ──────
                               ┌──────────────────────┐
  Tool call: $B snapshot -i    │  CLI (compiled binary)│
  ─────────────────────────→   │  • reads state file   │
                               │  • POST /command      │
                               │    to localhost:PORT   │
                               └──────────┬───────────┘
                                          │ HTTP
                               ┌──────────▼───────────┐
                               │  Server (Bun.serve)   │
                               │  • dispatches command  │
                               │  • talks to Chromium   │
                               │  • returns plain text  │
                               └──────────┬───────────┘
                                          │ CDP
                               ┌──────────▼───────────┐
                               │  Chromium (headless)   │
                               │  • persistent tabs     │
                               │  • cookies carry over  │
                               │  • 30min idle timeout  │
                               └───────────────────────┘
```

## 🔑 核心设计

### 状态文件

服务器写入 `.gstack/browse.json`（原子写入：tmp + rename，mode 0o600）：

```json
{ "pid": 12345, "port": 34567, "token": "uuid-v4", "startedAt": "...", "binaryVersion": "abc123" }
```

CLI 读取此文件找服务器。文件缺失或健康检查失败 → CLI 自动启动新服务器。

### 随机端口选择

10000-60000 随机端口（碰撞时最多重试 5 次）。10 个 Conductor workspace 各自运行独立 browse 守护进程，零配置零冲突。

### 版本自动重启

`bun build` 写 `git rev-parse HEAD` 到 `browse/dist/.version`。每次 CLI 调用时，如果二进制版本与运行服务器不匹配 → 自动杀死旧服务器并启动新的。**彻底消除"过期二进制"类 bug**。

### 崩溃恢复

服务器不自愈。Chromium 崩溃 → 服务器立即退出。CLI 在下次命令时检测到死服务器并自动重启。**比尝试重连半个死掉的浏览器进程更简单可靠**。

## 🧠 为什么是守护进程而非每次启动

| 方案 | 启动延迟 | 状态保持 | 适用场景 |
|------|---------|---------|---------|
| 每次冷启动 | 3-5s | 无（每次丢失 cookies/tab） | 单次截图 |
| 守护进程 | ~3s（首次）→ 100-200ms（后续） | 持久 | QA 会话 20+ 命令 |

守护进程模式意味着：
- **持久状态**：登录一次保持登录，tab 保持打开
- **子秒命令**：首次后每命令仅 HTTP POST
- **自动生命周期**：首次使用自动启动，30 分钟空闲自动关闭

## 📊 日志架构

三个环形缓冲区（50,000 条/个，O(1) push）：

```
Browser events → CircularBuffer (in-memory) → Async flush to .gstack/*.log
```

- HTTP 请求处理**永远不被磁盘 I/O 阻塞**
- 1 秒间隔异步刷盘（最多丢失 1 秒数据）
- 内存有界（50K × 3 缓冲区）
- 磁盘文件 append-only，外部工具可读

## 🔗 关联知识

- [[gstack 项目概览]] — gstack 整体架构
- [[gstack 安全架构]] — 双监听器隧道 + Bearer Token
- [[守护进程模式]] — 通用架构模式提取
- [[gstack Ref 交互系统]] — 浏览器内元素寻址

## 🏷️ 标签

#gstack #守护进程 #架构 #浏览器 #参考笔记