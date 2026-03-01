---
project: P004 · Swarm-IDE
file_type: learning-summary
tags: [swarm-ide, nextjs, sse, skill-loader, storage, frontend, framework-summary]
date: 2026-03-01
session_type: 深度研究
source: https://github.com/chmod777john/swarm-ide
related: p004_local-source-deep-dive_2026-03-01.md
---

# 学习总结：Swarm-IDE 前端架构完整拆解 + 全项目框架总结
# Learning Summary: Swarm-IDE Frontend Architecture Deep Dive + Full Project Framework

> **版本说明：** 本文是 `_3` 的续集。聚焦前端实现细节（page.tsx、storage.ts、skill-loader.ts）并输出全项目框架图。
> Sequel to `_3`. Focus: frontend implementation details and full project framework map.

---

## 会话元数据 / Session Metadata

| 字段 Field | 内容 Content |
|---|---|
| 日期 Date | 2026-03-01 |
| 时长（估算）Duration | ～ 1.5 小时 |
| 项目名称 Project | P004 Swarm-IDE（续三）|
| 主题 Topic | 前端完整实现 + 全项目框架总结 |
| 触发原因 Context | 继续本地源码精读，重点转向前端层 |
| 会话类型 Session Type | ☑ 深入研究 Deep Dive |
| 关联上次会话 Prev Session | `learning-summary_2026-03-01_3.md` |

---

## 一、前端技术栈全景 / Frontend Tech Stack Overview

Swarm-IDE 前端完全内嵌在 Next.js 的 `backend/app/` 目录下，是典型的**全栈一体化**项目：

```
backend/
├── app/
│   ├── im/
│   │   ├── page.tsx         ← ⭐ 主战场：2190 行单文件 React 客户端组件
│   │   ├── IMShell.tsx      ← 纯布局壳（三列 left/mid/right）
│   │   ├── IMMessageList.tsx← 消息气泡列表（~53 行，极简）
│   │   └── IMHistoryList.tsx← LLM history 折叠列表（~43 行，极简）
│   ├── graph/
│   │   └── page.tsx         ← Graph 静态快照页（独立路由，按需拉取）
│   ├── _components/
│   │   ├── create-workspace.tsx ← 创建 Workspace 按钮
│   │   └── clear-db.tsx         ← 管理操作按钮
│   └── api/                 ← Next.js Route Handlers（纯后端）
```

**引用的第三方库（前端）：**

| 库 | 用途 |
|---|---|
| `framer-motion` | Agent 节点动画、面板出现/消失动效 |
| `streamdown` | 流式 Markdown 渲染（实时 LLM 输出） |
| `@streamdown/code` | 代码高亮（`github-dark` 主题） |
| `@streamdown/mermaid` | 流式 Mermaid 图表渲染 |
| `lucide-react` | 图标（Briefcase, Network, User 等） |

---

## 二、前端核心：page.tsx 完整解析 / page.tsx Deep Dive

### 2.1 规模与架构

`backend/app/im/page.tsx`：**2190 行**，单文件 React 客户端组件（`"use client"`）。

这是一个刻意的**集中式设计**：无 Redux/Zustand，所有状态在 `IMPageInner()` 这一个组件函数内管理，通过 props 向子组件传递。

### 2.2 全状态清单（28 个 useState + 12 个 useRef）

**核心业务状态（useState）：**

