---
project: P003 · Engineering Blog Learning
file_type: learning-summary
tags: [claude-code, agent-teams, file-queue, reverse-engineering, multi-agent]
date: 2026-03-01
session_type: 初次学习
related: p003_sdlc-is-dead_2026-02-28.md
---

# 学习总结 / Learning Summary

> **文件命名 / Filename:** `learning-summary_2026-03-01.md`

> ⚠️ **阅读声明 / Reading Disclaimer**
> 本文内容代表作者（@alterxyz4）个人观点与逆向工程发现，不作为官方实现文档。
> Claude Code 仍在快速迭代，内部实现可能已变更。日后借鉴时以官方文档和 Terry 的实际验证为准。
> *This content represents the author's personal reverse-engineering findings. Claude Code is actively evolving; internal implementation may have changed. Refer to official docs and Terry's own verification when applying.*

---

## 会话元数据 / Session Metadata

| 字段 Field | 内容 Content |
|---|---|
| 日期 Date | 2026-03-01 |
| 时长（估算）Duration | ～ 0.5 小时 |
| 项目名称 Project | P003 · 工程师经验博文学习 Engineering Blog Learning |
| 主题 Topic | Claude Code Agent Teams 逆向分析——文件系统消息队列 / Reverse-Engineering Claude Code Agent Teams: Filesystem as Message Queue |
| 触发原因 Context | P003 系列继续——从宏观（SDLC 坍塌）、微观（个人工作流）延伸到 **系统内部架构**，理解多 Agent 协作的底层通信机制 |
| 会话类型 Session Type | ☑ 初次阅读 First Read |
| 关联上次会话 Prev Session | `learning-summary_2026-02-28_3.md`（Boris Tane SDLC is Dead） |

---

## 一、来源概览 / Source Overview

**本次阅读来源 / Source:**

| 文章 Article | 链接 Link | 作者 Author | 发布平台 |
|---|---|---|---|
| Claude Code 逆向了它自己，发现 agent 之间靠读写 JSON 文件聊天 | https://x.com/alterxyz4/status/2021892207574405386 | @alterxyz4 | X (Twitter) |

**文章核心方法论 / Methodology:**
作者让 Claude Code 对自己的 183MB Bun 编译二进制运行 `strings` 命令提取可读字符串（逆向），然后又让 Claude Code 实际 spawn 一个 teammate 并观察文件系统变化（实验验证）。一个 AI 先逆向了自己的源码，然后亲手验证了逆向结论。

The author asked Claude Code to run `strings` on its own 183MB binary to extract readable strings (reverse engineering), then had it spawn a real teammate and observe filesystem changes (experimental verification). An AI reverse-engineered its own source code, then verified the conclusion hands-on.

---

## 二、本次学习内容 / What I Studied Today

### 2.1 核心结论 / Core Finding

Claude Code 的 Agent Teams 系统建立在三个朴素原语上：

| 原语 Primitive | 实现 Implementation | 作用 Purpose |
|---|---|---|
| 文件系统消息队列 | 每个 agent 一个 inbox JSON 文件 | Agent 间通信 / Inter-agent messaging |
| AsyncLocalStorage | Node.js 原生异步上下文隔离 | 多 agent 共享进程时的上下文隔离 / Context isolation |
| 共享任务列表 | 每个任务一个 JSON 文件 | 任务协调与依赖管理 / Task coordination |

**没有** 消息中间件，**没有** 数据库，**没有** 网络通信。所有东西都是文件。

No message broker, no database, no network communication. Everything is files.

---

### 2.2 文件系统结构 / Filesystem Structure

```
~/.claude/
├── teams/<team-name>/
│   ├── config.json          ← 团队配置（成员列表、模型、backendType）
│   └── inboxes/
│       ├── team-lead.json   ← lead 的收件箱（JSON 数组，每条消息追加）
│       └── observer.json    ← teammate 的收件箱（按需创建，非预分配）
└── tasks/<team-name>/
    ├── 1.json               ← 任务文件（id, subject, status, blockedBy）
    └── 2.json               ← 编号递增
```

**关键设计选择 / Key Design Decisions:**

1. **inbox 按需创建** — 只有收到第一条消息时文件才出现，不预分配
   Inboxes are created on demand — file appears only when first message arrives
