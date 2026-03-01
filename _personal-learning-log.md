# 系统变更日志 / System Change Log

> 本文件记录知识库本身的结构性变更、设计讨论和决策。
> This file records structural changes, design discussions, and decisions about the knowledge base itself.
> 学习内容的日志请见 `_index.md` 的「会话时间线」。
> For session-level learning logs, see the "Session Timeline" in `_index.md`.

---

## 2026-03-01 · 知识库 v2.0 重构 / Knowledge Base v2.0 Restructure

**变更类型 / Change Type:** 结构重组 + 规范升级

**触发原因 / Trigger:**
Terry 提出四条改进建议，经讨论后采纳并调整执行：
1. 以项目名称为组织单位，学习时间作为 tag
2. `_index.md` 增加目录/日志，加入进度阶段、跳转链接
3. learning-summary 文件按项目整合入文件夹，命名更语义化
4. （调用次数建议调整为进度阶段标签，降低维护摩擦）

**未采纳项及理由 / Rejected Items:**
- ❌ 「学习条目被调用次数」→ 改为进度阶段标签（🌱/🔍/🔁/✅）。原方案手动维护成本高且容易腐烂；阶段标签能回答更有价值的问题："这个知识是纸面的还是实战过的？"

**具体变更 / Changes Made:**

| 变更项 | 变更前 | 变更后 |
|---|---|---|
| 文件组织 | 所有文件平铺在根目录 | 按项目归入文件夹 `p00X-[名称]/` |
| 命名格式 | `learning-summary_YYYY-MM-DD_N.md` | `[P编号]_[模块]_YYYY-MM-DD.md` |
| 模版文件 | 根目录散落 | 统一归入 `_templates/` |
| 杂项文件 | 根目录散落 | 统一归入 `_misc/` |
| `_index.md` 会话时间线 | 无文件链接，无进度阶段 | 加入相对路径链接 + 进度阶段列 |
| `_index.md` 顶部 | 无阶段说明 | 新增进度阶段 legend 表 |
| `_HOW-TO-USE.md` | v1.0，仅日期命名规范 | v2.0，新增文件夹结构、frontmatter 规范、进度阶段说明 |

**文件移动明细 / File Migration Map:**

| 原文件名 | 新路径 |
|---|---|
| `learning-summary_2026-02-27.md` | `p001-skill-evo/p001_skill-creator_2026-02-27.md` |
| `skill-creator-SKILL-archive_2026-02-27.md` | `p001-skill-evo/p001_skill-creator-archive_2026-02-27.md` |
| `learning-summary_2026-02-28.md` | `p002-openclaw/p002_frontend-overview_2026-02-28.md` |
| `learning-summary_2026-02-28_2.md` | `p003-engineering-blog/p003_boris-workflow_2026-02-28.md` |
| `learning-summary_2026-02-28_3.md` | `p003-engineering-blog/p003_sdlc-is-dead_2026-02-28.md` |
| `learning-summary_2026-03-01.md` | `p003-engineering-blog/p003_agent-teams-reverse_2026-03-01.md` |
| `learning-summary_2026-03-01_2.md` | `p004-swarm-ide/p004_core-architecture_2026-03-01.md` |
| `learning-summary_2026-03-01_3.md` | `p004-swarm-ide/p004_local-source-deep-dive_2026-03-01.md` |
| `learning-summary_2026-03-01_4.md` | `p004-swarm-ide/p004_frontend-architecture_2026-03-01.md` |
| `learning-summary_2026-03-01_5.md` | `p005-deerflow/p005_frontend-overview_2026-03-01.md` |
| `learning-summary_2026-03-01_6.md` | `p006-cat-cafe/p006_lesson00-08-overview_2026-03-01.md` |
| `template_learning-session.md` | `_templates/template_learning-session.md` |
| `skill-security-scan-template_2026-02-27.md` | `_templates/skill-security-scan-template_2026-02-27.md` |
| `2026-02-26.md` | `_misc/misc_adult-learning-methodology_2026-02-26.md` |
| `未命名.md` | `_misc/misc_unnamed.md` |
| `AI 时代个人与团队效能提升.md` | `_misc/misc_ai-era-productivity.md` |
| `Homebrew notice.md` | `_misc/misc_homebrew-notice.md` |
| `Homebrew和NPM(Node.js)区别.md` | `_misc/misc_homebrew-vs-npm.md` |

**新增文件 / New Files:**
- `_personal-learning-log.md`（本文件）
- `_quick-check.md`（规范快速核查清单）

**执行者 / Executed by:** Claude (Cowork)
**状态 / Status:** ✅ 完成

---

## 如何使用本日志 / How to Use This Log

**追加新条目的时机 / When to append:**
- 修改了 `_HOW-TO-USE.md` 或 `_index.md` 的结构性内容
- 重命名、移动、删除了文件
- 讨论并决定（或否决）了某个系统设计方案
- 升级了命名规范或工作流

**条目格式 / Entry Format:**

