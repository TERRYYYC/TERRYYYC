# 规范快速核查表 / Convention Quick Check

> 每次新建文件或会话结束后，用这张表扫一遍。30 秒内完成。
> Run through this checklist after creating files or ending a session. Takes under 30 seconds.
> 规范细节请见 `_HOW-TO-USE.md`。

---

## ✅ 新文件核查 / New File Check

### 1. 文件放对位置了吗？

```
学习总结      → p00X-[项目名]/
文档存档      → p00X-[项目名]/
安全扫描报告  → p00X-[项目名]/
模版          → _templates/
杂项笔记      → _misc/
系统文件      → 根目录（仅 _index.md / _HOW-TO-USE.md / _quick-check.md / _personal-learning-log.md）
```

- [ ] 文件在正确的项目文件夹内，不在根目录散落

---

### 2. 文件名格式正确吗？

**学习总结格式：** `[P编号]_[模块描述]_YYYY-MM-DD.md`

| ✅ 正确 | ❌ 错误 |
|---|---|
| `p004_core-architecture_2026-03-01.md` | `learning-summary_2026-03-01_2.md` |
| `p003_boris-workflow_2026-02-28.md` | `p003_boris_workflow_26-2-28.md` |

规则：
- [ ] 以 `p00X_` 开头（P编号 + 下划线）
- [ ] 模块描述用连字符 `-` 分词，全小写英文
- [ ] 日期格式 `YYYY-MM-DD`，不缩写
- [ ] 同项目同天多个文件：用**不同的模块描述**区分，不用 `_2` `_3` 序号

**文档存档格式：** `[P编号]_[文档名]-archive_YYYY-MM-DD.md`

- [ ] 末尾含 `-archive` 标记

**杂项文件格式：** `misc_[描述]_YYYY-MM-DD.md`（无日期时可省略日期）

---

### 3. Frontmatter 填了吗？

学习总结文件开头应包含（完整版）：

```yaml
---
project: P004 · Swarm-IDE
file_type: learning-summary
tags: [关键词1, 关键词2, 关键词3]
date: YYYY-MM-DD
session_type: 初次学习 / 深度研究 / 深度复现 / 实践验证
source: https://... （外部项目必填；内部项目删除此行）
related: 上一篇文件名（可选）
---
```

`file_type` 取值：`learning-summary` / `archive` / `homework-prompts` / `scan-report` / `misc`

- [ ] `project` 字段格式为 `P编号 · 项目名`（注意 `·` 分隔符）
- [ ] `file_type` 已填写（必填，见上方取值表）
- [ ] `tags` 包含 3 个以上核心关键词
- [ ] `session_type` 使用规定的四种之一
- [ ] 外部项目已填写 `source`（GitHub/文章 URL）
- [ ] 新文件是续集时，`related` 指向了上一篇

---

## ✅ `_index.md` 更新核查 / Index Update Check

每次学习会话结束后检查：

- [ ] 「会话时间线」已追加本次记录（含日期、项目、主题摘要、进度阶段、文件链接）
- [ ] 对应项目的「学习会话」表格已追加（含文件链接和进度阶段）
- [ ] 「待探索」清单中已完成的项目打了 ✅
- [ ] 如有新的待探索项，已追加到清单
- [ ] 顶部统计数字（总会话数、已学项目数、生成文件数）已更新
- [ ] 顶部「最后更新」日期已更新

---

## ✅ 新项目核查 / New Project Check

第一次为新项目创建文件时：

- [ ] 在 `_index.md` 「项目学习记录」中新建完整项目章节（含 ID、路径、主题、状态等字段）
- [ ] 创建了对应项目文件夹 `p00X-[名称]/`
- [ ] 分配了正确的 P 编号（续上一个，不重复）
- [ ] 在 `_index.md` 的「知识图谱」中补充了关联关系

---

## ✅ 系统文件变更核查 / System File Change Check

修改了 `_HOW-TO-USE.md` 或 `_index.md` 结构时：

- [ ] 在 `_personal-learning-log.md` 末尾追加了变更记录
- [ ] 记录了变更原因、具体内容、执行者

---

## 🔎 常见错误速查 / Common Mistakes

| 错误 | 正确做法 |
|---|---|
| 文件放在根目录 | 放入 `p00X-[项目名]/` 或 `_misc/` |
| 用 `_2` `_3` 区分同天多文件 | 改用不同的模块描述名 |
| 日期写 `26-3-1` 或 `2026/03/01` | 统一用 `2026-03-01` |
| 更新了文件但没更新 `_index.md` | 会话结束前必须更新索引 |
| 新项目没有在 `_index.md` 建章节 | 先建章节，再开始学习 |
| Frontmatter 缺失 | 每个学习总结文件都要有 frontmatter |
| `file_type` 字段缺失 | v2.1 起必填；取值见第3节 |
| `source` 字段缺失（外部项目）| 外部 GitHub/文章项目必须填写原始链接 |
| `project` 字段无 `·` 分隔符（如 `P004 Swarm-IDE`）| 改为 `P004 · Swarm-IDE` |
| 把模版文件直接当总结用 | 复制模版到项目文件夹，重命名后使用 |

---

*v1.1 · 2026-03-01（同步 _HOW-TO-USE.md v2.1：frontmatter 新增 file_type / source 字段；更新核查项和常见错误表）*
