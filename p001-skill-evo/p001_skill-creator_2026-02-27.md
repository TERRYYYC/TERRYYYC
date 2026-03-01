---
project: P001 · skill and e-evo
file_type: learning-summary
tags: [claude-skill, skill-creator, eval-loop, security, cowork]
date: 2026-02-27
session_type: 初次学习
---

# 今日学习总结 / Daily Learning Summary

> **日期 / Date:** 2026-02-27 (Friday)
> **项目 / Project:** `skill and e-evo`
> **学习主题 / Topic:** Claude Skill 系统设计、评估方法论、Skill 安全原则

---

## 一、今日学习核心内容 / Core Learning Today

### 1. Claude Skill 系统架构理解

今天深入阅读了 `skill-creator` 的 `SKILL.md`，理解了 Claude Skill 的完整生命周期：

**Skill 的核心结构 / Skill Anatomy:**
- `SKILL.md`（必须）：YAML frontmatter（name、description）+ Markdown 指令正文
- `scripts/`：可执行脚本，用于确定性/重复性任务
- `references/`：按需加载的文档上下文
- `assets/`：模版、图标、字体等输出用资源

**三层上下文加载（Progressive Disclosure）:**
1. Metadata（name + description）：始终在上下文中，约 100 词
2. SKILL.md 正文：技能触发时加载，建议 ≤ 500 行
3. Bundled resources：按需加载，无限制

**关键洞察 / Key Insight:** Description 字段是触发机制的核心。Claude 存在"欠触发"倾向，因此 description 应略带"推动性"，明确列出触发场景。

---

### 2. Skill 评估方法论（Eval Loop）

**完整循环 / Full Loop:**
```
设计意图 → 撰写 SKILL.md → 生成测试用例（2-3个）
→ 并行运行（with-skill + baseline）→ 抓取 timing 数据
→ 评分（grader subagent）→ 聚合基准（benchmark.json）
→ 分析师过滤（analyzer.md）→ 查看器（generate_review.py）
→ 用户反馈 → 改进 Skill → 迭代
```

**评估要点 / Eval Key Points:**
- with-skill 和 baseline 必须在**同一轮**并行启动
- Cowork 环境中：使用 `--static` 生成 HTML 而非启动服务器
- 先让人类看结果（generate_review.py），再自己评估
- timing.json 必须在 subagent 通知到达时立即保存

---

### 3. Skill 写作原则

**无惊喜原则（Principle of Lack of Surprise）:**
Skill 内容不能让用户感到意外。禁止包含恶意代码、利用漏洞代码、设计为误导用户的内容。

**写作风格原则:**
- 用命令式（imperative form）写指令
- 解释"为什么"而非只写"必须做什么"
- 避免过度刚性的 ALWAYS/NEVER（有理由时才用）
- 从测试案例中发现重复工作 → 提取为脚本放入 scripts/

**描述优化（Description Optimization）:**
- 生成 20 条评估查询（8-10 触发 / 8-10 不触发）
- 用 `run_loop.py` 自动迭代优化
- 选 test score 最优（非 train score），避免过拟合

---

### 4. 成人自主学习方法论（来自 2026-02-26 笔记）

复习上次积累的学习框架，关键要点：

- **目标拆解**：把模糊的"学会 XX"变成可执行、可验证的小任务
- **固定节奏**：用刚性时间规则替代"有空就学"
- **即时反馈**：费曼学习法（合上资料，用自己的话重述）
- **迭代复盘**：每周 15 分钟，回答 3 个问题（完成率/卡点/下周调整）
- **四轮学习法**：扫盲→靶向攻坚→补全细节→输出固化
- **主动搭建反馈网络**：找职场老师，带方案去请教而非空问"怎么学"

**与今日 Skill 学习的映射：**
- 扫盲（第一轮）= 阅读 SKILL.md 全文，建立整体框架
- 靶向攻坚（第二轮）= 重点理解 Eval Loop 和安全原则
- 输出固化（第四轮）= 本文件 + 安全扫描模版 + SKILL 存档

---

## 二、关键知识点速查 / Quick Reference

| 知识点 | 要点 |
|---|---|
| Skill 触发机制 | description 字段；推动性描述；复杂任务比简单任务更易触发 |
| Eval 并行原则 | with-skill + baseline 同一轮启动 |
| Cowork 查看器 | `--static <path>` 生成 HTML，不启动服务器 |
| timing 数据 | 只在 subagent 通知时存在，必须立即保存 |
| grading.json 字段 | `text`, `passed`, `evidence`（不是 name/met/details）|
| 无惊喜原则 | Skill 行为必须与 description 一致，不含恶意内容 |
| description 优化 | train/test 分割；选 test score 最优版本 |
| SKILL.md 长度 | 建议 ≤500 行；超出则分层（SKILL.md + references/）|

---

## 三、待深入研究 / Areas for Further Study

- [ ] `agents/grader.md` — 断言评估的具体逻辑
- [ ] `agents/comparator.md` — A/B 盲对比机制
- [ ] `agents/analyzer.md` — 基准结果分析方法
- [ ] `references/schemas.md` — JSON 结构完整规范
- [ ] `run_loop.py` 源码 — 描述优化的实现细节
- [ ] Skill 实际创建实践（用 skill-creator 创建一个新 skill）

---

## 四、文件索引 / File Index

> 本次会话生成的所有文件

| 文件名 | 类型 | 描述 | 路径 |
|---|---|---|---|
| `skill-security-scan-template_2026-02-27.md` | 模版 Template | Skill 安全扫描检查清单，含风险等级、检查项分类（A-G 类）、扫描历史表格 | `personal-learning/` |
| `skill-creator-SKILL-archive_2026-02-27.md` | 存档 Archive | `skill-creator` SKILL.md 完整存档，含目录结构、元数据标注、关联文件说明 | `personal-learning/` |
| `learning-summary_2026-02-27.md` | 总结 Summary | 本文件；今日学习总结 + 待研究清单 + 文件索引 | `personal-learning/` |

**历史文件 / Historical Files:**

| 文件名 | 日期 | 描述 |
|---|---|---|
| `2026-02-26.md` | 2026-02-26 | 成人自主学习方法论笔记（四轮学习法、费曼反馈、复盘框架）|
| `未命名.md` | — | 空文件（可清理）|

---

## 五、项目学习索引 / Project Learning Index

| 项目 | 主要 Skill | 相关文件 | 学习日期 |
|---|---|---|---|
| `skill and e-evo` | `skill-creator` | `skill-creator-SKILL-archive_2026-02-27.md` | 2026-02-27 |
| `skill and e-evo` | _(待扩展)_ | — | — |

---

*生成时间 / Generated: 2026-02-27 | 项目: skill and e-evo*
