# 项目学习主索引 / Master Learning Index

> **最后更新 / Last Updated:** 2026-03-01
> **总会话数 / Total Sessions:** 9
> **已学项目数 / Projects Studied:** 6
> **生成文件数 / Files Generated:** 13

此文件是整个学习知识库的目录。每次学习会话结束后更新。
This file is the table of contents for the entire learning knowledge base. Update after each session.

**进度阶段说明 / Progress Stage Legend:**

| 标签 | 含义 |
|---|---|
| 🌱 初次学习 | 首次接触，建立基本认知 |
| 🔍 深度研究 | 源码级或细节级深入 |
| 🔁 深度复现 | 亲手复现核心逻辑 |
| ✅ 实践验证 | 完成真实任务，验证理解 |
| 📌 高频参考 | 已被多次引用或迁移到其他项目 |

---

## 📁 系统文件 / System Files

| 文件 File | 用途 Purpose | 最后更新 |
|---|---|---|
| `_index.md` | 本文件，主索引 This file | 2026-03-01 |
| `_HOW-TO-USE.md` | 使用指南 Usage guide | 2026-03-01 |
| `_quick-check.md` | 规范快速核查表，会话结束前扫一遍 | 2026-03-01 |
| `_personal-learning-log.md` | 系统变更日志，记录结构调整与设计决策 | 2026-03-01 |
| `_templates/template_learning-session.md` | 学习会话模版 Session template | 2026-02-27 |
| `_templates/skill-security-scan-template_2026-02-27.md` | Skill 安全扫描检查清单 | 2026-02-27 |

---

## 📚 项目学习记录 / Project Learning Records

### P001 · skill and e-evo

| 字段 | 内容 |
|---|---|
| 项目 ID | P001 |
| 项目名称 | skill and e-evo |
| 项目路径 | `/mnt/.projects/019c9cf4-4af3-762e-9dde-95fa6668f9be` |
| 主要主题 | Claude Skill 系统设计、评估方法论、安全原则 |
| 首次学习 | 2026-02-27 |
| 最近更新 | 2026-02-27 |
| 状态 Status | 🟡 进行中 In Progress |

**学习会话 / Sessions:**

| 日期 | 文件 | 主题 | 进度阶段 | 状态 |
|---|---|---|---|---|
| 2026-02-27 | [p001_skill-creator_2026-02-27.md](./p001-skill-evo/p001_skill-creator_2026-02-27.md) | skill-creator 架构 + Eval Loop + 安全原则 | 🌱 初次学习 | ✅ 完成 |

**存档文档 / Archived Docs:**

| 原始文档 | 存档文件 | 日期 |
|---|---|---|
| `skill-creator/SKILL.md` | [p001_skill-creator-archive_2026-02-27.md](./p001-skill-evo/p001_skill-creator-archive_2026-02-27.md) | 2026-02-27 |

**待探索 / Still To Study:**

- [ ] `agents/grader.md`
- [ ] `agents/comparator.md`
- [ ] `agents/analyzer.md`
- [ ] `references/schemas.md`
- [ ] `run_loop.py` 实现细节
- [ ] 实践：用 skill-creator 创建一个新 skill

---

### P002 · OpenClaw Frontend

| 字段 | 内容 |
|---|---|
| 项目 ID | P002 |
| 项目名称 | OpenClaw Frontend Study |
| 项目路径 | `https://github.com/openclaw/openclaw` |
| 主要主题 | 前端架构（Lit Web Components）、WebSocket Gateway 协议、Canvas+A2UI、可借鉴设计模式 |
| 首次学习 | 2026-02-28 |
| 最近更新 | 2026-02-28 |
| 状态 Status | 🟡 进行中 In Progress |

**学习会话 / Sessions:**

| 日期 | 文件 | 主题 | 进度阶段 | 状态 |
|---|---|---|---|---|
| 2026-02-28 | [p002_frontend-overview_2026-02-28.md](./p002-openclaw/p002_frontend-overview_2026-02-28.md) | 前端全貌：Lit UI架构 + WebSocket协议 + Canvas/A2UI + 7大可借鉴模式 | 🌱 初次学习 | ✅ 完成 |

**存档文档 / Archived Docs:**

| 原始文档 | 存档文件 | 日期 |
|---|---|---|
| _(待补充，GitHub 暂无法直接访问)_ | — | — |

**待探索 / Still To Study:**

- [ ] `ui/src/ui/app.ts` 完整源码：Lit 组件树结构
- [ ] `vendor/a2ui/` 源码：A2UI 解析器与注册表实现
- [ ] `src/provider-web.ts`：WebChat provider 细节
- [ ] PinchChat `useGateway` hook 完整实现
- [ ] 实践：最小 WebSocket client 连接 Gateway，实现自制 WebChat

