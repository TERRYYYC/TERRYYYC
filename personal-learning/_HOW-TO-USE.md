# 使用指南：项目学习知识库 / How to Use: Project Learning Knowledge Base

> **创建于 / Created:** 2026-02-27
> **适用人：** Terry
> **目标：** 用最小摩擦力，把每次项目学习沉淀成可复用的知识资产

---

## 一、文件结构总览 / File Structure Overview

```
personal-learning/
│
├── _index.md                          ← ⭐ 主入口，每次从这里开始
├── _HOW-TO-USE.md                     ← 本文件
│
├── template_learning-session.md       ← 📝 每次新会话的起点
│
├── [模版库 Templates]
│   └── skill-security-scan-template_YYYY-MM-DD.md
│
├── [学习总结 Session Summaries]        ← 按日期命名
│   ├── learning-summary_2026-02-26.md
│   └── learning-summary_YYYY-MM-DD.md
│
└── [文档存档 Doc Archives]             ← 原始文档快照
    └── archive_[项目]_[文档]_YYYY-MM-DD.md
```

**文件名前缀含义 / Prefix Meaning:**

| 前缀 | 含义 |
|---|---|
| `_` | 系统文件，不要删除（`_index.md`, `_HOW-TO-USE.md`）|
| `template_` | 可复用模版，不填内容，复制后使用 |
| `learning-summary_` | 单次学习会话总结 |
| `archive_` 或 `[项目]-[文档]-archive_` | 原始文档快照存档 |
| `skill-security-scan-template_` | Skill 安全扫描专用模版 |

---

## 二、标准工作流 / Standard Workflow

### 🟢 开始新学习会话

**第 1 步：告诉 Claude**
```
「我想学习 [项目名]，从 [具体文档或功能] 开始。」
```

**第 2 步：Claude 会做**
- 检查 `_index.md` 确认该项目是否已有记录
- 阅读目标文档（SKILL.md 或其他）
- 帮你生成当日学习总结

**第 3 步：你需要确认**
- 检查生成的总结是否准确
- 在 `_index.md` 中确认新记录已追加

---

### 🔁 继续已有项目的学习

**告诉 Claude：**
```
「继续学习 P001（skill and e-evo），上次停在了 agents/grader.md，今天我想搞清楚断言评估的具体逻辑。」
```

Claude 会：
1. 读取 `_index.md` 中 P001 的历史记录
2. 读取上次 `learning-summary_*.md` 了解已有知识
3. 阅读目标文档，生成本次补充总结

---

### 🔍 做 Skill 安全扫描

**告诉 Claude：**
```
「帮我扫描一下 [skill名称]，用安全扫描模版检查。」
```

Claude 会：
1. 读取 `skill-security-scan-template_*.md`
2. 逐项检查目标 Skill
3. 输出填写好的扫描报告（新建文件 `security-scan_[skill]_YYYY-MM-DD.md`）
4. 在 `_index.md` 的对应项目下追加扫描记录

---

### 📦 存档新文档

当你读了一个重要的原始文档（SKILL.md、README、设计文档等），让 Claude 存档：
```
「把今天读的 [文档路径] 存档起来，打上今天的时间戳和项目标签。」
```

输出文件命名：`archive_[项目简称]_[文档名]_YYYY-MM-DD.md`

---

## 三、命名规范速查 / Naming Quick Reference

| 文件类型 | 命名格式 | 示例 |
|---|---|---|
| 学习总结 | `learning-summary_YYYY-MM-DD.md` | `learning-summary_2026-03-01.md` |
| 同日多次 | `learning-summary_YYYY-MM-DD_2.md` | `learning-summary_2026-03-01_2.md` |
| 文档存档 | `archive_[项目]_[文档]_YYYY-MM-DD.md` | `archive_p001_grader_2026-03-01.md` |
| 安全扫描报告 | `security-scan_[skill]_YYYY-MM-DD.md` | `security-scan_xlsx_2026-03-01.md` |
| 新模版 | `template_[用途].md` | `template_api-research.md` |

