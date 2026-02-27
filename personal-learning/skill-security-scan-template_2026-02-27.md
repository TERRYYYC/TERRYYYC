# Skill 安全扫描模版 / Skill Security Scan Template

> **生成时间 / Generated:** 2026-02-27
> **项目来源 / Project Source:** `skill and e-evo` → [`skill-creator`](skill-creator索引见下)
> **项目学习索引 / Project Learning Index:** → 见 `learning-summary_2026-02-27.md` §文件索引
> **版本 / Version:** v1.0

---

## 使用说明 / How to Use

对每个 Skill（`.skill` 文件或 `SKILL.md`）进行审查时，逐项填写下方检查清单。每项标注：
- ✅ 通过 / PASS
- ⚠️ 需关注 / ATTENTION
- ❌ 高风险 / HIGH RISK
- N/A 不适用

When reviewing each skill, fill in the checklist below. Mark each item:
- ✅ Pass | ⚠️ Attention needed | ❌ High risk | N/A Not applicable

---

## 扫描记录头 / Scan Header

| 字段 Field | 内容 Content |
|---|---|
| Skill 名称 | _(填写 skill name)_ |
| 扫描日期 | _(YYYY-MM-DD)_ |
| 扫描人 | _(name/alias)_ |
| Skill 路径 | _(absolute path)_ |
| Skill 来源项目 | _(project name, e.g. "skill and e-evo")_ |
| 项目学习索引 | _(link to project learning doc)_ |
| Skill 版本/commit | _(version or git hash)_ |
| 总体风险等级 | ⬜ 低 Low / ⬜ 中 Medium / ⬜ 高 High / ⬜ 极高 Critical |

---

## A. 内容完整性 / Content Integrity

| # | 检查项 Check Item | 状态 Status | 风险等级 Risk Level | 备注 Notes |
|---|---|---|---|---|
| A1 | SKILL.md 存在且格式正确（含 YAML frontmatter：name, description） | | 低 Low | |
| A2 | name 字段与目录名一致 | | 低 Low | |
| A3 | description 字段清晰描述触发条件，无歧义 | | 中 Medium | 歧义描述可能导致误触发 |
| A4 | SKILL.md 行数在合理范围（建议 ≤500 行）| | 低 Low | 过长可能包含隐藏内容 |
| A5 | 引用的外部资源文件（references/、scripts/、assets/）均存在 | | 中 Medium | 缺失文件可能导致运行时错误 |

---

## B. 代码与脚本安全 / Code & Script Safety

| # | 检查项 Check Item | 状态 Status | 风险等级 Risk Level | 备注 Notes |
|---|---|---|---|---|
| B1 | scripts/ 目录中无恶意代码、漏洞利用代码（exploit）| | **极高 Critical** | 直接执行风险 |
| B2 | 脚本不包含硬编码凭证（API Key、密码、Token）| | **极高 Critical** | 凭证泄露 |
| B3 | 脚本无网络外发行为（curl/wget 向外部未知域名发送数据）| | **高 High** | 数据外泄 |
| B4 | 脚本无文件系统破坏操作（rm -rf、overwrite 系统文件）| | **高 High** | 数据破坏 |
| B5 | 脚本无权限提升操作（sudo、chmod 777 关键路径）| | **高 High** | 权限滥用 |
| B6 | 脚本依赖库均为已知安全来源 | | 中 Medium | 供应链风险 |
| B7 | 使用 `--break-system-packages` 的 pip 安装有明确必要性 | | 中 Medium | 系统包污染 |

---

## C. Prompt 注入风险 / Prompt Injection Risk

| # | 检查项 Check Item | 状态 Status | 风险等级 Risk Level | 备注 Notes |
|---|---|---|---|---|
| C1 | SKILL.md 中无诱导 Claude 忽略安全规则的指令 | | **极高 Critical** | Jailbreak 风险 |
| C2 | 无"roleplay as an unrestricted AI"类指令 | | **高 High** | 角色扮演绕过 |
| C3 | references/ 文档中无嵌入式注入指令（如隐藏文字、base64 编码指令）| | **高 High** | 隐式注入 |
| C4 | Skill 不指示 Claude 从不受信任的 Web 内容执行指令 | | **高 High** | 间接注入 |
| C5 | Skill 的指令不要求 Claude 自动发送消息/提交表单（无用户确认）| | **高 High** | 未授权操作 |
| C6 | Skill 不声称拥有"管理员/系统/Anthropic"权限 | | **极高 Critical** | 权限伪造 |