---

### P003 · 工程师经验博文学习 Engineering Blog Learning

| 字段 | 内容 |
|---|---|
| 项目 ID | P003 |
| 项目名称 | 工程师经验博文学习 Engineering Blog Learning |
| 项目路径 | _(网络文章，无本地路径)_ |
| 来源类型 | 工程师公开博客 / 经验总结 |
| 主要主题 | AI 辅助编码方法论、工程师思维框架、工作流最佳实践 |
| ⚠️ 阅读原则 | **内容代表作者个人观点与经验推断，不作为普适定律。** 以 Terry 实际判断为主，作者观点仅供参考。 |
| 首次学习 | 2026-02-28 |
| 最近更新 | 2026-03-01 |
| 状态 Status | 🟡 进行中 In Progress |

**学习会话 / Sessions:**

| 日期 | 文件 | 主题 | 进度阶段 | 状态 |
|---|---|---|---|---|
| 2026-02-28 | [p003_boris-workflow_2026-02-28.md](./p003-engineering-blog/p003_boris-workflow_2026-02-28.md) | Boris Tane：纪律性 Claude Code 工作流（Research→Plan→Annotate→Implement） | 🌱 初次学习 | ✅ 完成 |
| 2026-02-28 | [p003_sdlc-is-dead_2026-02-28.md](./p003-engineering-blog/p003_sdlc-is-dead_2026-02-28.md) | Boris Tane：SDLC 已死——阶段坍塌、紧循环、上下文工程、可观测性成结缔组织 | 🌱 初次学习 | ✅ 完成 |
| 2026-03-01 | [p003_agent-teams-reverse_2026-03-01.md](./p003-engineering-blog/p003_agent-teams-reverse_2026-03-01.md) | @alterxyz4：Claude Code Agent Teams 逆向——文件系统消息队列、JSON inbox、5大可借鉴模式 | 🌱 初次学习 | ✅ 完成 |

**待探索 / Still To Study:**

- [x] Boris Tane：The SDLC is Dead ✅
- [x] @alterxyz4：Claude Code Agent Teams 逆向 ✅
- [ ] 实践验证：用 Research→Plan→Annotate→Implement 工作流完成一个真实任务
- [ ] 实践验证：在 Claude Code 中创建团队，观察 `~/.claude/teams/` 文件系统
- [ ] 探索其他工程师经验博文（来源待定）

---

### P004 · Swarm-IDE

| 字段 | 内容 |
|---|---|
| 项目 ID | P004 |
| 项目名称 | Swarm-IDE |
| 项目路径 | `https://github.com/chmod777john/swarm-ide` |
| 默认分支 | `chore/specs-mvp`（注意：不是 `main`）|
| 本地路径 | `/Users/terry/Desktop/code code/swarm-ide` |
| 主要主题 | 多 Agent 协作系统、IM 隐喻架构、Promise-deferred 事件驱动、双总线实时流 |
| 首次学习 | 2026-03-01 |
| 最近更新 | 2026-03-01 |
| 状态 Status | 🟡 进行中 In Progress |

**学习会话 / Sessions:**

| 日期 | 文件 | 主题 | 进度阶段 | 状态 |
|---|---|---|---|---|
| 2026-03-01 | [p004_core-architecture_2026-03-01.md](./p004-swarm-ide/p004_core-architecture_2026-03-01.md) | 源码级深潜：AgentRunner/AgentRuntime、Promise-deferred唤醒、IM系统prompt、双总线、PostgreSQL 5表Schema | 🌱 初次学习 | ✅ 完成 |
| 2026-03-01 | [p004_local-source-deep-dive_2026-03-01.md](./p004-swarm-ide/p004_local-source-deep-dive_2026-03-01.md) | 本地源码精读：specs/、runtime 控制流、前端 SSE、Upstash、.agents/skills — 6个悬案全破 | 🔍 深度研究 | ✅ 完成 |
| 2026-03-01 | [p004_frontend-architecture_2026-03-01.md](./p004-swarm-ide/p004_frontend-architecture_2026-03-01.md) | 前端完整实现（page.tsx/storage.ts/skill-loader.ts）+ 全项目框架总结 | 🔍 深度研究 | ✅ 完成 |

**待探索 / Still To Study:**

