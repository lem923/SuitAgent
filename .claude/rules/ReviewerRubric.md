# Reviewer Rubric 规范

**版本**: 1.0
**最后更新**: 2026-05-14
**说明**: Reviewer agent（v1.11.0c 升级版）的 8 维度结构化 rubric 详细子项规范 + web_search 核对锚点 + 评级阈值映射 + Diagnostic notes 格式 + auto-retry 协议详细规范

---

## 🎯 设计原则

1. **结构化 Y/N，不浮动评分**：避免 LeMAJ 论文报告的 LLM-as-a-Judge 在 legal 场景 58-88% hallucination 风险。每维度 Y/N，子项 Y/N，不打 1-5 分。
2. **硬核对优先**：可硬核对项（D1/D2/D4/D5）必须 web_search 白名单源核对，不靠训练数据 / LLM 主观判断。
3. **D8 zero tolerance**：保密硬约束任意 fail → 直接 D 级。
4. **Reflexion verbal feedback**：fail 项 diagnostic notes 反喂 orchestrator agent 作为 retry 输入（语义梯度）。
5. **max-retry=2**：硬上限避免死循环。

---

## 1. 8 维度 Rubric 详细子项

### D1 法条引用（硬核对，web_search 白名单）

**核查子项**（任一 N → D1 N）：

- [ ] D1.1 引用的法律 / 行政法规 / 司法解释 / 部门规章已 web_search 核对**现行有效版本**
- [ ] D1.2 引用版本与施行日期匹配（如"《民法典》第 577 条（2021.1.1 施行）"）
- [ ] D1.3 引用旧版本者已显式标注"已被 XX 修订"（如"《合同法》[已被 2021 民法典编入]"）
- [ ] D1.4 法条精确到**条款项**（不模糊到"《民法典》若干条"）
- [ ] D1.5 司法解释附完整文号（如"法释〔2020〕17 号"）
- [ ] D1.6 部门规章 / 规范性文件附完整文号（如"银发〔2026〕42 号"）
- [ ] D1.7 引用层级不混淆（区分法律 / 行政法规 / 部门规章 / 规范性文件 / 司法解释，不拔高）

**白名单引用源**（按 CLAUDE.md 引证源白名单）：
- PRC 法律 / 司法：gov.cn · npc.gov.cn · court.gov.cn · spp.gov.cn · 国务院各部委官网
- PRC 金融监管：nfra.gov.cn · pbc.gov.cn · csrc.gov.cn · safe.gov.cn
- HK：legislation.gov.hk · judiciary.hk · hklii.hk · sfc.hk · hkma.gov.hk · hkex.com.hk
- California / 联邦：leginfo.legislature.ca.gov · courts.ca.gov · law.cornell.edu · congress.gov · supremecourt.gov · courtlistener.com

**禁用源**：百度百科 / 微信公众号转载 / 内容农场聚合站 / AI 生成的法律咨询博客 / 未署名的二手摘抄。

### D2 案号 / 判例引用（硬核对）

**核查子项**：

- [ ] D2.1 案号格式正确（PRC：`[YYYY]+省+市+民初/初/终+号`；HK：`[YYYY] reporter cite`；US：Bluebook 格式）
- [ ] D2.2 案号实际存在已 web_search 核对（court.gov.cn / 仲裁机构官网 / courtlistener.com / hklii.hk）
- [ ] D2.3 引用判例的裁判要旨与原文一致（不歪曲、不断章取义）
- [ ] D2.4 引用层级正确（最高法院 vs. 中级 vs. 基层；不拔高）
- [ ] D2.5 引用判例的对照性已论证（与本案争议焦点相关）

**注意**：D2 不要求每个文书都有案号引用——如果文书内无案号 / 判例引用，D2 默认 Y（不适用）。

### D3 事实陈述一致性（软评估）

**核查子项**：

- [ ] D3.1 文书内事实陈述与上游 DocAnalyzer 解析结果一致
- [ ] D3.2 文书内事实陈述与 EvidenceAnalyzer 证据三性分析一致
- [ ] D3.3 时间线节点无矛盾（前后日期 / 顺序无冲突）
- [ ] D3.4 数字 / 金额前后一致（如诉求金额 = 损失计算 = 证据所示金额）
- [ ] D3.5 当事人陈述与代理人陈述一致（不互相矛盾）
- [ ] D3.6 与 matter.yaml / matter_dashboard.md 关键事实字段一致