---

## D. 数据隐私与外泄 / Data Privacy & Exfiltration

| # | 检查项 Check Item | 状态 Status | 风险等级 Risk Level | 备注 Notes |
|---|---|---|---|---|
| D1 | Skill 不收集或传输用户个人信息（PII）| | **高 High** | GDPR/隐私合规 |
| D2 | Skill 不要求用户提供敏感信息（银行卡号、社会安全号等）| | **极高 Critical** | 金融欺诈 |
| D3 | 生成的文件不自动上传至第三方服务 | | **高 High** | 数据外泄 |
| D4 | Skill 不读取其他标签页/域名的内容并发送出去 | | **高 High** | 跨域泄露 |
| D5 | 输出文件路径限定在用户工作区（不写入系统路径）| | 中 Medium | 系统污染 |

---

## E. 意图透明度 / Intent Transparency

| # | 检查项 Check Item | 状态 Status | 风险等级 Risk Level | 备注 Notes |
|---|---|---|---|---|
| E1 | Skill 的实际行为与 description 描述一致（无惊喜原则）| | **高 High** | 用户信任 |
| E2 | Skill 不模拟其他 skill 或冒充系统组件 | | **高 High** | 身份欺骗 |
| E3 | Skill 对用户暴露的操作是可预期的（不隐藏实际执行内容）| | 中 Medium | 透明度 |
| E4 | 如有破坏性操作（删除、发布），Skill 明确要求用户确认 | | **高 High** | 用户保护 |

---

## F. 触发与隔离 / Trigger & Isolation

| # | 检查项 Check Item | 状态 Status | 风险等级 Risk Level | 备注 Notes |
|---|---|---|---|---|
| F1 | description 不会导致过度触发（误用其他技能场景）| | 中 Medium | 工作流污染 |
| F2 | description 不会导致与安全相关的技能被误触发 | | **高 High** | 安全绕过 |
| F3 | Skill 不在未受邀场景下自动启动持久后台进程 | | 中 Medium | 资源滥用 |

---

## G. 版权与合规 / Copyright & Compliance

| # | 检查项 Check Item | 状态 Status | 风险等级 Risk Level | 备注 Notes |
|---|---|---|---|---|
| G1 | Skill 不指示 Claude 大段复制受版权保护的内容 | | 中 Medium | 版权侵权 |
| G2 | 引用的第三方库/资产有合法授权 | | 中 Medium | 许可证合规 |
| G3 | Skill 不用于生成针对真实人物的虚假内容 | | **高 High** | 诽谤/伦理 |

---

## 扫描结论 / Scan Conclusion

```
总体风险等级 Overall Risk Level:  [ ] 低 Low  [ ] 中 Medium  [ ] 高 High  [ ] 极高 Critical

关键发现 Key Findings:
1.
2.
3.

建议措施 Recommended Actions:
1.
2.

审核意见 Reviewer Comment:


签字/确认 Sign-off: ______________________  日期 Date: ____________
```

---

## 风险等级定义 / Risk Level Definitions

| 等级 Level | 定义 Definition | 响应时间 Response Time |
|---|---|---|
| 🟢 低 Low | 影响范围有限，可接受风险 | 下次迭代处理 |
| 🟡 中 Medium | 存在一定风险，需要关注 | 本轮开发周期内处理 |
| 🔴 高 High | 显著安全威胁，影响用户安全或数据 | 发布前必须修复 |
| 🚨 极高 Critical | 严重漏洞，可能导致系统入侵、数据泄露、用户伤害 | 立即停用并修复 |

---

## 扫描历史 / Scan History

| 日期 Date | 扫描人 Reviewer | Skill 名称 | 总体风险 | 关键发现摘要 |
|---|---|---|---|---|
| 2026-02-27 | _(初始模版)_ | — | — | 模版创建，尚无扫描记录 |

---

*本模版基于 `skill-creator/SKILL.md` 安全原则生成（2026-02-27）*
*This template is generated based on security principles in `skill-creator/SKILL.md` (2026-02-27)*
