---
project: P005 · DeerFlow Frontend
file_type: learning-summary
tags: [nextjs, langgraph, react-query, artifact-panel, xy-flow, sse, streaming]
date: 2026-03-01
session_type: 初次学习
source: https://github.com/bytedance/deer-flow
---

# 学习总结 / Learning Summary
## P005 · DeerFlow Frontend Architecture

> **日期 / Date:** 2026-03-01
> **项目 / Project:** P005 · DeerFlow（ByteDance 开源）
> **来源 / Source:** https://github.com/bytedance/deer-flow
> **聚焦 / Focus:** 前端架构全貌与可复现方案 (Frontend architecture & replicable patterns)
> **状态 / Status:** ✅ 完成第一次深潜（GitHub 源码远程阅读）

---

## 一、项目定位 / Project Overview

DeerFlow 是字节跳动开源的 **"Super Agent Harness"**，定位是多 Agent 编排平台——让 AI Agent 能在隔离的 Docker 容器中执行长达数分钟甚至数小时的复杂任务。

**与已学项目的差别：**
- P002 OpenClaw：Lit Web Components + WebSocket + A2UI 声明式 UI
- P004 Swarm-IDE：IM 隐喻 + PostgreSQL + SSE 双总线 + Promise-deferred 唤醒
- **P005 DeerFlow：Next.js App Router + LangGraph SDK + React Query + Artifact Panel**

核心设计哲学：**前端不自己写 Agent 状态机，把 Agent 执行状态完全托管给 LangGraph SDK，前端只负责"显示流 + 交互操作"。**

---

## 二、完整技术栈 / Full Tech Stack

### 框架层
| 层级 | 选型 | 版本 |
|---|---|---|
| Meta-framework | **Next.js** | 16.1.4（App Router）|
| UI Library | **React** | 19.0.0 |
| 语言 | **TypeScript** | 5.8.2 |
| 包管理 | **pnpm** | 10.26.2（workspace 模式）|
| 运行时 | Node.js | 22+ |

### UI & 样式
| 技术 | 说明 |
|---|---|
| **Tailwind CSS v4** | 原子化 CSS，postcss 处理 |
| **Radix UI**（20+ 组件） | 无障碍原语，底层支撑 |
| **Shadcn/UI** | 基于 Radix 的可复制组件库（`components.json` 配置）|
| **Lucide React** | 图标库 |
| **Class Variance Authority** | 变体样式管理（cva）|

### 状态与数据
| 技术 | 用途 |
|---|---|
| **LangGraph SDK** (`@langchain/langgraph-sdk@1.5.3`) | Agent 线程 streaming 状态（**核心**）|
| **LangChain Core** (`@langchain/core@1.1.15`) | BaseMessage、消息类型定义 |
| **TanStack Query v5** | 服务端状态（线程列表、缓存、乐观更新）|
| **Vercel AI SDK 6.0.33** | 辅助流式 AI 接口 |

### 编辑器
- **CodeMirror 6**：嵌入式代码编辑器，支持 JS/Python/HTML/CSS/JSON/Markdown
- UIW CodeMirror 主题库

### 可视化与动画
| 技术 | 用途 |
|---|---|
| **XY Flow v12** | 节点/边图（可视化 Agent DAG / 执行流程图）|
| **GSAP 3.13** | 高性能动画 |
| **Motion 12** | React 动画声明式 API |
| **Canvas Confetti** | 任务完成庆祝动画 |

### 内容处理
| 技术 | 用途 |
|---|---|
| **Shiki 3.15** | 服务端语法高亮（SSR-first）|
| **Remark/Rehype + GFM** | Markdown 渲染完整管线 |
| **KaTeX** | 数学公式渲染 |

### 其他
| 技术 | 用途 |
|---|---|
| **Better Auth 1.3** | OAuth 认证（`/api/auth/[...all]` catch-all）|
| **ESLint 9 + Prettier** | 代码质量工具 |

---

## 三、目录架构 / Directory Architecture