2. **团队名 sanitize** — 非字母数字字符全变连字符
   Team names are sanitized: non-alphanumeric chars become hyphens
3. **任务依赖仅靠 `blockedBy` 数组** — 极简依赖管理
   Task dependencies managed solely by `blockedBy` array

---

### 2.3 通信协议 / Communication Protocol

**普通消息格式 / Regular Message:**
```json
{
  "from": "observer",
  "text": "你好 lead，我是 observer，我已经启动了！",
  "summary": "Observer reporting in",
  "timestamp": "2026-02-12T09:21:46.491Z",
  "color": "blue",
  "read": true
}
```

**协议消息 / Protocol Message（JSON-in-JSON）:**
```json
{
  "from": "observer",
  "text": "{\"type\":\"idle_notification\",\"from\":\"observer\",\"idleReason\":\"available\"}",
  "timestamp": "...",
  "read": true
}
```

协议消息（空闲通知、关闭请求等）将 JSON 序列化为字符串塞进 `text` 字段。接收方先 parse `text`，检测 `type` 字段，再分发处理。

Protocol messages serialize JSON into the `text` field. Receiver parses `text`, checks `type`, then dispatches accordingly.

**完整生命周期消息序列 / Full Lifecycle Sequence:**
```
Lead inbox:  普通DM → 普通DM → idle_notification → idle_notification → shutdown_approved
Observer inbox:  普通DM(from lead) → shutdown_request
```

---

### 2.4 消息投递机制 / Message Delivery Mechanism

从逆向发现的函数名 `injectUserMessageToTeammate` 揭示了核心机制：

**Teammate 消息被注入为 user message。** 对接收方 agent 来说，来自 teammate 的消息和来自人类用户的消息在对话历史里地位相同。

Teammate messages are injected as user messages — from the receiver's perspective, a teammate message has the same status as a human message in conversation history.

**投递时机：只在 conversation turn 之间。**
- 一个 Claude API 调用 = 一个 turn
- Agent 在执行一个长 turn 时（如写大量代码），期间收到的消息**不会被实时处理**
- 必须等当前 turn 完成后，系统才检查 inbox

Delivery timing: only between conversation turns. Messages received during a long turn are NOT processed in real-time.

---

### 2.5 两种运行模式 / Two Backend Types

| 模式 Mode | 特性 Characteristics | 终止方式 | 适用场景 |
|---|---|---|---|
| `in-process` | 主进程内，AsyncLocalStorage 隔离 | `AbortController.abort()` | 默认模式，性能更好 |
| `tmux` | 独立 tmux pane，完全独立进程 | `process.exit()` | 更隔离，但有轮询启动 bug |

两者共用相同的 inbox 文件通信机制。

Both share the same inbox file communication mechanism.

---

### 2.6 已知问题 / Known Issues（截至文章发布时，全部 OPEN）

| GitHub Issue | 问题 | 根因分析 |
|---|---|---|
| #23620 | Context compaction 杀死团队感知 | 压缩后 lead 忘记团队存在 |
| #25131 | Agent 生命周期管理混乱 | 重复 spawn、mailbox polling 浪费 |
| #24130 | Auto memory 不支持并发 | 多 teammate 同时写 MEMORY.md 互相覆盖 |
| #24977 | 任务完成通知淹没上下文 | TaskUpdate 在 lead context 留痕，加速 compaction |
| #23629 | 任务状态不同步 | 团队层面 vs agent 会话内状态不一致 |
| #24108 | tmux 模式下新 teammate 卡死 | 启动后停在欢迎界面，无首次 turn，不轮询 inbox |

社区开发了 **Cozempic** 工具缓解 #23620——在压缩后从 config.json 读取团队状态并重新注入。但官方还没有 PostCompact hook。

---

## 三、洞察与收获 / Insights & Takeaways

### 3.1 最重要的 3 个发现 / Top 3 Insights

1. **文件系统即消息队列，是一个极其合理的架构选择。** CLI 工具不能要求用户装 Redis 或 RabbitMQ。文件系统每个 OS 都有，不需要安装/配置/端口/权限。`mkdir` + `writeFile` 就是全部基础设施。这不是"凑合"，是在约束条件下的最优解。
   *Filesystem as message queue is a remarkably reasonable architectural choice.* A CLI tool can't require users to install Redis. The filesystem exists on every OS with zero setup cost. This isn't a hack — it's optimal under the constraints.