### D4 时效计算（硬核对，民诉法 + 司法解释）

**核查子项**：

- [ ] D4.1 诉讼时效起算日 + 届满日已具体计算并 web_search 核对现行《民法典》第 188 条
- [ ] D4.2 上诉期已具体计算（国内 15 日 / 涉外 30 日，自送达之次日起算；现行民诉法第 175 条）
- [ ] D4.3 再审申请期已核对（判决/裁定生效后 6 个月内；现行民诉法第 209 条）
- [ ] D4.4 检察监督期已核对（裁判生效之日起 2 年内；民诉监督规则）
- [ ] D4.5 执行异议期已核对（15 日；民诉法第 232 条）
- [ ] D4.6 举证期已具体注明（按本案举证期限通知书 / 法院指定）
- [ ] D4.7 涉外延长期已主动评估（涉外 30 日 vs. 国内 15 日）
- [ ] D4.8 节假日顺延规则已考虑（最后一日为节假日的顺延至次工作日）
- [ ] D4.9 ≤30 天救济期限已触发响应顶部红色加粗预警（若适用）

### D5 程序节点（硬核对，民诉法 + 司法解释）

**核查子项**：

- [ ] D5.1 管辖法院级别正确（基层 / 中级 / 高级 / 最高，依案件性质 + 标的额）
- [ ] D5.2 管辖法院地域正确（合同纠纷 vs. 侵权纠纷管辖规则不同；民诉法第 24-35 条）
- [ ] D5.3 立案条件已核查（民诉法第 122 条）
- [ ] D5.4 程序前置已评估（再审需经二审；检察监督次于再审）
- [ ] D5.5 仲裁条款援引完整（仲裁条款号 + 仲裁机构全称；仲裁法第 16 条）
- [ ] D5.6 反诉牵连性论证已具体引用《民诉法解释》第 233 条
- [ ] D5.7 财产保全条款准确（诉前 103 / 诉中 104；现行民诉法）

### D6 主体清晰且统一（软评估）

**核查子项**：

- [ ] D6.1 全篇当事人姓名 / 公司全称一致（不混用"甲""乙公司""被告"等指代）
- [ ] D6.2 公司主体已注明统一社会信用代码（如有）
- [ ] D6.3 多原告 / 多被告时已注明诉讼地位（普通共同诉讼 vs. 必要共同诉讼）
- [ ] D6.4 代理关系链清晰（代理人 → 当事人 → 案件）
- [ ] D6.5 法定代表人 / 负责人 / 主要负责人 区分准确

### D7 内部逻辑一致（软评估）

**核查子项**：

- [ ] D7.1 诉求金额 = 损失计算列示总和（不舍入 / 不夹带未列示项目）
- [ ] D7.2 法律依据与事实主张对应（不引用与本案无关的法条充数）
- [ ] D7.3 证据清单编号与文书正文引用编号完全一致
- [ ] D7.4 段落间无相互矛盾（如"已履行义务" vs. "未履行义务"前后不一致）
- [ ] D7.5 请求权基础唯一明确（主请求 + 至多 2 备选，不模糊"违约/侵权择一主张"）
- [ ] D7.6 论证逻辑链完整（事实 → 构成要件 → 法律后果，不跳跃）

### D8 保密硬约束（zero tolerance）

> **任一子项 N → D8 N → 整体降级为 D 级，不论其他 7 维度如何。这是 CLAUDE.md 保密硬约束的实现。**

**核查子项**：

- [ ] D8.1 文书内未出现完整客户姓名（核查全文以 [原告] [被告] [当事人] 替代）
- [ ] D8.2 文书内未出现完整案号（核查以 [案号] 替代或仅在交付当事人副本中填回）
- [ ] D8.3 文书内未出现完整公司名称（核查以 [XX 公司] 替代）
- [ ] D8.4 未出现身份证号 / 护照号完整（最多保留末 4 位）
- [ ] D8.5 未出现联系方式 / 地址完整
- [ ] D8.6 未出现合同金额具体数字（替换为 [金额] 或区间词）
- [ ] D8.7 客户原文片段未在 web_search / web_fetch query 中泄露
- [ ] D8.8 涉港 / 涉跨境敏感事项未调用境内不可用的境外服务做内容处理

