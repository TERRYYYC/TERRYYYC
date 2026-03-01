---
project: P003 · Engineering Blog Learning
file_type: learning-summary
tags: [claude-code, workflow, research-plan-implement, annotate, methodology]
date: 2026-02-28
session_type: 初次学习
---

# 学习总结 / Learning Summary

> **文件命名 / Filename:** `learning-summary_2026-02-28_2.md`

> ⚠️ **阅读声明 / Reading Disclaimer**
> 本文内容代表作者（Boris Tane）个人观点与经验总结，不作为普适方法论或行业事实。
> 日后借鉴时，以 Terry 或 reviewer 的实际判断与场景需求为优先；作者观点仅供参考与启发。
> *This content represents the author's personal opinion and experience. When applying ideas, Terry's or the reviewer's judgment takes precedence over the author's recommendations.*

---

## 会话元数据 / Session Metadata

| 字段 Field | 内容 Content |
|---|---|
| 日期 Date | 2026-02-28 |
| 时长（估算）Duration | ～ 0.5 小时 |
| 项目名称 Project | P003 · 工程师经验博文学习 Engineering Blog Learning |
| 主题 Topic | AI 时代 Claude Code 的纪律性工作流 / Disciplined Claude Code Workflow |
| 触发原因 Context | 学习顶尖工程师如何高效使用 AI 编程工具，提炼可复用的思维框架 |
| 会话类型 Session Type | ☑ 初次阅读 First Read |
| 关联上次会话 Prev Session | 无 None（P003 新建） |

---

## 一、项目概览 / Project Overview

**项目描述 / What is it:**

P003 是一个持续性的「工程师经验博文」学习项目。来源为行业工程师公开分享的思考总结、工作流方法论、工具使用经验等。目标：提炼精华、建立可复用的心智模型，如有通用 Skill 可直接存档。

**本次阅读来源 / Source:**

| 文章 Article | 链接 Link | 作者 Author |
|---|---|---|
| How I Use Claude Code | https://boristane.com/blog/how-i-use-claude-code/ | Boris Tane |
| The SDLC is Dead（待读） | https://boristane.com/blog/the-software-development-lifecycle-is-dead/ | Boris Tane |

> ⚠️ 注：第一篇 SDLC 文章因网络限制暂未读取，待下次会话补充。

---

## 二、本次学习内容 / What I Studied Today

### 2.1 阅读的文档 / Documents Read

- [x] `How I Use Claude Code` (Boris Tane, 2026-02-10) — 作者使用 Claude Code 约 9 个月后沉淀的完整工作流，核心是「先规划再执行」的纪律性流程

### 2.2 核心概念 / Core Concepts

**概念 1：纪律性分离原则 / Disciplined Separation Principle**

永远不让 Claude 写代码，直到你审阅并批准了一份书面计划。规划（thinking）和执行（typing）必须物理分离。这不是建议，是工作流的基石。

Never let Claude write code until you've reviewed and approved a written plan. Planning (thinking) and execution (typing) must be physically separated. This is not a suggestion — it's the foundation of the entire workflow.

**概念 2：书面制品作为审查面 / Written Artifacts as Review Surfaces**

`research.md` 和 `plan.md` 不只是"让 Claude 做作业"——它们是你的审查面（review surface）。如果 Claude 对系统的理解是错的，你在计划阶段就能发现并纠正，而不是在实现阶段付出巨大代价。这是 AI 辅助编码中最昂贵的失败模式：实现了一个孤立可运行、但破坏周边系统的功能。

`research.md` and `plan.md` are not homework — they are your review surfaces. If Claude misunderstands the system, you catch it in planning, not after 15 minutes of wrong implementation. The most expensive failure mode in AI-assisted coding: implementations that work in isolation but break the surrounding system.

**概念 3：标注循环 / Annotation Cycle**

最有特色的部分。计划写好后，直接在文档里内联添加注释（行内批注），然后把 Claude 送回文档：「我在文档里添了一些注释，请处理所有注释并更新文档，先不要实现。」循环 1-6 次。这是注入「我的判断」到计划中的核心机制——Claude 不了解我的产品优先级、用户痛点、和我愿意接受的工程权衡。

The most distinctive part of the workflow. After Claude writes the plan, add inline notes directly in the document, then send Claude back: "I added a few notes, address all notes and update the document accordingly. don't implement yet." Repeat 1-6 times. This is the core mechanism for injecting your own judgment — Claude doesn't know your product priorities, user pain points, or the engineering trade-offs you're willing to make.

**概念 4：标准实现提示词 / Standard Implementation Prompt**

```
implement it all. when you're done with a task or phase, mark it as completed in
the plan document. do not stop until all tasks and phases are completed. do not
add unnecessary comments or jsdocs, do not use any or unknown types. continuously
run typecheck to make sure you're not introducing new issues.
```