- [x] GitHub 仓库源码 + 本地源码精读 ✅
- [x] `specs/` 目录设计意图文档 ✅
- [x] 前端 SSE 消费层 ✅
- [ ] `.agents/` 内置 Agent 定义深入（role / skill 配置对比）
- [ ] 实践：本地部署 Swarm-IDE，体验液态拓扑和微信式交互
- [ ] 对比分析：Claude Code / OpenAI Swarm / Swarm-IDE 三种通信机制横向对比

---

### P005 · DeerFlow Frontend

| 字段 | 内容 |
|---|---|
| 项目 ID | P005 |
| 项目名称 | DeerFlow Frontend Study |
| 项目路径 | `https://github.com/bytedance/deer-flow` |
| 主要主题 | Next.js App Router + LangGraph SDK + React Query + Artifact Panel + XY Flow DAG |
| 首次学习 | 2026-03-01 |
| 最近更新 | 2026-03-01 |
| 状态 Status | 🟡 进行中 In Progress |

**学习会话 / Sessions:**

| 日期 | 文件 | 主题 | 进度阶段 | 状态 |
|---|---|---|---|---|
| 2026-03-01 | [p005_frontend-overview_2026-03-01.md](./p005-deerflow/p005_frontend-overview_2026-03-01.md) | 前端全貌：Next.js + LangGraph useStream + React Query + Subtask Card + Artifact Panel + 7大亮点 | 🌱 初次学习 | ✅ 完成 |

**待探索 / Still To Study:**

- [ ] `frontend/src/core/api/` 完整源码：streaming fetch 实现细节
- [ ] `frontend/src/components/workspace/artifacts/` 源码：Artifact 渲染组件树
- [ ] `frontend/src/core/streamdown/plugins/` 源码：每个 plugin 处理哪种数据类型
- [ ] XY Flow 集成：Agent DAG 可视化渲染位置
- [ ] `backend/` 目录：LangGraph lead_agent 架构（与前端数据契约）
- [ ] 实践：本地部署，用 DevTools 观察 LangGraph streaming 数据格式

---

### P006 · Cat Café Tutorials

| 字段 | 内容 |
|---|---|
| 项目 ID | P006 |
| 项目名称 | Cat Café Tutorials — AI 多 Agent 协作系统完整复盘 |
| 项目路径 | `https://github.com/zts212653/cat-cafe-tutorials/tree/main` |
| 主要主题 | 异构多 Agent 协作（Claude/Codex/Gemini）、CLI subprocess 架构、MCP callback、A2A 路由、数据安全三层防护、Session Chain |
| 首次学习 | 2026-03-01 |
| 最近更新 | 2026-03-01 |
| 状态 Status | 🟡 进行中 In Progress（代码库未开源） |

**学习会话 / Sessions:**

| 日期 | 文件 | 主题 | 进度阶段 | 状态 |
|---|---|---|---|---|
| 2026-03-01 | [p006_lesson00-08-overview_2026-03-01.md](./p006-cat-cafe/p006_lesson00-08-overview_2026-03-01.md) | Lesson 00-08 全量通读：CLI架构+MCP callback+A2A路由+数据防护+Session Chain+7大可借鉴模式 | 🌱 初次学习 | ✅ 完成 |
| 2026-03-01 | [p006_homework-prompts_2026-03-01.md](./p006-cat-cafe/p006_homework-prompts_2026-03-01.md) | 6 课 24 个提示词完整收录：含目录/命名/索引，原文保留，可直接复用 | 🌱 初次学习 | ✅ 完成 |

**存档文档 / Archived Docs:**

| 原始文档 | 存档文件 | 日期 |
|---|---|---|
| `docs/lessons/01-08-homework.md`（全 6 课） | [p006_homework-prompts_2026-03-01.md](./p006-cat-cafe/p006_homework-prompts_2026-03-01.md) | 2026-03-01 |

**待探索 / Still To Study:**

- [ ] 配套代码库开源后，阅读 AgentService / CliTransformer / A2A routing 源码
- [ ] `docs/decisions/` 后续 ADR（当前只有 ADR-001）
- [ ] 新增 Lesson（Sessions management context engineering 等）
- [ ] 实践：用类似架构设计自己的异构多 Agent 系统
- [ ] 对比：Cat Café CLI spawn 延迟 vs P004 Promise-deferred 实际差异
- [ ] 完成 P01-MAIN 作业，运行 `minimal-claude.js` 验证 CLI 调用
- [ ] 完成 P05-MAIN 作业，体验内心独白 vs 主动发言的区别

---

_(新项目在此继续添加 / Add new projects below)_

---

## 🗓️ 会话时间线 / Session Timeline