2. **"JSON 套 JSON" 协议设计是务实的分层。** 普通消息和协议消息共用同一个信道（inbox JSON 数组），通过 text 字段是否可再次 JSON.parse 来区分类型。这避免了引入额外的消息类型系统，虽然不优雅但足够用。
   *JSON-in-JSON protocol is pragmatic layering.* Regular and protocol messages share the same channel, differentiated by whether the `text` field is further JSON-parseable. Not elegant, but sufficient.

3. **可观测性是免费的副产品。** `cat inbox.json` 就能看到所有通信历史，`ls teams/` 就知道当前状态。不需要监控面板或 log aggregator——文件系统本身就是调试工具。这是"透明性即可观测性"的极致案例。
   *Observability is a free byproduct.* `cat inbox.json` shows all message history. The filesystem IS your debugging tool. This is the ultimate case of "transparency as observability."*

### 3.2 让我意外的地方 / What Surprised Me

- **消息注入为 user message** — 我原以为 teammate 消息会有某种特殊的 system/assistant 角色标识，但实际上直接注入为 user message，这意味着 LLM 无法在原始层面区分"人类说的"和"teammate 说的"，区别只靠文本格式包裹。
  I expected teammate messages to have a special role tag, but they're injected as user messages — the LLM can't natively distinguish human vs teammate, only by text wrapping.

- **完全没有实时推送** — 只在 turn 间轮询，不是 watch/notify 机制。这与现代消息系统的直觉完全相反，但在 agent 场景下（消息量小、延迟不敏感）完全够用。
  No real-time push at all — polling between turns only, counter-intuitive for messaging systems but sufficient for agent scenarios.

- **逆向方法本身** — 让 AI 对自己的二进制跑 `strings`，再让它实际创建团队验证，这个"自我逆向+自我验证"的方法论本身就是一个有趣的研究范式。
  The methodology itself — having an AI `strings` its own binary then self-verify — is an interesting research paradigm.

### 3.3 与已有知识的连接 / Connections to Prior Knowledge

**与 P003 前两篇的关联：**

| 前文 | 本文连接点 |
|---|---|
| Boris Tane "纪律性工作流" | 那篇讲**人如何与单个 Agent 协作**；本文揭示了 **Agent 之间如何协作的底层机制**，是更底层的视角 |
| Boris Tane "SDLC is Dead" | 那篇讲可观测性是"结缔组织"；本文的 inbox 文件天然提供了 agent 间通信的完整可观测性，是对该观点的实例印证 |

**与更广泛的系统设计知识的连接：**

- **Unix 哲学的回响** — "一切皆文件"在 50 年后依然是有效的通信抽象。/proc 文件系统、Plan 9、D-Bus 的早期版本都用过类似思路。
  The Unix philosophy of "everything is a file" remains a valid communication abstraction 50 years later.

- **消息队列的本质抽象** — 本文清晰地映射了文件系统操作到队列语义：JSON 数组追加 = enqueue，readUnreadMessages = dequeue，`"read": true` = ack。这种映射帮助理解任何消息系统的核心抽象。
  The article clearly maps filesystem ops to queue semantics, helping understand the core abstraction of any messaging system.

---

## 四、安全与风险观察 / Security & Risk Notes

| 观察 Observation | 风险等级 Risk Level | 备注 |
|---|---|---|
| inbox JSON 文件明文存储在 `~/.claude/` | 低 | 本地文件，权限受 OS 用户权限控制 |
| 没有严格互斥锁，.lock 文件是"君子协定" | 中 | 高并发场景下（多 teammate 同时写同一文件）可能数据损坏 |
| 协议消息与普通消息共用信道，仅靠 JSON parse 区分 | 低 | 理论上可被构造的恶意文本混淆，但在受控环境下风险极低 |
| Context compaction 后团队感知丢失 | 中 | 可能导致 agent 行为不可预测，需依赖外部工具（Cozempic）缓解 |

**是否需要进行完整安全扫描？** ⬜ 否 No — 这是对外部系统的学习分析，非自建 Skill。

