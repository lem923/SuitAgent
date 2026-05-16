# N\* 协议规范（工具边界诚实处置 + 接管闭环）

**版本**: v1.0
**最后更新**: 2026-05-16
**说明**: 把 v1.11.0 P0 四 pass（260508 / 260511 / 260507 / 260510）实证、跨 Writer / Researcher / JudgmentAnalyzer / ContractReviewer 四类 orchestrator agent 一致复现的 **N\* 工具边界诚实处置 + 有 web 权层接管闭环**模式，从隐性涌现升为可据以实现的一等规范。新 agent 接入三层防御必须遵循本协议。

## 🎯 为什么需要 N\*

硬核对项（法条现行性 / 案号 / 时效计算 / 程序节点）必须 web_search 白名单源核对，**不得凭训练数据假定现行有效**（外部研究确认：知识截断导致的"时间性幻觉"——如适用已被推翻的旧法/旧条号——是结构性问题，本项目 260508 失效 SJ、260507 旧条号、260511 误引均属此类）。

但**仅 Researcher 与 Reviewer 两个 agent 被授予 WebSearch / WebFetch 工具**；6 个 orchestrator agent（Writer / ContractReviewer / JiubufaAnalyst / JudgmentAnalyzer / TrialPrep / Postmortem）线程内**无 web 工具**。当这些 agent 遇到硬核对项时，面临三选一：

- ❌ **凭训练数据伪 Y**：严重缺陷（D1/D4 现行性幻觉），最危险，绝对禁止
- ❌ **静默删除该项**：掩盖不确定性，下游无从知晓，禁止
- ✅ **标 N\* 并显式 escalate**：工具边界诚实处置——这就是 N\* 协议

## 🔑 N\* 的定义（三态，区别于 Y / N）

| 态 | 含义 |
|----|------|
| **Y** | 已实际核对通过（有 web 工具者 web_search 白名单确认；或本就无需硬核对的软项达标）|
| **N** | 已实际核对**未通过**（确有错误/缺失）→ 进入 Enhance 修订或判 fail |
| **N\*** | **本线程工具能力不足以核对**该硬核对项；未伪 Y、未静默；显式标注 + escalate 有 web 权层接管 |

N\* ≠ N：N 是"查了，错了"；N\* 是"该项需 web 核对但本层无此工具，移交"。二者下游处理路径不同。

## 📋 适用条件（同时满足才用 N\*）

1. 该项是**硬核对项**：对应 ReviewerRubric.md 的 **D1（法条引用）/ D2（案号判例）/ D4（时效计算）/ D5（程序节点）** 维度，或 skill QC 的 search-first 强制项（如 cn-litigation-drafting 共享2/共享6）；
2. 执行 agent 的**线程内无 WebSearch / WebFetch 工具**（6 个 orchestrator agent 默认属此情形）；
3. 该项**无法仅凭已落盘上游产物充分确证**（若上游 v1.11.0d Researcher 已核并在「引用规范现行性核验表」给出结论，应优先采信上游，标 Y 并注明依据，不必 N\*）。

> 软评估项（D3 事实一致 / D6 主体清晰 / D7 内部逻辑）**不适用 N\***——它们靠上下文比对即可判定，无 web 依赖。

## 🔄 N\* 处置与接管闭环（标准流程）

```
[orchestrator agent 遇硬核对项 + 本线程无 web 工具]
  → 标 N*（不伪 Y / 不静默）
  → 该项写入「## Examine 自检结果」/「## QC 自检结果」并显式列为 N*
  → agent 不因 N* 阻断落盘（N* 非草稿质量缺陷，是工具边界）
  → agent 响应末尾显式 escalate：列出全部 N* 项 + 移交 Reviewer 哪个 D 维度
     ↓
[主 agent 落盘后自动调起 Reviewer（v1.11.0c）]
  → Reviewer 具 WebSearch/WebFetch + D1/D2/D4/D5 白名单核对权
  → 对每个 N* 项实际执行白名单 web_search（npc.gov.cn / court.gov.cn / spp.gov.cn /
     gov.cn / 各部委及省级官网 / 现行有效版本），双源交叉
  → 补核结论：N* → Y（核实现行有效，给出可定稿的精确条号/文号/期限）
              或 N* → N（核实确已失效/错误，给出现行替代 + diagnostic notes 反喂）
     ↓
[闭环判据]：Reviewer 在评估结果段给出可直接回填下游文书的精确结论
  （现行条号 / 文号 / 施行日 / 期限），下游 Writer/承办律师据此定稿。
  闭环达成 = N* 项已被有 web 权层实际核实并产出可定稿结论。
```

