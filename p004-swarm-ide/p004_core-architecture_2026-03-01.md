---
project: P004 · Swarm-IDE
file_type: learning-summary
tags: [multi-agent, im-metaphor, promise-deferred, event-bus, postgresql, typescript]
date: 2026-03-01
session_type: 初次学习
source: https://github.com/chmod777john/swarm-ide
---

# 学习总结：Swarm-IDE 深度源码分析 / Deep Source Code Analysis

> **⚠️ 版本说明：** 本文是对同日初稿的**重大更新**，初稿仅有表面概念描述，本版基于 GitHub 源码直接阅读，包含完整的实现层技术细节。
> This is a **major rewrite** of the same-day draft. The original lacked technical depth. This version is based on direct source code reading from GitHub.

---

## 会话元数据 / Session Metadata

| 字段 Field | 内容 Content |
|---|---|
| 日期 Date | 2026-03-01 |
| 时长（估算）Duration | ～ 3 小时 hours |
| 项目名称 Project | P004 Swarm-IDE |
| 主题 Topic | Swarm-IDE 多 Agent 协作系统——源码级实现拆解 |
| 触发原因 Context | 初稿信息量不足，通过 Chrome 直接阅读 GitHub 源码补充 |
| 会话类型 Session Type | ☑ 深入研究 Deep Dive |
| 关联上次会话 Prev Session | `learning-summary_2026-03-01.md`（Claude Code Agent Teams）|
| 代码来源 Source | `github.com/chmod777john/swarm-ide`，分支 `chore/specs-mvp` |

---

## 一、项目概览 / Project Overview

**项目描述：**

Swarm-IDE 是一个开源的多 Agent 协作 IDE，核心隐喻是**把 AI Agent 嵌入一个类微信群聊 IM 系统**。Agent 之间通过发送消息（而非函数调用）协作，支持嵌套创建子 Agent、群组通信、P2P 通信，并通过 SSE 实时流将所有活动推送给前端可视化。

**关键文档位置：**

| 文档 | 路径/链接 |
|---|---|
| 主仓库 | `github.com/chmod777john/swarm-ide` |
| 默认分支 | `chore/specs-mvp`（注意：不是 `main`） |
| 核心运行时 | `backend/src/runtime/agent-runtime.ts` |
| 事件总线 | `backend/src/runtime/event-bus.ts` + `ui-bus.ts` |
| 数据库 Schema | `backend/src/db/schema.ts` |
| 技术栈环境变量 | `backend/README.md` |

**项目结构速览：**

```
swarm-ide/
├── backend/
│   ├── src/
│   │   ├── runtime/
│   │   │   ├── agent-runtime.ts   ← ⭐ 核心：Agent 执行引擎
│   │   │   ├── event-bus.ts       ← Agent 内部事件流（per-agent）
│   │   │   ├── ui-bus.ts          ← 前端 UI 事件流（per-workspace）
│   │   │   ├── upstash-realtime.ts← 跨进程持久化（可选）
│   │   │   ├── mcp.ts             ← MCP 工具加载
│   │   │   └── skill-loader.ts    ← Skill 加载器
│   │   └── db/
│   │       ├── schema.ts          ← 5 张表：workspaces/agents/groups/group_members/messages
│   │       ├── init.ts            ← 建表脚本
│   │       └── client.ts          ← Drizzle ORM 客户端
│   └── README.md                  ← 环境变量说明
├── skills/                        ← 全局 Skill 目录
└── .agents/                       ← Agent 专属 skill/配置
```

---

## 二、本次学习内容 / What I Studied Today

### 2.1 阅读的文档

- [x] `backend/src/runtime/agent-runtime.ts` — 完整 Agent 执行引擎（AgentRunner + AgentRuntime 两个类）
- [x] `backend/src/runtime/event-bus.ts` — Agent 内部事件总线，in-memory 环形缓冲区 + Upstash 可选持久化
- [x] `backend/src/runtime/ui-bus.ts` — 面向前端的 UI 事件总线，workspace 级别
- [x] `backend/src/db/schema.ts` — Drizzle ORM 数据库 Schema，5 张表

---

### 2.2 核心概念 / Core Concepts

#### 概念 1：两类核心类——AgentRunner vs AgentRuntime

**AgentRunner**（每 Agent 一个实例，负责单个 Agent 的执行循环）：

