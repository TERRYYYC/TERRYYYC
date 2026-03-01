---
project: P004 · Swarm-IDE
file_type: learning-summary
tags: [swarm-ide, source-code, specs, upstash, sse, skills, agent-runtime]
date: 2026-03-01
session_type: 深度研究
source: https://github.com/chmod777john/swarm-ide
related: p004_core-architecture_2026-03-01.md
---

# 学习总结：Swarm-IDE 本地源码深读——6 个悬案全部破解
# Learning Summary: Swarm-IDE Local Source Deep Read — All 6 Open Questions Resolved

> **版本说明：** 本文是 `_2` 的续集。上次 6 个待解问题全部通过本地源码阅读得到答案。
> This is the sequel to `_2`. All 6 open questions from last session resolved via local source code.

---

## 会话元数据 / Session Metadata

| 字段 Field | 内容 Content |
|---|---|
| 日期 Date | 2026-03-01 |
| 时长（估算）Duration | ～ 1.5 小时 hours |
| 项目名称 Project | P004 Swarm-IDE（续）|
| 主题 Topic | 本地源码精读：specs、runtime 控制流、前端 SSE、upstash-realtime、.agents/skills |
| 触发原因 Context | 项目已下载到本地，可以直接阅读完整代码 |
| 会话类型 Session Type | ☑ 深入研究 Deep Dive |
| 关联上次会话 Prev Session | `learning-summary_2026-03-01_2.md` |
| 代码来源 Source | `/Users/terry/Desktop/code code/swarm-ide`（本地） |

---

## 一、上次 6 个待解问题的答案 / Answers to 6 Open Questions

---

### Q1：`specs/` 目录写了什么？

**答案：** `specs/implementation-plan/` 下有两个文档，揭示了设计意图层：

**`prd.md`** — 产品需求文档，定义了：
- 首次进入自动创建 `human agent` + `assistant agent` + 默认 P2P 群
- 侧边栏只展示"当前 human agent 所在的群"（会话可见性约束）
- 前端完整 API 接口规范（REST + SSE），包含断线重连策略
- IM 隐喻的操作路径（12 步 MVP 用户旅程）

**`tech_stack.md`** — 技术方案文档，明确写出：
> **"去中心化：用户和 agent 完全等价。用户只是一种特殊的 agent。所有视角可切换。"**

这是全项目最重要的哲学宣言，比代码注释更清晰。

另外在 `spells/` 目录发现了两个**多 Agent 协作模式的 Prompt 模板**：

| Spell | 用途 |
|---|---|
| `map-reduce.md` | 大任务拆分为 N 个子任务并行处理，汇总结果 |
| `router-experts.md` | 入口 Agent 按关键词路由到专家 Agent（pm/coder/designer），自动创建不存在的专家 |

这说明 swarm-ide 不只是基础设施，还附带了**可复用的 Agent 协作 Prompt 模板库**（称为 "spells"，魔法咒语）。

---

### Q2：`runWithTools` 的 `maxToolRounds` 是多少？是否可配置？

**答案（直接从源码第 528 行）：**

```typescript
// backend/src/runtime/agent-runtime.ts, line 528
const maxToolRounds = 3;
```

**`maxToolRounds = 3`，硬编码，不可配置。**

含义：每次 `runWithTools` 调用，LLM 最多有 3 轮工具调用机会。第 3 轮之后，即使 LLM 还想调用工具，也会强制退出循环返回当前结果。

**重要补充**：`runWithTools` 被调用**两次**（正常轮 + 强制发送 followup 轮），因此单次 `processGroupUnread` 最多触发 **3+3=6 轮 LLM 工具调用**。

---

### Q3：`interruptRequested` 在 `processUntilIdle` 中如何被检测？是 per-step 还是仅在 wakeup 时？

**答案（源码第 395-510 行）：**

是 **per-step 级别检测**，检测点分布极为密集：

```typescript
// processUntilIdle 开始时
if (this.consumeInterruptRequest()) return;    // ← 检测点 1：进入前

while (true) {
  if (this.consumeInterruptRequest()) return;  // ← 检测点 2：每次拉取未读前
  const batches = await store.listUnreadByGroup(...);
  if (batches.length === 0) return;

  for (const batch of batches) {
    if (this.consumeInterruptRequest()) return;  // ← 检测点 3：每个 group batch 前
    await this.processGroupUnread(batch.groupId, batch.messages);
    if (this.consumeInterruptRequest()) return;  // ← 检测点 4：每个 group batch 后
  }
}
```

**总计 4 个检测点**，确保中断信号在 batch 级别粒度内响应（不会等到整个任务结束）。

`requestInterrupt()` 同时：
1. 设置 `interruptRequested = true`
2. `resolve()` 当前的 `wake.promise`（使 `loop()` 立即醒来）