```
frontend/
├── src/
│   ├── app/                              ← Next.js App Router
│   │   ├── layout.tsx                    ← 根 layout
│   │   ├── page.tsx                      ← 重定向（静态模式→demo；正常→新会话）
│   │   ├── api/auth/[...all]/            ← Better Auth OAuth catch-all 路由
│   │   ├── mock/api/                     ← 开发/demo 用的 mock API
│   │   └── workspace/
│   │       ├── layout.tsx                ← workspace 共享布局
│   │       ├── page.tsx                  ← 重定向至 /workspace/chats/[thread_id]
│   │       └── chats/
│   │           ├── page.tsx              ← 聊天列表页
│   │           └── [thread_id]/          ← 单线程对话页（动态路由）
│   │
│   ├── components/
│   │   ├── ai-elements/                  ← AI 专用 UI 元素
│   │   ├── landing/                      ← 落地页组件
│   │   ├── ui/                           ← Shadcn/UI 基础组件库
│   │   ├── theme-provider.tsx            ← 主题切换
│   │   └── workspace/                    ← 工作台专用组件
│   │       ├── messages/
│   │       │   ├── message-list.tsx
│   │       │   ├── message-group.tsx     ← 按发送者/时间分组
│   │       │   ├── message-list-item.tsx
│   │       │   ├── markdown-content.tsx  ← Remark/Rehype 渲染管线
│   │       │   ├── subtask-card.tsx      ← Agent 子任务状态卡片 ⭐
│   │       │   ├── skeleton.tsx          ← 流式加载占位符
│   │       │   └── context.ts
│   │       ├── artifacts/                ← 产出物渲染（代码/文档/报告）⭐
│   │       ├── citations/                ← 引用标注组件
│   │       ├── settings/                 ← 设置面板
│   │       ├── input-box.tsx             ← 用户输入（含文件上传）
│   │       ├── todo-list.tsx             ← Agent 自动生成待办清单 ⭐
│   │       ├── streaming-indicator.tsx   ← 流式进度指示器
│   │       ├── workspace-container.tsx   ← 主布局容器
│   │       ├── workspace-header.tsx      ← 顶栏（面包屑 + GitHub 链接）
│   │       ├── workspace-sidebar.tsx     ← 侧边栏
│   │       ├── workspace-nav-chat-list.tsx
│   │       └── workspace-nav-menu.tsx
│   │
│   ├── core/                             ← 业务逻辑层（19 个模块）
│   │   ├── api/          ← HTTP / streaming API 客户端
│   │   ├── artifacts/    ← 产出物管理
│   │   ├── config/       ← 应用配置
│   │   ├── i18n/         ← 国际化
│   │   ├── mcp/          ← Model Context Protocol 集成 ⭐
│   │   ├── memory/       ← 状态 & 持久化
│   │   ├── messages/     ← 消息领域逻辑
│   │   ├── models/       ← 数据模型定义
│   │   ├── notification/ ← 通知系统
│   │   ├── rehype/       ← HTML/Markdown 处理插件
│   │   ├── settings/     ← 用户偏好
│   │   ├── skills/       ← Skill 加载与管理
│   │   ├── streamdown/   ← 自定义流式数据处理（插件化）⭐
│   │   ├── tasks/        ← 任务编排
│   │   ├── threads/      ← 对话线程（核心模块）⭐
│   │   │   ├── types.ts   → AgentThreadState / AgentThread 定义
│   │   │   ├── hooks.ts   → useThreadStream / useSubmitThread 等
│   │   │   ├── utils.ts
│   │   │   └── index.ts
│   │   ├── todos/        ← Todo 跟踪
│   │   ├── tools/        ← 工具集成
│   │   ├── uploads/      ← 文件上传
│   │   └── utils/        ← 通用工具函数
│   │
│   ├── hooks/            ← 应用级 React Hooks
│   ├── lib/              ← 共享工具库
│   ├── server/better-auth/ ← 服务端认证
│   ├── styles/           ← 全局 CSS
│   ├── typings/          ← TypeScript 类型声明
│   └── env.js            ← 启动时环境变量验证
│
├── public/demo/threads/  ← 静态演示数据（静态站模式）⭐
├── AGENTS.md             ← 架构说明
└── CLAUDE.md             ← Claude 使用指引
```

---

## 四、核心模块深潜 / Core Module Deep Dive

### 4.1 线程状态机 (Threads + LangGraph SDK)

**AgentThreadState 数据结构：**
```typescript
interface AgentThreadState extends Record<string, unknown> {
  title: string;
  messages: BaseMessage[];    // LangChain BaseMessage（含 Human/AI/Tool）
  artifacts: string[];        // 产出物 ID 列表
  todos?: Todo[];             // Agent 自动生成的待办清单
}
```

**四个核心 Hook：**

| Hook | 用途 | 关键技术 |
|---|---|---|
| `useThreadStream` | 初始化 streaming 连接，处理自定义事件（subtask 更新），完成后缓存失效 | LangGraph SDK `useStream<AgentThreadState>` |
| `useSubmitThread` | 用户消息提交、文件上传、传递 recursion limit 和 stream modes | LangGraph SDK |
| `useThreads` | 分页获取线程列表（search + 排序） | TanStack Query |
| `useDeleteThread` / `useRenameThread` | 线程操作，**乐观更新**（`setQueriesData` 本地先更新） | TanStack Query mutations |