---

## 2. 评级阈值映射

| 评级 | Fail 维度数 | D8 状态 | 处理 |
|------|------------|---------|------|
| **A 优秀** | 0 fail | Y | pass，本轮完成 |
| **B 良好** | 1 fail（非 D8）| Y | pass，diagnostic notes 提示但不 retry |
| **C 合格但需修改** | 2-3 fail（非 D8）| Y | trigger retry，diagnostic notes 反喂 orchestrator |
| **D 不合格** | ≥4 fail，**或任意 D8 fail** | — | trigger retry |

**D8 special rule**：任意 D8 fail → 直接降级 D（不论其他维度评级）。Reviewer 在 Step 3 优先检查 D8，若 fail 立即降级，跳过其他 Step。

**评级计算示例**：
- 8 Y → A
- 7 Y / 1 N (非 D8) → B  
- 6 Y / 2 N (非 D8) → C
- 5 Y / 3 N (非 D8) → C
- 4 Y / 4 N (非 D8) → D
- 任意 D8 N → D（特殊降级）

---

## 3. auto-retry handshake 协议详细规范

### 触发流程

```
[orchestrator A 落盘] → 主 agent 自动调起 Reviewer →
  Step 1：Reviewer 接收 A 落盘文件 + 上游 context
  Step 2：Reviewer 读 .claude/rules/ReviewerRubric.md
  Step 3：D8 zero tolerance 优先检查
  Step 4：D1/D2/D4/D5 硬核对（web_search 白名单）
  Step 5：D3/D6/D7 软评估
  Step 6：生成 Y/N 矩阵 + Diagnostic notes
  Step 7：决定 pass / retry / escalate
    ├─ A/B → pass + 完成标识 + 返回 orchestrator → 流程结束
    └─ C/D → retry handshake：
         Reviewer 输出：
           - 当前评级 [C/D]
           - Y/N 矩阵
           - Fail 项 diagnostic notes（具体到子项 + 建议修订方向）
           - retry 计数 [1/2]
         主 agent 接收 Reviewer 输出 →
           主 agent 调起 orchestrator A 进入"retry 模式" →
             - 把 Reviewer 的 diagnostic notes 注入 A 的 Examine 步（v1.11.0b 嵌入的 3E）
             - A 在 Enhance 步针对 Reviewer 指出的 fail 项修订（不整体重写）
             - A 再次落盘
         主 agent 再次调起 Reviewer →
           Reviewer 再评 →
             ├─ A/B → pass + 完成标识 → 流程结束
             └─ C/D + retry < 2 → 再 retry handshake
             └─ C/D + retry == 2 → escalate（max-retry=2 硬上限）
```

### Retry 边界条件

- **max-retry = 2 硬上限**：第 3 次（即 retry 计数 2 后仍 fail）必出 escalate，不论 D 还是 C
- **D8 fail 在 retry 1 即可挽救**（修订脱敏后即可）；仍 D8 fail → 直接 escalate（保密 zero tolerance 不接受 retry）
- **同一 fail 维度连续 retry 2 次仍 N**：escalate 并明示"Reviewer 无法在 max-retry 内通过该维度"
- **Reviewer 评级不应反向**（如 retry 1 是 C，retry 2 不应回到 D）；若反向，说明 orchestrator 修订引入了新错误 → escalate

### Escalate 输出格式

```
## Reviewer ESCALATE（max-retry=2 已耗尽）

**当前评级**：[C / D]
**累计 retry 次数**：2
**累计 Y/N 矩阵演变**：
  retry 0：[Y/N/Y/N/Y/Y/N/Y] → D
  retry 1：[Y/Y/Y/N/Y/Y/N/Y] → C
  retry 2：[Y/Y/Y/N/Y/Y/N/Y] → C （未改善）
**累计 fail 维度**：D4 时效计算 / D7 内部逻辑
**累计 fail 子项**：
  - D4.2 上诉期计算错误（已 retry 但仍误算）
  - D7.4 诉求金额与损失列示不一致

**升级建议**：
[1] 用户裁定（最常见）：本 fail 项是否本案不适用 / 是否接受当前版本
[2] 由资深律师人工复核
[3] 推迟落盘并补充 context 再启动新一轮

**当前最佳版本**：[落盘路径，未撤回]
```

