---
name: cn-case-postmortem
description: >
  案件结案复盘技能。在案件最终判决 / 调解 / 撤诉 / 终本后触发，整合 matter.yaml 全字段
  + 各 agent 最终产物 + 工时记录.md + 判决书（如有），输出案件复盘报告（事实回顾 / 关键决策点 /
  胜败原因 / 工作流改进），并把可复用经验**人 in the loop 沉淀**回对应 skill 的 memory.md
  （cn-litigation-drafting / cn-contract-review / cn-jiubufa-case-analysis /
  cn-judgment-analysis 等）。覆盖：结案、已结案、案件复盘、案件归档、复盘沉淀、
  案件总结、postmortem、工作流改进、胜败原因分析。
license: GNU AGPL v3（详见项目根 LICENSE）
---

# 中国诉讼案件结案复盘技能

## 角色定位

案件结案后的方法论复盘工具。把单一案件的实战经验**抽象 + 去标识化**为可复用的"问题
模式 + 采纳方案"，沉淀回对应 skill 的 memory.md，让下一个同类案件直接受益。

不仅仅是"写一份结案报告"——是**把私有经验转化为系统资产**的关键环节。

## 触发条件

满足以下**任一**条件即应使用：
- matter.yaml 的 `当前阶段` 字段更新为"已结案"（含判决 / 调解 / 撤诉 / 终本 / 和解履行完毕）
- 用户明示"结案 / 已结案 / 案件复盘 / 案件归档 / 复盘沉淀"
- 案件超过 6 个月无新动作，用户主动归档

## 5-Stage Workflow

```
Stage 0：Prepare（准备）
  → 读取 matter.yaml 全字段（含关键日期 / 阶段 / 当事人 / 案号 / 案由）
  → 读取 工时记录.md 全部条目（含累计工时 / 费用）
  → 读取案件根目录下所有 agent 产物：
      - 02 - 法律研究/案件分析/* （含 IssueIdentifier / Strategist / JiubufaAnalyst /
                                 JudgmentAnalyzer / ContractReviewer 等）
      - 03 - 我方证据/* + 04 - 对方证据/*
      - 05 - 我方法律文书/* （Writer 输出）
      - 07 - 法院法律文书/* （含最终判决 / 裁定 / 调解书）
      - 08 - 庭审笔录/*
      - 10 - 综合报告/*
  → 识别案件结案方式（判决 / 调解 / 撤诉 / 终本）
  → 启动读经验库（v1.12.0 闭环读侧）：读本 skill 目录 memory.md（**复盘方法论自身经验**，非分发给他 skill 的 lessons）；若有非"（暂无条目）"真实条目，按结案方式 / 案件类型筛选 top-3 纳入复盘参考；不存在或全空则跳过（no-op，不阻断、不污染 context）

Stage 1：Recap（事实回顾）
  → 时间线整合：从立案 → 关键节点 → 结案
  → 关键决策点提取（基于 Strategist 历次决策建议 + 客户决策记录）
  → 法庭 / 仲裁庭关键判定提取（如有判决 → JudgmentAnalyzer 已做的 IRAC 还原）

Stage 2：Analyze（胜败原因分析）
  → references/win-loss-analysis-framework.md 5 维度框架：
      法律层 / 事实层 / 程序层 / 策略层 / 资源层
  → 每维度评估：我方胜势 / 对方胜势 / 中性
  → 综合归纳：本案胜负的核心驱动因素

Stage 3：Improve（工作流改进）
  → 识别本案中**应该做而未做** / **做了但效果不佳**的事项
  → 对应到具体 skill / agent / Workflow 场景
  → 出具改进建议清单（短期可改 + 长期需调）

Stage 4：Distill（人 in the loop 沉淀）
  ⚠️ **严格保密硬约束**：以下步骤必须人 in the loop，不得自动执行
  
  → 按 references/memory-distillation.md 提取可复用条目，分发到以下目标
    （v1.12.0 WP1 起 7 个 legal skill 的 memory.md 均已 schema 预建于
    skill 根目录，**不再是"如有/references 下"——直接定位 `<skill>/memory.md`**）：
      - `cn-litigation-drafting/memory.md`：起草高频问题 + 可复用结论（按 A-K 文书类型 section）
      - `cn-contract-review/memory.md`：合同审查高频问题（按 14 类合同 section）
      - `cn-jiubufa-case-analysis/memory.md`：要件归入 / 请求权基础经验（按请求权类型 section）
      - `cn-judgment-analysis/memory.md`：判决书评审 / 救济路径经验（按救济路径 section）
      - `cn-trial-preparation/memory.md`：庭审准备实战经验（按庭审阶段 section）
      - `cn-client-communications/memory.md`：客户沟通措辞口径经验（按文书类型 section）
      - `cn-case-postmortem/memory.md`：**复盘方法论自身经验**（5 维度归因 / 分发踩坑，自沉淀）
  → 显式输出"沉淀草稿"到对话内（标注每条 → 目标 skill memory.md 的哪个 section）
  → **等待用户明确确认** 沉淀草稿无 client identifier / 无具体合同金额 / 无客户名
  → 用户确认后才执行写入：**按目标文件既有 schema 格式（触发条件/问题模式/可复用结论/适用场景/脱敏校验），倒序追加到对应 section 顶部（替换该 section 的"（暂无条目）"占位或追加到已有条目之上）；不新建文件、不覆盖既有条目、不破坏 schema 头**。重复模式更新已有条目而非重复追加。

Stage 5：Archive（归档）
  → 主交付物：YYMMDD 案件复盘报告.md（落 99 - 复盘沉淀/）
  → 次交付物：YYMMDD 工作流改进建议.md（落 99 - 复盘沉淀/）
  → memory 沉淀清单：YYMMDD memory 沉淀清单.md（落 99 - 复盘沉淀/，
    记录哪些条目被沉淀到哪个 skill 的 memory.md，便于追溯）
  → matter.yaml 字段更新：`当前阶段` → "已结案归档"
```

