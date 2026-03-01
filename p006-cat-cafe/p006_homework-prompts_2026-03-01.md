---
project: P006 · Cat Café Tutorials
file_type: homework-prompts
tags: [multi-agent, cli, mcp, session-chain, rich-blocks, prompt-collection]
date: 2026-03-01
session_type: 初次学习
source: https://github.com/zts212653/cat-cafe-tutorials
related: p006_lesson00-08-overview_2026-03-01.md
---

# Cat Café Tutorials — Homework Prompt 索引
# Cat Café 课后作业提示词完整收录

> **来源 / Source:** https://github.com/zts212653/cat-cafe-tutorials/tree/main
> **收录范围:** Lesson 01、02、05、06、07、08 课后作业（共 6 课，24 个提示词）
> **注意:** Lesson 03（元规则）和 Lesson 04（A2A路由）无专属 homework 文件
> **原文语言:** 中文（完整保留原始提示词文字，不翻译不改写）

---

## 📋 目录 / Table of Contents

| 编号       | 提示词名称                  | 所属课程      | 类型        | 页内锚点           |
| -------- | ---------------------- | --------- | --------- | -------------- |
| P01-MAIN | 最小 CLI 调用 Claude Agent | Lesson 01 | 核心作业      | [→](#p01-main) |
| P01-C1   | 支持 Codex CLI 调用        | Lesson 01 | 进阶挑战      | [→](#p01-c1)   |
| P01-C2   | 统一 CLI 调用接口            | Lesson 01 | 进阶挑战      | [→](#p01-c2)   |
| P01-C3   | CLI Session 恢复         | Lesson 01 | 进阶挑战（描述性） | [→](#p01-c3)   |
| P02-MAIN | CLI 工程化自检              | Lesson 02 | 核心作业      | [→](#p02-main) |
| P02-C1   | 心跳机制替代固定超时             | Lesson 02 | 进阶挑战（描述性） | [→](#p02-c1)   |
| P02-C2   | 优雅两阶段关机                | Lesson 02 | 进阶挑战      | [→](#p02-c2)   |
| P02-C3   | AI 幻觉检测设计              | Lesson 02 | 进阶挑战（描述性） | [→](#p02-c3)   |
| P05-MAIN | 最小 MCP 回传系统（给猫装上嘴巴）    | Lesson 05 | 核心作业      | [→](#p05-main) |
| P05-C1   | Prompt 注入版 Callback    | Lesson 05 | 进阶挑战（描述性） | [→](#p05-c1)   |
| P05-C2   | MCP vs curl 对比实验       | Lesson 05 | 进阶挑战（描述性） | [→](#p05-c2)   |
| P05-C3   | 隐私模式（猫猫杀）              | Lesson 05 | 进阶挑战（描述性） | [→](#p05-c3)   |
| P06-MAIN | Redis 数据丢失演练环境         | Lesson 06 | 核心作业      | [→](#p06-main) |
| P06-C1   | 故意选错备份演练               | Lesson 06 | 进阶挑战      | [→](#p06-c1)   |
| P06-C2   | 目录大小防腐门                | Lesson 06 | 进阶挑战      | [→](#p06-c2)   |
| P06-C3   | 证据闸门脚本                 | Lesson 06 | 进阶挑战      | [→](#p06-c3)   |
| P07-MAIN | 最小 Rich Blocks 管线      | Lesson 07 | 核心作业      | [→](#p07-main) |
| P07-C1   | Game Block（角色卡）        | Lesson 07 | 进阶挑战      | [→](#p07-c1)   |
| P07-C2   | 双路由 Rich Blocks        | Lesson 07 | 进阶挑战      | [→](#p07-c2)   |
| P07-C3   | Rich Block 格式容错        | Lesson 07 | 进阶挑战      | [→](#p07-c3)   |
| P08-MAIN | 最小 Session Chain 模拟器   | Lesson 08 | 核心作业      | [→](#p08-main) |
| P08-C1   | 压缩 vs 交接信息保真度对比        | Lesson 08 | 进阶挑战      | [→](#p08-c1)   |
| P08-C2   | 跨 Thread 污染复现          | Lesson 08 | 进阶挑战      | [→](#p08-c2)   |
| P08-C3   | Context 健康度监控          | Lesson 08 | 进阶挑战      | [→](#p08-c3)   |

---

## 图例 / Legend

- **核心作业**：每课的主提示词，完整可执行，包含所有技术细节
- **进阶挑战**：扩展性挑战，部分是完整提示词，部分是描述性指引
- **描述性**：原作业提供场景描述而非完整提示词，需自行扩展

---

## Lesson 01 · 用 CLI 调用 Claude Agent

> **对应课程:** 第一课 — 选型之路：从 SDK 到 CLI
> **核心收获:** 用 `spawn()` 调用 CLI，解析 NDJSON，理解 CLI 调用的基本原理

---

### P01-MAIN · 最小 CLI 调用 Claude Agent {#p01-main}

**目标:** 写一个 Node.js 脚本，能用 `spawn()` 调用 `claude` CLI，解析 NDJSON 流式输出，打印 Claude 的回复。

**前置条件:**
- Node.js (v18+)
- Claude Code CLI（`claude --version` 能跑）
- 已登录 Claude CLI

```
我想写一个 Node.js 脚本，用 child_process.spawn() 调用 Claude CLI，并解析它的流式输出。

## 背景知识

Claude CLI 支持以下调用方式：
- `claude -p "你的问题"` — 非交互模式
- `--output-format stream-json` — 输出 NDJSON（每行一个 JSON）
- `--verbose` — 必须和 stream-json 一起用

输出格式示例：
{"type":"system","subtype":"init","session_id":"abc123"}
{"type":"assistant","message":{"content":[{"type":"text","text":"Hello!"}]}}
{"type":"result","subtype":"success","session_id":"abc123"}

## 要求

1. 创建一个 `minimal-claude.js` 文件
2. 使用 Node.js 原生的 `child_process.spawn()`
3. 使用 `readline` 模块逐行解析 stdout
4. 解析 JSON，提取 `assistant` 类型事件中的文本内容
5. 打印出 Claude 的回复
6. 处理进程退出

## 运行方式

node minimal-claude.js "你好，请用一句话介绍自己"

## 不需要

- 不需要 TypeScript
- 不需要任何 npm 依赖（纯原生 Node.js）
- 不需要错误重试、超时处理（保持简单）

请帮我写这个脚本，并解释关键部分的代码。
```

**验收标准:** `node minimal-claude.js "用一句话介绍自己"` 看到 Claude 的回复被打印出来。

---

### P01-C1 · 支持 Codex CLI 调用 {#p01-c1}

**目标:** 修改脚本，让它也能调用 `codex exec`。

> **注：** 本挑战为描述性指引，核心提示为：Codex 的 NDJSON 格式不同，事件类型是 `thread.started`、`item.completed` 等。

**运行目标:**
```bash
node minimal-codex.js "你好"
```

---

### P01-C2 · 统一 CLI 调用接口 {#p01-c2}

**目标:** 创建一个 `invoke(cli, prompt)` 函数，能同时支持 Claude 和 Codex。

> **注：** 本挑战为描述性指引。

**运行目标:**
```javascript
await invoke('claude', '你好');
await invoke('codex', '你好');
```

---

### P01-C3 · CLI Session 恢复 {#p01-c3}

**目标:** 让脚本能记住 session，下次调用时用 `--resume` 继续对话。

> **注：** 本挑战为描述性指引，无完整提示词。

---

## Lesson 02 · CLI 工程化自检

> **对应课程:** 第二课 — 从玩具到生产：一场辩论赛引发的连环惨案
> **核心收获:** 检查 CLI 调用代码的 6 大生产就绪问题；了解 stderr 监听、超时、环境隔离等关键点

---

### P02-MAIN · CLI 工程化自检 {#p02-main}

**目标:** 用 AI 帮你检查现有的 CLI 调用代码，找出潜在的生产就绪问题。

**适用场景:** 你已有一个能跑的 CLI 调用实现，想知道它离"生产可用"还差多远。

```markdown
# CLI 调用代码自检

请帮我检查我的 CLI 子进程调用代码，看看有没有以下问题：

## 检查清单

### 1. stderr 活跃信号
- [ ] 超时检测是否同时监听了 stdout 和 stderr？
- [ ] CLI 在 thinking/工具调用时输出到 stderr，只监听 stdout 会误判超时

**问题代码示例**：
```javascript
// 只监听 stdout — 有 bug！
child.stdout.on('data', () => { lastActivity = Date.now(); });
```

**正确代码示例**：
```javascript
// 同时监听 stdout 和 stderr
child.stdout.on('data', () => { lastActivity = Date.now(); });
child.stderr.on('data', () => { lastActivity = Date.now(); });
```

### 2. 超时设置
- [ ] 超时时间是否合理？（复杂任务可能需要 10-30 分钟）
- [ ] 是否有任务级别的超时配置？
- [ ] 超时后的处理是否优雅？（先 SIGTERM，等几秒再 SIGKILL）

### 3. 进程生命周期
- [ ] 是否处理了 SIGTERM/SIGINT 信号？
- [ ] 父进程退出时，子进程是否会被正确清理？
- [ ] 是否有僵尸进程防护？

### 4. 流式解析
- [ ] NDJSON 解析是否处理了不完整行？
- [ ] 是否处理了粘包情况（多个 JSON 在一个 chunk 里）？
- [ ] 解析错误时是否有容错处理？

### 5. 环境隔离
- [ ] 开发环境和生产环境的数据库是否隔离？
- [ ] 是否有防止开发代码连接生产数据库的机制？
- [ ] 环境变量配置是否清晰？

### 6. 错误处理
- [ ] CLI 进程异常退出时如何处理？
- [ ] 是否有重试机制？
- [ ] 错误信息是否足够用于调试？

## 我的代码

[在这里粘贴你的 CLI 调用代码]

## 请帮我：

1. 逐项检查上述清单
2. 指出我的代码中存在的问题
3. 给出具体的修复建议
4. 如果某项检查通过，也请确认

输出格式：
- ✅ 通过的检查项
- ❌ 存在问题的检查项 + 问题描述 + 修复建议
- ⚠️ 无法确认的检查项（需要更多上下文）
```

**验收标准（6 项全✅）:**

| 检查项 | 期望状态 |
|---|---|
| stderr 和 stdout 都被监听 | ✅ |
| 超时时间可配置且合理 | ✅ |
| 进程生命周期管理完善 | ✅ |
| NDJSON 解析有容错 | ✅ |
| 开发/生产环境隔离 | ✅ |
| 错误处理完善 | ✅ |

---

### P02-C2 · 优雅两阶段关机 {#p02-c2}

**目标:** 实现 SIGTERM → 等待 5 秒 → SIGKILL 的两阶段关机。

> **注：** 原文提供了代码模板，可直接描述给 AI 实现：

```javascript
// 第一阶段：发 SIGTERM，让进程有机会清理
child.kill('SIGTERM');

// 等待 5 秒
setTimeout(() => {
  // 第二阶段：如果还没退出，强制 SIGKILL
  if (!child.killed) {
    child.kill('SIGKILL');
  }
}, 5000);
```

---

### P02-C1 · 心跳机制替代固定超时 {#p02-c1}

**目标:** 不用固定超时，改用心跳机制——CLI 定期输出心跳信号，超过 N 秒没收到才判超时。

> **注：** 本挑战为描述性指引。

---

### P02-C3 · AI 幻觉检测设计 {#p02-c3}

**目标:** 设计"暗号测试"验证 AI 是否真的能获取信息，而不只是"有没有回答"。

> **注：** 本挑战为描述性指引，关键点：
> - 设计暗号测试验证 AI 是否真的能获取信息
> - 检查 AI 回答的正确性
> - 在 system prompt 中强调"不确定时说不知道"

---

## Lesson 05 · 给猫装上嘴巴（MCP 回传）

> **对应课程:** 第五课 — MCP Callback 让 Agent 主动发言
> **核心收获:** 搭建 MCP 回传系统；理解 CLI 内心独白 vs 主动发言的本质区别

---

### P05-MAIN · 最小 MCP 回传系统 {#p05-main}

**目标:** 搭建一个最小的 MCP 回传系统：猫能在执行任务的过程中，主动往聊天室发消息。

**前置:** 完成 Lesson 01 作业，安装 `@modelcontextprotocol/sdk`

```
我想搭建一个最小的 MCP 回传系统，理解"猫主动说话"的机制。

## 背景

我在学习 AI Agent 协作系统。核心概念是：
- AI 通过 CLI 子进程执行任务
- CLI 内部的输出是 AI 的"内心独白"，默认不可见
- AI 可以通过 MCP 工具主动调用 HTTP callback，把消息发到聊天室
- 这样 AI 就有了"选择说什么"的自主权

## 要求

帮我创建三个文件：

### 1. callback-server.js — HTTP 回调服务器

一个简单的 Node.js HTTP 服务器（不用框架，用原生 http 模块），监听 3200 端口：

- POST /api/callbacks/post-message
  - 接收 JSON body: { invocationId, callbackToken, content }
  - 验证 invocationId 和 callbackToken 是否匹配预设值
  - 如果验证通过，把 content 打印到终端（模拟"消息出现在聊天室"）
  - 返回 { status: "ok" }
  - 如果验证失败，返回 401

- GET /api/callbacks/thread-context
  - 接收 query params: invocationId, callbackToken
  - 验证通过后，返回一段模拟的对话历史 JSON
  - 比如: { messages: [{ role: "user", content: "请写一首关于猫的诗" }] }

启动时生成一对 UUID 作为 invocationId 和 callbackToken，打印出来。

### 2. cat-cafe-mcp.js — 最小 MCP Server

一个 MCP Server（使用 @modelcontextprotocol/sdk），提供两个工具：

- cat_cafe_post_message(content: string)
  - 向 callback-server 的 POST /api/callbacks/post-message 发 HTTP 请求
  - 从环境变量读取 CAT_CAFE_API_URL, CAT_CAFE_INVOCATION_ID, CAT_CAFE_CALLBACK_TOKEN

- cat_cafe_get_context()
  - 向 callback-server 的 GET /api/callbacks/thread-context 发 HTTP 请求
  - 返回对话上下文

### 3. run-cat.js — 调用脚本

用 spawn 调用 claude CLI，动态挂载 MCP Server：

```bash
claude -p "你的任务是写一首关于猫的诗。
在开始写之前，先用 cat_cafe_get_context 获取上下文。
写完后，用 cat_cafe_post_message 把诗发到聊天室。
注意：你的思考过程不需要发送，只把最终的诗发到聊天室即可。" \
  --output-format stream-json \
  --verbose \
  --mcp-config '{"mcpServers":{"cat-cafe":{"command":"node","args":["cat-cafe-mcp.js"]}}}'
```

脚本需要：
- 设置环境变量 CAT_CAFE_API_URL, CAT_CAFE_INVOCATION_ID, CAT_CAFE_CALLBACK_TOKEN
- 这些值从 callback-server 启动时打印的值获取（可以硬编码或通过参数传入）
- 解析 Claude CLI 的 NDJSON 输出（参考第一课作业）

## 运行方式

终端 1：
```bash
node callback-server.js
# 输出：Server listening on :3200
# 输出：invocationId: xxx
# 输出：callbackToken: yyy
```

终端 2：
```bash
CAT_CAFE_API_URL=http://localhost:3200 \
CAT_CAFE_INVOCATION_ID=xxx \
CAT_CAFE_CALLBACK_TOKEN=yyy \
node run-cat.js
```

## 预期结果

- 终端 2 会显示 Claude CLI 的内部输出（"内心独白"）
- 终端 1 会收到并打印 Claude 通过 post_message 发送的诗（"主动发言"）
- 两个终端的输出是不同的！终端 2 是 AI 的全部思考过程，终端 1 只有 AI 选择公开的内容

## 技术要求

- callback-server.js: 纯原生 Node.js，不用框架
- cat-cafe-mcp.js: 使用 @modelcontextprotocol/sdk（需要 npm install）
- run-cat.js: 纯原生 Node.js

请帮我写这三个文件，并解释核心设计。
```

**验收标准:**
1. 启动 callback-server.js，看到 invocationId 和 callbackToken 被打印
2. 运行 run-cat.js，Claude 执行任务
3. **终端 1**（服务器）收到并打印了 Claude 发来的诗
4. **终端 2**（客户端）显示了 Claude 的完整思考过程
5. **两个终端的输出不同**——这就是"内心独白 vs 主动发言"的区别

---

### P05-C1 · Prompt 注入版 Callback {#p05-c1}

**目标:** 不使用 `--mcp-config`，而是把 HTTP callback 指令写进系统提示词（模拟 Codex/Gemini 的做法）。

**关键提示（注入到 system prompt）:**
```
你可以通过以下 curl 命令发送消息：
curl -X POST http://localhost:3200/api/callbacks/post-message \
  -H "Content-Type: application/json" \
  -d '{"invocationId":"xxx","callbackToken":"yyy","content":"你的消息"}'
```

**观察要点:** AI 真的会照着 prompt 里的 curl 命令去执行吗？

---

### P05-C2 · MCP vs curl 对比实验 {#p05-c2}

**目标:** 同一个任务，分别用两种方式调用，对比行为差异。

> - 方式 A：通过 `--mcp-config` 挂载 MCP Server（Claude 原生方式）
> - 方式 B：通过系统提示词注入 curl 命令（Codex/Gemini 的兜底方式）
>
> 对比：哪种更可靠？哪种的调用格式更稳定？

---

### P05-C3 · 隐私模式（猫猫杀）{#p05-c3}

**目标:** 修改 run-cat.js，让它**不打印 Claude CLI 的内部输出**，只在 callback-server 的终端上看到 Claude 主动发送的消息。

> **说明:** 这就是"猫猫杀模式"——其他玩家只能看到猫主动说的话，看不到猫在想什么。

---

## Lesson 06 · 数据丢失演练

> **对应课程:** 第六课 — 消失的 28 秒（Redis 数据丢失事故）
> **核心收获:** 亲手体验数据丢失→取证→恢复全过程；建立三层防护意识

---

### P06-MAIN · Redis 数据丢失演练环境 {#p06-main}

**目标:** 搭建一个 Redis 数据丢失演练环境，走通"写入→取证→备份→灾难→恢复→验证"完整流程。

```
帮我搭建一个 Redis 数据丢失演练环境。要求：

1. write-test-data.mjs：
   - 往 Redis 写入 50 条测试消息，key 格式 `demo:msg:{id}`
   - 写入 5 个 thread 索引，key 格式 `demo:thread:{name}`
   - 写完后打印 dbsize 和 key 分布统计
   - 用 ioredis，连接 redis://localhost:6399

2. redis-forensics.sh：
   - 连接指定端口的 Redis（默认 6399）
   - 打印 dbsize
   - 按 namespace 统计 key 数量（demo:msg:*, demo:thread:* 等）
   - 抽样显示 3 个 key 的内容
   - 必须是只读操作，不能写入任何东西

3. simulate-disaster.sh：
   - 先触发 BGSAVE（确保有一份"灾前备份"）
   - 等待 BGSAVE 完成
   - 打印警告："即将执行 FLUSHDB，这会清空所有数据"
   - 要求用户输入 FLUSH 6399 确认（不是 y/n）
   - 执行 FLUSHDB
   - 打印灾后 dbsize

4. restore-from-rdb.sh：
   - 接受 --source 参数指定 RDB 文件路径
   - 恢复前先备份当前 dump（复制到 .bak 文件）
   - 关闭 Redis → 替换 dump.rdb → 重启 Redis
   - 恢复后打印 dbsize 和 key 分布统计
   - 必须有 --yes 参数才执行，否则只打印计划

全部脚本用 bash 写（除了 write-test-data.mjs 用 Node.js）。
注释用中文。每个脚本开头写清楚"这个脚本做什么"。
```

**验收流程:**
```bash
node write-test-data.mjs      # → dbsize: 55
bash redis-forensics.sh       # → demo:msg:* = 50, demo:thread:* = 5
redis-cli -p 6399 BGSAVE      # 触发备份
bash simulate-disaster.sh     # 输入 FLUSH 6399 → dbsize: 0 😱
bash redis-forensics.sh       # → demo:msg:* = 0
bash restore-from-rdb.sh --source /path/to/dump.rdb --yes  # → dbsize: 55 ✅
bash redis-forensics.sh       # → 50 + 5 ✅
```

---

### P06-C1 · 故意选错备份演练 {#p06-c1}

**目标:** 模拟"第一次选错备份"的失误，理解恢复前验证备份内容的重要性。

```
帮我扩展演练环境，模拟"选错备份"的场景：

1. 修改 write-test-data.mjs，支持 --batch 参数：
   - batch=1：写入 50 条消息（完整数据）
   - batch=2：只写入 20 条消息（不完整数据）

2. 演练流程：
   - 执行 batch=1，BGSAVE → 得到 dump-full.rdb
   - 执行 batch=2（覆盖部分数据），BGSAVE → 得到 dump-partial.rdb
   - FLUSHDB 清空
   - 先用 dump-partial.rdb 恢复 → 发现只有 20 条 ❌
   - 再用 dump-full.rdb 恢复 → 50 条全回来 ✅

3. 教训：恢复前要先验证备份内容，不要盲选。
```

---

### P06-C2 · 目录大小防腐门 {#p06-c2}

**目标:** 给项目加一个目录大小检查脚本，防止目录悄悄膨胀。

```
帮我写一个 check-dir-size.sh 脚本：

1. 扫描指定目录下所有子目录
2. 统计每个子目录中的源文件数量（排除 index.ts 和 .d.ts）
3. 双阈值：
   - >= 15 个文件：黄色警告
   - >= 25 个文件：红色报错，exit 1
4. 支持豁免机制：
   - 读取 .dir-exceptions.json 文件
   - 每条豁免必须有 owner（谁注册的）和 expiresAt（到期日）
   - 过期的豁免自动变成报错

用法示例：
  bash check-dir-size.sh src/
  bash check-dir-size.sh --root packages/api/src

把它加到你的 package.json scripts 里，
这样每次提交前跑一下就知道有没有目录在悄悄膨胀。
```

---

### P06-C3 · 证据闸门脚本 {#p06-c3}

**目标:** 生成标准化的构建+测试证据报告，让 PR 里的证据可复现。

```
帮我写一个 generate-evidence.sh 脚本：

1. 运行项目的构建命令（如 npm run build）
2. 运行项目的测试命令（如 npm test）
3. 解析测试输出，提取：
   - 总测试数、通过数、失败数
4. 生成一个 Markdown 表格：

| 指标 | 值 |
|------|-----|
| 时间 (UTC) | 2026-02-18T12:00:00Z |
| 分支 | feat/my-feature |
| Commit | abc1234 |
| 构建状态 | 通过 |
| 总测试 | 42 |
| 通过 | 42 |
| 失败 | 0 |
| 通过率 | 100% |

5. 安全检查：如果总测试为 0 但退出码为 0，
   打印警告"测试解析可能失败，请手动确认"

6. 支持 --out report.md 输出到文件

这个脚本的目的是让 PR 里的测试证据标准化、可复现，
而不是靠开发者手动数测试结果。
```

---

## Lesson 07 · Rich Blocks 富文本

> **对应课程:** 第七课 — 从 Café 到平台：Rich Blocks 与 Context Cleaner
> **核心收获:** AI 输出结构化 block；Context Cleaner 分离显示层与推理层

---

### P07-MAIN · 最小 Rich Blocks 管线 {#p07-main}

**目标:** 搭建一个最小的 AI 聊天 + Rich Blocks 系统，体验"AI 输出不只是文字"。

```
帮我搭建一个最小的 AI 聊天 + Rich Blocks 系统。要求：

1. 约定格式：AI 回复中可以包含 ```cc_rich {...} ``` 代码块，
   格式是 JSON，必须有 kind 字段。支持两种 kind：
   - card：{ kind: "card", title: "标题", body: "内容", tone: "info" | "success" | "warning" }
   - checklist：{ kind: "checklist", title: "标题", items: [{ text: "内容", done: boolean }] }

2. rich-extract.js：
   - 输入：AI 回复的原始文本
   - 输出：{ cleanText: "去掉 cc_rich 块后的纯文本", blocks: [...提取出的 block 数组] }
   - 用正则提取 ```cc_rich ... ``` 块
   - 解析 JSON，验证 kind 字段存在
   - 容错：如果 JSON 解析失败，跳过该块（不要崩溃）

3. rich-digest.js：
   - 输入：一条包含 rich blocks 的消息
   - 输出：适合放回 prompt 的摘要文本
   - 规则：
     - card → [卡片: {title}]
     - checklist → [清单: {title}, {done}/{total} 完成]
   - 这是 Context Cleaner 的核心——让 AI 知道"我之前发过什么"，
     但不浪费 token 存完整 JSON

4. chat-server.js：
   - 用 Express 或原生 http
   - POST /chat 接受 { message: "用户输入", history: [...] }
   - 调用 AI API（OpenAI/Anthropic/本地模型都行）
   - 系统提示词里告诉 AI：
     "你可以用 ```cc_rich {...} ``` 格式发送富文本卡片。
      当你想强调某个信息时，用 card。
      当你要列出任务时，用 checklist。"
   - AI 回复后，用 rich-extract.js 提取 blocks
   - 存储消息时分开存：{ text: cleanText, blocks: [...] }
   - 组装下一轮 prompt 时，用 rich-digest.js 替换历史中的 blocks

5. index.html + styles.css：
   - 简单的聊天界面
   - 普通文本正常渲染
   - card 块渲染为带颜色边条的卡片（info=蓝, success=绿, warning=橙）
   - checklist 块渲染为带勾选框的列表（只读）
   - 刷新页面后，从 history 重新渲染（验证持久化）

全部用 Node.js + 原生 HTML/CSS，不用框架。注释用中文。
```

**关键体验:**
- AI 的回复不再只是纯文字——有了结构化的卡片和清单
- 打开浏览器开发者工具看网络请求，观察：prompt 里的历史消息**没有**完整的 JSON blocks，只有 `[卡片: ...]` 摘要——这就是 Context Cleaner 在工作

---

### P07-C1 · Game Block（互动角色卡）{#p07-c1}

**目标:** 给 Rich Blocks 加一个 game 类型，实现 AI 文字冒险的基础。

```
帮我给 Rich Blocks 系统加一个 game 类型的 block：

1. 数据格式：
   {
     kind: "game",
     character: "勇者猫猫",
     hp: { current: 85, max: 100 },
     stats: { attack: 15, defense: 12, speed: 8 },
     status: ["中毒", "加速"]
   }

2. 前端渲染：
   - 角色名 + 像素风头像（可以用 emoji 代替）
   - 血条（绿色/黄色/红色根据比例变化）
   - 属性数值
   - 状态标签（buff 绿色，debuff 红色）

3. Context Cleaner 摘要：
   [角色: 勇者猫猫, HP 85/100, 状态: 中毒+加速]

4. 测试场景：
   让 AI 当 DM，你当玩家，打一场简单的战斗。
   AI 每轮输出更新后的角色卡。
```

---

### P07-C2 · 双路由 Rich Blocks {#p07-c2}

**目标:** 实现 MCP 风格（HTTP 回调）和文本提取两条路由，模拟 Cat Café 的双路由设计。

```
帮我实现 Rich Blocks 的双路由模式：

Route A（MCP 风格 — HTTP 回调）：
- 新增 POST /api/create-block 端点
- AI 可以通过"工具调用"直接发送 block
- 用一个 BlockBuffer 暂存，消息写入时合并

Route B（文本提取 — 当前模式）：
- 从 AI 回复文本中提取 ```cc_rich ... ``` 块
- 这是 fallback 路径

合并逻辑：
- 消息写入时，Route A 的 blocks + Route B 的 blocks 合并
- 去重（按 block id）

测试：
- 只用 Route A 发送一个 card
- 只用 Route B 发送一个 checklist
- 同时用两个路由发送，验证合并和去重
```

---

### P07-C3 · Rich Block 格式容错 {#p07-c3}

**目标:** 写 `normalizeRichBlock()` 函数，处理 AI 不听话时的各种格式错误。

```
AI 有时候不听话，不按你的格式输出。
帮我写一个 normalizeRichBlock() 函数，处理这些情况：

1. "type" 误用为 "kind" → 自动映射
2. 缺少 "v" 字段 → 默认填 1
3. kind 不在已知列表里 → 返回 null（丢弃）
4. JSON 格式错误 → 跳过，不崩溃
5. card 的 tone 不在已知列表里 → 默认为 "info"

写 5 个测试用例，每个对应一种容错场景。
```

---

## Lesson 08 · Session Chain

> **对应课程:** 第八课 — Session Management：空间、时间、战略三维挑战
> **核心收获:** 亲手体验 context 满了→封存→新 session 按需搜索旧 session 的完整流程

---

### P08-MAIN · 最小 Session Chain 模拟器 {#p08-main}

**目标:** 搭建一个最小的 Session Chain 系统，体验"context 满了怎么换命"。

```
帮我搭建一个最小的 Session Chain 模拟器，理解"context 满了怎么换命"。

## 背景

AI Agent 的 context window 有上限。满了以后有两种做法：
A) 压缩：把对话摘要化，信息有损
B) Session Chain：封存当前 session，开新 session，新猫按需搜索旧 session

我要体验方案 B。

## 要求

### 1. session-store.mjs — Session 存储 + HTTP API

一个 Node.js HTTP 服务器（原生 http 模块，不用框架），端口 3210：

数据模型：
- sessions: Map<sessionId, { id, seq, status, messages[], sealedAt? }>
- status: "active" | "sealed"

API 端点：
- POST /api/sessions — 创建新 session，返回 { id, seq }
- POST /api/sessions/:id/messages — 往 session 追加一条消息 { role, content }
- POST /api/sessions/:id/seal — 封存 session（status → sealed，记录 sealedAt）
- GET /api/sessions — 列出所有 session（id, seq, status, messageCount, sealedAt）
- GET /api/sessions/:id — 获取 session 详情（含完整 messages）
- GET /api/sessions/:id/digest — 返回 session 的 digest（自动从 messages 生成摘要）
  digest 格式：按角色分类统计消息数，列出所有 assistant 消息的前 80 字符，最后附"未完成事项"提取（从最后 3 条消息中找含"TODO"/"待做"/"下一步"的内容）
- GET /api/sessions/search?q=关键词 — 全文搜索所有 sealed session 的消息内容
  返回匹配的 { sessionId, seq, messageIndex, snippet（前后各 50 字符）}

启动时创建一个 Session 1 并预填充 8 条模拟对话（模拟一个开发任务的上下文）：
- user: "帮我实现一个用户注册功能"
- assistant: "好的，我来设计数据模型。User 表需要 id, email, passwordHash, createdAt 四个字段。选择 bcrypt 做密码哈希，因为它有内置的 salt 且计算成本可调。"
- user: "用什么框架？"
- assistant: "推荐 Fastify，原因：1. 比 Express 快 2 倍 2. 原生 TypeScript 支持 3. JSON Schema 验证内置。TODO: 后续需要加 rate limiting 防暴力破解。"
- user: "好的，先写 API 路由"
- assistant: "POST /api/register 路由已实现。校验规则：email 格式 + 密码至少 8 位。返回 201 + userId。下一步：需要加邮箱验证流程，暂时跳过。"
- user: "加个登录接口"
- assistant: "POST /api/login 已实现。用 JWT 做 token，过期时间 24h。待做：refresh token 机制还没做，当前是硬过期。"

然后将 Session 1 封存（seal）。

### 2. session-tools-mcp.js — MCP Server

使用 @modelcontextprotocol/sdk，提供四个工具：

- list_sessions() — 调 GET /api/sessions，返回所有 session 列表
- read_session(sessionId) — 调 GET /api/sessions/:id，返回完整消息
- get_digest(sessionId) — 调 GET /api/sessions/:id/digest，返回 digest 摘要
- search_sessions(query) — 调 GET /api/sessions/search?q=...，全文搜索

所有工具从环境变量 SESSION_STORE_URL 读取服务地址（默认 http://localhost:3210）。

### 3. run-session-chain.mjs — 模拟 Session Chain 交接

用 spawn 调用 claude CLI，模拟"Session 2 的猫"醒来后的行为：

系统提示词（关键！）：
"""
你是 Session 2 的布偶猫。你刚从一次 session 交接中醒来。

你知道的事实：
- 你是第 2 个 session，前面有 1 个已封存的 session
- 你有一套 MCP 工具可以查询旧 session 的内容
- 你不记得 Session 1 里发生了什么——但你可以查

你的任务：
1. 先用 list_sessions 看看有哪些旧 session
2. 用 get_digest 读取上一个 session 的摘要
3. 找出 Session 1 里提到但还没完成的事项（TODO / 待做 / 下一步）
4. 用 search_sessions 搜索一个你感兴趣的技术决策细节
5. 整理一份简短的"交接报告"，格式如下：

## 交接报告
### Session 1 做了什么
（摘要）
### 未完成事项
（列表）
### 我查到的关键决策
（你搜索到的细节）
### Session 2 的计划
（你打算先做什么）

注意：不要猜测 Session 1 的内容。一切信息必须来自工具查询。
"""

运行后：
- 解析 Claude CLI 的 NDJSON 输出
- 打印 Claude 调用了哪些 MCP 工具（工具名 + 参数）
- 打印最终的交接报告

## 运行方式

终端 1：
```bash
node session-store.mjs
# → Session 1 created with 8 messages (sealed)
# → Server listening on :3210
```

终端 2：
```bash
SESSION_STORE_URL=http://localhost:3210 node run-session-chain.mjs
# → [MCP] list_sessions → 1 session found
# → [MCP] get_digest(session_1) → ...
# → [MCP] search_sessions("bcrypt") → 1 hit
# → === 交接报告 ===
# → ...
```

## 技术要求

- session-store.mjs: 纯原生 Node.js，不用框架
- session-tools-mcp.js: 使用 @modelcontextprotocol/sdk
- run-session-chain.mjs: 纯原生 Node.js
- 注释用中文
```

**关键体验:** Session 2 的猫面对一片空白的 context，靠搜索工具重建了对前世的理解。这就是"读旧 session = 搜代码"的直观感受。

---

### P08-C1 · 压缩 vs 交接信息保真度对比 {#p08-c1}

**目标:** 设计实验对比"压缩"和"Session Chain 交接"哪个丢的信息更多。

```
帮我设计一个对比实验，测试"压缩"和"Session Chain 交接"哪个丢的信息更多。

实验设计：
1. 准备一段包含 15 条消息的模拟对话（开发一个功能的完整过程），
   其中包含 5 个关键技术决策（每个决策有明确的 WHY）。

2. 方案 A — 压缩模式：
   把 15 条消息压缩成 3 条摘要（模拟 CLI 压缩），
   然后问 AI："之前为什么选择了 X 而不是 Y？"（针对 5 个决策各问一个）
   记录 AI 能准确回答几个。

3. 方案 B — Session Chain 模式：
   把 15 条消息存入 session-store，封存。
   新 session 的 AI 用 search_sessions + read_session 查询后回答同样的 5 个问题。
   记录 AI 能准确回答几个。

4. 对比：
   - 哪种方式回答准确率更高？
   - 哪种方式的 AI 更会说"我不确定"而不是编答案？
   - 哪种方式消耗的 token 更少？

用 Markdown 表格展示对比结果。
```

**预期发现:** 方案 B 准确率更高，因为原始信息完整保留；方案 A 的猫更容易"编造"一个看似合理但实际错误的 WHY。

---

### P08-C2 · 跨 Thread 污染复现 {#p08-c2}

**目标:** 亲手重现 Thread Affinity Bug（茶话会夺魂 bug）。

```
帮我搭建一个最小的跨 thread 污染演示。

1. 修改 session-store.mjs：
   - 加入 thread 概念（每个 session 属于一个 threadId）
   - 但 session 查找故意用 BUG 版本：只按 catId 匹配，不看 threadId

2. 模拟两个 thread：
   - Thread A "写代码"：session 里有 5 条关于 Phase 5 开发的消息
   - Thread B "茶话会"：session 里有 5 条关于哲学讨论的消息

3. 演示污染：
   - 在 Thread B 里调用猫，但因为 session 查找不带 threadId，
     猫 resume 了 Thread A 的 session
   - 猫在茶话会里突然开始讨论 Phase 5 😱

4. 修复：
   - session 查找加上 threadId 过滤
   - 再次在 Thread B 里调用猫，这次正常了

打印对比：
- [BUG] Thread B 的猫说了什么（应该会提到 Phase 5）
- [FIX] Thread B 的猫说了什么（应该只聊哲学）
```

**关键体验:** 亲眼看到"少了一个字段"如何导致灵魂串线。

---

### P08-C3 · Context 健康度监控 {#p08-c3}

**目标:** 给 CLI 调用项目加一个 context 健康度监控管道。

```
帮我写一个 context-health-monitor.mjs：

1. 监听 Claude CLI 的 NDJSON 输出流
2. 从中提取 token 使用信息（如果有 usage 事件）
3. 实时计算 fillRatio = usedTokens / contextWindow
4. 双阈值告警：
   - >= 75%：黄色警告 "⚠️ Context 75%，注意控制对话长度"
   - >= 90%：红色警告 "🔴 Context 90%，建议结束当前 session"
5. 输出格式：每次 AI 回复后打印一行
   [Context] 85,234 / 200,000 (42.6%) ████████░░░░░░░░░░░░ OK
   [Context] 172,000 / 200,000 (86.0%) █████████████████░░░ ⚠️ WARN

这个脚本可以用管道接在 CLI 调用后面：
  node run-cat.mjs | node context-health-monitor.mjs

如果拿不到 token 数据，用消息字符数做近似估算（1 token ≈ 4 字符）。
```

---

## 附：各课思考题汇总 / Reflection Questions

> 原版作业附带的思考题，适合在完成练习后自我检验。

**Lesson 06 思考题:**
1. 你的项目有数据恢复方案吗？如果你的数据库今晚丢了 95% 的数据，你能在 5 分钟内恢复吗？
2. 你的 AI 助手在操作数据库时有隔离吗？它连的是生产数据库还是开发数据库？
3. 你的代码库有"悄悄变烂"的检测机制吗？有没有目录已经堆了 50+ 文件但没人注意？
4. 如果要在"快速开发"和"安全护栏"之间选一个先做，你选哪个？

**Lesson 07 思考题:**
1. 你的 AI 助手回复过"纯文字太长读不下去"的内容吗？哪些部分适合变成卡片或清单？
2. Context Cleaner 的本质是什么？（提示：不只是"省 token"——是"显示层和上下文层的分离"）
3. 如果要给你的 AI 聊天加一种全新的 block，你会加什么？
4. 从"工具"到"平台"的转变需要什么？

**Lesson 08 思考题:**
1. 你的 AI 助手被压缩过吗？你有没有遇到 AI 突然"忘了"之前讨论的决策？
2. 如果让你设计 session 交接，你会选"写遗书"还是"留尸体"？
3. "按需拉取优于一次性灌入"这个原则还能用在哪？
4. 你的多轮对话系统有 thread affinity 问题吗？

---

*收录时间：2026-03-01 | Cat Café Tutorials P006 · 课后作业提示词完整存档*
*原始来源：https://github.com/zts212653/cat-cafe-tutorials*