**最重要的架构决策：**
Agent 执行状态（streaming、工具调用链、检查点恢复）= LangGraph SDK 负责
UI 数据（列表、缓存、后台同步）= TanStack Query 负责
两者通过 `AgentThreadContext` 传递元数据连接，**完全解耦**。

### 4.2 消息渲染管线

```
message-list.tsx
  └── message-group.tsx           ← 按发送者/时间分组
        └── message-list-item.tsx
              ├── markdown-content.tsx  ← Remark → Rehype → KaTeX → Shiki
              ├── subtask-card.tsx      ← Agent 子任务执行状态卡片
              └── skeleton.tsx          ← 流式加载骨架占位
```

### 4.3 StreamDown 插件管线

`core/streamdown/` 是自定义流式数据处理器，采用插件化架构（`index.ts` 导出所有 `plugins`）。推测是对 LangGraph SSE 流数据的解析分发层——不同的事件类型（工具调用、思考、结果）路由到不同的处理插件。

### 4.4 MCP 集成层

`core/mcp/` 实现 Model Context Protocol，支持 OAuth 流程（client_credentials、refresh_token）。这是 DeerFlow 扩展性的核心：通过标准 MCP 协议接入任意外部工具。

### 4.5 静态演示模式

`NEXT_PUBLIC_STATIC_WEBSITE_ONLY=true` 时，`/workspace/page.tsx` 读取 `public/demo/threads/` 下的预录制数据，无需后端即可运行完整演示。根路由重定向到第一个 demo thread。

---

## 五、路由与布局架构 / Routing & Layout

```
/
└── /workspace
      └── /workspace/chats
            └── /workspace/chats/[thread_id]   ← 核心对话页
```

WorkspaceContainer 布局结构：
- **WorkspaceHeader**（高度 4rem，backdrop blur）：动态面包屑（URL路径 → i18n label），右上角 GitHub 链接
- **WorkspaceBody**（剩余高度，居中响应式）：主内容区
- **WorkspaceSidebar**：对话历史列表 + 导航菜单

---

## 六、7 大可借鉴亮点 / 7 Replicable Highlights

### ⭐ 1. LangGraph `useStream` 作为前端 Agent 状态机

**模式：** 不自己写 WebSocket/SSE 消费逻辑，直接用 `useStream<AgentThreadState>` 订阅 Agent 执行状态。

**核心价值：** 复杂的 streaming reconnect、状态合并、检查点恢复全部由 SDK 处理。与 P004 Swarm-IDE 的手写 SSE 双总线相比，这个方案抽象层次更高、维护成本更低。

**可复现最小代码：**
```typescript
const { values, isLoading } = useStream<AgentThreadState>({
  apiUrl: process.env.NEXT_PUBLIC_LANGGRAPH_API_URL,
  assistantId: "lead_agent",
  threadId,
  onCustomEvent: (event) => handleSubtaskUpdate(event),
});
```

---

### ⭐ 2. React Query 乐观更新 + LangGraph 状态分离

**模式：** 服务端数据（线程列表）用 React Query 管理，Agent 流式执行状态用 LangGraph SDK 管理，两者分工明确，不混用。

**核心价值：** 解决"UI 响应快"（乐观更新）和"Agent 状态准确"（SDK 保证一致性）的矛盾。删除线程立刻体现在 UI，无需等服务端确认。

---

### ⭐ 3. Subtask Card —— Agent 进度可视化

**模式：** Agent 执行的每个子任务渲染为嵌入在消息流中的独立状态卡片，实时更新。

**核心价值：** 把 Agent 的"黑盒执行"变为用户可观测的透明流程，大幅降低等待焦虑（类似 P004 的强制发送透明度机制，但用户体验更友好）。

---

### ⭐ 4. Artifact 面板与消息流分离

**模式：** 普通对话显示在消息流；结构化产出（代码、报告、文档）显示在独立 Artifact 面板（内嵌 CodeMirror 可直接编辑）。

**核心价值：** 避免长代码/文档淹没对话流。用户可在 Artifact 面板直接编辑 Agent 生成的代码，形成"对话生成 → 面板编辑"的完整工作流。

---

### ⭐ 5. 插件化 StreamDown 管线

**模式：** `core/streamdown/plugins/` —— 对 Agent 输出流的每种数据类型注册独立处理插件。

**核心价值：** Agent 流式输出来源多样（工具调用结果、思考过程、最终回答），插件架构让每种类型的解析/展示逻辑互不干扰，方便按需扩展。

---

### ⭐ 6. 静态演示模式（Static Site Mode）