```typescript
// 会话
const [session, setSession] = useState<WorkspaceDefaults | null>(null);
// session = { workspaceId, humanAgentId, assistantAgentId, defaultGroupId }

// 数据
const [groups, setGroups] = useState<Group[]>([]);      // 侧边栏会话列表
const [agents, setAgents] = useState<AgentMeta[]>([]);  // 全部 Agent 列表
const [messages, setMessages] = useState<Message[]>([]); // 当前会话消息
const [draft, setDraft] = useState("");                  // 输入框草稿

// 状态机
const [status, setStatus] = useState<
  "boot" | "groups" | "messages" | "send" | "idle"
>("boot");

// Agent 实时流状态（右侧面板）
const [contentStream, setContentStream] = useState("");    // LLM 正在输出的内容
const [reasoningStream, setReasoningStream] = useState(""); // 思维链输出
const [toolStream, setToolStream] = useState("");           // 工具调用参数流
const [llmHistory, setLlmHistory] = useState("");          // LLM 完整历史（JSON）
const [agentStatusById, setAgentStatusById] = useState<   // Agent 状态：IDLE/BUSY/WAKING
  Record<string, AgentStatus>
>({});

// Agent Graph 可视化状态
const [vizEvents, setVizEvents] = useState<VizEvent[]>([]);  // 事件日志（UI侧边）
const [vizBeams, setVizBeams] = useState<VizBeam[]>([]);     // 飞行光束（动态连线）
const [vizSize, setVizSize] = useState({ width: 640, height: 260 }); // Canvas 尺寸
const [vizScale, setVizScale] = useState(0.9);               // 缩放比例
const [vizOffset, setVizOffset] = useState({ x: 0, y: 0 }); // 平移偏移
const [nodeOffsets, setNodeOffsets] = useState<             // 可拖拽节点偏移
  Record<string, { x: number; y: number }>
>({});

// 布局控制
const [rightPanels, setRightPanels] = useState<RightPanelState[]>([
  { id: "history",   title: "LLM history",       size: 320, collapsed: false },
  { id: "content",   title: "Realtime content",   size: 220, collapsed: false },
  { id: "reasoning", title: "Realtime reasoning", size: 220, collapsed: false },
  { id: "tools",     title: "Realtime tools",     size: 200, collapsed: false },
]);
const [midSplitRatio, setMidSplitRatio] = useState(0.55); // 聊天区/Graph 高度比
```

**关键 useRef（绕过闭包捕获）：**

```typescript
const esRef = useRef<EventSource | null>(null);          // context-stream 连接
const uiEsRef = useRef<EventSource | null>(null);        // ui-stream 连接
const activeGroupIdRef = useRef<string | null>(null);    // 当前群 ID（避免旧闭包）
const agentRoleByIdRef = useRef<Map<string, string>>();  // agentId→role 映射（SSE handler 内使用）
const toolCallBuffersRef = useRef<Map<string, string>>(); // tool_call 流式拼接缓冲
const toolResultBuffersRef = useRef<Map<string, string>>(); // tool_result 流式拼接缓冲
const groupsRef = useRef<Group[]>([]);                   // groups 的 ref 副本（SSE handler 内使用）
const beamTimeoutsRef = useRef<number[]>([]);            // 光束自动消失的 timeout IDs
```

---

### 2.3 Agent Graph 布局算法

`vizLayout` 是一个纯函数 `useMemo`，输入 `agents`，输出每个 Agent 的 (x, y) 坐标。

算法是**树状层次布局**（Reingold-Tilford 简化版）：

```
1. 构建父子关系树（parentId → children[]）
2. 深度优先遍历（DFS walk）：
   - 叶节点：xIndex = leafIndex++（从左到右分配槽位）
   - 内部节点：xIndex = (leftmostChild + rightmostChild) / 2（居中于子节点）
3. 坐标映射：
   x = paddingX + xIndex * xStep   （水平，按叶节点槽位）
   y = paddingY + depth * yStep    （垂直，按层级深度）
4. 用户手动拖拽的偏移量叠加（nodeOffsets state）
5. 父子层级累计偏移（子节点随父节点移动）
```

**特殊处理：** human agent 永远排在 roots[] 的第一位（左上角），保证布局一致性。

---

### 2.4 前端可视化的「飞行光束」（VizBeam）

Agent 之间通信时，Graph 上会出现动态连线（光束）。实现机制：

```typescript
// 推入一个 beam（在 ui-stream onmessage 中调用）
const pushBeam = (beam: Omit<VizBeam, "id" | "createdAt">) => {
  const b = { ...beam, id: crypto.randomUUID(), createdAt: Date.now() };
  setVizBeams((prev) => [...prev, b]);
  // 2秒后自动移除（CSS 动画时长匹配）
  const t = window.setTimeout(() => {
    setVizBeams((prev) => prev.filter((x) => x.id !== b.id));
  }, 2000);
  beamTimeoutsRef.current.push(t);
};
```

beam 类型有两种：`"create"`（虚线箭头）和 `"message"`（实线箭头）。颜色与 Agent 状态颜色编码系统一致。

---

### 2.5 斜杠命令系统（/create, /hire）

输入框支持斜杠命令，`onSend` 函数在发送前先做前缀检测：

```typescript
if (text.startsWith("/create") || text.startsWith("/hire")) {
  const role = text.replace(/^\/(create|hire)\s*/i, "").trim();
  // 调用 POST /api/agents，创建 human→新Agent 的 P2P 群
  // 自动切换到新群，连接该 Agent 的 context-stream
  return;
}
```

