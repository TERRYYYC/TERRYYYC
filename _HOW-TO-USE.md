# 使用指南：项目学习知识库 / How to Use: Project Learning Knowledge Base

> **创建于 / Created:** 2026-02-27
> **更新于 / Updated:** 2026-03-01
> **适用人：** Terry
> **目标：** 用最小摩擦力，把每次项目学习沉淀成可复用的知识资产

---

## 一、文件结构总览 / File Structure Overview

```
personal-learning/
│
├── _index.md                          ← ⭐ 主入口，每次从这里开始
├── _HOW-TO-USE.md                     ← 本文件（含 Frontmatter 规范）
├── _quick-check.md                    ← 会话结束前快速核查表
├── _personal-learning-log.md          ← 系统变更日志
├── _idiot-moments.md                  ← 🔍 失误复盘日志（错误模式 + 防护原则）
├── create-prompt.md                   ← 🔧 课后作业 Prompt 收录操作指引
│
├── _templates/                        ← 📝 可复用模版（不直接填写，复制后使用）
│   ├── template_learning-session.md   ← 含 Frontmatter 模版头
│   └── skill-security-scan-template_YYYY-MM-DD.md
│
├── _misc/                             ← 📦 杂项笔记（不归属特定项目）
│   └── misc_[描述]_YYYY-MM-DD.md
│
├── p001-skill-evo/                    ← 🗂️ 项目文件夹（P 编号 + 简称）
│   ├── p001_skill-creator_2026-02-27.md
│   └── p001_skill-creator-archive_2026-02-27.md
│
├── p002-openclaw/
│   └── p002_frontend-overview_2026-02-28.md
│
├── p003-engineering-blog/
│   ├── p003_boris-workflow_2026-02-28.md
│   ├── p003_sdlc-is-dead_2026-02-28.md
│   └── p003_agent-teams-reverse_2026-03-01.md
│
├── p004-swarm-ide/
│   ├── p004_core-architecture_2026-03-01.md
│   ├── p004_local-source-deep-dive_2026-03-01.md
│   └── p004_frontend-architecture_2026-03-01.md
│
├── p005-deerflow/
│   └── p005_frontend-overview_2026-03-01.md
│
└── p006-cat-cafe/
    ├── p006_lesson00-08-overview_2026-03-01.md
    └── p006_homework-prompts_2026-03-01.md
```

---

## 二、命名规范 / Naming Conventions

### 文件夹命名 / Folder Naming

```
[P编号]-[项目简称]/
```

示例：`p001-skill-evo/`、`p004-swarm-ide/`

规则：P编号保证有序，简称短且可搜索，用连字符 `-` 分词。

---

### 文件命名 / File Naming

**学习总结（主要产出）:**

```
[P编号]_[模块/主题]_YYYY-MM-DD.md
```

示例：`p004_core-architecture_2026-03-01.md`

同项目同天多次学习，在模块名后加描述区分，不用数字序号：

```
p004_core-architecture_2026-03-01.md
p004_local-source-deep-dive_2026-03-01.md   ← 同天续集，描述区分
p004_frontend-architecture_2026-03-01.md
```

**文档存档:**

```
[P编号]_[文档名]-archive_YYYY-MM-DD.md
```

示例：`p001_skill-creator-archive_2026-02-27.md`

**安全扫描报告:**

```
[P编号]_security-scan_[skill名]_YYYY-MM-DD.md
```

**模版文件（放 `_templates/`，不直接使用）:**

```
template_[用途].md
```

**杂项笔记（放 `_misc/`）:**

```
misc_[描述]_YYYY-MM-DD.md
```

---

### 完整前缀含义速查 / Prefix Quick Reference

| 前缀/格式 | 含义 |
|---|---|
| `_index.md` / `_HOW-TO-USE.md` | 系统文件，不要删除 |
| `_templates/` | 模版库文件夹 |
| `_misc/` | 杂项笔记文件夹 |
| `p00X-[名称]/` | 项目文件夹 |
| `p00X_[模块]_日期.md` | 学习总结（主要产出） |
| `p00X_[文档]-archive_日期.md` | 原始文档快照 |
| `p00X_security-scan_[skill]_日期.md` | Skill 安全扫描报告 |
| `misc_[描述]_日期.md` | 不归属特定项目的杂项笔记 |

---

## 三、学习总结文件 Frontmatter / Session File Frontmatter

**每个笔记文件（学习总结、存档、homework-prompts 等）开头必须加入以下 YAML 元数据块。**
这是新会话中 Claude 定位上下文、跨文件搜索的核心依据。

### 完整字段模版

```yaml
---
project: P004 · Swarm-IDE
file_type: learning-summary
tags: [multi-agent, promise-deferred, event-bus, typescript]
date: 2026-03-01
session_type: 深度研究
source: https://github.com/chmod777john/swarm-ide
related: p004_core-architecture_2026-03-01.md
---
```

### 字段说明

| 字段 | 必填 | 说明 |
|---|---|---|
| `project` | ✅ 必填 | `P编号 · 项目名`，与 `_index.md` 项目标题保持一致 |
| `file_type` | ✅ 必填 | 见下方类型表 |
| `tags` | ✅ 必填 | 核心主题关键词（3–8 个），供跨项目 grep 搜索 |
| `date` | ✅ 必填 | 学习/创建日期，格式 `YYYY-MM-DD` |
| `session_type` | ✅ 必填 | 见进度阶段说明（初次学习/深度研究/深度复现/实践验证） |
| `source` | 外部项目必填 | 原始仓库或文章 URL；内部项目（无公开链接）可省略 |
| `related` | 可选 | 同项目的前一次文件或强关联文件；无则省略整行 |