两者配合，实现"**发出中断信号 → loop 醒来 → processUntilIdle 在下一个 batch 边界检测到并退出**"的机制。

---

### Q4：前端如何消费 `WorkspaceUIBus` SSE？有没有 `useWorkspace` Hook？

**答案：** 没有专用 Hook，直接在组件内使用原生 `EventSource` API。

代码在 `backend/app/im/page.tsx`（1069-1130 行），`"use client"` 组件内：

```typescript
// 订阅 UI 事件流
useEffect(() => {
  if (!session) return;
  const es = new EventSource(
    `/api/ui-stream?workspaceId=${encodeURIComponent(session.workspaceId)}`
  );
  uiEsRef.current = es;

  es.onmessage = (evt) => {
    const payload = JSON.parse(evt.data) as UiStreamEvent;

    if (payload.event === "ui.agent.created") {
      // 更新 agentStatusById，触发 Graph 可视化更新
    } else if (payload.event === "ui.message.created") {
      // 触发侧边栏刷新 + 当前会话消息刷新
      void refreshGroups(session);
      // 还会触发 pushBeam() 在 Graph 上显示消息流光效果
    } else if (payload.event === "ui.agent.llm.start") {
      // 设置该 Agent 状态为 "BUSY"
    } else if (payload.event === "ui.agent.llm.done") {
      // 设置该 Agent 状态为 "IDLE"
    }
    // ...
  };
}, [session]);
```

**第二个 SSE 流**：Agent 详情面板另外订阅 `context-stream`（per-agent LLM 流式输出）：

```typescript
// 订阅特定 Agent 的 LLM 流式输出
const es = new EventSource(`/api/agents/${agentId}/context-stream`);
es.onmessage = (evt) => {
  const payload = JSON.parse(evt.data) as AgentStreamEvent;
  if (payload.event === "agent.stream") {
    if (payload.data.kind === "content") setContentStream(t => t + chunk);
    if (payload.data.kind === "reasoning") setReasoningStream(t => t + chunk);
    if (payload.data.kind === "tool_calls" || "tool_result") { /* 渲染工具调用流 */ }
  }
};
```

**关键发现**：后端的 `context-stream` 路由在 Agent 订阅时会**主动触发一次 `wakeAgent("context_stream")`**，即"打开详情面板"这个动作本身就会唤醒 Agent。这是设计上很微妙的一点。

---

### Q5：`upstash-realtime.ts` 的跨进程重播如何工作？是否有竞态？

**答案：** 实际底层是 **Redis Streams + Redis Pub/Sub** 组合，不是 Upstash 专有 SDK。

```typescript
// 发送事件
async emit(event, payload) {
  await client.XADD(streamKey, '*', { event, data: JSON.stringify(payload) });
  await client.PUBLISH(streamKey, '1');  // ← 立即通知订阅者
}

// 订阅
async subscribe(opts) {
  // 1. 创建 Consumer Group（XGROUP CREATE）
  // 2. 如果 history.start === "-"，先读历史（XREADGROUP 从 0 开始）
  // 3. 订阅 Pub/Sub channel，收到通知后 readGroup() 拉取新事件
}
```

**断线重连机制**：
- `ui-stream` 路由订阅时传 `history: { start: "-" }`，即**从头重播所有事件**
- `context-stream` 路由同样传 `history: { start: "-", limit: 2000 }`
- 每个订阅者创建**独立的 Consumer Group**（`sse-${randomUUID()}`），互不干扰

**竞态分析**：
- **pub/sub 通知** 和 **XREADGROUP 读取** 之间存在极小的竞态窗口：
  - 若 PUBLISH 后订阅者还未建立，通知丢失；但 `history.start: "-"` 的重播会在建立时补齐
  - 若两个通知几乎同时到达，`readGroup` 用 `COUNT: 2000` 批量读取，会一次读完所有积压
- **结论**：竞态存在但已通过"重播 + 批量读取"机制被充分缓解，生产环境中几乎不会有实际影响

**与 in-memory EventBus 的区别**：
- `event-bus.ts` / `ui-bus.ts` 的 in-memory 环形缓冲是**单进程**（Next.js 同一进程内的 SSE）
- `upstash-realtime.ts` 是**跨进程**（多实例部署、Codespaces 等场景）
- 两者通过配置选择，但当前 `ui-stream` 路由**直接使用 upstash-realtime**（Redis），而非 in-memory bus

---

### Q6：`.agents/` 目录预置了哪些 Agent？

**答案：** `.agents/` 下**没有预置 Agent 定义**，只有 `skills/` 子目录：

