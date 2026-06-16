---
date: 2026-06-15
tags:
  - 方法
  - Agent框架
  - 安全
  - Hermes
  - 审批机制
source: "[[OpenClaw与Hermes源码架构深度复盘]]"
---

# Smart Approval 三态审批

> [!quote] 核心定义
> Hermes 的 Smart Approval 用辅助 LLM 自动评估命令风险，返回三种结果：APPROVE（自动放行）、DENY（直接阻止）、ESCALATE（交给用户决定）。相当于在规则匹配和人工审批之间增加了一个"AI安全审查员"层——低风险自动放行，高风险自动阻止，不确定的才叫人。

---

## 三种结果的处理

| 结果 | 处理方式 |
|------|----------|
| **APPROVE** | 自动批准 + **会话级免审**（同一命令后续不再询问） |
| **DENY** | 直接阻止 + 返回"BLOCKED by smart approval" + **禁止重试** |
| **ESCALATE** | 降级为手动审批流程（交给用户决定） |

---

## 与 OpenClaw 审批对比

| 维度 | OpenClaw Exec Approval | Hermes Smart Approval |
|------|------------------------|----------------------|
| 方法 | 纯规则匹配+人工审批 | **LLM辅助风险评估+规则+人工** |
| 安全默认 | deny-by-default，逐命令审批 | Default/Auto/Plan三态模式 |
| 白名单 | 交互式加入 | 持久化到config.yaml |
| YOLO模式 | 无 | `enable_session_yolo()` 绕过所有审批 |
| 混淆检测 | 5种模式（curl-pipe-shell等） | 35条DANGEROUS_PATTERNS |

> [!tip] 设计取舍
> OpenClaw偏保守——宁可多问用户也不让AI决定。Hermes偏效率——让AI先分诊，只有不确定的才叫人。这是平台(OpenClaw)和工具(Hermes)定位差异在安全层的体现。

---

## 设计哲学：先 LLM 分诊再叫人

关键洞见：**不是所有命令都需要人审批**。`ls -la` 不需要，`rm -rf /` 不需要（应该直接阻止），只有像 `pip install new-package` 这种"不确定是否有风险"的才需要人判断。LLM 分诊把大多数低风险命令过滤掉，只把真正需要人判断的推给用户。

---

## 状态管理

- **per-session 状态**：线程安全，使用 `threading.Lock` + `contextvars`
- **持久白名单**：`config.yaml` 的 `command_allowlist` 跨会话存活
- **YOLO 模式**：`enable_session_yolo()` 绕过所有审批（仅当前会话）

---

## Tirith 预执行安全扫描

Hermes 还有 **Tirith**（Rust编写的外部安全扫描器），在命令执行前检测内容级威胁。退出码语义：`0=allow, 1=block, 2=warn`。安装时通过 SHA-256 校验验证完整性，有 `cosign` 还会验证 GitHub Actions 工作流签名（供应链验证）。

---

## 相关笔记

- [[Harness Engineering]] — 执行治理的通用框架，Smart Approval 是执行前拦截层
- [[OpenClaw vs Hermes 架构对比]] — 安全模型对比
- [[工程设计与安全]] — Agent安全的通用设计原则
- [[Channel Plugin 25+ Adapter 契约]] — OpenClaw的审批在Channel层实现