**模式：** `NEXT_PUBLIC_STATIC_WEBSITE_ONLY=true` 时读取预录制数据，无需后端即可运行完整 UI。

**核心价值：** 零成本 Demo 部署（GitHub Pages 等）、CI 截图测试、产品演示。极低成本展示效果。

---

### ⭐ 7. XY Flow 可视化 Agent DAG

**模式：** 使用 XY Flow（前身 React Flow）绘制 Agent 执行有向无环图。

**核心价值：** 对于 LangGraph 这类图结构 Agent 系统，DAG 可视化对调试、演示、理解执行流程都很有价值。与 P004 Swarm-IDE 的 IM 隐喻相比，这个方案更适合有复杂分支的 Research Agent。

---

## 七、与已学项目的对比 / Comparison with P002 & P004

| 维度 | P002 OpenClaw | P004 Swarm-IDE | **P005 DeerFlow** |
|---|---|---|---|
| 前端框架 | Lit Web Components | Next.js | **Next.js App Router v16** |
| 样式方案 | CSS vars + Zustand | Tailwind | **Tailwind v4 + Shadcn/UI** |
| 状态管理 | Web Components 内置 | Drizzle + SSE 双总线 | **LangGraph SDK + React Query** |
| 实时通信 | WebSocket JSON frames | SSE（WorkspaceUIBus）| **LangGraph Streaming** |
| Agent 唤醒 | A2UI 声明式 | Promise-deferred | **useStream 订阅** |
| 消息渲染 | 声明式 JSON UI（A2UI）| 待研究 | **Markdown + Subtask Card** |
| 产出物 | — | — | **Artifact Panel + CodeMirror** |
| 可视化 | — | — | **XY Flow DAG** |
| 认证 | Challenge-nonce | — | **Better Auth OAuth** |
| 演示模式 | — | — | **静态站模式（无后端）** |
| 扩展协议 | — | MCP（计划）| **MCP（已实现）+ OAuth** |

---

## 八、待探索清单 / Still To Study

- [ ] `frontend/src/core/api/` 完整源码：streaming fetch 实现细节
- [ ] `frontend/src/components/workspace/artifacts/` 源码：Artifact 渲染组件树
- [ ] `frontend/src/core/streamdown/plugins/` 源码：每个 plugin 处理哪种数据类型
- [ ] `frontend/src/app/workspace/chats/[thread_id]/` 完整页面组件实现
- [ ] XY Flow 集成：Agent DAG 如何映射到节点/边图（在哪个组件渲染）
- [ ] `backend/` 目录：LangGraph lead_agent 后端架构（与前端 useStream 的数据契约）
- [ ] `skills/public/` 目录：前端 Skill 加载机制（与 P001 skill-creator 对比）
- [ ] 实践：本地部署 DeerFlow，用 Wireshark/DevTools 观察 LangGraph streaming 数据格式

---

## 九、快速复现方案 / Quick Replication Guide

**最小技术栈（复现 DeerFlow 风格前端）：**

```bash
# 初始化
pnpm create next-app my-agent-ui --typescript

# 核心依赖
pnpm add @langchain/langgraph-sdk @langchain/core
pnpm add @tanstack/react-query
pnpm add @radix-ui/react-dialog @radix-ui/react-scroll-area
pnpm add tailwindcss @tailwindcss/typography

# 可选增强
pnpm add @xyflow/react          # Agent DAG 可视化
pnpm add @uiw/react-codemirror  # 代码编辑器
pnpm add shiki                  # 语法高亮
pnpm add gsap                   # 动画
pnpm add better-auth            # 认证
```

**关键架构模式：**
```typescript
// types.ts
interface AgentState {
  messages: BaseMessage[];
  todos?: { id: string; content: string; done: boolean }[];
  artifacts?: string[];
}

// hooks.ts - 核心 streaming hook
export function useAgentStream(threadId: string) {
  return useStream<AgentState>({
    apiUrl: process.env.NEXT_PUBLIC_LANGGRAPH_URL!,
    assistantId: "main_agent",
    threadId,
    onCustomEvent: (event) => {
      // 处理 subtask 更新等自定义事件
    },
  });
}

// hooks.ts - 线程列表（乐观更新）
export function useDeleteThread() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (id: string) => client.threads.delete(id),
    onMutate: async (id) => {
      // 乐观更新：本地先删除，不等服务端
      queryClient.setQueriesData({ queryKey: ["threads"] }, (old) =>
        old?.filter((t) => t.thread_id !== id)
      );
    },
  });
}
```

---

*总结 by Claude · 2026-03-01 · P005 DeerFlow 首次远程源码深读*
*Summary by Claude · 2026-03-01 · P005 DeerFlow First Remote Source Study*
