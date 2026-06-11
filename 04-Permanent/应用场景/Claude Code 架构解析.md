

# AI Agent 架构演进：从单代理到自主团队

本文档梳理了一个 AI Agent 系统从简单到复杂的完整演进过程，展示每个阶段解决的核心问题和架构变化。

---

## 目录

1. [基础组件：待办与子代理](#1-基础组件待办与子代理)
2. [上下文管理：三层压缩系统](#2-上下文管理三层压缩系统)
3. [异步执行：后台任务管理](#3-异步执行后台任务管理)
4. [团队协作：持久化队友](#4-团队协作持久化队友)
5. [协作协议：请求-响应模式](#5-协作协议请求-响应模式)
6. [自主性：自驱动工作](#6-自主性自驱动工作)
7. [任务隔离：Worktree 并行执行](#7-任务隔离worktree-并行执行)

---

## 1. 基础组件：待办与子代理

### 1.1 AgentLoop-React

最简单的代理核心是一个 while 循环加上一个工具

```
+--------+      +-------+      +---------+

|  User  | ---> |  LLM  | ---> |  Tool   |

| prompt |      |       |      | execute |

+--------+      +---+---+      +----+----+

                    ^                |

                    |   tool_result  |

                    +----------------+

                    (loop until stop_reason != "tool_use")
```

### 1.2 TodoManager - 简单待办系统

**解决问题**：管理代理的工作清单，确保一次只专注一件事。

**核心设计**：

- 最多 20 条待办
- 状态：pending / in_progress / completed
- 约束：同时只能有一个 in_progress

```python
class TodoManager:
    def __init__(self):
        self.items = []

    def update(self, items: list) -> str:
        if len(items) > 20:
            raise ValueError("Max 20 todos allowed")
        
        in_progress_count = 0
        for item in items:
            if item["status"] == "in_progress":
                in_progress_count += 1
        
        if in_progress_count > 1:
            raise ValueError("Only one task can be in_progress at a time")
        
        self.items = items
        return self.render()

    def render(self) -> str:
        lines = []
        for item in self.items:
            marker = {"pending": "[ ]", "in_progress": "[>]", "completed": "[x]"}[item["status"]]
            lines.append(f"{marker} #{item['id']}: {item['text']}")
        return "\n".join(lines)
```

**使用示例**：

```python
todo = TodoManager()
todo.update([
    {"id": "1", "text": "设计 API 接口", "status": "completed"},
    {"id": "2", "text": "实现业务逻辑", "status": "in_progress"},
    {"id": "3", "text": "编写测试用例", "status": "pending"}
])
```

### 1.3 Subagent - 子代理执行器

**解决问题**：将复杂任务委派给隔离的执行环境，避免污染主上下文。

**核心设计**：
- 独立的上下文（不继承父代理历史）
- 最多 30 轮执行
- 只返回最终摘要（中间过程丢弃）

```python
def run_subagent(prompt: str) -> str:
    # 全新上下文
    sub_messages = [{"role": "user", "content": prompt}]
    
    for _ in range(30):  # 安全限制
        response = client.messages.create(
            model=MODEL,
            system=SUBAGENT_SYSTEM,
            messages=sub_messages,
            tools=CHILD_TOOLS,
            max_tokens=8000,
        )
        
        sub_messages.append({"role": "assistant", "content": response.content})
        
        # 任务完成
        if response.stop_reason != "tool_use":
            break
        
        # 执行工具
        results = []
        for block in response.content:
            if block.type == "tool_use":
                output = TOOL_HANDLERS[block.name](**block.input)
                results.append({"type": "tool_result", "tool_use_id": block.id, "content": str(output)})
        
        sub_messages.append({"role": "user", "content": results})
    
    # 只返回最终文本
    return "".join(b.text for b in response.content if hasattr(b, "text"))
```

**委派模式**：

```python
# 父代理决定委派
response = client.messages.create(
    model=MODEL,
    system=SYSTEM,
    messages=messages,
    tools=[{"name": "task", "description": "Delegate to subagent", ...}]
)

# 如果调用 task 工具
if tool_name == "task":
    output = run_subagent(tool_input["prompt"])
```

---

## 2. 上下文管理：三层压缩系统

### 解决问题

对话上下文无限增长会超出 token 限制，需要智能压缩。

### 三层压缩架构

```
每轮执行
    │
    ▼
[Layer 1: micro_compact]
静默执行，替换旧的 tool_result
    │
    ▼
检查 tokens > 50000 ?
    │           │
   no          yes
    │           │
    │           ▼
    │    [Layer 2: auto_compact]
    │    存档 + 生成摘要
    │           │
    │           ▼
    │    [Layer 3: compact]
    │    代理主动触发
    │
    └───────► 继续执行
```

### Layer 1: micro_compact

**策略**：保留最近 3 条 tool_result，更早的替换为占位符。

```python
KEEP_RECENT = 3

def micro_compact(messages: list) -> list:
    # 收集所有 tool_result
    tool_results = []
    for msg_idx, msg in enumerate(messages):
        if msg["role"] == "user" and isinstance(msg.get("content"), list):
            for part_idx, part in enumerate(msg["content"]):
                if isinstance(part, dict) and part.get("type") == "tool_result":
                    tool_results.append((msg_idx, part_idx, part))
    
    if len(tool_results) <= KEEP_RECENT:
        return messages
    
    # 建立工具名映射
    tool_name_map = {}
    for msg in messages:
        if msg["role"] == "assistant":
            for block in msg.get("content", []):
                if block.type == "tool_use":
                    tool_name_map[block.id] = block.name
    
    # 替换旧结果
    for _, _, result in tool_results[:-KEEP_RECENT]:
        tool_name = tool_name_map.get(result.get("tool_use_id"), "unknown")
        result["content"] = f"[Previous: used {tool_name}]"
    
    return messages
```

**效果**：

```
原始（50000 tokens）:
  tool_result: "完整的文件内容..." (40000 tokens)
  tool_result: "搜索结果..." (8000 tokens)
  tool_result: "命令输出..." (2000 tokens)

压缩后（5000 tokens）:
  tool_result: "[Previous: used read_file]"
  tool_result: "[Previous: used bash]"
  tool_result: "命令输出..." (2000 tokens)
```

### Layer 2: auto_compact

**触发条件**：tokens > 50000

```python
THRESHOLD = 50000

def auto_compact(messages: list) -> list:
    # 1. 存档完整历史
    TRANSCRIPT_DIR.mkdir(exist_ok=True)
    transcript_path = TRANSCRIPT_DIR / f"transcript_{int(time.time())}.jsonl"
    for msg in messages:
        with open(transcript_path, "a") as f:
            f.write(json.dumps(msg, default=str) + "\n")
    
    # 2. 生成摘要
    response = client.messages.create(
        model=MODEL,
        messages=[{
            "role": "user",
            "content": "Summarize: 1) What accomplished 2) Current state 3) Key decisions"
        }]
    )
    summary = response.content[0].text
    
    # 3. 重置消息
    return [
        {"role": "user", "content": f"[Compressed. Transcript: {transcript_path}]\n{summary}"},
        {"role": "assistant", "content": "Understood. Continuing."}
    ]
```

### Layer 3: compact 工具

**用途**：代理主动触发压缩。

```python
# 工具定义
{"name": "compact", "description": "Force context compression", ...}

# 执行逻辑
if tool_name == "compact":
    messages[:] = auto_compact(messages)
    output = "Compressed"
```

### 整合到主循环

```python
def agent_loop(messages: list):
    while True:
        # 每轮执行微压缩
        micro_compact(messages)
        
        # 检查是否需要自动压缩
        if estimate_tokens(messages) > THRESHOLD:
            messages[:] = auto_compact(messages)
        
        # 调用 LLM
        response = client.messages.create(...)
        
        # 处理工具调用
        if response.stop_reason != "tool_use":
            return
        
        # 执行工具...
        # 如果是 compact，触发压缩
        if tool_name == "compact":
            messages[:] = auto_compact(messages)
```

---

## 3. 异步执行：后台任务管理

### 解决问题

长时间命令（构建、测试）会阻塞主循环，需要异步执行。

### 核心设计

```python
class BackgroundManager:
    def __init__(self):
        self.tasks = {}  # task_id -> {status, result}
        self._notification_queue = []  # 完成通知队列
        self._lock = threading.Lock()
    
    def run(self, command: str) -> str:
        """启动后台任务，立即返回 task_id"""
        task_id = str(uuid.uuid4())[:8]
        self.tasks[task_id] = {"status": "running", "result": None}
        
        # 启动守护线程
        thread = threading.Thread(
            target=self._execute,
            args=(task_id, command),
            daemon=True
        )
        thread.start()
        
        return f"Background task {task_id} started"
    
    def _execute(self, task_id: str, command: str):
        """后台线程执行"""
        try:
            r = subprocess.run(command, shell=True, capture_output=True, timeout=300)
            output = (r.stdout + r.stderr).strip()
            status = "completed"
        except subprocess.TimeoutExpired:
            output = "Timeout"
            status = "timeout"
        except Exception as e:
            output = str(e)
            status = "error"
        
        # 更新状态
        self.tasks[task_id]["status"] = status
        self.tasks[task_id]["result"] = output
        
        # 推送通知
        with self._lock:
            self._notification_queue.append({
                "task_id": task_id,
                "status": status,
                "result": output[:500]
            })
    
    def check(self, task_id: str) -> str:
        """查询任务状态"""
        t = self.tasks.get(task_id)
        return f"[{t['status']}] {t.get('result', 'running')}"
    
    def drain_notifications(self) -> list:
        """获取所有完成通知"""
        with self._lock:
            notifs = list(self._notification_queue)
            self._notification_queue.clear()
        return notifs
```

### 通知注入机制

```python
def agent_loop(messages: list):
    while True:
        # 每轮检查后台通知
        notifs = BG.drain_notifications()
        if notifs:
            # 注入为对话消息
            notif_text = "\n".join(
                f"[bg:{n['task_id']}] {n['status']}: {n['result']}"
                for n in notifs
            )
            messages.append({
                "role": "user",
                "content": f"<background-results>\n{notif_text}\n</background-results>"
            })
            messages.append({"role": "assistant", "content": "Noted."})
        
        # 调用 LLM...
```

### 使用场景

```python
# 启动多个后台任务
background_run("npm run build")
background_run("npm test")

# 代理继续做其他事...

# 稍后自动收到通知
<background-results>
[bg:abc123] completed: Build succeeded
[bg:def456] completed: All tests passed
</background-results>
```

---

## 4. 团队协作：持久化队友

### 解决问题

Subagent 是一次性的，需要持久化的协作伙伴。

### 核心区别

| 特性     | Subagent | Teammate     |
| -------- | -------- | ------------ |
| 生命周期 | 一次性   | 持久化       |
| 上下文   | 用完即弃 | 跨任务保持   |
| 通信     | 单向返回 | 双向消息     |
| 状态     | 无       | idle/working |

### 消息系统

```python
class MessageBus:
    def __init__(self, inbox_dir: Path):
        self.dir = inbox_dir
        self.dir.mkdir(parents=True, exist_ok=True)
    
    def send(self, sender: str, to: str, content: str, msg_type: str = "message"):
        msg = {
            "type": msg_type,
            "from": sender,
            "content": content,
            "timestamp": time.time()
        }
        inbox_path = self.dir / f"{to}.jsonl"
        with open(inbox_path, "a") as f:
            f.write(json.dumps(msg) + "\n")
    
    def read_inbox(self, name: str) -> list:
        inbox_path = self.dir / f"{name}.jsonl"
        messages = [json.loads(line) for line in inbox_path.read_text().splitlines()]
        inbox_path.write_text("")  # 读后清空
        return messages
```

### 队友管理器

```python
class TeammateManager:
    def __init__(self, team_dir: Path):
        self.config = self._load_config()  # config.json
        self.threads = {}
    
    def spawn(self, name: str, role: str, prompt: str) -> str:
        # 登记到花名册
        member = {"name": name, "role": role, "status": "working"}
        self.config["members"].append(member)
        self._save_config()
        
        # 启动线程
        thread = threading.Thread(
            target=self._teammate_loop,
            args=(name, role, prompt),
            daemon=True
        )
        self.threads[name] = thread
        thread.start()
        
        return f"Spawned '{name}'"
    
    def _teammate_loop(self, name: str, role: str, prompt: str):
        sys_prompt = f"You are '{name}', role: {role}"
        messages = [{"role": "user", "content": prompt}]
        
        for _ in range(50):
            # 检查收件箱
            inbox = BUS.read_inbox(name)
            for msg in inbox:
                messages.append({"role": "user", "content": json.dumps(msg)})
            
            # 调用 LLM
            response = client.messages.create(
                model=MODEL,
                system=sys_prompt,
                messages=messages,
                tools=TEAMMATE_TOOLS
            )
            
            # ... 执行工具 ...
            
            if response.stop_reason != "tool_use":
                break
        
        # 标记为空闲
        self._set_status(name, "idle")
```

### 文件结构

```
.team/
├── config.json              # 花名册
│   {
│     "team_name": "dev-team",
│     "members": [
│       {"name": "coder1", "role": "backend", "status": "working"},
│       {"name": "coder2", "role": "frontend", "status": "idle"}
│     ]
│   }
│
└── inbox/                   # 收件箱
    ├── lead.jsonl
    ├── coder1.jsonl
    └── coder2.jsonl
```

---

## 5. 协作协议：请求-响应模式

### 解决问题

简单消息无法表达复杂的协商流程。

### 协议设计

**关闭协议**：

```
Lead                            Teammate
  │                                 │
  │ shutdown_request                │
  │ {request_id: "abc"}             │
  │ ─────────────────────────────► │
  │                                 │ 检查是否可以关闭
  │                                 │
  │ shutdown_response               │
  │ {request_id: "abc",             │
  │  approve: true}                 │
  │ ◄───────────────────────────── │
  │                                 │
  │ 执行关闭                         │
```

**计划审批协议**：

```
Teammate                      Lead
  │                              │
  │ plan_approval                │
  │ {request_id: "xyz",          │
  │  plan: "重构登录模块"}        │
  │ ───────────────────────────► │
  │                              │ 审核计划
  │                              │
  │ plan_approval_response       │
  │ {request_id: "xyz",          │
  │  approve: true}              │
  │ ◄─────────────────────────── │
  │                              │
  │ 开始执行计划                  │
```

### 代码实现

```python
# 请求追踪器
shutdown_requests = {}
plan_requests = {}
_tracker_lock = threading.Lock()

def handle_shutdown_request(teammate: str) -> str:
    # 生成请求 ID
    req_id = str(uuid.uuid4())[:8]
    
    # 记录状态
    with _tracker_lock:
        shutdown_requests[req_id] = {"target": teammate, "status": "pending"}
    
    # 发送请求
    BUS.send("lead", teammate, "Please shut down", "shutdown_request", 
             {"request_id": req_id})
    
    return f"Shutdown request {req_id} sent"

def handle_plan_review(request_id: str, approve: bool, feedback: str) -> str:
    # 查找请求
    with _tracker_lock:
        req = plan_requests.get(request_id)
    
    # 更新状态
    with _tracker_lock:
        req["status"] = "approved" if approve else "rejected"
    
    # 发送响应
    BUS.send("lead", req["from"], feedback, "plan_approval_response",
             {"request_id": request_id, "approve": approve})
    
    return f"Plan {req['status']}"
```

### request_id 关联

**为什么需要？**

```
同时发送多个请求：
Lead → Teammate1: shutdown_request {request_id: "aaa"}
Lead → Teammate2: shutdown_request {request_id: "bbb"}

收到响应时，用 request_id 匹配：
Teammate1 → Lead: shutdown_response {request_id: "aaa", approve: true}
Teammate2 → Lead: shutdown_response {request_id: "bbb", approve: false}
```

---

## 6. 自主性：自驱动工作

### 解决问题

队友不再被动等待，而是主动找活干。

### 生命周期

```
spawn（创建）
    │
    ▼
WORK（工作）
    │
    │ 任务完成 / 调用 idle
    ▼
IDLE（空闲）
    │
    ├─► 检查收件箱 ──有消息──► WORK
    │
    ├─► 扫描任务板 ──有未认领任务──► 认领 ──► WORK
    │
    └─► 60秒超时 ──► shutdown
```

### 任务板扫描

```python
def scan_unclaimed_tasks() -> list:
    unclaimed = []
    for f in TASKS_DIR.glob("task_*.json"):
        task = json.loads(f.read_text())
        # 找满足条件的任务
        if (task.get("status") == "pending"
                and not task.get("owner")
                and not task.get("blockedBy")):
            unclaimed.append(task)
    return unclaimed

def claim_task(task_id: int, owner: str) -> str:
    with _claim_lock:
        task = json.loads((TASKS_DIR / f"task_{task_id}.json").read_text())
        task["owner"] = owner
        task["status"] = "in_progress"
        (TASKS_DIR / f"task_{task_id}.json").write_text(json.dumps(task))
    return f"Claimed task #{task_id}"
```

### IDLE 轮询

```python
def _teammate_loop(self, name: str, role: str, prompt: str):
    while True:
        # WORK 阶段
        for _ in range(50):
            # ... 标准 agent loop ...
            
            # 任务完成或调用 idle
            if idle_requested or stop_reason != "tool_use":
                break
        
        # IDLE 阶段
        self._set_status(name, "idle")
        resume = False
        
        for _ in range(12):  # 60秒 / 5秒
            time.sleep(5)
            
            # 检查收件箱
            inbox = BUS.read_inbox(name)
            if inbox:
                messages.extend([{"role": "user", "content": json.dumps(m)} for m in inbox])
                resume = True
                break
            
            # 扫描任务板
            unclaimed = scan_unclaimed_tasks()
            if unclaimed:
                task = unclaimed[0]
                claim_task(task["id"], name)
                
                # 注入任务提示
                messages.append({
                    "role": "user",
                    "content": f"<auto-claimed>Task #{task['id']}: {task['subject']}</auto-claimed>"
                })
                resume = True
                break
        
        # 超时关闭
        if not resume:
            self._set_status(name, "shutdown")
            return
        
        # 恢复工作
        self._set_status(name, "working")
```

### 身份重新注入

```python
def make_identity_block(name: str, role: str, team_name: str) -> dict:
    return {
        "role": "user",
        "content": f"<identity>You are '{name}', role: {role}, team: {team_name}</identity>"
    }

# 压缩后重新注入
if len(messages) <= 3:  # 刚压缩完
    messages.insert(0, make_identity_block(name, role, team_name))
```

---

## 7. 任务隔离：Worktree 并行执行

### 解决问题

多个任务同时修改同一文件会产生冲突。

### 两层平面

```
控制平面          执行平面
       │                                     │
       ▼                                     ▼
.tasks/task_12.json                .worktrees/auth-refactor/
{                                  ├── src/
  "id": 12,                        │   └── auth.js (独立副本)
  "subject": "重构认证",            └── tests/
  "worktree": "auth-refactor"
}
```

**核心思想**：用目录隔离，用任务 ID 协调。

### WorktreeManager

```python
class WorktreeManager:
    def __init__(self, repo_root: Path):
        self.repo_root = repo_root
        self.dir = repo_root / ".worktrees"
        self.index_path = self.dir / "index.json"
    
    def create(self, name: str, task_id: int = None, base_ref: str = "HEAD") -> str:
        # 验证名称
        if not re.fullmatch(r"[A-Za-z0-9._-]{1,40}", name):
            raise ValueError("Invalid name")
        
        # 创建 git worktree
        path = self.dir / name
        branch = f"wt/{name}"
        
        # git worktree add -b wt/auth-refactor .worktrees/auth-refactor HEAD
        subprocess.run(["git", "worktree", "add", "-b", branch, str(path), base_ref])
        
        # 更新索引
        entry = {
            "name": name,
            "path": str(path),
            "branch": branch,
            "task_id": task_id,
            "status": "active"
        }
        idx = json.loads(self.index_path.read_text())
        idx["worktrees"].append(entry)
        self.index_path.write_text(json.dumps(idx, indent=2))
        
        # 绑定到任务
        if task_id:
            TASKS.bind_worktree(task_id, name)
        
        return json.dumps(entry, indent=2)
    
    def run(self, name: str, command: str) -> str:
        """在 worktree 目录中执行命令"""
        wt = self._find(name)
        path = Path(wt["path"])
        
        r = subprocess.run(
            command,
            shell=True,
            cwd=path,  # 关键：在 worktree 中执行
            capture_output=True,
            text=True
        )
        return r.stdout + r.stderr
    
    def remove(self, name: str, complete_task: bool = False) -> str:
        wt = self._find(name)
        
        # 移除 git worktree
        subprocess.run(["git", "worktree", "remove", wt["path"]])
        
        # 可选标记任务完成
        if complete_task and wt.get("task_id"):
            TASKS.update(wt["task_id"], status="completed")
            TASKS.unbind_worktree(wt["task_id"])
        
        # 更新索引状态
        idx = json.loads(self.index_path.read_text())
        for item in idx["worktrees"]:
            if item["name"] == name:
                item["status"] = "removed"
        self.index_path.write_text(json.dumps(idx, indent=2))
        
        return f"Removed {name}"
```

### 事件总线

```python
class EventBus:
    def __init__(self, event_log_path: Path):
        self.path = event_log_path
    
    def emit(self, event: str, task: dict = None, worktree: dict = None):
        payload = {
            "event": event,
            "ts": time.time(),
            "task": task or {},
            "worktree": worktree or {}
        }
        with self.path.open("a") as f:
            f.write(json.dumps(payload) + "\n")
```

**事件日志示例**：

```
{"event": "worktree.create.before", "ts": 1711234567, "task": {"id": 12}, "worktree": {"name": "auth-refactor"}}
{"event": "worktree.create.after", "ts": 1711234570, "task": {"id": 12}, "worktree": {"name": "auth-refactor", "status": "active"}}
{"event": "task.completed", "ts": 1711234700, "task": {"id": 12, "status": "completed"}, "worktree": {"name": "auth-refactor"}}
```

### 完整工作流

```python
# 1. 创建任务
task_create(subject="重构认证模块")
# task_id = 12

# 2. 创建 worktree
worktree_create(name="auth-refactor", task_id=12)
# 创建 .worktrees/auth-refactor/

# 3. 在 worktree 中工作
worktree_run("auth-refactor", "npm install jsonwebtoken")
worktree_run("auth-refactor", "npm test")

# 4. 完成
worktree_remove("auth-refactor", complete_task=True)
# 任务 12 标记为 completed
```

---

## 架构演进总结

### 演进路径

```
单代理
    │
    ├─► 添加 Subagent（任务委派）
    │
    ├─► 添加上下文压缩（无限工作）
    │
    ├─► 添加后台任务（异步执行）
    │
    ├─► 添加 Teammate（持久协作）
    │
    ├─► 添加协议层（规范协作）
    │
    ├─► 添加自主性（自驱动工作）
    │
    └─► 添加 Worktree（并行隔离）
```

### 核心能力对比

| 阶段      | 并行能力 | 协作能力 | 持久性 | 自主性 |
| --------- | -------- | -------- | ------ | ------ |
| 基础      | 无       | 无       | 无     | 无     |
| +Subagent | 有限     | 单向     | 无     | 无     |
| +Teammate | 有       | 双向     | 有     | 无     |
| +自主性   | 有       | 双向     | 有     | 有     |
| +Worktree | 强       | 双向     | 有     | 有     |

### 设计原则

1. **渐进增强**：每个阶段只解决一个核心问题
2. **组合优于继承**：功能通过组合实现，不是层层继承
3. **文件作为接口**：JSON/JSONL 文件作为模块间通信协议
4. **线程隔离**：每个队友独立线程，互不干扰
5. **事件驱动**：状态变化通过事件通知，解耦模块

---

## 附录：工具清单

### 基础工具
- `bash` - 执行命令
- `read_file` / `write_file` / `edit_file` - 文件操作

### 任务工具
- `task_create` / `task_list` / `task_get` / `task_update` - 任务管理

### 团队工具
- `spawn_teammate` / `list_teammates` - 队友管理
- `send_message` / `read_inbox` / `broadcast` - 消息通信

### 协议工具
- `shutdown_request` / `shutdown_response` - 关闭协议
- `plan_approval` - 计划审批

### Worktree 工具
- `worktree_create` / `worktree_list` / `worktree_status` - 工作树管理
- `worktree_run` - 在工作树中执行命令
- `worktree_remove` / `worktree_keep` - 清理工作树
- `worktree_events` - 生命周期事件