# 项目学习主索引 / Master Learning Index

> **最后更新 / Last Updated:** 2026-02-27
> **总会话数 / Total Sessions:** 2
> **已学项目数 / Projects Studied:** 1
> **生成文件数 / Files Generated:** 6

此文件是整个学习知识库的目录。每次学习会话结束后更新。
This file is the table of contents for the entire learning knowledge base. Update after each session.

---

## 📁 系统文件 / System Files

| 文件 File | 用途 Purpose | 最后更新 |
|---|---|---|
| `_index.md` | 本文件，主索引 This file | 2026-02-27 |
| `_HOW-TO-USE.md` | 使用指南 Usage guide | 2026-02-27 |
| `template_learning-session.md` | 学习会话模版 Session template | 2026-02-27 |
| `skill-security-scan-template_2026-02-27.md` | Skill 安全扫描检查清单 | 2026-02-27 |

---

## 📚 项目学习记录 / Project Learning Records

### P001 · skill and e-evo

| 字段 | 内容 |
|---|---|
| 项目 ID | P001 |
| 项目名称 | skill and e-evo |
| 项目路径 | `/mnt/.projects/019c9cf4-4af3-762e-9dde-95fa6668f9be` |
| 项目库链接 | _(内部 Cowork 项目)_ |
| 主要主题 | Claude Skill 系统设计、评估方法论、安全原则 |
| 首次学习 | 2026-02-27 |
| 最近更新 | 2026-02-27 |
| 状态 Status | 🟡 进行中 In Progress |

**学习会话 / Sessions:**

| 日期 | 文件 | 主题 | 状态 |
|---|---|---|---|
| 2026-02-27 | `learning-summary_2026-02-27.md` | skill-creator 架构 + Eval Loop + 安全原则 | ✅ 完成 |

**存档文档 / Archived Docs:**

| 原始文档 | 存档文件 | 日期 |
|---|---|---|
| `skill-creator/SKILL.md` | `skill-creator-SKILL-archive_2026-02-27.md` | 2026-02-27 |

**待探索 / Still To Study:**

- [ ] `agents/grader.md`
- [ ] `agents/comparator.md`
- [ ] `agents/analyzer.md`
- [ ] `references/schemas.md`
- [ ] `run_loop.py` 实现细节
- [ ] 实践：用 skill-creator 创建一个新 skill

---

_(新项目在此继续添加 / Add new projects below)_

---

## 🗓️ 会话时间线 / Session Timeline

| 日期 | 项目 | 主题摘要 | 文件 |
|---|---|---|---|
| 2026-02-26 | _(成人学习方法论)_ | 四轮学习法、费曼反馈、复盘框架 | `2026-02-26.md` |
| 2026-02-27 | P001 skill and e-evo | skill-creator 架构、Eval Loop、安全扫描模版生成 | `learning-summary_2026-02-27.md` |

---

## 🗂️ 可复用模版库 / Reusable Template Library

| 模版文件 Template | 适用场景 When to Use | 版本 |
|---|---|---|
| `template_learning-session.md` | 每次新学习会话开始时 | v1.0 |
| `skill-security-scan-template_2026-02-27.md` | 审查任何新 Skill 的安全性 | v1.0 |

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
```

---

*更新说明：每次学习会话结束后，在「项目学习记录」和「会话时间线」中追加记录。*
*Update note: After each session, append to "Project Learning Records" and "Session Timeline".*
