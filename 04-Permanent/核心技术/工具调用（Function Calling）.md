---
title: Function Calling
description: 让LLM从"生成文本"到"调用工具"的关键技术——模型输出结构化调用指令而非自然语言
tags: [概念卡片, 永久笔记, 概念, 智能体, 工具, Function Calling]
date: 2026-05-22
aliases:
  - 函数调用
  - Tool Use
  - 工具调用
---

# Function Calling

## 📖 定义

Function Calling 是让 LLM 在"该调用工具时"输出==结构化的调用指令==（函数名+参数JSON），而非自然语言文本的技术。核心要点：**LLM本身并不执行工具，它只负责"决策"（选择工具和生成参数），实际执行由应用侧完成**。这种决策-执行分离的设计确保了安全性和可控性。

## 🧠 我的理解

Function Calling 就像给LLM装上了"手脚的遥控器"——LLM不会自己动手，但它能准确告诉系统"该用哪个工具、该怎么用"。本质上是在LLM的输出空间中增加了一种新格式：除了文本回复，还可以输出工具调用指令。

## 🎯 为什么重要

这是Agent演进中从"说"到"做"的关键一步。没有Function Calling，LLM只能告诉你答案但无法执行操作；有了它，LLM可以调用天气API、操作数据库、发送邮件——从信息提供者变成行动执行者。

## 🔑 Function Calling 的核心痛点

> 来源：[[Agent核心技术概念与范式发生了哪些演变以及背后的思考]]

Function Calling 作为早期 Agent 工具调用的主流范式，存在显著的痛点：

| 痛点 | 说明 |
|------|------|
| **极高的开发维护成本** | ==现实中大量的系统或数据源并没有现成的 API 可供调用==，团队往往需要投入大量精力去"补全" API |
| **API Schema 管理膨胀** | ==随着工具数量的膨胀，API Schema 的管理变得极其复杂==，维护成本居高不下 |
| **人为适配模型** | 需要针对具体业务场景，将系统能力封装成标准的 API——本质是"人为适配模型"，而非利用模型原生能力 |

> [!important] 痛点的根源
> Function Calling 的核心问题在于：它要求人类为模型"铺路"——把所有能力都封装成 API 才能让模型使用。但现实中大量系统没有现成 API，这就导致了巨大的"铺路成本"。而 CLI + Script 模式从根本上改变了这一逻辑：==从"人为适配模型"转向"利用模型原生能力"==。

## 🔑 演进时间线

| 时间 | 里程碑 | 关键能力 |
|------|--------|---------|
| 2023.06 | OpenAI 引入 Function Calling | 单函数调用，模型输出函数名+参数JSON |
| 2023.11 | Parallel Function Calling | 一次响应可并行调用多个函数 |
| 2024.01 | 各厂商跟进 | Anthropic Tool Use、Google Function Calling、通义千问 |
| 2024.08 | Structured Outputs | 保证输出100%符合JSON Schema |
| 2024.11 | MCP发布（Anthropic） | 标准化工具协议，脱离具体LLM平台 |
| 2025.03 | MCP Streamable HTTP | 新传输层，支持无状态服务器 |

## 🔑 交互流程

### 基础五步流程

```
Step 1: 定义工具 schema → tools列表
Step 2: LLM 返回工具调用指令 → tool_calls: [{name, arguments}]
Step 3: 应用侧执行工具 → query_weather(city="杭州")
Step 4: 工具结果回传 → messages.append(role="tool", content=result)
Step 5: LLM 生成最终回答 → "杭州今天天气多云，气温25°C"
```

### FC 生命周期四步模型（Harness 视角）

> 来源：[[万字干货：理解 Harness Engineering，看这一篇就够了]]

从 Harness（驾驭层）视角，Function Calling 的完整生命周期包含四个严密且脆弱的环节：

| 步骤 | 说明 | 脆弱性 |
|------|------|--------|
| **① Schema 序列化** | Harness 将可用工具列表及其参数定义（JSON Schema）序列化为特定格式文本，注入 Prompt。这是 LLM 理解其"能力边界"的唯一途径 | 工具描述的准确性直接影响 LLM 的工具选择 |
| **② 触发生成** | LLM 在参数空间中"模式匹配"，认为某工具能满足当前规划时，生成一段包含工具名和参数值的特定语法文本 | LLM 可能选择错误工具或生成格式不规范的参数 |
| **③ 确定性反序列化** | Harness 捕获 LLM 生成的文本，尝试将其反序列化为结构化调用请求 | **最脆弱的环节**——LLM 生成可能不完全符合语法（JSON格式错误、参数类型错误），5-10%格式错误率 |
| **④ 观测注入** | Harness 执行调用，将结果（成功或失败）封装成"观测"文本，注入 Prompt 完成闭环 | 执行可能失败，需将错误信息反馈给 LLM |