```
.agents/
└── skills/
    └── remotion-best-practices/   ← ⭐ 唯一预置 Skill
        ├── SKILL.md               ← Skill 入口（元数据 + 规则索引）
        └── rules/                 ← 20+ 个规则文件
            ├── animations.md
            ├── audio.md
            ├── compositions.md
            ├── text-animations.md
            ├── videos.md
            └── ...（共 20+ 个）
```

**`remotion-best-practices` Skill 的内容**：Remotion（React 视频创作框架）的最佳实践集合，包括：
- 动画、音视频处理、字幕显示
- 3D 渲染（Three.js + React Three Fiber）
- 图表、字体、GIF、过渡效果
- 组合时序、参数动态计算

这揭示了 Swarm-IDE 的**实际使用场景之一**：视频生成 Agent 系统（演示视频 Demo 中正是用 Remotion 制作的宣传视频，`promo-video/` 目录也是一个 Remotion 项目）。

---

## 二、新发现 / New Discoveries

### 2.1 `context-stream` 路由的"连接即唤醒"行为

```typescript
// backend/app/api/agents/[agentId]/context-stream/route.ts
const agent = await store.getAgent({ agentId });
if (agent.role !== 'human') {
  void runtime.wakeAgent(agentId, 'context_stream');  // ← ！
}
```

**打开 Agent 详情面板 = 唤醒该 Agent**（若有未读则触发推理）。

这不是 bug，是设计：用户"关注"某个 Agent，系统确保它立刻活跃起来。

---

### 2.2 前端三区架构的真实实现

PRD 里的 ASCII 草图在代码里完全实现：

```
/im 页面（page.tsx，一个巨型 "use client" 组件）
├── IMShell（left | mid | right 三列布局）
│   ├── 左：侧边栏（对话列表 + 搜索 + Workspace 切换）
│   ├── 中：聊天窗口（消息气泡 + 输入框）
│   └── 右：Agent 详情（LLM history + reasoning + tool-call 流）
└── Graph（/graph 独立路由）
    └── 可视化拓扑图（API: /api/agent-graph → nodes + edges）
```

前端**没有 Redux / Zustand**，全部状态用 `useState` + `useRef` 管理在单组件内。

---

### 2.3 Skill 自动加载机制

`AgentRunner.start()` 在启动时调用 `ensureSkillsLoaded()`：

```typescript
private async ensureSkillsLoaded() {
  const history = JSON.parse(agent.llmHistory);
  if (historyHasSkills(history)) return;           // 已有则跳过
  const skillsBlock = await buildSkillsBlock();    // 构建 skill 内容块
  history.push({ role: "system", content: skillsBlock });
  await store.setAgentHistory({ agentId, llmHistory: JSON.stringify(history) });
}
```

`buildSkillsBlock()` 加载两种 skill：
1. **Metadata prompt**：所有 skill 的名称/描述索引（让 LLM 知道有哪些 skill 可用）
2. **Auto-load skills**：标记为自动加载的 skill 内容（如 `remotion-best-practices`）

Skill 通过 `SKILLS_MARKER = "[skills:loaded]"` 标记防止重复注入。

---

### 2.4 Agent 状态机（从前端推断）

前端维护 `AgentStatus = "IDLE" | "BUSY" | "WAKING"` 三态：

| 状态 | 触发事件 | 视觉呈现 |
|---|---|---|
| `IDLE` | `ui.agent.created` / `ui.agent.llm.done` | 默认态 |
| `BUSY` | `ui.agent.llm.start` | 显示"正在思考"指示器 |
| `WAKING` | （推断：`agent.wakeup` SSE 事件）| 中间态 |

---

## 三、完整知识图谱更新 / Updated Knowledge Map

上次总结的"Agent 处理流程"现在可以加入 interrupt 细节：

```
[挂起：await wake.promise]
         ↓ requestInterrupt() 或 wakeup()
loop() 醒来
    ↓
processUntilIdle()
    ├─ 检测点 1：进入时检查 interrupt
    ├─ while(true)
    │   ├─ 检测点 2：拉 unread 前检查 interrupt
    │   ├─ 拉取所有 unread batches（按 group 分组）
    │   ├─ if batches.length == 0 → return（回到挂起）
    │   └─ for each batch:
    │       ├─ 检测点 3：batch 前检查 interrupt
    │       ├─ processGroupUnread(groupId, messages)
    │       │   ├─ 构建 system prompt（含 skills block）
    │       │   ├─ 注入 unread 为 user 消息
    │       │   ├─ 标记已读（先标记再推理）
    │       │   ├─ runWithTools(maxRounds=3) ← 正常轮
    │       │   ├─ if !didSend → 注入提醒 → runWithTools(maxRounds=3) ← followup 轮
    │       │   └─ 保存 llmHistory → emit ui.agent.history.persisted
    │       └─ 检测点 4：batch 后检查 interrupt
[回到挂起]
```

