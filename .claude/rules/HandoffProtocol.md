# Handoff 简报协议规范（v1.13.0 单一权威源）

**版本**: v1.0
**最后更新**: 2026-05-16
**说明**: agent 间上下文传递的单一权威源。把"下游 orchestrator 读上游 .md 全文"改为"读案件根 `handoff_ledger.md` 小账本 + 按指针 lazy-load 仅需的 full 产物"，砍长 context 链路（如华雄 post-judgment：多判决+多证据）的 token 膨胀与丢失风险。**走结构化简报路线，不做 LLM 摘要**（外部研究证实 LLM 摘要 lossy 且误差累积）。

## 🎯 设计原则

- **结构化简报，非摘要**：定 typed 字段，照填，不做自由文本压缩/转述。
- **指针驱动 lazy-load**：briefing 给 full 产物路径 + 关键 slot，下游按需只读需要的那几份，不全量塞入。
- **持久账本**：每 producing agent 落盘 full 后，把自己的 briefing 块**倒序追加**到案件根 `handoff_ledger.md`（仿 Anthropic `claude-progress.txt` + git 模式），跨 session 不丢。
- **向后安全 gated**：`handoff_ledger.md` 不存在/为空 → 下游**回退现状全读**（no-op，不阻断），与 v1.12.0 已验证的 gated 模式同构。

## 📋 HandoffBriefing schema（每 agent 落盘后追加一块）

```
### [YYYY-MM-DD] [产出 agent] — [一句话本轮干成了什么]

- 全产物指针：`<相对路径>`（+ 关键 slot：如 05 - 我方法律文书/...）
- 目标达成：[1-3 句，本 agent 产出的核心结论/成果]
- 关键约束：[下游必须遵守的，如"诉求金额以 X 为准""管辖法院已定 Y"；无则写"无"]
- 已决事项：[下游不要重复决策的，如"请求权基础已锁定违约不走侵权"；无则写"无"]
- 待决 / N* 项：[下游必须接手的未决项 + 工具边界 N* 项（关联 NStarProtocol.md）；无则写"无"]
- 下游建议：[谁该读哪份 full、起草走哪个 skill 模板、重点核哪段]
```

### 软约束（不设硬 token 门槛）

- **简洁不啰嗦**为原则；**随案件复杂度适度伸缩**——简单案件每块几行即可；大型案件（华雄类多判决/多证据/多请求权）可适当展开，宁可多带一条关键约束/待决项，不可漏。
- 不复制 full 产物正文、不转述大段论证；只给**结论 + 指针 + 下游必须知道的约束/未决**。
- 字段可空但**必须显式写"无"**（不留空，避免下游误判为"未提供"）。

## 🚦 硬边界（zero tolerance，违反即防御退化）

1. **Reviewer 豁免**：Reviewer 的 D1-D8 核查**必须读 full 产物**，**绝不**以 briefing 替代全文（briefing 仅供 orchestrator→orchestrator 的 context 经济）。简报化下游不得波及 Reviewer 的对抗式 Verifier 全文核查——否则 v1.11.0c 三层防御退化。Reviewer.md 显式声明本豁免。
2. **待决 / N\* 项强制非空判断**：该字段是漏判防线——关键未决/工具边界项必须在此显式列出，下游据此接手；为空必须写"无"且确属无。
3. **gated 回退优先安全**：下游不确定 briefing 是否完整、或 ledger 缺失 → **回退读 full 产物**（安全侧默认），不得因省 token 漏读关键内容。

## 🔄 读写流程

**写侧（producing agent，WP3）**：落盘 full 产物 → 按 schema 追加 briefing 块到案件根 `handoff_ledger.md`（倒序，最新在顶）。指针引用本规范，不在各 agent.md 复述 schema（DRY，对齐 SelfCheck3E/NStarProtocol 体例）。

**读侧（orchestrator 3E Explore，WP4）**：Explore 步先读 `handoff_ledger.md`（小）→ 据各 briefing 的"下游建议/指针"**只 lazy-load 本任务真正需要的 full 产物** → 再进入 Examine。`handoff_ledger.md` 不存在/为空 → 回退现状"读相关上游 .md 全文"（gated no-op）。

## 🔗 与其他规范的关系

- **SelfCheck3E.md（Stage 2）**：3E 的 Explore 步读取方式由本协议规定（ledger-first + lazy-load）；SelfCheck3E.md「Explore」段指针引用本文。
- **NStarProtocol.md**：briefing「待决 / N* 项」字段是 N* 工具边界项向下游传递的载体；二者协同。
- **AgentMapping.md**：`handoff_ledger.md` 为案件根第 5 个 root level 文件（matter.yaml / matter_dashboard.md / AGENTS.md / 工时记录.md / **handoff_ledger.md**）。
- **ReviewerRubric.md**：Reviewer 全文核查硬边界（§硬边界 1）与 ReviewerRubric D 维度核查口径一致。

## 🔄 变更历史

| 版本 | 日期 | 更新内容 |
| :--- | :--- | :--- |
| v1.0 | 2026-05-16 | v1.13.0 WP1：结构化 handoff 简报协议初版——typed schema + 软约束（无硬 token 门槛，随案件复杂度伸缩）+ 三条硬边界（Reviewer 全文豁免 / 待决N*强制 / gated 回退）+ ledger-first lazy-load 读写流程 |

---

*本文档是 SuitAgent v1.13.0 结构化 handoff 简报协议的单一权威源，与 SelfCheck3E.md / NStarProtocol.md / AgentMapping.md / ReviewerRubric.md 交叉引用，自动加载到项目记忆中。*
