---
name: cn-client-communications
description: >
  律所对客户的日常沟通文书技能。覆盖 ongoing 流程的非正式沟通：进度通报（周报 /
  月报）、阶段性总结（开庭后 / 立案后 / 判决后）、风险预警（突发事件 / 时效届满）、
  决策建议书（需要客户拍板时）、客户问询回复（标准化）。与 cn-firm-documents（律所
  对客户**正式**文书：律师函 / 委托代理协议 / 法律意见书 / 谈话笔录等）严格分工：
  本 skill 仅做日常沟通层面，正式文书走 cn-firm-documents。覆盖：周报、月报、进度
  通报、阶段总结、风险预警、决策建议、决策建议书、客户问询回复、客户沟通、ongoing
  通报。
license: GNU AGPL v3（详见项目根 LICENSE）
---

# 律所对客户日常沟通文书技能

## 角色定位

律师与客户之间 ongoing 沟通的文书制作助手。把分散在 matter.yaml / 工时记录.md / 各
agent 输出中的案件最新进展，**抽象 + 整合 + 适度脱敏**为客户能读懂、能据以决策的简洁
沟通文档。

不替代律师与客户的口头沟通；不替代正式法律意见书（那是 cn-firm-documents 的范围）。

## 与 cn-firm-documents 的边界（必读）

| 文书类型 | 归属 | 说明 |
|---------|------|------|
| 律师函 / 委托代理协议 / 授权委托书 / 谈话笔录 / 法律意见书 / 离婚协议审阅意见 / 刑事格式文书 | **cn-firm-documents**（外置）| 正式 + 对外 + 有法律或合同效力 |
| 周报 / 月报 / 进度通报 / 阶段总结 / 风险预警 / 决策建议书 / 客户问询回复 | **本 skill cn-client-communications**（内置）| 日常 + ongoing + 沟通性质 |

边界判断口径：
- **是否对外（除客户外的第三人）展示？** 是 → cn-firm-documents；否 → 本 skill
- **是否产生独立法律 / 合同效力？** 是 → cn-firm-documents；否 → 本 skill
- **决策建议书是否包含简版法律意见？** **否**——决策建议书仅引用已有的正式 LegalOpinion，不重复其内容；如客户尚无 LegalOpinion，先请 cn-firm-documents 出具，再用本 skill 出决策建议

## 启动读经验库（v1.12.0 闭环读侧）

起草任何客户沟通文书前，先读取本 skill 目录下 `memory.md`。若存在**非"（暂无条目）"的真实条目**，按文书类型（周报月报 / 阶段总结 / 风险预警 / 决策建议书 / 问询回复）筛选**最相关 top-3** 纳入起草参考（尤其措辞口径类经验）；若 `memory.md` 不存在或相关 section 全为"（暂无条目）"，**跳过本步（no-op，不阻断、不污染 context）**。本步**只读不写**；写入由 cn-case-postmortem 结案复盘统一负责。

## 5 类文书设计

### 1. 进度通报（周报 / 月报）

- 触发：定期（每周一 / 每月初）+ 客户要求
- 模板：[`references/progress-report-template.md`](references/progress-report-template.md)
- 核心字段：本周/月已完成 + 待办 + 期限提醒 + 工时统计 + 阶段判定
- 长度：1-2 页 A4

### 2. 阶段性总结

- 触发：关键节点后（立案 / 开庭 / 判决 / 调解 / 撤诉 / 执行启动 / 终本）
- 模板：[`references/stage-summary-template.md`](references/stage-summary-template.md)
- 核心字段：节点事件 + 实际进展 vs 预期 + 后续策略调整 + 客户决策事项
- 长度：1-3 页 A4

### 3. 风险预警

- 触发：突发事件（对方异常动作 / 法院发现新事实 / 时效紧迫 / 客户证据问题）
- 模板：[`references/risk-alert-template.md`](references/risk-alert-template.md)
- 核心字段：风险类型 + 严重程度（红 / 橙 / 黄）+ 触发原因 + 即时应对 + 客户配合事项
- 长度：1 页 A4（紧急简短）+ 必要时附详细分析

### 4. 决策建议书

- 触发：需要客户拍板的关键节点（接受调解 / 提起反诉 / 上诉 / 再审 / 和解金额）
- 模板：[`references/decision-recommendation-template.md`](references/decision-recommendation-template.md)
- 核心字段：决策事项 + 选项（至少 3 个）+ 各选项利弊 + 律师推荐 + 推荐理由 + 决策时限
- **关键限定**：仅引用已有 LegalOpinion 的结论，**不重复或重写**法律意见书的法律分析层
- 长度：2-3 页 A4