```markdown
## YYYY-MM-DD · [变更标题]

**变更类型:** 结构重组 / 规范升级 / 新增功能 / 决策记录 / Bug修复

**触发原因:** ...

**具体变更:**
...

**执行者:** Terry / Claude
**状态:** ✅ 完成 / 🚧 进行中 / ❌ 否决
```

---

---

## 2026-03-01 · Frontmatter 规范升级 v2.1 + Prompt 收录工作流

**变更类型 / Change Type:** 规范升级 + 新增功能

**触发原因 / Trigger:**
P006 cat-cafe-tutorials 的 `homework-prompts` 文件生成时，Claude 自动加入了 `file_type` 和 `source` 字段，效果良好。Terry 指出现有规范（v2.0）缺少这两个字段，且历史笔记均未补充，决定全库统一升级。

**具体变更 / Changes Made:**

| 变更项 | 变更前（v2.0）| 变更后（v2.1）|
|---|---|---|
| frontmatter `file_type` 字段 | 不存在 | 新增，必填；取值：`learning-summary / archive / homework-prompts / scan-report / misc / workflow-guide` |
| frontmatter `source` 字段 | 不存在 | 新增，外部项目必填（GitHub/文章 URL）|
| frontmatter `project` 格式 | `P004 Swarm-IDE` | `P004 · Swarm-IDE`（加 `·` 分隔符）|
| `_HOW-TO-USE.md` 第三节 | 5 字段规范 | 7 字段规范，含完整字段说明表和 file_type 取值表 |
| `_templates/template_learning-session.md` | 无 frontmatter 区块 | 顶部添加完整 frontmatter 模版块 |
| `_quick-check.md` Section 3 | 旧 5 字段示例 | 更新为 7 字段示例 + 新增 `file_type`/`source`/`·` 检查项 |

**历史文件补录 / Retroactive Frontmatter Added（11 个文件）:**

| 文件 | file_type | 新增字段 |
|---|---|---|
| `p001_skill-creator_2026-02-27.md` | learning-summary | file_type, ·分隔符 |
| `p001_skill-creator-archive_2026-02-27.md` | archive | file_type, ·分隔符 |
| `p002_frontend-overview_2026-02-28.md` | learning-summary | file_type, source, ·分隔符 |
| `p003_boris-workflow_2026-02-28.md` | learning-summary | file_type, source, ·分隔符 |
| `p003_sdlc-is-dead_2026-02-28.md` | learning-summary | file_type, source, ·分隔符 |
| `p003_agent-teams-reverse_2026-03-01.md` | learning-summary | file_type, source, ·分隔符 |
| `p004_core-architecture_2026-03-01.md` | learning-summary | file_type, source, ·分隔符 |
| `p004_local-source-deep-dive_2026-03-01.md` | learning-summary | file_type, source, ·分隔符 |
| `p004_frontend-architecture_2026-03-01.md` | learning-summary | file_type, source, ·分隔符 |
| `p005_frontend-overview_2026-03-01.md` | learning-summary | file_type, source, ·分隔符 |
| `p006_lesson00-08-overview_2026-03-01.md` | learning-summary | file_type, source, ·分隔符 |

**新增文件 / New Files:**
- `personal-learning/create-prompt.md`（homework prompt 收录工作流指引，`file_type: workflow-guide`）
- `p006-cat-cafe/p006_homework-prompts_2026-03-01.md`（24 个提示词，P001-MAIN 至 P08-C3，含完整目录索引）

**根因备注 / Root Cause Note:**
本次升级完成后，`_quick-check.md` 和 `_personal-learning-log.md` 本身未被及时同步更新，系统日志出现断档。原因：任务范围定义为"p00X 笔记 + 模版 + HOW-TO-USE"，`_` 前缀系统文件未被纳入范围检查。已于同日补录。

**执行者 / Executed by:** Claude (Cowork)
**状态 / Status:** ✅ 完成

---

---

## 2026-03-01 · 新增 _idiot-moments.md（失误复盘日志）

**变更类型 / Change Type:** 新增功能

**触发原因 / Trigger:**
本次 v2.1 升级遗漏 `_quick-check.md` 和 `_personal-learning-log.md` 更新，Terry 提议将此类"有价值的失误"单独记录，形成可复用的错误模式数据库，用于后续项目复盘和经验分享。

**具体变更 / Changes Made:**
- 新建 `_idiot-moments.md`（系统根目录），首条记录 IM-001 记录本次遗漏事件
- 文件含字段说明（现象/根因/发现方式/影响/修复/提炼原则/防护措施）、类型标签体系、复发标记规则和定期回顾建议
- `_HOW-TO-USE.md` 文件结构树新增 `_idiot-moments.md` 条目

**新增文件 / New Files:**
- `_idiot-moments.md`（v1.0，含 IM-001）

**执行者 / Executed by:** Claude (Cowork)
**状态 / Status:** ✅ 完成

---

_(新变更记录在此文件末尾追加 / Append new entries at the bottom)_