这意味着用户可以直接在聊天框输入 `/create coder` 来创建一个新的 `coder` Agent 并自动进入其对话界面，**无需通过界面按钮**。

---

### 2.6 流式 Markdown 渲染

LLM 输出内容通过 `Streamdown` 库渲染，支持**实时流式更新**（不等到输出完成）：

```tsx
function MarkdownContent({ content }: { content: string }) {
  return (
    <Streamdown plugins={{ code, mermaid }}>
      {content}
    </Streamdown>
  );
}
```

这意味着 LLM 在输出 Mermaid 图表时，图表会随着字符的到来逐步渲染成型。

---

### 2.7 乐观更新（Optimistic UI）

发送消息时，前端立即插入一条带 `optimistic-${Date.now()}` 前缀 id 的假消息，然后才发起 API 请求：

```typescript
const optimistic: Message = {
  id: `optimistic-${Date.now()}`,
  senderId: session.humanAgentId,
  content: text,
  sendTime: new Date().toISOString(),
};
setMessages((m) => [...m, optimistic]);
setDraft("");
// 之后发请求，再拉取真实消息替换
```

---

### 2.8 角色颜色编码系统

```typescript
const roleColor = (role?: string) => {
  if (role === "human")          return "#f8fafc"; // 白色
  if (role === "assistant")      return "#38bdf8"; // 蓝色
  if (role === "productmanager") return "#fb7185"; // 粉红
  if (role === "coder")          return "#34d399"; // 绿色
  return "#fbbf24";                                 // 其他：金色
};

const statusColor = (status?: AgentStatus) => {
  if (status === "BUSY")   return "#ef4444"; // 红色（正在推理）
  if (status === "WAKING") return "#facc15"; // 黄色（刚被唤醒）
  return "#22c55e";                           // 绿色（IDLE）
};
```

---

## 三、storage.ts 的 `listUnreadByGroup` SQL 详解

**答案（上次 Q 留下的悬案）：**

```typescript
// 步骤 1：查出该 Agent 所有群成员资格（含 lastReadMessageId）
const memberships = await db
  .select({ groupId, lastReadMessageId })
  .from(groupMembers)
  .where(eq(groupMembers.userId, agentId));

// 步骤 2：对每个群，查出 cutoff 时间（lastReadMessageId 对应消息的 sendTime）
// 步骤 3：查出 sendTime > cutoff 且 senderId != agentId 的未读消息
for (const m of memberships) {
  const cutoff = lastReadMessageId
    ? db.select({ sendTime }).from(messages).where(eq(id, m.lastReadMessageId))
    : new Date(0);

  const rows = db.select(...)
    .where(
      and(
        eq(messages.groupId, m.groupId),
        gt(messages.sendTime, cutoff),      // ← 时间戳比较（非 ID 比较）
        ne(messages.senderId, agentId)      // ← 排除自己发的消息
      )
    );
}
```

**关键细节：**
- 未读边界用**时间戳**（`sendTime > cutoff`）而非消息 ID 大小比较
- 消息 ID 是 UUID v7（按时间排序），但代码选择用时间戳比较，可能是为了兼容性
- `ne(messages.senderId, agentId)`：Agent 不会"读"到自己发的消息，避免自触发
- **N+1 查询问题存在**：每个 group membership 触发 2 次额外查询，不做批量优化

---

## 四、skill-loader.ts 完整解析

### 4.1 Skill 自动加载判断

**`auto-load` 在 SKILL.md frontmatter 中声明：**

```yaml
---
name: remotion-best-practices
description: Best practices for Remotion
auto-load: true   ← 有这个才自动注入到所有 Agent 的 system prompt
---
```

代码：

```typescript
const autoLoad = parseBoolean(
  frontmatter["auto-load"] ?? frontmatter["auto_load"]
);
// parseBoolean 接受: true/false/1/0/yes/y/no/n
```

### 4.2 Skill 发现路径

按以下优先级查找 `skills/` 目录：

```
1. 环境变量 AGENT_SKILLS_DIR
2. process.cwd() + "/skills"
3. process.cwd() + "/backend/skills"
```

使用第一个存在的目录。对应项目中的 `backend/skills/`（如果存在）或 `.agents/skills/`（不是自动查找路径，需要通过 Agent 工具按名称加载）。

### 4.3 Skill 路径替换（processSkillPaths）

