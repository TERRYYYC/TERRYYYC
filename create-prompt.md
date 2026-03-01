---
file_type: workflow-guide
tags: [prompt-collection, homework, workflow, template]
date: 2026-03-01
related: _HOW-TO-USE.md
---

# create-prompt · 课后作业 Prompt 收录操作指引
# Workflow Guide: Collecting & Indexing Homework Prompts

> **创建于 / Created:** 2026-03-01
> **适用人：** Terry
> **用途：** 当学习一个包含课后作业（homework）的教程项目时，
>           用本指引把所有 prompt 完整收录、命名、建立索引，形成可直接复用的 prompt 库文件。
>
> **触发条件 / When to use:**
> - 项目包含 homework 文件（如 `01-homework.md`、`02-homework.md` 等）
> - 需要把 prompt 整理成可检索、可直接复用的格式
> - 想建立跨项目通用的 prompt 参考库

---

## 一、操作流程（告诉 Claude 这些就够了）/ How to Invoke

**标准调用语句：**
```
学习 [项目名/仓库链接]，读取所有 homework 文件，
按照 create-prompt.md 的规范生成 prompt 收录文件，
存入 p00X-[项目简称]/ 文件夹，并更新 _index.md。
```

**Claude 收到指令后会执行：**
1. 读取 `_HOW-TO-USE.md` 了解当前命名规范
2. 读取 `_index.md` 确认项目 P 编号
3. 获取所有 homework 文件（如通过 `curl` 或 WebFetch 抓取）
4. 读取每个 homework 文件原文（**不总结，不改写**）
5. 按本指引的格式生成 `p00X_homework-prompts_YYYY-MM-DD.md`
6. 更新 `_index.md` 的会话记录和存档文档区块

---

## 二、输出文件规范 / Output File Specification

### 文件命名

```
[P编号]_homework-prompts_YYYY-MM-DD.md
```

示例：`p006_homework-prompts_2026-03-01.md`

存放位置：`p00X-[项目简称]/` 文件夹内

---

### Frontmatter（必填）

> 完整字段规范见 `_HOW-TO-USE.md` 第三节。以下是 homework-prompts 类型的标准写法。

```yaml
---
project: P00X · [项目名]
file_type: homework-prompts
tags: [项目核心技术标签, prompt-collection]
date: YYYY-MM-DD
session_type: 初次学习
source: https://... (项目原始仓库链接)
related: p00X_[概览文件]_YYYY-MM-DD.md
---
```

**`file_type` 固定填 `homework-prompts`**，这是与 `learning-summary`、`archive` 等区分的关键字段，方便后续 grep 过滤。

---

### 文件整体结构

```
# [项目名] — Homework Prompt 索引

> 来源 / 收录范围 / 注意事项

---

## 📋 目录 / Table of Contents
（索引表格，所有 prompt 一览）

---

## 图例 / Legend
（类型说明）

---

## [每课标题]
（课程背景一行说明）

### [编号] · [提示词标题]
**目标:** 一句话说明这个 prompt 做什么
**前置:** （如有）

[原始 prompt 完整文本，放在代码块内]

**验收标准 / 关键体验:** （如有，原文保留）

---

## 附：思考题汇总
（所有课程的思考题集中整理）
```

---

## 三、Prompt 编号规则 / Numbering Convention

| 编号格式 | 含义 |
|---|---|
| `P01-MAIN` | Lesson 01 核心作业（每课只有一个 MAIN）|
| `P01-C1` | Lesson 01 进阶挑战 1（Challenge 1）|
| `P01-C2` | Lesson 01 进阶挑战 2 |
| `P05-MAIN` | Lesson 05 核心作业 |
| … | 以此类推 |

**规则：**
- 课程编号和原始文件保持一致（`01-homework.md` → `P01-*`）
- 跳号正常（如无 03、04 的作业，就跳过 P03-*、P04-*）
- MAIN 只有一个，Challenge 从 C1 开始计数

---

## 四、目录表格格式 / TOC Table Format

```markdown
| 编号 | 提示词名称 | 所属课程 | 类型 | 页内锚点 |
|---|---|---|---|---|
| P01-MAIN | [提示词标题] | Lesson 01 | 核心作业 | [→](#p01-main) |
| P01-C1   | [提示词标题] | Lesson 01 | 进阶挑战 | [→](#p01-c1) |
| P01-C2   | [提示词标题] | Lesson 01 | 进阶挑战（描述性）| [→](#p01-c2) |
```

**类型说明：**
- `核心作业`：每课的主 prompt，完整可执行，包含所有技术细节
- `进阶挑战`：有完整 prompt 文本的扩展作业
- `进阶挑战（描述性）`：原文只有场景说明，无完整 prompt，需自行扩展

---

## 五、关键原则 / Key Principles

### ✅ 必须做

- **完整保留原始 prompt 文字**：不翻译、不改写、不缩略
  - 原文是中文则保持中文；原文有代码块则完整复制
- **放入 markdown 代码块**：所有 prompt 放在 ` ``` ` 内，保持可直接复制
- **类型标注**：在目录和每个 prompt 标题旁标注类型（核心/进阶/描述性）
- **保留验收标准**：每课的验收 checklist 或关键体验描述原文保留
- **保留思考题**：思考题单独汇总到文末，方便复盘时使用

### ❌ 不要做

- 不要把 prompt 总结或改写成"要点"
- 不要省略"看起来重复"的部分（验收标准、常见问题等可省略，但 prompt 本身不可省）
- 不要只收录核心作业而跳过进阶挑战（即使进阶是描述性的也要记录标题和意图）
- 不要把多个 prompt 合并到一个条目里

---

## 六、_index.md 更新要点 / How to Update _index.md

在 P00X 项目的「学习会话」中追加一行：

```markdown
| YYYY-MM-DD | [p00X_homework-prompts_YYYY-MM-DD.md](./p00X-[名称]/p00X_homework-prompts_YYYY-MM-DD.md) | X 课 XX 个提示词完整收录：含目录/命名/索引，原文保留，可直接复用 | 🌱 初次学习 | ✅ 完成 |
```

在「存档文档」中追加：

```markdown
| `docs/lessons/XX-homework.md`（全 X 课） | [p00X_homework-prompts_YYYY-MM-DD.md](./p00X-[名称]/p00X_homework-prompts_YYYY-MM-DD.md) | YYYY-MM-DD |
```

---

## 七、真实示例 / Real Example Reference

**已生成的参考文件：**

- [`p006_homework-prompts_2026-03-01.md`](./p006-cat-cafe/p006_homework-prompts_2026-03-01.md)
  - 来源：`cat-cafe-tutorials` GitHub 项目，Lesson 01/02/05/06/07/08
  - 收录 6 课 24 个提示词，含完整目录索引

可以把这个文件作为格式标杆，在做新项目时对照使用。

---

## 八、快速自检 / Quick Self-Check

生成文件后，对照以下 checklist 验证：

- [ ] 文件名符合 `p00X_homework-prompts_YYYY-MM-DD.md` 规范
- [ ] Frontmatter 已填写（project / tags / date / session_type / related）
- [ ] 顶部目录表格覆盖了所有 prompt（包括进阶挑战）
- [ ] 所有 prompt 原文完整保留在代码块内
- [ ] 类型标注正确（核心作业 / 进阶挑战 / 描述性）
- [ ] 思考题已汇总到文末
- [ ] `_index.md` 已更新（学习会话 + 存档文档两个区块）

---

*v1.0 · 2026-03-01 · 基于 P006 cat-cafe-tutorials 实践总结*