## 输出标准

- 复盘报告**结构化**：事实回顾 + 5 维度胜败分析 + 关键决策评估 + 改进清单
- 胜败分析**实事求是**：不夸大胜利 / 不掩饰失败；如部分胜诉 / 部分败诉，分项评估
- 改进建议**可操作**：每项含"具体动作 + 责任主体（律所 / 律师本人 / 系统）+ 优先级"
- memory 沉淀**严格脱敏**：参 references/memory-distillation.md 的脱敏检查清单

## 三条铁律

1. **保密硬约束 zero tolerance**：memory 沉淀严禁含 client identifier（当事人姓名 / 案号 /
   合同金额原文 / 身份证号 / 银行账户）；脱敏检查由 Reviewer 二次复核
2. **诚实分析**：胜败原因不为客户辩护，不为律师团队辩护；指向系统性原因 / 个人决策原因
3. **可复用导向**：所有改进建议和 memory 条目必须能被**下一个同类案件**直接受益

## 不允许的输出

- ❌ "本案律师付出了大量努力" 等无信息含量的赞誉
- ❌ "对方律师不够专业 / 法官有偏见" 等推卸性归因（除非有客观证据）
- ❌ 包含 client identifier 的 memory 条目（必返回重做）
- ❌ "下次注意" 等空泛改进建议
- ❌ 单独评判客户决策的对错（客户决策需要尊重，复盘聚焦律师工作流）

## 与既有 agent 的边界

- **不替代** Reporter 的综合报告——Reporter 是结案前/中的客户向报告；本 skill 是结案后的内部复盘
- **不替代** JudgmentAnalyzer 的判决书评审——本 skill 在 JudgmentAnalyzer 已完成评审基础上做更宏观的"案件全周期"复盘
- **不替代** Reviewer 的跨 agent 质量审查——Reviewer 是产物层 QA；本 skill 是流程层复盘

## 与 SuitAgent 的集成

本 skill 由 Postmortem agent 调起。agent 负责工程包装层（Stage 0 上下文加载、Stage 4
人 in the loop 触发、Stage 5 落盘 + matter.yaml 更新）。

详见 SuitAgent 仓库的 `.claude/agents/Postmortem.md`。

## License

本 skill 受项目根 LICENSE（GNU AGPL v3）约束。详见 `/LICENSE`。