Skill 内容中的相对路径会被替换成绝对路径，确保 Agent 执行 `bash` 时能找到文件：

```
python scripts/example.py  →  python /abs/path/to/skill/scripts/example.py
see rules/animations.md    →  see `/abs/path/.../rules/animations.md` (use read_file to access)
[link text](relative.md)   →  [link text](`/abs/path/relative.md`) (use read_file to access)
```

这是**在 Prompt 层自动处理相对路径**的工程技巧，让 Agent 无需自己解析路径。

---

## 五、全项目框架图 / Full Project Architecture Map

### 5.1 目录结构全景

```
swarm-ide/
├── backend/                    ← Next.js 全栈应用（Bun 运行）
│   ├── app/
│   │   ├── im/                ← ⭐ IM 界面（主入口）
│   │   │   ├── page.tsx       ← 2190行单组件，全部前端逻辑
│   │   │   ├── IMShell.tsx    ← 三列布局壳
│   │   │   ├── IMMessageList.tsx ← 消息气泡（极简）
│   │   │   └── IMHistoryList.tsx ← LLM历史折叠列表
│   │   ├── graph/
│   │   │   └── page.tsx       ← Graph 快照页（独立路由）
│   │   ├── api/
│   │   │   ├── ui-stream/     ← GET：workspace 级 SSE
│   │   │   ├── agents/
│   │   │   │   ├── [agentId]/
│   │   │   │   │   ├── context-stream/ ← GET：per-agent LLM 流 SSE
│   │   │   │   │   └── route.ts        ← GET agent info
│   │   │   │   ├── route.ts            ← GET/POST agents
│   │   │   │   └── interrupt-all/      ← POST：中断所有 Agent
│   │   │   ├── groups/
│   │   │   │   ├── [groupId]/
│   │   │   │   │   └── messages/       ← GET/POST messages
│   │   │   │   └── route.ts            ← GET groups
│   │   │   ├── workspaces/             ← GET/POST workspaces
│   │   │   ├── agent-graph/            ← GET：nodes+edges 快照
│   │   │   ├── search/                 ← GET：搜索 agents/groups
│   │   │   └── admin/                  ← init-db/reset/clear-db
│   │   └── page.tsx           ← 首页（Workspace 选择/创建）
│   └── src/
│       ├── runtime/
│       │   ├── agent-runtime.ts ← ⭐ 核心：AgentRunner + AgentRuntime
│       │   ├── event-bus.ts     ← per-agent 内部事件总线
│       │   ├── ui-bus.ts        ← per-workspace UI 事件总线
│       │   ├── upstash-realtime.ts ← Redis Streams 跨进程层
│       │   ├── skill-loader.ts  ← Skill 发现+加载+路径替换
│       │   ├── mcp.ts           ← MCP 工具动态加载
│       │   ├── utils.ts         ← createDeferred 等工具函数
│       │   └── agent-logger.ts  ← 调试日志记录
│       ├── lib/
│       │   ├── storage.ts       ← ⭐ 所有 DB 操作（store 对象）
│       │   ├── config.ts        ← 环境变量读取（LLM 配置）
│       │   ├── glm-stream.ts    ← GLM（智谱）SSE 流解析
│       │   └── openai-stream.ts ← OpenAI/OpenRouter 流解析
│       └── db/
│           ├── schema.ts        ← Drizzle ORM 表定义（5 张表）
│           ├── init.ts          ← 建表 SQL
│           ├── client.ts        ← PostgreSQL 连接
│           └── index.ts         ← getDb() 函数
├── .agents/
│   └── skills/
│       └── remotion-best-practices/ ← Remotion 专属 Skill（20+ 规则文件）
├── specs/
│   └── implementation-plan/
│       ├── prd.md               ← 产品需求文档
│       └── tech_stack.md        ← 技术方案文档
├── spells/                      ← 可复用 Prompt 模板
│   ├── map-reduce.md            ← MapReduce 协作模式
│   ├── router-experts.md        ← 路由专家模式
│   └── tree-executor.md         ← 树状执行模式（未读）
└── promo-video/                 ← Remotion 宣传视频项目
    └── src/
        ├── DemoComposition.tsx
        └── Root.tsx
```

---

### 5.2 数据流全图（从用户发消息到 Agent 响应）