---

## 4. Diagnostic Notes 输出格式

Reviewer 响应末尾必含"## Reviewer 评估结果"段：

```
## Reviewer 评估结果

**评级**：[A / B / C / D]
**累计 retry 次数**：[0 / 1 / 2]
**最终决策**：[pass / retry / escalate]

### Y/N 矩阵

| 维度 | 类型 | Y/N | 子项 fail 数 |
|------|------|-----|------------|
| D1 法条引用 | 硬核对 | [Y/N] | [0/N] |
| D2 案号 / 判例引用 | 硬核对 | [Y/N] | [0/N] |
| D3 事实陈述一致性 | 软评估 | [Y/N] | [0/N] |
| D4 时效计算 | 硬核对 | [Y/N] | [0/N] |
| D5 程序节点 | 硬核对 | [Y/N] | [0/N] |
| D6 主体清晰 | 软评估 | [Y/N] | [0/N] |
| D7 内部逻辑 | 软评估 | [Y/N] | [0/N] |
| D8 保密硬约束 | zero tolerance | [Y/N] | [0/N] |

### Fail 项 Diagnostic Notes（按维度展开）

#### D[N] [维度名] - fail

**子项 fail**：
- [子项编号] [子项内容简述] - [fail 理由]
- ...

**fail 理由具体描述**：[详细说明哪一处出错、为什么错、对法律效果的影响]

**建议修订方向**：
- [具体修订指令 1，可执行]
- [具体修订指令 2，可执行]
- [是否需 web_search 重新核对，如是给出 query 建议]

---

### Retry 决策

- A/B → pass，本轮完成
- C/D → 触发 retry [1 / 2]，本 diagnostic notes 反喂 orchestrator agent [Writer / ContractReviewer / ...] 进入 retry 模式
- 第 3 次仍 C/D → escalate 用户裁定（max-retry=2 硬上限）

**本次决策**：[pass / retry 1 / retry 2 / escalate]
```

---

## 5. 与 v1.11.0b 3E 自检的衔接

- **v1.11.0b orchestrator 内嵌 3E**：agent 内部一次性自检（max_iter=1），拦截**低级错误**（漏字段 / 与上游产物冲突 / skill 调起错误）
- **v1.11.0c Reviewer**：跨 agent 一致性 + 引用源 web_search 核对，抓**高级错误**（法条版本错 / 案号编造 / 时效误算 / 保密泄露）

**两层防御不重叠原则**：
- 3E Examine 已核查过的项（如 cn-litigation-drafting QC 自检 + Writer 3E Examine W4 "QC 自检结果段已读"），Reviewer **不重复核查**该项是否被读取，但**仍核查其内容是否合规**（如 QC 自检结果段是否含 fail 项 / 是否如实呈现）
- D 维度按文书类型不同重点不同（详见 Reviewer.md "审查范围（按 agent 类型）"表）

**N\* 接管职责**：上游 orchestrator agent 因线程无 WebSearch 而对硬核对项标 **N\*** 并 escalate 时，Reviewer 是其接管层——必须对每个 N\* 项实际执行 D1/D2/D4/D5 白名单 web_search 双源核对，产出可定稿结论（精确现行条号/文号/期限），完成闭环。N\* 项经 Reviewer 补核后，若仅属工具边界（非实质缺陷）不计为质量 fail。完整协议（定义/适用条件/记法/反例/闭环判据）见 [`NStarProtocol.md`](./NStarProtocol.md)。

---

## 6. 与既有 OutputStandards.md 的关系

- 本 rubric **不取代** OutputStandards.md 的文件命名 / 格式规范——D7 内部逻辑 + D8 保密硬约束部分子项与 OutputStandards.md 配套
- D1-D8 是**质量层 rubric**，OutputStandards.md 是**形式层规范**；Reviewer 同时核查两层（形式层 fail 通常归入 D7 内部逻辑）

---

## 7. 变更历史

| 版本 | 日期 | 更新内容 |
| :--- | :--- | :--- |
| v1.0 | 2026-05-14 | v1.11.0c 初始版本：8 维度 Y/N rubric + 50 子项 + auto-retry 协议 + max-retry=2 |