> [!important] 失败面与降级路径
> 由于 LLM 生成的非确定性，FC 每一步都可能失败。稳健的 Harness 必须为这些失败设计降级路径：
>
> **反序列化失败**：
> - 重试：向 LLM 提供错误信息（如 "Invalid JSON format"），要求其重新生成
> - 回退到文本：放弃 FC，转而要求 LLM 生成自然语言指令，由传统解析器处理
>
> **执行失败**：
> - 交互式补充：因参数缺失导致失败时，向用户请求补充信息
> - 反思与重规划：将详细错误信息注入上下文，引导 Agent 反思失败原因并选择其他路径

> [!note] FC 生命周期与本文已有的修复策略的关系
> FC 生命周期四步模型是**理论框架**——描述了 FC 从定义到执行的完整流程及其脆弱性。本文已有的工具名修复四策略和参数 JSON 修复三策略是**具体实现**——针对步骤③反序列化失败的具体修复方案。两者互补：生命周期模型告诉你"在哪一步会出什么错"，修复策略告诉你"出了错怎么修"。

> [!example] 代码交互示例
> ```python
> # Step 1: 定义 schema
> tools = [{"type":"function","function":{"name":"query_weather",
>   "description":"查询指定城市的天气","parameters":{"type":"object",
>   "properties":{"city":{"type":"string","description":"城市名"}},
>   "required":["city"]}}}]
> # Step 2: LLM返回调用指令（非文本）
> resp = llm.chat(messages=[{"role":"user","content":"杭州天气如何"}], tools=tools)
> # → tool_calls: [{"function":{"name":"query_weather","arguments":"{\"city\":\"杭州\"}"}}]
> # Step 3-4: 执行并回传结果
> result = query_weather(city="杭州")  # "杭州今天25°C，多云"
> messages.append({"role":"tool","content":result,"tool_call_id":"..."})
> # Step 5: LLM综合结果生成最终回答
> final = llm.chat(messages=messages, tools=tools)
> # → "杭州今天天气多云，气温25°C，适合出行。"
> ```

## 🔑 Function Calling vs MCP

| 维度 | Function Calling | MCP |
|------|-----------------|-----|
| **标准化** | 各厂商格式不同 | 统一协议标准 |
| **工具发现** | 硬编码在应用中 | 动态 tools/list（运行时发现） |
| **可复用性** | 换平台需改代码 | 一个Server服务所有Agent |
| **额外能力** | 仅函数调用 | Resources + Prompts + Sampling |
| **部署复杂度** | 无额外组件 | 需要部署MCP Server |
| **适用场景** | 工具少于5个且只对接一个LLM | 工具超过10个、需要跨平台复用 |

> [!tip] 选择建议
> - 工具 < 5个 + 单LLM平台 → 直接用 Function Calling
> - 工具 > 10个 + 需跨平台复用 + 多团队提供工具 → MCP 更好

### MCP 架构与协议

MCP 采用 **Host-Client-Server** 三层架构，实现工具的即插即用：

```
┌── Host (Claude Desktop / IDE / Agent框架) ──────────────┐
│  Client A ──── Client B ──── Client C                    │
└──────┬──────────────┬──────────────┬─────────────────────┘
       ▼              ▼              ▼
  Server A          Server B        Server C
  (文件系统)         (数据库)        (SLS日志)
  read/write        query/insert    search/alert
```

- **Host**：运行环境，管理多个 Client 实例的生命周期
- **Client**：维护与单个 Server 的 1:1 连接，处理协议协商和消息路由
- **Server**：工具提供方，暴露 capabilities

协议层使用 **JSON-RPC 2.0**，核心操作：

```json
// 发现工具: {"method":"tools/list", "params":{}}
// → {"tools":[{"name":"query_logs","description":"查询SLS日志","inputSchema":{...}}]}
// 调用工具: {"method":"tools/call", "params":{"name":"query_logs","arguments":{"query":"ERROR"}}}
// → {"content":[{"type":"text","text":"找到3条错误..."}]}
```

MCP 提供四类能力，远超 Function Calling 的单一函数调用：

| 能力 | 说明 | 对应操作 |
|------|------|---------|
| **Tools** | 可执行的函数/操作 | `tools/list`, `tools/call` |
| **Resources** | 可读取的数据源（文件、数据库） | `resources/list`, `resources/read` |
| **Prompts** | 预定义的 prompt 模板 | `prompts/list`, `prompts/get` |
| **Sampling** | Server 反向调用 LLM（需要 AI 能力时） | `sampling/createMessage` |

## 🔑 常见挑战与解决方案

| 挑战 | 问题 | 解决方案 |
|------|------|---------|
| **工具选择准确性** | 工具多时LLM选错 | 渐进式加载（Skill方案）、优化工具描述 |
| **参数提取可靠性** | JSON格式错误、缺必填字段 | Structured Outputs、参数修复、重试 |
| **嵌套参数包装** | LLM展平嵌套参数 `{query:...}` → 应为 `{request:{query:...}}` | 自动包装检测（本项目方案） |
| **工具名拼写** | 大小写/格式错误 | 模糊匹配 + 自动修复（大小写归一化、camelCase→snake_case、strip_tool_suffix） |
| **上下文占用** | 几十个工具schema占满prompt | 按需注入 + Skill分级加载 |