```typescript
class AgentRunner {
  private wake = createDeferred<void>();  // Promise-based 挂起
  private started = false;
  private running = false;
  private interruptRequested = false;

  wakeup(reason: "manual" | "group_message" | "direct_message" | "context_stream")
  requestInterrupt()   // 设 flag + resolve Promise，让 loop 感知并退出

  private async loop() {
    while (true) {
      await this.wake.promise;   // 在此挂起，直到被 wakeup() 唤醒
      if (this.running) continue;
      this.running = true;
      await this.processUntilIdle();
    }
  }
}
```

**AgentRuntime**（全局单例，管理所有 AgentRunner）：

```typescript
export class AgentRuntime {
  private readonly runners = new Map<UUID, AgentRunner>();
  static readonly VERSION = 2;
  // 全局单例存储在 globalThis.__agentWechatRuntime
  // VERSION 检查防止 Next.js 热更新后的 stale 实例

  ensureRunner(agentId)                    // 懒创建 AgentRunner，idempotent
  wakeAgentsForGroup(groupId, senderId)    // 群发时唤醒所有非发送方 Agent
  wakeAgent(agentId, reason)              // 唤醒特定 Agent（P2P/context stream）
  interruptAll({workspaceId?})             // 强制中断所有 Agent
}
```

---

#### 概念 2：Promise-based 事件驱动唤醒（非轮询）

这是与 Claude Code 最大的架构差异。Swarm-IDE 用 `createDeferred<void>()` 实现**零轮询的阻塞式等待**：

```
Agent A 调用 send_group_message
    └─ executeToolCall() 写入 DB → 调用 wakeAgentsForGroup()
           └─ ensureRunner(memberB).wakeup("group_message")
                  └─ memberB.wake.resolve()   ← Promise 被立即 resolve
                         └─ memberB 的 loop() 从 await 处继续执行
                                └─ processUntilIdle() 开始处理新消息
```

对比 Claude Code：Agent B 需要等到自己的下一个 turn，才去轮询 inbox JSON 文件。Swarm-IDE 的 Agent B 在消息到达的**同一事件循环**内就被唤醒。

---

#### 概念 3：IM 隐喻系统提示词 + 强制发送机制

每个 Agent 的 system prompt 核心框架（概括）：

> "You are an agent in an IM system. Your agent_id is: `${agentId}`. Your role is: `${role}`. To send messages, you MUST call tools like `send_group_message` or `send_direct_message`."

**强制发送机制**：发送工具集合 `SEND_TOOL_NAMES = new Set(["send", "send_group_message", "send_direct_message"])`。如果 LLM 完成一轮推理后没有调用任何发送工具，系统注入中文提醒并重新运行 LLM：

> "Reminder: 本轮未调用 send_*。先判断是否需要对外可见..."

这确保 Agent 的思考结果总会通过消息传递出去，不会"沉默完成"。

---

#### 概念 4：Agent 工具集（10 个内置 + MCP 扩展）

```typescript
const AGENT_TOOLS = [
  "create",              // 生成子 Agent，返回 {agentId, groupId}
  "self",                // 查询自身 identity（agentId, role）
  "get_skill",           // 按名称加载 skill 内容
  "send_direct_message", // P2P 发送，立即唤醒目标 Agent
  "list_groups",         // 列出自己所在的所有群组
  "list_group_members",  // 列出群成员
  "create_group",        // 创建群组并拉入成员
  "send_group_message",  // 群发消息（发送方需是成员），唤醒所有成员
  "get_group_messages",  // 获取群消息历史（catch-up 重播）
  "bash",                // 执行 shell 命令，沙箱限制在 workspace root
  // + MCP 动态工具（通过 mcp.ts 加载，作为 fallback）
];
```

`create` 工具是"液态拓扑"的底层实现：Agent 运行时自主决定是否生成子 Agent，整个拓扑从单一起点动态演化，无需预先定义。

---

#### 概念 5：双总线架构（AgentEventBus + WorkspaceUIBus）

**AgentEventBus**（per-agent，内部通信用，发往 SSE `/api/agents/:agentId/context-stream`）：

| 事件 | 含义 |
|---|---|
| `agent.wakeup` | Agent 被唤醒（含唤醒原因） |
| `agent.unread` | 有新的未读消息 batches（含 groupId + messageIds） |
| `agent.stream` | LLM 流式输出片段（reasoning / content / tool_calls / tool_result） |
| `agent.done` | LLM 完成一轮（含 finishReason） |
| `agent.error` | 运行时错误 |