### 5. 客户问询回复

- 触发：客户主动提问（口头 / 微信 / 邮件）
- 模板：[`references/decision-recommendation-template.md`](references/decision-recommendation-template.md)（共用决策建议书模板的局部）
- 核心字段：客户问题 + 简明回答 + 法律依据（如适用）+ 后续配合事项
- 长度：视问题复杂度，单一问题 ≤ 0.5 页 A4

## 4-Stage Workflow

```
Stage 0：Prepare（准备）
  → 读取 matter.yaml（关键日期 / 阶段 / 当前状态）
  → 读取 工时记录.md（本周 / 本月已工作内容）
  → 读取上游 agent 最新产物（DocAnalyzer / IssueIdentifier / Strategist 等）
  → 识别文书类型（5 类之一）
  → 客户身份核对（确认收件人）

Stage 1：Build（按文书类型生成）
  → 周报 / 月报：从工时记录.md 抽取已完成 + 从 matter.yaml 关键日期抽取下周 / 下月待办
  → 阶段总结：从 matter.yaml 阶段字段 + 上游产物提取节点信息
  → 风险预警：从 Strategist 风险评估或 Scheduler 时效预警提取
  → 决策建议书：从已有 LegalOpinion / Strategist 策略 / JudgmentAnalyzer 救济路径表提取选项
  → 客户问询回复：基于客户原问 + 既有产物精准回答

Stage 2：Discuss（人 in the loop）
  → 输出对话内草稿，等待用户审阅与微调（重点：脱敏程度 / 措辞克制 / 决策表述）
  → 客户身份信息**保留**（毕竟收件人是客户本人），但不带其他案件 / 第三方信息

Stage 3：Execute（落盘）
  → 主交付物：YYMMDD [文书类型].md（落 10 - 综合报告/客户沟通/）
  → 必要时调起 docx skill（套用 china_law_firm_template.md 排版参数）
  → 不入仓（10 - 综合报告/ 整体由 .gitignore 兜底排除）

Stage 4：Learn（学习）
  → 客户偏好沉淀（如客户偏好简短 / 详细 / 表格）写入 personal-preferences 类文件
  → 不写入 client identifier / 案号 / 金额原文
```

## 输出标准

- **措辞克制**：避免胜负宣告（"必胜" / "稳赢"）；避免过度乐观（"基本没问题"）；派生金额用范围词（"约 X 万元 ± Y%"）
- **结构清晰**：用 Markdown 标题 / 列表 / 表格组织；客户能 1-3 分钟读完核心
- **决策可执行**：每份文书末尾列"客户需要做的事"或"等待客户确认事项"；不含 "请客户酌情考虑" 等空泛措辞
- **语言对接**：不用过深的法律术语（如必须用，配合简明解释）；可在末尾附"术语注释"
- **保密硬约束**：本案外的其他案件 / 客户不出现；律所内部信息不出现

## 与 Writer agent 的集成

本 skill 由 Writer agent 在收到"周报 / 月报 / 进度通报 / 阶段总结 / 风险预警 / 决策建议 / 客户问询回复"等触发关键词时调起。

Writer agent 的文书路由表中，本 skill 是第三类（律所对客户**日常**沟通文书）；与
cn-litigation-drafting（诉讼文书）、cn-firm-documents（律所对客户**正式**文书）三足
鼎立。

## 三条铁律

1. **不重写法律意见**：决策建议书仅引用已有 LegalOpinion 结论，不重复其法律分析
2. **客户视角**：用客户能理解的语言，不展示律师内部分歧 / 律所内部讨论
3. **决策可操作**：每份沟通文书都给客户**具体可执行的下一步**，不留空泛悬念

## 不允许的输出

- ❌ "请咨询律师" / "请专业律师酌定" 等回避（你就是律师）
- ❌ 重复 LegalOpinion 的法律分析（不是本 skill 范围）
- ❌ 律所内部讨论 / 主办律师与协办律师分歧
- ❌ 其他案件 / 其他客户信息（保密硬约束）
- ❌ 未审阅就直接发送客户的文书

## License

本 skill 受项目根 LICENSE（GNU AGPL v3）约束。详见 `/LICENSE`。