---

## 四、`_index.md` 维护规则 / Index Maintenance Rules

`_index.md` 是整个系统的核心，请保持它的准确性：

**每次新会话后必须更新 / Always update after each session:**
- [ ] 在「会话时间线」追加一行
- [ ] 在对应项目的「学习会话」追加一行
- [ ] 如有新存档文档，在「存档文档」追加一行
- [ ] 勾选已完成的「待探索」项目，添加新的待探索项
- [ ] 更新顶部「最后更新」日期和统计数字

**添加新项目时 / When adding a new project:**
- 分配新的 P 编号（P002、P003…）
- 在「项目学习记录」新建完整章节
- 更新「知识图谱」中的关联关系

---

## 五、让 Claude 的提示词示例 / Example Prompts for Claude

> 通用公式 / Universal Formula：**[情境] + [目标] + [聚焦（可选）] + [输出]**
> 括号里可选的部分省略也没关系，Claude 会根据系统文件自动判断该做什么。

### 场景 A：第一次学一个新项目

```
学习 [项目路径 或 描述]，阅读 [具体文档，如 SKILL.md / README]，
帮我生成今天的学习总结和文档存档，并更新 _index.md。
```

**实际例子：**
```
学习 /mnt/.skills/skills/docx，阅读 SKILL.md，
帮我生成今天的学习总结和文档存档，并更新 _index.md。
```

---

### 场景 B：继续已有项目

```
继续 [P001 / 项目名] 的学习，上次停在 [上次进度]，
今天重点看 [目标文档或功能]，生成补充总结并更新索引。
```

**实际例子：**
```
继续 P001 skill-and-e-evo，上次停在 skill-creator 整体架构，
今天重点搞清楚 agents/grader.md 的断言评估逻辑，生成补充总结并更新索引。
```

---

### 场景 C：做 Skill 安全扫描

```
用安全扫描模版检查 [skill路径]，生成扫描报告并存档。
```

**实际例子：**
```
用安全扫描模版检查 /mnt/.skills/skills/xlsx，生成扫描报告并存档。
```

---

### 场景 D：快速唤醒上次记忆，然后继续工作

```
先读 _index.md 里 [P001] 的记录帮我快速回顾，然后 [具体任务]。
```

隔了几天回来时特别有用——Claude 会先给你两三句"上次你学到了……"的摘要，再开工。

---

### 场景 E：深入某个模块（原有示例）

```
继续 P001 的学习，今天重点看 agents/grader.md，
帮我把关键内容加入今天的 learning-summary，并更新待探索清单。
```

### 场景 F：比较两次学习进展

```
对比我在 P001 的两次学习记录（2026-02-27 和今天），
看看理解有没有加深，有没有之前误解的地方。
```

---

> **💡 小技巧：** 每次新会话开头加一句「先读一下 `_index.md`」，Claude 就能立刻进入你的知识库上下文，而不是从零开始。这是成本最低、效果最好的连续性保障。

---

## 六、设计原则 / Design Principles

这套系统遵循以下原则，请在使用中保持：

**1. 最小摩擦 / Minimum Friction**
任何记录动作超过 3 分钟就会被放弃。所以模版要简，能自动化的交给 Claude。

**2. 原文存档 + 自己的理解分离**
`archive_` 文件保持原文，不加评论。理解和洞察只写在 `learning-summary_` 里。这样原文永远可查，自己的认知也独立演化。

**3. 索引优先 / Index First**
每次开始前先看 `_index.md`，避免重复学习已掌握内容，也能快速定位上次进度。

**4. 输出驱动 / Output-Driven**
每次学习结束必须有文件产出（哪怕只是一个问题列表）。没有输出的学习很难内化。

**5. 关联成网 / Build a Web**
在 `_index.md` 的知识图谱中维护概念间的关联，帮助形成体系而非孤立知识点。

---

*本文档随系统演进更新 | This document evolves with the system*
*v1.0 · 2026-02-27*