**WorkspaceUIBus**（per-workspace，面向 UI 刷新）：

| 事件 | 含义 |
|---|---|
| `ui.agent.created` | 新 Agent 被创建（含 parentId，可构建树形拓扑图） |
| `ui.group.created` | 新群组被创建（含初始成员列表） |
| `ui.message.created` | 新消息到达 |
| `ui.agent.llm.start` | LLM 开始推理（含 round 轮次） |
| `ui.agent.llm.done` | LLM 推理完成（含 finishReason） |
| `ui.agent.history.persisted` | 对话历史已保存到 DB（含 historyLength） |
| `ui.agent.tool_call.start` | 工具调用开始 |
| `ui.agent.interrupt_all` | 全部 Agent 被中断 |
| `ui.db.write` | 数据库写入（insert/update/delete，含 table + recordId） |

两个总线结构完全对称：
- in-memory 环形缓冲区（max 2000 事件）
- `getSince(id, afterId)` 支持 SSE 断线重连重播
- `persistQueue` 串行化写入，防止 Upstash 乱序
- 可选 Upstash Realtime 持久化（跨进程场景）

---

#### 概念 6：数据库 Schema（5 张表，Drizzle ORM + PostgreSQL）

```
workspaces
  id (uuid PK), name, created_at

agents
  id (uuid PK), workspace_id (FK→workspaces), role (text),
  parent_id (uuid),              ← 子 Agent 树结构（parent_id → 创建者 Agent）
  llm_history (TEXT NOT NULL),   ← ⭐ 完整 LLM 对话历史，JSON 序列化单字段存储
  created_at

groups
  id (uuid PK), workspace_id (FK→workspaces), name (text),
  context_tokens (INT default 0),  ← 累计 token 用量追踪
  created_at

group_members                        ← 多对多关系表
  (group_id, user_id) 复合主键
  group_id (FK→groups), user_id (uuid),
  last_read_message_id (uuid),       ← ⭐ 驱动"未读消息"逻辑（类 IM 读状态）
  joined_at

messages
  id (uuid PK), workspace_id (FK→workspaces), group_id (FK→groups),
  sender_id (uuid), content_type (text), content (TEXT), send_time
```

**关键设计决策**：Agent 的完整 LLM 对话历史存在 `agents.llm_history` 的单个 text 字段（JSON blob），每轮覆盖写回。这是"简单可靠优先于规范化"的决策，避免了历史分片管理的复杂性，代价是行大小随对话增长。

---

### 2.3 关键流程 / Key Workflow

#### Agent 处理一轮消息的完整流程

```
[挂起中：await wake.promise]
         ↓ wakeup() 调用，Promise resolve
1. processUntilIdle() 开始
   ├─ 从 DB 加载 agent 行（llmHistory, role）
   ├─ 查 group_members.last_read_message_id
   │   → 找出所有未读 messages
   │   → 注入到历史作为 "user" 角色消息
         ↓
2. runWithTools(maxToolRounds) 循环
   ├─ callLlmStreaming(history) → GLM/OpenRouter SSE 流
   ├─ 边接收边 emit AgentEvent（reasoning / content / tool_calls）
   ├─ 所有 tool_calls 同步执行（executeToolCall）
   │   ├─ bash              → exec(cmd, {cwd: workspaceRoot})
   │   ├─ create            → 写 DB agents，ensureRunner，wakeup("manual")
   │   ├─ send_group_message → 写 DB messages，wakeAgentsForGroup()
   │   ├─ send_direct_message→ 写 DB messages，wakeAgent()
   │   ├─ get_group_messages → 从 DB 读消息历史
   │   ├─ create_group / list_groups 等 → DB 操作
   │   └─ MCP 工具         → 调外部 MCP server（fallback）
   └─ 直到 LLM finish_reason = "stop"（无更多 tool_calls）
         ↓
3. 检查本轮是否调用了 send_* 工具
   ├─ 是 → 保存 llmHistory 到 DB，emit agent.done
   └─ 否 → 注入 "本轮未调用 send_*..." 提醒 → 重回步骤 2
         ↓
4. 更新 last_read_message_id（标记已读）
5. 保存 token 用量到 groups.context_tokens
[回到挂起：await wake.promise]
```