每一个词都是精心设计的：
- "implement it all"：不要自作主张地挑选任务
- "mark it as completed"：计划文档是进度唯一真相来源
- "do not stop"：不要中途暂停等确认
- "no unnecessary comments"：保持代码整洁
- "no any types"：保持严格类型
- "continuously run typecheck"：早发现，而不是最后才发现

**概念 5：共享可变状态 / Shared Mutable State**

Plan.md 充当了人和 Claude 之间的「共享可变状态」。你可以按自己的节奏思考，精确地标注哪里有问题，并写下纠正。这和通过聊天信息来引导实现根本不同——计划是完整的结构化规范，可以整体审阅；聊天记录需要翻滚才能重建决策。

Plan.md acts as shared mutable state between human and Claude. You can think at your own pace, annotate precisely where something is wrong. This is fundamentally different from steering through chat — a plan is a structured, complete specification; a chat is something you'd have to scroll through to reconstruct decisions.

### 2.3 关键流程 / Key Workflow

```
1. Research Phase（研究阶段）
   └─ 深度阅读相关代码 → 写入 research.md → 人工审阅，验证 Claude 的理解

2. Planning Phase（规划阶段）
   └─ 生成 plan.md（含代码片段、文件路径、权衡分析）

3. Annotation Cycle（标注循环） × 1-6 次
   └─ 人在编辑器里写内联批注 → Claude 处理批注更新计划 → "don't implement yet"

4. Todo List（任务清单）
   └─ 在计划中添加详细的分阶段任务列表 → 实现时逐一标记完成

5. Implementation（实现）
   └─ 标准提示词一键触发 → Claude 持续执行直到完成 → 自动类型检查

6. Feedback & Iterate（反馈迭代）
   └─ 简短纠正（一句话）→ 截图辅助视觉问题 → 引用已有代码作参考 → 错了就 revert + 重新聚焦
```

### 2.4 重要提示词片段 / Key Prompts

**Research（研究阶段）:**
```
read this folder in depth, understand how it works deeply, what it does and all
its specificities. when that's done, write a detailed report of your learnings
and findings in research.md
```
> 关键词："deeply"、"in great details"、"intricacies"——没有这些词，Claude 会浏览而不是深读。

**Annotation Return（标注返回）:**
```
I added a few notes to the document, address all the notes and update the
document accordingly. don't implement yet
```

**Todo List Request（请求任务清单）:**
```
add a detailed todo list to the plan, with all the phases and individual tasks
necessary to complete the plan - don't implement yet
```

**Revert & Re-scope（回滚重聚焦）:**
```
I reverted everything. Now all I want is to make the list view more minimal — nothing else.
```

---

## 三、洞察与收获 / Insights & Takeaways

### 3.1 最重要的 3 个发现 / Top 3 Insights

1. **"实现应该是无聊的"（Implementation should be boring）** ——如果实现阶段需要创造性决策，说明规划阶段不够彻底。这是一个可用来自我评估规划质量的金标准。

   "If implementation requires creative decision-making, planning wasn't thorough enough." — This is a gold standard for evaluating the quality of your planning.

2. **最贵的失败模式不是语法错误，而是「孤立可运行、破坏周边」** ——一个忽略了已有缓存层的函数、一个不符合 ORM 约定的数据迁移、一个在别处已有逻辑的 API。Research 阶段专门用来防止这种失败。

   The most expensive failure mode is not syntax errors — it's "works in isolation, breaks the surrounding system." The research phase exists specifically to prevent this.

3. **标注循环是注入人类判断的核心机制** ——Claude 可以理解代码、提出方案、写出实现，但它不知道你的产品方向、用户痛点和工程权衡标准。Annotation cycle 是你作为架构师在场的体现。

   The annotation cycle is the core mechanism for injecting human judgment. Claude can understand code and write implementations, but it doesn't know your product priorities. The annotation cycle is where you, as the architect, show up.

### 3.2 让我意外的地方 / What Surprised Me

- **作者不用 Claude Code 内置的 plan mode**，而是用自定义 `.md` 文件代替——因为内置 plan mode 给的控制权不足，自己的 markdown 文件可以在编辑器中操作、添加注释、永久保存。

  The author explicitly says "Claude Code's built-in plan mode sucks" and replaces it with custom `.md` files — more control, editor-editable, persistent as real project artifacts.

- **作者不经历长对话后性能下降**（很多人反映 50% context window 后质量明显下降）——作者认为是因为 Claude 在研究和标注阶段已经深度建立了对系统的理解，context 被高质量内容填满了。

  Author doesn't experience the performance degradation at 50% context window that others report — attributes this to Claude having built deep understanding during research and annotation phases; context is filled with high-quality, relevant content.

- **错了就 revert，而不是 patch**——这和很多人的直觉相反（总是想"再修一下"），但作者指出重新聚焦范围的结果几乎总是比在错误基础上修补更好。

  When something goes wrong, revert + re-scope rather than patch. Counter-intuitive but almost always produces better results than incrementally fixing a bad approach.