| 日期 | 项目 | 主题摘要 | 进度阶段 | 文件链接 |
|---|---|---|---|---|
| 2026-02-26 | _(成人学习方法论)_ | 四轮学习法、费曼反馈、复盘框架 | 🌱 初次学习 | [misc_adult-learning-methodology](./_misc/misc_adult-learning-methodology_2026-02-26.md) |
| 2026-02-27 | P001 skill and e-evo | skill-creator 架构、Eval Loop、安全扫描模版生成 | 🌱 初次学习 | [p001_skill-creator_2026-02-27](./p001-skill-evo/p001_skill-creator_2026-02-27.md) |
| 2026-02-28 | P002 OpenClaw Frontend | Lit Web Components UI、WebSocket Gateway协议、Canvas+A2UI、7大借鉴模式 | 🌱 初次学习 | [p002_frontend-overview_2026-02-28](./p002-openclaw/p002_frontend-overview_2026-02-28.md) |
| 2026-02-28 | P003 工程师经验博文 | Boris Tane：纪律性 Claude Code 工作流，Research→Plan→Annotate→Implement 六阶段流程 | 🌱 初次学习 | [p003_boris-workflow_2026-02-28](./p003-engineering-blog/p003_boris-workflow_2026-02-28.md) |
| 2026-02-28 | P003 工程师经验博文 | Boris Tane：SDLC 已死，上下文工程是新核心技能，可观测性成整个循环的结缔组织 | 🌱 初次学习 | [p003_sdlc-is-dead_2026-02-28](./p003-engineering-blog/p003_sdlc-is-dead_2026-02-28.md) |
| 2026-03-01 | P003 工程师经验博文 | @alterxyz4：Claude Code Agent Teams 逆向分析，文件系统消息队列，5大可借鉴设计模式 | 🌱 初次学习 | [p003_agent-teams-reverse_2026-03-01](./p003-engineering-blog/p003_agent-teams-reverse_2026-03-01.md) |
| 2026-03-01 | **P004 Swarm-IDE** | 源码级深潜——Promise-deferred事件驱动、IM系统prompt、双总线(AgentEventBus+WorkspaceUIBus)、PostgreSQL 5表Schema | 🌱 初次学习 | [p004_core-architecture_2026-03-01](./p004-swarm-ide/p004_core-architecture_2026-03-01.md) |
| 2026-03-01 | **P004 Swarm-IDE** | 本地源码精读：specs/、runtime控制流、前端SSE、Upstash、.agents/skills — 6个悬案全破 | 🔍 深度研究 | [p004_local-source-deep-dive_2026-03-01](./p004-swarm-ide/p004_local-source-deep-dive_2026-03-01.md) |
| 2026-03-01 | **P004 Swarm-IDE** | 前端完整实现（page.tsx/storage.ts/skill-loader.ts）+ 全项目框架总结 | 🔍 深度研究 | [p004_frontend-architecture_2026-03-01](./p004-swarm-ide/p004_frontend-architecture_2026-03-01.md) |
| 2026-03-01 | **P005 DeerFlow** | 前端全貌：Next.js App Router + LangGraph SDK useStream + React Query 乐观更新 + Subtask Card + Artifact Panel + XY Flow DAG + 7大亮点 | 🌱 初次学习 | [p005_frontend-overview_2026-03-01](./p005-deerflow/p005_frontend-overview_2026-03-01.md) |
| 2026-03-01 | **P006 Cat Café Tutorials** | Lesson 00-08 全量：CLI subprocess架构、MCP callback双路径、A2A路由统一(F27)、28秒事故三层防护、Session Chain、Context Cleaner、7大可借鉴模式 | 🌱 初次学习 | [p006_lesson00-08-overview_2026-03-01](./p006-cat-cafe/p006_lesson00-08-overview_2026-03-01.md) |

---

## 🗂️ 可复用模版库 / Reusable Template Library

| 模版文件 Template | 适用场景 When to Use | 版本 |
|---|---|---|
| [template_learning-session.md](./_templates/template_learning-session.md) | 每次新学习会话开始时 | v1.0 |
| [skill-security-scan-template_2026-02-27.md](./_templates/skill-security-scan-template_2026-02-27.md) | 审查任何新 Skill 的安全性 | v1.0 |

---

## 📊 知识图谱（文字版）/ Knowledge Map