```
用户在浏览器输入框输入文字
        ↓
IMPageInner.onSend()
        ↓  乐观更新：立即在本地显示消息
POST /api/groups/:groupId/messages
        ↓
store.sendMessage() → INSERT INTO messages
        ↓  emitDbWrite → ui.db.write 事件
getAgentRuntime().wakeAgentsForGroup(groupId, senderId)
        ↓
对每个非 human 成员：ensureRunner(memberId).wakeup("group_message")
        ↓
AgentRunner.wake.resolve() ← Promise 立即 resolve
        ↓
AgentRunner.loop() 从 await 处继续
        ↓
processUntilIdle() → processGroupUnread()
        ↓
1. 拉取 unread messages（listUnreadByGroup SQL）
2. 构建 system prompt + unread 注入为 user 消息
3. 标记已读（markGroupReadToMessage）
4. callLlmStreaming() → GLM/OpenRouter SSE
   ├── 每个 chunk → emit AgentEventBus(agent.stream)
   │       ↓ Redis XADD + PUBLISH
   │   context-stream SSE → 浏览器右侧面板实时更新
   └── 工具调用 → executeToolCall()
          ├── send_group_message → INSERT messages → 唤醒其他 Agent
          └── create → INSERT agent → ensureRunner → wakeup
5. 落库：setAgentHistory（UPDATE agents.llm_history）
6. emit WorkspaceUIBus(ui.agent.history.persisted)
        ↓ Redis XADD + PUBLISH
ui-stream SSE → 浏览器收到 → refreshGroups() + refreshMessages()
        ↓
侧边栏更新未读数，当前会话显示 Agent 回复消息
```

---

### 5.3 前端层次关系图

```
浏览器
├── [/ 首页]  page.tsx
│   ├── createWorkspace() → POST /api/workspaces
│   └── 进入 /im
│
└── [/im] IMPageInner（2190行单组件）
    │
    ├── 左侧栏（left）
    │   ├── Workspace 切换器
    │   ├── 全局搜索（GET /api/search）
    │   ├── Agent 树形列表（collapsibleAgents）
    │   └── 对话列表（groups，含未读小红点）
    │
    ├── 中间区（mid，可垂直拖拽调整）
    │   ├── 上：聊天窗口
    │   │   ├── IMMessageList（消息气泡）
    │   │   └── 输入区（草稿 + 发送按钮 + /create 命令 + Hire 按钮）
    │   └── 下：Agent Graph 可视化
    │       ├── SVG 画布（节点 + 父子边 + 飞行光束）
    │       ├── 节点可拖拽（startNodeDrag）
    │       ├── 整体可平移/缩放（vizOffset + vizScale）
    │       └── 事件日志列表（vizEvents，右侧小窗）
    │
    └── 右侧栏（right，4个可折叠面板）
        ├── [history]   IMHistoryList（LLM history JSON，<details> 折叠）
        ├── [content]   Streamdown（实时 LLM 输出 Markdown）
        ├── [reasoning] Streamdown（实时思维链输出）
        └── [tools]     pre 标签（工具调用参数流 + 结果流）
```

---

### 5.4 后端运行时关系图

```
┌─────────────────────────────────────────────────────────────┐
│                    AgentRuntime（全局单例）                   │
│  runners: Map<UUID, AgentRunner>                             │
│  bus: AgentEventBus                                          │
│                                                              │
│  ┌─────────────────────────┐  ┌─────────────────────────┐   │
│  │  AgentRunner (human)    │  │  AgentRunner (assistant) │   │
│  │  wake = Deferred<void>  │  │  wake = Deferred<void>   │   │
│  │  [永不推理，role过滤]    │  │  loop() 等待 → 推理      │   │
│  └─────────────────────────┘  └─────────────────────────┘   │
│                                        ↕ wakeup()           │
│  ┌─────────────────────────┐  ┌─────────────────────────┐   │
│  │  AgentRunner (coder)    │  │  AgentRunner (reviewer)  │   │
│  │  loop() 等待 → 推理      │  │  loop() 等待 → 推理      │   │
│  └─────────────────────────┘  └─────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
           ↕ emit()                         ↕ emit()
┌──────────────────────┐       ┌──────────────────────────────┐
│  AgentEventBus       │       │  WorkspaceUIBus               │
│  (per-agent)         │       │  (per-workspace)              │
│  环形缓冲 2000 事件   │       │  环形缓冲 2000 事件            │
│  + Redis Streams     │       │  + Redis Streams              │
└──────────┬───────────┘       └──────────────┬───────────────┘
           ↓ SSE                              ↓ SSE
  /api/agents/:id/context-stream      /api/ui-stream?workspaceId=...
           ↓                                  ↓
  前端右侧面板实时流                   前端侧边栏刷新触发器
```

