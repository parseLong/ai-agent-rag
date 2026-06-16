---
date: 2026-06-15
tags:
  - 概念
  - Agent框架
  - 源码分析
  - OpenClaw
  - 凭证管理
  - 错误处理
source: "[[OpenClaw与Hermes源码架构深度复盘]]"
---

# Auth Profile 与 FailoverError

> [!quote] 核心定义
> OpenClaw 把凭证管理从"API Key数组"升级为"带健康状态的对象"（Auth Profile），把错误分类从"抓异常重试"升级为"闭合枚举驱动策略"（FailoverError）。两者组合实现了**可恢复错误的处理是静态可证明的，不靠LLM猜**。

---

## Auth Profile — 不只是"API Key数组"

### 真实场景

你有3个Anthropic账号——个人Pro、公司Max、AWS Bedrock。想自动管理：Pro额度用完切Max，Max被限频切Bedrock，恢复了自动切回来。

**Hermes做不到**——Credential Pool只是API Key数组，按顺序试，不知道为什么失败，也不记得"上次哪个key挂了"。

### 数据结构

```
type AuthProfileCredential =
  | ApiKeyCredential       // { key, provider }          ← 最简单
  | TokenCredential        // { token, expiresAt }       ← 会过期
  | OAuthCredential;       // { clientId, refreshToken } ← 能自动刷新，最持久

type ProfileUsageStats = {
  lastUsed: number;
  cooldownUntil: number;          // 临时退避：30s→1min→5min（指数退避）
  cooldownReason: "rate_limit" | "overloaded" | "billing" | ...;
  disabledUntil: number;          // 永久型错误（key被吊销）
  failureCounts: Record<FailureReason, number>;
};
```

### 选取策略（4原则）

1. **类型偏好**：`oauth > token > api_key`（能自动刷新的优先）
2. **均衡轮转**：同类型按 `lastUsed` 升序（不让一个key被打爆）
3. **冷却探针**：冷却中的排末尾，到期自动试一次——恢复了回主队列
4. **用户锁定**：显式指定的 `preferredProfile` 永远优先

### 实际行为差异

```
场景：用Profile A调Claude，返回429 rate_limit

Hermes的做法：
  retry → retry → retry → 超时报错（不知道该切key）

OpenClaw的做法：
  ① 识别错误类型 = rate_limit（凭证类）
  ② Profile A标记冷却30s
  ③ 50ms内切到Profile B
  ④ 用户无感知，对话继续
  ⑤ 30s后Profile A自动"探针重试"——如果恢复加回可用队列
```

### 与Hermes Credential Pool的关键差异

| 维度 | Hermes | OpenClaw |
|------|--------|----------|
| 抽象层级 | API Key数组 | 凭据+健康状态+来源 |
| 凭据类型 | 只支持api_key | api_key, token(过期)/oauth(刷新) |
| 持久化 | 进程内内存（重启丢失） | **磁盘store**（重启保留冷却状态） |
| 外部同步 | 无 | 自动发现本地claude-cli/codex已登录账号 |
| 选取逻辑 | 线性从头到尾试 | **round-robin+冷却队列+类型偏好+用户锁定** |
| 冷却策略 | 统一计数 | **按FailoverReason分级退避** |

> [!important] 生产体验差异
> Hermes重启后→不记得哪个key上次挂了→又去撞已知欠费的key→白等30s
> OpenClaw重启后→磁盘store里Profile A还标着"billing冷却到14:30"→直接跳过→0ms恢复

---

## FailoverError — 把错误分类做成结构化契约

### 13种闭合枚举

```
type FailoverReason =
  | "billing"          // 402
  | "rate_limit"       // 429
  | "overloaded"       // 503
  | "auth"             // 401（可刷新）
  | "auth_permanent"   // 403（禁用）
  | "timeout"          // 408 / ETIMEDOUT / ECONNRESET...
  | "format"           // 400（payload问题）
  | "model_not_found"  // 404
  | "session_expired"; // 410
  // ... 等13种
```

### 分类器是递归的

逐级走 HTTP status → 符号码 → errno → cause链 → timeout heuristics，容忍不同Provider SDK的错误表达差异。

### 分类结果驱动不同策略

| 策略 | 适用reason | 效果 |
|------|-----------|------|
| 允许探针式重试冷却profile | rate_limit, overloaded, billing |
| 走临时slot | 瞬时错误(rate_limit, overloaded) |
| 保留slot | 永久错误(auth_permanent, session_expired) |

> **甚至不是API调用的错误也会被翻译成FailoverError**——context_length_exceeded, session_expired, model_not_found全走同一条路。这样 `runWithModelFallback` 能用统一契约处理所有可恢复错误。

---

## 与外层的唯一契约

主循环里所有可恢复错误最终都 `throw new FailoverError(reason, ...)`，调用者 `runWithModelFallback` 只接 FailoverError（instanceof 匹配就换模型，否则直接抛）。

> **`runEmbeddedPiAgent` 是"FailoverError工厂"，`runWithModelFallback` 是"消费者"——两者只通过这一个错误类型交流**

---

## Live Model Switch 的幂等条件

只有**完全干净的attempt**（没发消息、没执行工具、没产生assistant文本、没审批提示、没工具错误）才允许实时切模型。一旦对外产生过影响，切模型重来会导致重复发送或不可撤销操作。

---

## 相关笔记

- [[OpenClaw架构深度解析]] — Gateway和执行引擎的整体架构
- [[OpenClaw vs Hermes 架构对比]] — 执行引擎对比表
- [[上下文预算与资源分配]] — OpenClaw 10种稀缺资源量化
- [[Harness Engineering]] — 执行治理的通用框架