---

## 五、可借鉴的设计模式 / Borrowable Design Patterns

> 💡 **本节是本文的核心产出** — 从 Claude Code Agent Teams 的实现中提炼出可复用的设计模式。

### 模式 1：文件系统消息队列 / Filesystem Message Queue

**适用场景：** CLI 工具、本地多进程协作、零依赖要求的场景
**核心思想：** 用文件系统实现消息队列语义

| 队列操作 | 文件系统映射 |
|---|---|
| 创建 channel | 创建 `<agent-name>.json` 文件 |
| enqueue | 追加 JSON 数组元素 |
| dequeue | 读取 `read: false` 的消息 |
| ack | 将 `read` 设为 `true` |
| 消息持久化 | 文件本身就是持久化 |

**约束条件：** 消息量小（几十条）、延迟不敏感（秒级）、并发有限（2-4个进程）
**代价：** 无原子性、无实时推送、无 backpressure

### 模式 2：按需资源创建 / On-Demand Resource Creation

**思想：** inbox 文件在首次收到消息时才创建，而非 spawn 时预分配。
**好处：** 减少不必要的文件创建，让文件存在本身成为状态信号（"有这个文件 = 有人给这个 agent 发过消息"）。
**推广：** 任何资源（线程、连接、缓存条目）都可以考虑按需创建而非预分配。

### 模式 3：协议消息复用数据通道 / Protocol Messages Over Data Channel

**思想：** 不为协议消息（idle、shutdown 等）建立独立信道，而是塞进同一个 inbox 数组，靠 text 字段的可解析性区分。
**好处：** 零额外基础设施，协议消息自动获得与数据消息相同的持久化和可观测性。
**代价：** 接收方需要做类型检测（try parse → 检查 type 字段）。

### 模式 4：透明性即可观测性 / Transparency as Observability

**思想：** 当你的所有状态都是可直接 `cat`/`ls` 的文件时，你不需要专门的监控系统。
**条件：** 状态必须是人类可读的格式（JSON/YAML），而非二进制 blob。
**限制：** 只适用于规模小、状态简单的系统。大规模系统仍需结构化监控。

### 模式 5：先跑起来再优化 / Start Simple, Optimize Later

**思想：** 先用最简单的方式（文件）实现功能，让真实用户踩真实的坑，再决定哪里需要变复杂。比起一上来就搞分布式消息系统，这种路径风险小得多。
**作者原话引用：** "文件系统这个东西，40 年了，还没挂过。"
**适用：** 任何早期产品/原型阶段，尤其是不确定规模需求时。

---

## 六、问题与待解答 / Open Questions

- [ ] Claude Code 后续版本是否已修复 #23620（context compaction 杀死团队感知）？
- [ ] `injectUserMessageToTeammate` 的具体文本包裹格式是什么？（逆向未找到，可能是运行时拼接）
- [ ] 实际使用 Agent Teams 时，最佳的 team 规模和任务拆分粒度是什么？
- [ ] Cozempic 工具的具体实现方式和效果如何？

---

## 七、下一步计划 / Next Steps

| 优先级 Priority | 行动 Action | 预计时间 Est. Time | 状态 Status |
|---|---|---|---|
| 中 Medium | 实践验证：在 Claude Code 中创建团队，观察 `~/.claude/teams/` 目录结构 | ～0.5h | ⬜ 待做 |
| 低 Low | 跟踪 GitHub issues #23620 #25131 的进展 | 持续 | ⬜ 待做 |
| 低 Low | 探索 Cozempic 工具源码 | ～0.5h | ⬜ 待做 |

---

## 八、本次生成的文件 / Files Generated This Session

| 文件名 Filename | 类型 Type | 描述 Description |
|---|---|---|
| `learning-summary_2026-03-01.md` | 总结 Summary | Claude Code Agent Teams 逆向分析学习总结 |

---

## 九、关键词 / Keywords

`Claude Code` `Agent Teams` `文件系统消息队列` `JSON inbox` `AsyncLocalStorage` `多Agent协作` `逆向工程` `进程间通信` `可观测性` `Unix哲学`

---

*模版版本 Template Version: v1.0 | 基于 template_learning-session.md*