> [!important] LLM 工具调用的 5–10% 格式错误率
> 在生产环境中，LLM 返回的工具调用指令约有 **5–10%** 存在格式问题——工具名拼写不一致、参数 JSON 格式错误、嵌套参数被展平等。如果不做修复，这些调用会直接失败，导致用户体验断崖式下降。下面的修复策略完全透明：用户看不到错误，LLM 也不感知修复，调用链无缝继续。

> [!example] 工具名自动修复——四策略递进
> LLM 经常输出与 schema 定义不一致的工具名（大小写差异、后缀多余、拼写偏差），需逐级尝试修复：
>
> | 策略 | 逻辑 | 示例 |
> |------|------|------|
> | **策略 1：直接小写 + snake_case 匹配** | 将 LLM 输出的名称转为小写 snake_case，与已注册工具名比对 | `QueryWeather` → `query_weather` ✓ |
> | **策略 2：标准化替换** | 替换 `-` 和空格为 `_`，再匹配 | `query-weather` → `query_weather` ✓ |
> | **策略 3：候选集交叉组合** | 对同一名称同时做 camelToSnake、stripSuffix（去掉 `_tool`/`_api` 等后缀）、normalize，与工具列表交叉匹配 | `query_weather_tool` → `query_weather` ✓ |
> | **策略 4 兜底：difflib 模糊匹配** | 用 difflib 计算相似度，≥70% 即命中 | `qeury_weather` → `query_weather` ✓（拼写偏差仍可修复） |
>
> **修复案例汇总**：
> | LLM 输出 | 修复后 | 命中策略 |
> |----------|--------|----------|
> | `QueryWeather` | `query_weather` | 策略 1 |
> | `QUERY_WEATHER` | `query_weather` | 策略 1 |
> | `query_weather_tool` | `query_weather` | 策略 3 |
> | `qeury_weather` | `query_weather` | 策略 4 |

> [!example] 参数 JSON 自动修复——三策略递进
> LLM 生成的 arguments JSON 同样存在格式问题，需依次尝试修复：
>
> | 策略 | 逻辑 | 示例 |
> |------|------|------|
> | **策略 1：清理 surrogate 字符** | LLM 常生成无效 Unicode（surrogate），需清除后再解析 | 含 `\uD800` 等无效字符 → 清除后正常解析 |
> | **策略 2：补全缺失右括号** | 检测括号不匹配，自动补全 `}` | `{"key": "val"` → `{"key": "val"}` ✓ |
> | **策略 3：删除尾随逗号** | 移除 JSON 末尾或元素间的多余逗号 | `{"key": "val",}` → `{"key": "val"}` ✓ |
> | **全部失败兜底** | 返回原始字符串包裹为 `{raw_input: ...}`，供 LLM 在下一轮自行查看并修正 | 无法修复 → `{"raw_input": "原始字符串"}` |
>
> 三策略按顺序执行，任一成功即停止；全部失败则走兜底，让 LLM 在后续对话中自行纠错。

> [!example] 嵌套参数自动包装
> MCP 工具的 inputSchema 常有嵌套包装结构，例如 `{request: {query: ...}}`，但 LLM 倾向于将嵌套展平为 `{query: ...}`，导致参数校验失败。
>
> **自动检测与包装逻辑**：
> 1. 检查 inputSchema 的 `properties` 是否只有 **1 个 key**，且其 `type = "object"`
> 2. 如果是，取出该 key 下属的 `inner_props`（即嵌套内部的属性集合）
> 3. 比对 LLM 传入的 arguments 的 keys 是否与 `inner_props` 匹配
> 4. 匹配则自动包装：`{query: ...}` → `{request: {query: ...}}`
>
> ```python
> # 示例：inputSchema = {request: {query: str, page: int}}
> # LLM arguments  = {"query": "error", "page": 1}
> # 自动包装后     = {"request": {"query": "error", "page": 1}}
> ```
>
> 这一修复确保了 MCP 工具调用的参数结构始终符合 schema 定义，无需在 prompt 中反复提醒 LLM 保持嵌套格式。

## 🔗 关联知识

- **演进方向**：[[MCP]] — Function Calling的标准化升级版
- **应用范式**：[[ReAct 范式]] — Function Calling是ReAct中"Act"的具体实现
- **管理策略**：[[Skills技能]] — 用Skill渐进式加载解决工具过多问题
- **安全层面**：[[驾驭工程]] — 工具调用的权限检查和自动修复属于Harness
- **通信协议**：[[智能体通信协议]] — Function Calling在协议层级中的定位

## 📝 个人笔记

Function Calling 和 MCP 本质上没有技术差异——MCP是Function Calling的标准化版本，核心是约定一套标准规范让大家使用更方便。从Function Calling到MCP的演进，和从HTTP到REST的演进类似：不是技术突破，而是规范统一。

## 🏷️ 标签

#概念卡片 #永久笔记 #概念 #智能体 #工具 #FunctionCalling