```
成人自主学习方法论（2026-02-26）
    └─ 四轮学习法 / 费曼法 / 复盘框架
           │
           ▼
Claude Skill 系统 · P001（2026-02-27）
    ├─ Skill 架构：SKILL.md + scripts/ + references/ + assets/
    ├─ 三层加载：Metadata → SKILL.md → Resources
    ├─ Eval Loop：design → run → grade → benchmark → review → improve
    ├─ 安全原则：无惊喜 / 无注入 / 无恶意代码
    └─ 描述优化：run_loop.py / train-test split / 选 test score
           │
           ▼ (待学 / To Study)
    ├─ grader / comparator / analyzer subagents
    └─ 实践创建一个新 Skill
           │
           ▼
OpenClaw Frontend · P002（2026-02-28）
    ├─ Control UI：Lit Web Components + Shadow DOM + URL routing
    ├─ 协议层：WebSocket JSON frames（req/res/event/invoke）+ TypeBox Schema
    ├─ 认证：Challenge-nonce → DeviceToken 持久化
    ├─ Canvas/A2UI：声明式 JSON UI + 白名单组件注册 + a2ui-* HTML属性
    └─ 7大借鉴模式：单端口部署/Lit选型/WS状态机/TypeBox共享/A2UI白名单/进程隔离/多主题
           │
           ▼ (待学 / To Study)
    ├─ ui/src/ui/app.ts Lit 组件树 + vendor/a2ui/ 源码
    └─ 实践：最小 WebSocket client 接入 Gateway
           │
           ▼
工程师经验博文 · P003（2026-02-28 ~ 2026-03-01）
    ├─ [Boris Tane] 纪律性 Claude Code 工作流
    │   ├─ Research Phase：深度阅读→research.md→人工审阅
    │   ├─ Planning Phase：plan.md（含代码片段、文件路径、权衡）
    │   ├─ Annotation Cycle：内联批注→Claude 更新→循环 1-6 次
    │   ├─ Todo List：分阶段任务清单，实现时标记完成
    │   ├─ Implementation：标准提示词，一键持续执行，类型检查
    │   └─ 核心原则：实现应是无聊的；错了 revert 不要 patch
    ├─ [Boris Tane] SDLC is Dead → 上下文工程 + 可观测性闭环
    └─ [@alterxyz4] Claude Code Agent Teams 逆向分析
        ├─ 三原语：文件系统消息队列 + AsyncLocalStorage + 共享任务列表
        └─ 5大可借鉴模式：文件系统队列/按需创建/协议复用数据通道/透明性即可观测性/先跑起来再优化
           │
           ▼ (与 P004 Swarm-IDE 形成对照)
           │
Swarm-IDE · P004（2026-03-01，源码级 × 3 sessions）
    ├─ 核心隐喻：Agent = IM系统中的成员
    ├─ AgentRunner（per-agent）+ AgentRuntime（全局单例）
    ├─ 唤醒机制：createDeferred<void>()，Promise resolve → 立即唤醒
    ├─ 双总线：AgentEventBus（内部流）+ WorkspaceUIBus（UI推送）
    ├─ DB Schema（Drizzle ORM + PostgreSQL）：5张核心表
    └─ 与 Claude Code（P003/@alterxyz4）对比：
        Promise-deferred vs 文件轮询 / PostgreSQL vs JSON文件
           │
           ▼ (对比参考 / Compare)
           │
DeerFlow Frontend · P005（2026-03-01）
    ├─ 框架：Next.js 16 App Router + React 19 + TypeScript 5.8
    ├─ 状态双轨：LangGraph SDK（Agent流）+ TanStack Query（UI服务端状态）
    ├─ useStream<AgentThreadState> hook：订阅 Agent 执行状态，零自写 SSE 逻辑
    ├─ Artifact Panel：结构化产出与消息流分离展示 + CodeMirror 编辑
    └─ 与 P004 对比：
        useStream vs SSE双总线 / React Query vs PostgreSQL /
        Artifact Panel vs 消息流合一 / XY Flow DAG vs IM消息列表
           │
           ▼ (横向对比 / Cross-Compare)
           │
Cat Café Tutorials · P006（2026-03-01，全量 Lesson 00-08）
    ├─ 核心目标：消除"活路由"——让三个异构 AI 自主协作（Claude/Codex/Gemini）
    ├─ 技术选型：CLI subprocess（非 SDK）
    ├─ MCP Callback 双路径：Claude原生 vs HTTP注入
    ├─ A2A 路由（F27 统一后）：worklist + callback → 统一路径
    ├─ 数据安全三层：物理隔离 + 结构检查 + 属性测试
    ├─ Session Chain（5层）：检测→封存→归档→MCP查询→重生
    └─ 与 P003/P004 横向对比：
        异构多模型 vs 同质多实例 /
        CLI spawn(500ms) vs Promise.resolve(即时) vs 文件轮询(次轮)
```

---

*更新说明：每次学习会话结束后，在「项目学习记录」和「会话时间线」中追加记录。*
*Update note: After each session, append to "Project Learning Records" and "Session Timeline".*