---

## 四、洞察与收获 / Insights & Takeaways

### 4.1 三个新洞察

**洞察 1：`maxToolRounds = 3` 的含义**

3 轮上限配合"未发送则强制 followup"的双重机制，意味着每轮消息最多 6 次 LLM 调用。这是**稳定性 vs. 能力的权衡**：轮数太少 Agent 无法完成复杂工具链；太多则 API 成本失控、响应变慢。3 轮是经验值，并不是因为 3 有什么特殊含义。

**洞察 2：`upstash-realtime.ts` 名字具有误导性**

文件名叫 `upstash-realtime` 但实际是**标准 Redis Streams 实现**，并没有用 Upstash 专有 SDK。理由推测：作者最初可能计划用 Upstash（托管 Redis），后来改为可自托管的标准 Redis，但文件名未更新。

**洞察 3：Skills 是 Agent 领域知识的注入机制**

`remotion-best-practices` Skill 的存在说明：Swarm-IDE 的 Agent 可以通过 Skill 获得**特定领域的结构化专业知识**（20+ 个规则文件，每个聚焦一个 Remotion 子话题）。这类似于 RAG，但更简单：直接把精心整理的文档注入 system prompt。

---

### 4.2 设计哲学的完整版本

结合 `tech_stack.md` 的官方表述和代码的实际实现：

| 哲学原则 | 代码体现 |
|---|---|
| 去中心化：人类 = 特殊 Agent | `role = "human"` 过滤，共享 messages 表 |
| 极简原语：create + send | 10 个工具，核心只有 2 个 |
| 液态拓扑：运行时自演化 | `create` 工具触发 `ensureRunner` |
| 扁平协作：人可介入任意层级 | 侧边栏显示所有 human 所在的群 |
| 透明度强制：强制发送机制 | `didSend` 检测 + 注入 Reminder |

---

## 五、已解决 / 新增问题 / Questions Resolved & New

### ✅ 已解决（上次 6 个全部）

- [x] `specs/` 目录内容 → PRD + 技术方案文档，含完整设计意图
- [x] `maxToolRounds` → **3**，硬编码不可配置
- [x] 前端 SSE 消费 → 原生 `EventSource`，两个流（`ui-stream` + `context-stream`）
- [x] `upstash-realtime.ts` → Redis Streams + Pub/Sub，历史重播已处理竞态
- [x] `interruptRequested` → per-step 4 个检测点
- [x] `.agents/` → 无预置 Agent，有 `remotion-best-practices` Skill

### 🆕 新的待解问题

- [ ] `skill-loader.ts` 如何判断哪些 Skill 是 "auto-load"？有无配置文件？
- [ ] `storage.ts`（`store` 对象）如何实现 `listUnreadByGroup`？SQL 查询细节？
- [ ] Graph 页面（`/api/agent-graph`）如何构建 nodes + edges？是实时还是快照？
- [ ] `IMMessageList.tsx` 和 `IMHistoryList.tsx` 具体实现什么？

---

## 六、下一步计划（更新）/ Next Steps (Updated)

| 优先级 | 行动 | 状态 |
|---|---|---|
| 高 | **本地部署 Swarm-IDE**（`docker compose up` + `bun dev`）实际运行 | ⬜ 待做 |
| 高 | 阅读 `storage.ts` — `listUnreadByGroup` SQL 实现 | ⬜ 待做 |
| 中 | 阅读 `skill-loader.ts` — Skill 自动加载判断逻辑 | ⬜ 待做 |
| 中 | 撰写对比文章：Claude Code / Swarm-IDE / OpenAI Swarm 三者横向对比 | ⬜ 待做 |
| 低 | 阅读 `IMMessageList.tsx` + `IMHistoryList.tsx` 前端细节 | ⬜ 待做 |

---

## 七、本次生成的文件 / Files Generated

| 文件名 | 描述 |
|---|---|
| `learning-summary_2026-03-01_3.md` | 本文件，Swarm-IDE 本地源码深读，6 个悬案全部破解 |

---

## 八、关键词 / Keywords

`maxToolRounds=3` `interruptRequested` `per-step-interrupt` `EventSource` `ui-stream` `context-stream` `wakeAgent-on-subscribe` `Redis-Streams` `XREADGROUP` `XADD` `Consumer-Group` `history-replay` `remotion-best-practices` `Skill-auto-load` `ensureSkillsLoaded` `SKILLS_MARKER` `spells` `map-reduce` `router-experts` `AgentStatus` `IDLE-BUSY-WAKING` `processUntilIdle` `runWithTools`

---

*基于本地源码 `/Users/terry/Desktop/code code/swarm-ide`，阅读日期 2026-03-01*
*Based on local source code at swarm-ide project, date 2026-03-01*