### 3.3 与已有知识的连接 / Connections to Prior Knowledge

- **与 P001 Skill 系统的共鸣**：Cowork 模式中的 skill 也遵循「先读 SKILL.md 再行动」——这和 research.md 优先的原则一致。两者都强调：没有充分的上下文理解，执行必然出错。

  Parallels with P001 Skill system: Cowork mode also mandates "read SKILL.md before acting" — same principle as research.md first. Both emphasize: without sufficient context understanding, execution will fail.

- **与 P002 OpenClaw 的共鸣**：OpenClaw 大量使用"引用现有实现"作为类型共享机制（TypeBox schema 共享），Tane 的工作流里同样强调「引用已有代码模式」作为最有效的沟通方式（"this table should look exactly like the users table"）。

  Parallels with P002 OpenClaw: OpenClaw uses shared TypeBox schemas as reference implementations. Tane's workflow similarly emphasizes referencing existing code patterns as the most efficient communication ("this should look exactly like X").

- **「输出驱动学习」（来自 2026-02-26）**：每次学习必须有文件产出。Tane 的 research.md 和 plan.md 都是这一原则在工程实践中的体现——没有书面输出的 AI 对话等于没有留下任何可复查的资产。

  "Output-driven learning" (from 2026-02-26 session): Every learning session must produce a file artifact. Tane's research.md and plan.md are this principle applied to engineering practice — AI conversations without written output leave no reviewable assets.

---

## 四、安全与风险观察 / Security & Risk Notes

| 观察 Observation | 风险等级 | 参考 |
|---|---|---|
| 文中未涉及安全相关工作流 | — | — |

**是否需要进行完整安全扫描？** ☑ 否 No

---

## 五、问题与待解答 / Open Questions

- [ ] **第一篇文章待读**：SDLC is Dead — https://boristane.com/blog/the-software-development-lifecycle-is-dead/ ，需用户在浏览器打开后复制内容
- [ ] **作者如何处理多人协作场景**？plan.md 方法论在团队里如何落地？（博文未提及）
- [ ] **标注循环的"最优轮数"** 如何判断？什么信号表明计划已经足够好、可以进入实现？
- [ ] **research.md 的粒度**：对于一个大型系统（数十万行代码），如何界定 research 的边界？

---

## 六、下一步计划 / Next Steps

| 优先级 | 行动 | 状态 |
|---|---|---|
| 高 High | 读第一篇博文（SDLC is Dead）——用户复制内容后继续 | ⬜ 待做 |
| 高 High | **实践验证**：在下一个编程任务中，严格按 Research → Plan → Annotate → Implement 流程执行，观察效果 | ⬜ 待做 |
| 中 Medium | 将标准实现提示词保存为 Cowork 可复用的 Skill 模版 | ⬜ 待做 |
| 低 Low | 查找 Boris Tane 其他文章（如有），纳入 P003 学习清单 | ⬜ 待做 |

---

## 七、可直接借鉴的提示词清单 / Reusable Prompt Templates

> 以下提示词已在实际项目中验证，可直接复用：

### 📋 研究阶段
```
read [folder/file] in depth, understand how it works deeply, what it does and all
its specificities. when that's done, write a detailed report of your learnings
and findings in research.md
```

### 📝 规划阶段（带参考实现）
```
I want to build [feature]. Here's a reference implementation: [paste code].
Write a detailed plan.md explaining how we can adopt a similar approach in our codebase.
Read source files before suggesting changes, base the plan on the actual codebase.
```

### ✏️ 标注返回
```
I added a few notes to the document, address all the notes and update the
document accordingly. don't implement yet
```

### ✅ 任务清单
```
add a detailed todo list to the plan, with all the phases and individual tasks
necessary to complete the plan - don't implement yet
```

### 🚀 标准实现提示词（核心）
```
implement it all. when you're done with a task or phase, mark it as completed
in the plan document. do not stop until all tasks and phases are completed.
do not add unnecessary comments or jsdocs, do not use any or unknown types.
continuously run typecheck to make sure you're not introducing new issues.
```

### 🔄 回滚重聚焦
```
I reverted everything. Now all I want is to [narrowly scoped task] — nothing else.
```

---

## 八、本次生成的文件 / Files Generated This Session

| 文件名 | 类型 | 描述 |
|---|---|---|
| `learning-summary_2026-02-28_2.md` | 总结 Summary | 本文件，P003 首次会话 |

---

## 九、关键词 / Keywords

`Claude Code` `纪律性工作流` `plan.md` `research.md` `标注循环 annotation cycle` `AI辅助编码` `规划执行分离` `共享可变状态` `标准实现提示词` `Boris Tane`

---

*P003 · 工程师经验博文学习 · 会话 1 of n*
*2026-02-28 · claude-sonnet-4-6*