### file_type 取值

| 值 | 含义 |
|---|---|
| `learning-summary` | 学习总结（主要产出） |
| `archive` | 原始文档快照存档 |
| `homework-prompts` | 课后作业提示词收录（create-prompt.md 工作流产出） |
| `scan-report` | Skill 安全扫描报告 |
| `misc` | 不归属特定项目的杂项笔记 |

### 最简版（只有必填字段）

```yaml
---
project: P001 · skill and e-evo
file_type: learning-summary
tags: [claude-skill, eval-loop]
date: 2026-02-27
session_type: 初次学习
---
```

---

## 四、进度阶段说明 / Progress Stage Guide

| 阶段 | 标签 | 触发条件 |
|---|---|---|
| 初次学习 | 🌱 | 首次阅读文档，建立基本认知 |
| 深度研究 | 🔍 | 深入源码或细节，解决具体疑问 |
| 深度复现 | 🔁 | 亲手复现核心逻辑或运行代码 |
| 实践验证 | ✅ | 把学到的模式用于真实任务 |
| 高频参考 | 📌 | 被多个其他项目引用，是核心知识锚点 |

---

## 五、标准工作流 / Standard Workflow

### 🟢 开始新学习会话

**第 1 步：告诉 Claude**
```
「我想学习 [项目名]，从 [具体文档或功能] 开始。」
```

**第 2 步：Claude 会做**
- 检查 `_index.md` 确认该项目是否已有记录
- 阅读目标文档（SKILL.md 或其他）
- 帮你生成学习总结，存入对应项目文件夹

**第 3 步：你需要确认**
- 检查生成的总结是否准确
- 在 `_index.md` 中确认新记录已追加

---

### 🔁 继续已有项目的学习

**告诉 Claude：**
```
「继续 P001（skill and e-evo）的学习，上次停在了 agents/grader.md，
今天我想搞清楚断言评估的具体逻辑。」
```

Claude 会：
1. 读取 `_index.md` 中 P001 的历史记录和文件链接
2. 读取上次总结文件了解已有知识
3. 阅读目标文档，生成本次补充总结，存入 `p001-skill-evo/`

---

### 🔍 做 Skill 安全扫描

**告诉 Claude：**
```
「帮我扫描一下 [skill名称]，用安全扫描模版检查。」
```

Claude 会：
1. 读取 `_templates/skill-security-scan-template_*.md`
2. 逐项检查目标 Skill
3. 输出扫描报告，存入对应项目文件夹

---

### 📦 存档新文档

```
「把今天读的 [文档路径] 存档起来，打上今天的时间戳和项目标签。」
```

输出文件命名：`[P编号]_[文档名]-archive_YYYY-MM-DD.md`，存入对应项目文件夹。

---

## 六、让 Claude 的提示词示例 / Example Prompts for Claude

> 通用公式 / Universal Formula：**[情境] + [目标] + [聚焦（可选）] + [输出]**

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

---

### 场景 C：做 Skill 安全扫描

```
用安全扫描模版检查 [skill路径]，生成扫描报告并存档。
```

---

### 场景 D：快速唤醒上次记忆，然后继续工作

```
先读 _index.md 里 [P001] 的记录帮我快速回顾，然后 [具体任务]。
```

隔了几天回来时特别有用——Claude 会先给你两三句"上次你学到了……"的摘要，再开工。

---

### 场景 E：跨项目对比

```
对比 P003（工程师经验博文）和 P004（Swarm-IDE）中的多 Agent 通信机制，
看看有哪些可以迁移到我的新项目里。
```

---

## 七、`_index.md` 维护规则 / Index Maintenance Rules

**每次新会话后必须更新 / Always update after each session:**
- [ ] 在「会话时间线」追加一行（含文件链接和进度阶段）
- [ ] 在对应项目的「学习会话」表格追加一行
- [ ] 如有新存档文档，在「存档文档」追加一行
- [ ] 勾选已完成的「待探索」项目，添加新的待探索项
- [ ] 更新顶部「最后更新」日期和统计数字

**添加新项目时 / When adding a new project:**
- 分配新的 P 编号（P007、P008…）
- 创建对应项目文件夹 `p007-[名称]/`
- 在 `_index.md` 的「项目学习记录」新建完整章节
- 更新「知识图谱」中的关联关系

---

## 八、设计原则 / Design Principles

**1. 最小摩擦 / Minimum Friction**
任何记录动作超过 3 分钟就会被放弃。所以模版要简，能自动化的交给 Claude。

**2. 原文存档 + 自己的理解分离**
`[P编号]_*-archive_*.md` 保持原文，不加评论。理解和洞察只写在学习总结里。

**3. 索引优先 / Index First**
每次开始前先看 `_index.md`，避免重复学习已掌握内容，也能快速定位上次进度。

**4. 输出驱动 / Output-Driven**
每次学习结束必须有文件产出（哪怕只是一个问题列表）。没有输出的学习很难内化。

**5. 关联成网 / Build a Web**
在 `_index.md` 的知识图谱中维护概念间的关联，帮助形成体系而非孤立知识点。

**6. 进度阶段可见 / Progress Visibility**
用阶段标签（🌱/🔍/🔁/✅）区分"看过"和"真的掌握"，避免自我欺骗。

---

> **💡 小技巧：** 每次新会话开头加一句「先读一下 `_index.md`」，Claude 就能立刻进入你的知识库上下文，而不是从零开始。这是成本最低、效果最好的连续性保障。

---

*本文档随系统演进更新 | This document evolves with the system*
*v2.1 · 2026-03-01（Frontmatter 规范扩展：新增 file_type + source 字段；模版同步更新；全库 11 个历史文件补充 frontmatter）*