## ✅ 记法约定

- **skill QC / agent Examine 清单**：该项写 `N*`（不是 `N`，不是 `Y`），并在「未通过项」区注明"工具边界，escalate Reviewer Dx"
- **agent 响应末尾**：必含一段显式 escalate，如：`⚠️ 现行性 escalate：[列项] 判 N*，本线程无 WebSearch，移交 Reviewer D1/D4/D5 白名单硬核对`
- **Reviewer 评估结果**：被接管的维度若仅因 N* 而非实质缺陷，按 ReviewerRubric 阈值处理——Reviewer 完成补核后该 N* 不计为质量 fail（属"工具边界诚实标注"，4-pass 一致认定为正确处置而非缺陷）；Reviewer 在 diagnostic notes 给出可定稿结论

## 🚫 反例（严重缺陷，绝对禁止）

| 反例 | 后果 | 正确做法 |
|------|------|---------|
| 凭训练数据把硬核对项写 Y | D1/D4 现行性幻觉（如沿用已废止旧条号），最危险 | 标 N\* + escalate |
| 把无法核对的项静默删除 | 下游无从知晓不确定性，掩盖 | 标 N\* + 显式列出 |
| 标 N\* 但响应末尾不 escalate | 接管链断裂，Reviewer 不知该重点核哪项 | N\* 必配显式 escalate 清单 |
| Reviewer 对 N\* 项也凭训练数据判定 | 接管层失防，整链失守 | Reviewer 必须实际 web_search 白名单双源 |

## 🔗 与其他规范的关系

- **与 v1.11.0d 上游前移（Researcher 规范现行性强制核验）互补**：Researcher **有** WebSearch，在上游主动核验并产出现行性核验表（早拦截）；6 个无 web 工具的 orchestrator 在下游用 N\* escalate（晚兜底）。两者构成纵深，不取消彼此。若上游 Researcher 已核，下游优先采信上游结论（不必 N\*）。
- **与 SelfCheck3E.md（Stage 2）**：3E 的 Examine 清单中的硬核对项遇工具边界即走本协议；SelfCheck3E.md「与 N\* 协议的衔接」段指向本文。
- **与 ReviewerRubric.md（Stage 3）**：Reviewer 是 N\* 的接管层；D1/D2/D4/D5 硬核对子项 + auto-retry handshake 是闭环的执行机制。
- **与三层防御 N\* 串联**：skill QC fail 升级 agent → agent Examine N\* 升级 Reviewer → Reviewer 补核闭环（详见 ReviewerRubric.md §3）。

## 📌 实证依据（P0 四 pass）

| pass | agent | N\* 表现 | 接管结果 |
|------|-------|---------|---------|
| 260508 | Writer | 共享2/6 判 N\* 未伪 Y | Reviewer 核出 2 处失效 SJ，C→B→pass |
| 260511 | Researcher+Writer | Researcher 有 web 上游拦截 3 误引；Writer 规避 | A 级 0 retry |
| 260507 | JudgmentAnalyzer | JA2 判 N\* 未伪造民诉法 2023 条号 | Reviewer 补现行第211/216/220/236条，B 级 |
| 260510 | ContractReviewer | 全法条判 N\* + 主动标注法释〔2025〕12号 | Reviewer 补核 12 处全现行有效，B 级 |

跨 4 类 orchestrator agent 一致复现 → N\* 是已验证的稳定设计模式，故升为一等规范。

## 🔄 变更历史

| 版本 | 日期 | 更新内容 |
| :--- | :--- | :--- |
| v1.0 | 2026-05-16 | v1.11.1 工程债清偿 D2：把 4-pass 实证的 N\* 工具边界诚实处置 + 接管闭环升为一等规范，供新 agent 据以实现 |

---

*本文档是 SuitAgent v1.11.0 三层纵深防御的 N\* 协议单一权威源，与 SelfCheck3E.md / ReviewerRubric.md 交叉引用，自动加载到项目记忆中。*
