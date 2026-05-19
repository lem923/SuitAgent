# 3E 自检流程规范（Stage 2 单一权威源）

**版本**: v1.1
**最后更新**: 2026-05-16
**说明**: v1.11.0 三层纵深防御 Stage 2 的通用规范单一权威源。6 个 orchestrator agent（Writer / ContractReviewer / JiubufaAnalyst / JudgmentAnalyzer / TrialPrep / Postmortem）的 3E 自检共享本规范；各 agent.md 只保留**专属 Examine 校验问题清单**与**专属自检结果段格式**，不再重复本文通用部分（DRY 单一权威源，v1.11.1 抽取）。

## 🎯 核心理念

调起 skill 后、落盘前**必须插入一次自检（Examine 步）**。Self-Refine 范式（NeurIPS 2023, arXiv 2303.17651）+ CoVe 校验问题清单（arXiv 2309.11495），拦截低级工程错误（漏字段 / 路径错 / skill 调起错 / 与上游产物冲突），作为 Reviewer 对抗式 Verifier（v1.11.0c）前置的第一道防御。**不可跳过**。

与 Stage 1（skill 内 QC）、Stage 3（Reviewer 跨 agent 对抗式 Verifier）构成纵深；与 Stage 3 遵循"两层防御不重叠原则"——Reviewer 不重复核查 3E 已查项是否被执行，但仍核查其内容是否合规。

## 🔄 3E 三步

### Explore（已有）

**上游 context 承接（v1.13.0 ledger-first）**：进入本步前，先读案件根 `handoff_ledger.md`（小账本）→ 按各 briefing 的"下游建议 / 指针"**只 lazy-load 本任务真正需要的 full 上游产物**，不全量读所有上游 .md（砍长 context 膨胀）。`handoff_ledger.md` 不存在 / 为空 / briefing 不足以支撑本任务 → **回退现状"读相关上游 .md 全文"（gated no-op，不阻断、安全侧默认）**。规范见 [`HandoffProtocol.md`](./HandoffProtocol.md)。

然后按本 agent "工作流程" 章节调起对应 skill 完成草稿 / 分析。**skill 内部自身的 QC（Stage 1）必须先完成**（各 agent 调起的具体 skill 及其内嵌 QC 见该 agent 的工作流程章节，此处不复述以避免漂移）。

### Examine（强制，max_iter=1）

落盘前对照本 agent 的"专属校验问题清单"逐项核查。每项答 **Y/N**，**任一项 N 进入 Enhance 修订一次**（max_iter=1，不得多轮自循环）。

### Enhance（条件触发）

- Examine 全 Y → 直接落盘（不进入修订）
- Examine 任一项 N → 针对 N 项修订草稿一次（不整体重写）→ 重新执行 Examine
- 修订后仍 N → **不擅自落盘**，输出末尾显式列"未通过 Examine 项" + 修订摘要 + 升级建议（升级用户裁定 / Reviewer 接管）

## 📋 Examine 自检结果段（落盘前响应必含）

每个 orchestrator agent 落盘前响应**必含**一段"## Examine 自检结果"，通用格式如下（`[N]` = 该 agent 专属清单项数，由各 agent.md 自带的专属格式块给出具体项数与编号）：

```
## Examine 自检结果
专属 [N] 项：[Y/Y/.../Y]
修订次数：[0 / 1]
未通过项（如有）：[列具体编号 + fail 理由 + 修订摘要 + 是否升级]
```

## 🔗 与 N* 协议的衔接

Examine 校验清单中涉及**硬核对项**（法条现行性 / 案号 / 时效 / 程序节点），若本 agent 线程无 WebSearch 工具，**不得凭训练数据伪 Y**，应按 [`NStarProtocol.md`](./NStarProtocol.md) 标 N\* 并 escalate 有 web_search 权的 Reviewer（D1/D4/D5）接管补核，闭环判据见该协议。

## 📌 各 agent 专属清单索引

| Agent | 专属 Examine Q 编号 | 项数 |
|-------|---------------------|------|
| Writer | W1-W7 | 7 |
| ContractReviewer | CR1-CR7 | 7 |
| JiubufaAnalyst | JB1-JB8 | 8 |
| JudgmentAnalyzer | JA1-JA8 | 8 |
| TrialPrep | TP1-TP7 | 7 |
| Postmortem | PM1-PM8 | 8 |

具体清单内容见各对应 `.claude/agents/<Agent>.md` 的"Examine 校验问题清单（agent 专属）"段。

## 🔄 变更历史

| 版本 | 日期 | 更新内容 |
| :--- | :--- | :--- |
| v1.1 | 2026-05-16 | v1.13.0 WP4：Explore 步新增 ledger-first 上游 context 承接（读 handoff_ledger.md → 按指针 lazy-load 仅需 full 产物，gated 回退全读），DRY 覆盖 6 orchestrator 读侧 |
| v1.0 | 2026-05-16 | v1.11.1 工程债清偿：从 6 个 orchestrator agent.md 抽取 3E 通用规范为单一权威源（DRY），各 agent 改为指针引用 + 仅保留专属清单 |

---

*本文档是 SuitAgent v1.11.0 三层纵深防御 Stage 2 的单一权威源，6 个 orchestrator agent 共享，自动加载到项目记忆中。*