---

### 2.4 重要代码 / Key Code

**全局单例 + Hot-Reload 安全模式：**

```typescript
declare global {
  var __agentWechatRuntime: AgentRuntime | undefined;
  var __agentWechatRuntimeVersion: number | undefined;
}

export function getAgentRuntime() {
  if (globalThis.__agentWechatRuntime &&
      globalThis.__agentWechatRuntimeVersion === AgentRuntime.VERSION) {
    return globalThis.__agentWechatRuntime;
  }
  globalThis.__agentWechatRuntime = new AgentRuntime();
  globalThis.__agentWechatRuntimeVersion = AgentRuntime.VERSION;
  return globalThis.__agentWechatRuntime;
}
```

理解：Next.js 热更新会多次执行模块代码。VERSION 号确保只有 runtime 版本匹配时才复用现有实例，否则创建新实例，防止 runner 状态泄漏。

**`ensureRunner` 懒初始化 + 依赖注入回调：**

```typescript
ensureRunner(agentId: UUID) {
  const existing = this.runners.get(agentId);
  if (existing) return existing;
  const runner = new AgentRunner(
    agentId,
    this.bus,
    (id) => { this.ensureRunner(id); },              // createAgent 回调
    (id) => { this.ensureRunner(id).wakeup("manual"); } // wakeAgent 回调
  );
  this.runners.set(agentId, runner);
  return runner;
}
```

理解：AgentRunner 通过回调获得创建/唤醒能力，不直接持有 AgentRuntime 引用，是轻量级依赖反转（DI）。

**`wakeAgentsForGroup` — 群消息广播唤醒：**

```typescript
async wakeAgentsForGroup(groupId: UUID, senderId: UUID) {
  await this.bootstrap();
  const memberIds = await store.listGroupMemberIds({ groupId });
  for (const memberId of memberIds) {
    if (memberId === senderId) continue;        // 不唤醒发送方自己
    const role = await store.getAgentRole({ agentId: memberId }).catch(() => null);
    if (role === "human" || role === null) continue;  // 不唤醒人类用户
    this.ensureRunner(memberId).wakeup("group_message");
  }
}
```

理解：`role === "human"` 的过滤揭示了系统设计：人类用户和 AI Agent 共享同一套消息基础设施，人类只是一种特殊成员类型，不会被自动唤醒执行推理。

---

## 三、洞察与收获 / Insights & Takeaways

### 3.1 最重要的 3 个发现

1. **"IM 群聊"是完整的架构决策，不只是比喻**：数据库有完整的 groups + group_members + messages 表，`last_read_message_id` 驱动未读逻辑，这是完整的 IM 系统实现。Agent 的 system prompt 第一句就是"You are an agent in an IM system"，隐喻从设计层到代码层到 prompt 层完全统一。

2. **Promise deferred vs 文件轮询是两种完全不同的并发哲学**：Claude Code 接受"文件系统是共享内存"的 UNIX 哲学，每 turn 轮询；Swarm-IDE 用 Promise + event-driven 实现真正的"推送到接收"语义。两者各有权衡：Swarm-IDE 响应延迟更低，Claude Code 模型更简单可调试。

3. **`agents.llm_history` 存 TEXT 是刻意的简化权衡**：放弃规范化，换取极简的读写逻辑。代价是行大小随对话增长，但 LLM context window 本身就有上限，实际影响有限。这是"让简单的事情简单"的工程判断。

### 3.2 让我意外的地方

**"未调用 send 工具时注入提醒"**超出预期。Swarm-IDE 把"必须对外通信"作为 Agent 行为的硬约束而非建议——如果 Agent 想沉默完成任务，系统会强制打断要求其声明意图。这是一种**协议层面的透明度强制机制**，与 Claude Code 的自然 tool-call 循环截然不同。

**`role === "human"` 的显式过滤**也让我意外：人类用户和 AI Agent 共享同一套消息基础设施。人类不是从外部注入命令的控制者，而是系统内一种特殊成员类型，只是不会被自动唤醒而已。这个设计隐含了一个平等观：人类是"慢速 Agent"。

### 3.3 与已有知识的连接