---

## 六、洞察与收获 / Key Insights

### 6.1 最重要的 3 个新发现

**发现 1：`page.tsx` 是一个刻意的"巨石组件"**

2190 行单组件是**故意为之**，不是技术债。理由：所有状态需要交叉引用（Graph 需要 messages 数据，SSE handler 需要 agentRoleById，光束动画需要 groups 数据），提取成子组件需要大量 Context 或 prop drilling。作者选择了"简单粗暴但可读"的方案。代价是难以测试，优点是逻辑集中、调试清晰。

**发现 2：`listUnreadByGroup` 是 N+1 查询但有意为之**

对每个 group membership，分别发 2 次额外查询（查 cutoff 时间、查未读消息）。在 Agent 数量少、群组有限的 MVP 场景下，N+1 不是问题。这与 `agents.llm_history` 用 TEXT 存储的决策一脉相承：**为简单性牺牲规范性**。

**发现 3：Skill 系统是"Prompt RAG"的极简实现**

`skill-loader.ts` 做了三件事：YAML frontmatter 解析、相对路径→绝对路径替换、auto-load 过滤。最终结果是把精心整理的文档注入到 Agent 的 system prompt。这是 RAG（检索增强）的最简版本：不做向量检索，直接全量注入，利用 LLM 的 context window 做"检索"。对于专业领域知识（如 Remotion 的 20+ 规则），效果不逊于 RAG，且零基础设施依赖。

---

### 6.2 前端与后端的对称性

Swarm-IDE 有一个优雅的设计对称性：

| 维度 | 后端 | 前端 |
|---|---|---|
| 事件推送 | AgentEventBus（per-agent）| context-stream SSE（per-agent 面板）|
| 事件推送 | WorkspaceUIBus（per-workspace）| ui-stream SSE（侧边栏刷新触发）|
| 状态存储 | llm_history TEXT（per-agent）| llmHistory string（per-agent 状态）|
| 唤醒机制 | wakeup() resolve Promise | connectAgentStream() 打开 EventSource |
| 中断机制 | requestInterrupt() + flag | onInterruptAllAgents() + POST API |

后端的 Promise deferred 和前端的 EventSource 是同一个"等待-被触发"哲学在不同层次的体现。

---

## 七、最终待解问题 / Remaining Open Questions

- [ ] `spells/tree-executor.md` 的内容是什么？（还未读）
- [ ] `agent-logger.ts` 具体记录什么？日志存在哪里？
- [ ] `glm-stream.ts` 和 `openai-stream.ts` 的差异点在哪？（推测：GLMStreamAssembler 需要特殊处理 `reasoning_content`）
- [ ] `promo-video/` 项目本身是用 Swarm-IDE 生成的，还是手工写的？
- [ ] `search/route.ts` 实现了什么搜索逻辑？

---

## 八、下一步计划（最终版）/ Next Steps (Final)

| 优先级 | 行动 | 状态 |
|---|---|---|
| 🔥 高 | **本地部署运行**：`docker compose up` + `bun dev` + 实际对话 | ⬜ 待做 |
| 高 | 撰写横向对比文章：Claude Code / Swarm-IDE / OpenAI Swarm | ⬜ 待做 |
| 中 | 读 `glm-stream.ts`：GLM 特殊流格式（reasoning_content）| ⬜ 待做 |
| 中 | 读 `agent-logger.ts`：调试日志机制 | ⬜ 待做 |
| 低 | 读 `spells/tree-executor.md` | ⬜ 待做 |

---

## 九、关键词 / Keywords

`page.tsx` `IMShell` `IMMessageList` `IMHistoryList` `useState-28个` `useRef-12个` `vizLayout` `Reingold-Tilford` `VizBeam` `飞行光束` `乐观更新` `Optimistic-UI` `Streamdown` `streamdown-mermaid` `slash-command` `/create` `/hire` `roleColor` `statusColor` `listUnreadByGroup` `N+1查询` `skill-loader` `auto-load` `processSkillPaths` `路径替换` `Prompt-RAG` `巨石组件` `monolithic-component`

---

*基于本地源码 `/Users/terry/Desktop/code code/swarm-ide`，阅读日期 2026-03-01*
