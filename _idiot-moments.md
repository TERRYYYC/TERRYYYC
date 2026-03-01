# 复盘笔记 · Idiot Moments Log

> 本文件记录工作流执行中出现的失误、盲点与判断失误。
> 目标不是自责，而是从每次失误中提炼出**可复用的防护原则**，并将其归入系统流程。
>
> This file logs workflow mistakes, blind spots, and judgment errors.
> Goal: not self-criticism, but extracting **reusable defensive principles** and wiring them into the system.

---

## 字段说明 / Field Glossary

| 字段 | 含义 |
|---|---|
| **现象** | 表面上发生了什么，谁发现的 |
| **根因** | 为什么会发生——思维或流程上的漏洞 |
| **发现方式** | 自查 / 用户指出 / 系统检查 |
| **影响** | 造成了哪些实际后果（数据断档、返工、信任损耗等）|
| **修复** | 事后做了哪些补救 |
| **提炼原则** | 从这次失误中抽象出的、可在未来不同情境下复用的规则 |
| **防护措施** | 应该加入哪个 checklist 或流程，让这条错误真正"闭环" |
| **类型标签** | 见下方类型表 |

**类型标签 / Type Tags:**

| 标签 | 含义 |
|---|---|
| `scope-miss` | 任务范围定义遗漏了应包含的项 |
| `rule-violation` | 有明文规定但没有遵守 |
| `assumption-error` | 基于错误假设行动 |
| `context-lost` | 上下文丢失导致决策错误 |
| `tooling-gap` | 使用了不合适的工具或方法 |
| `communication-gap` | 误解了用户意图 |

---

## 记录列表 / Entries

---

### IM-001 · 2026-03-01 · 系统文件不在"改造范围"内

**现象 / What happened:**
执行 frontmatter v2.1 升级任务时，成功更新了全部 11 个项目笔记、模版文件和 `_HOW-TO-USE.md`，但遗漏了 `_quick-check.md` 和 `_personal-learning-log.md`。用户在会话结束后指出。

**根因 / Root cause:**
任务范围被心理上定义为 **"p00X 笔记 + 模版 + HOW-TO-USE"**，`_` 前缀的系统文件被隐性排除。更关键的是：没有在任务开始前或结束前重读 `_quick-check.md`——如果读了，里面明确有一条"修改 HOW-TO-USE 后必须追加 `_personal-learning-log.md`"，就不会遗漏。即：**有文档的规则，但没有执行文档要求的自查步骤。**

**发现方式 / How detected:** 用户（Terry）在会话结束后主动指出

**影响 / Impact:**
- `_quick-check.md` 的 frontmatter 示例停留在旧格式（v2.0），5字段，缺 `file_type` / `source`
- `_personal-learning-log.md` 的系统变更日志出现断档，v2.1 升级记录缺失
- 知识库自文档不一致，降低了系统可信度

**修复 / Fix:**
- `_quick-check.md`：更新第3节 frontmatter 示例为7字段，新增 `file_type`/`source`/`·` 检查项，扩充常见错误表，版本升 v1.1
- `_personal-learning-log.md`：补录完整 v2.1 变更条目（含11文件清单、新增文件列表、根因备注）

**提炼原则 / Extracted principle:**
> **"任务范围内修改了规范文档，必须同步更新所有依赖该规范的文档。"**
>
> 更通用版本：**"系统文件之间有隐性依赖关系。改了 A（规范源），就要更新所有引用 A 内容的 B（规范镜像）。"**
>
> 特别是：`_HOW-TO-USE.md`（规范源）→ `_quick-check.md`（规范摘要）→ `_personal-learning-log.md`（变更日志）是一条需要同步的链。

**防护措施 / Prevention:**
- 已在 `_quick-check.md` 新增"系统文件变更核查"节（v1.0 时已有，但未被执行）
- **建议在 `_quick-check.md` 顶部加显式提醒**：每次任务结束前必须过一遍核查表，而不是选择性阅读
- 后续所有涉及规范变更的任务，应将"读 `_quick-check.md`"列为最后一个 Todo 项（验证步骤）

**类型标签:** `scope-miss` · `rule-violation`

---

## 使用建议 / How to Use

**何时添加新条目 / When to add:**
- 用户指出了一个遗漏、错误或不一致
- 自查时发现了本不应该出现的问题
- 同一类错误出现第二次（必须追加，并注明"复发"）

**编号规则 / Numbering:**
`IM-XXX`，三位数字，顺序递增，不按项目分组（错误类型比项目更重要）

**复发标记 / Recurrence flag:**
若某条原则对应的错误再次发生，在新条目末尾注明：
```
⚠️ 复发：参见 IM-XXX（首次记录）
```

**定期回顾 / Periodic review:**
每完成 5 个新项目后，浏览一次所有 IM 条目，检查其中的"提炼原则"是否已真正内化进工作流。

---

*v1.0 · 2026-03-01 · 首条记录：IM-001*