| 比较维度 | Claude Code Agent Teams | Swarm-IDE |
|---|---|---|
| 通信基础设施 | 文件系统 JSON inbox | PostgreSQL messages 表 |
| Agent 挂起机制 | Turn 间轮询文件 | Promise deferred，事件驱动唤醒 |
| 历史存储 | 内存 + JSONL 文件 | agents.llm_history TEXT（JSON blob）|
| 通信时序 | 异步：下轮才读取 | 同步：发送后立即唤醒对方 |
| 群组支持 | 无原生群组概念 | 原生 group_members 表 |
| 前端可视化 | CLI 输出 | SSE + 双总线实时流 |
| 跨进程 | tmux 独立进程 | Upstash Realtime（可选）|
| 人类接入 | 外部 CLI | "human" role Agent，共享消息通道 |
| 运行时 | Node.js | Bun |
| 部署复杂度 | 低（无外部依赖）| 高（需 PostgreSQL + Redis）|

连接到 Boris Tane 的"Context Engineering 是核心技能"：Swarm-IDE 在数据库层就内置了 context 管理——`context_tokens` 追踪 token 用量，`last_read_message_id` 追踪未读状态，`get_group_messages` 支持历史重播。不是事后补救，是架构原语。

---

## 四、安全与风险观察 / Security & Risk Notes

| 观察 | 风险等级 | 备注 |
|---|---|---|
| `bash` 工具仅沙箱到 workspace root，无进一步权限隔离 | 中 | 任意 Agent 均可执行 shell 命令 |
| `agents.llm_history` 为纯 TEXT，无加密 | 低-中 | 对话历史含敏感上下文，DB 需严格访问控制 |
| `send_group_message` 只检查发送方是否是群成员 | 低 | 无消息内容过滤，任意 Agent 可广播任意内容 |
| 无 LLM 调用频率限制 | 中 | Agent 链式唤醒可能造成 API 调用风暴 |

**是否需要进行完整安全扫描？** ⬜ TBD（若要生产部署则需要）

---

## 五、问题与待解答 / Open Questions

- [ ] `specs/` 目录里的实现规格文档写了什么？是否有更多设计意图？
- [ ] `runWithTools` 的 `maxToolRounds` 默认值是多少？是否可配置？
- [ ] 前端如何消费 `WorkspaceUIBus` SSE？是否有 `useWorkspace` 之类的 hook？
- [ ] `upstash-realtime.ts` 的跨进程重播如何与 in-memory buffer 协同？是否有竞态？
- [ ] `interruptRequested` flag 在 `processUntilIdle` 中如何被检测？是 per-step 还是仅在 wakeup 时？
- [ ] `.agents/` 目录下预置了哪些 Agent？它们的 role 和 skill 是什么？

---

## 六、下一步计划 / Next Steps

| 优先级 | 行动 | 预计时间 | 状态 |
|---|---|---|---|
| 高 | 阅读 `specs/` 目录，补充设计意图层理解 | 1h | ⬜ 待做 |
| 高 | 实践：本地部署 Swarm-IDE，观察 Agent 创建 + 群聊流程 | 2h | ⬜ 待做 |
| 中 | 阅读前端代码：如何消费 WorkspaceUIBus SSE | 1h | ⬜ 待做 |
| 中 | 对比文章：Claude Code / Swarm-IDE / OpenAI Swarm 三者通信机制横向对比 | 1h | ⬜ 待做 |
| 低 | 阅读 `.agents/` 内置 Agent 定义 | 0.5h | ⬜ 待做 |

---

## 七、本次生成的文件 / Files Generated

| 文件名 | 类型 | 描述 |
|---|---|---|
| `learning-summary_2026-03-01_2.md` | 总结（重写版）| 本文件，Swarm-IDE 完整源码级分析 |

---

## 八、关键词 / Keywords

`Swarm-IDE` `AgentRuntime` `AgentRunner` `createDeferred` `event-driven` `IM-metaphor` `Drizzle-ORM` `WorkspaceUIBus` `AgentEventBus` `SSE` `Promise-deferred` `group_members` `llm_history` `send_group_message` `wakeAgentsForGroup` `bash-tool` `skills-autoload` `MCP-integration` `Upstash-Realtime` `context-tokens`

---

*基于 GitHub `chmod777john/swarm-ide` 源码，分支 `chore/specs-mvp`，阅读日期 2026-03-01*
*Based on direct source code reading from GitHub, branch chore/specs-mvp, date 2026-03-01*
