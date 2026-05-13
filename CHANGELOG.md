# 变更记录

> Last updated: 2026-05-08
> 所有对用户或其他协作者有影响的变更都会在此记录。使用 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.0.0/) 格式。

---

## [v1.10.1] - 2026-05-11 — 案件文件夹重命名工序 + .gitignore 兜底

### ➕ 新增 (Added)

#### new-case skill 扩展为 dual-mode（重大更新）

- **Mode 1 创建模式**（已有）：从空白生成新案件框架
- **Mode 2 重整理模式**（新增）：对已有案件文件夹做合规检查 + 必要时 mv 重命名 + 内部结构整理
  - 现状扫描 → 5 项合规检查（YYMMNN / 空格 / 与 / 简称 / 案由）→ 缺字段人 in the loop 询问 → 生成 mv 命令 → 用户确认后执行 → 内部结构整理 → matter.yaml 字段更新
  - **不自动 mv**（保密硬约束 + 客户案件不可逆操作）

#### 严格命名规范

新增 `.claude/rules/OutputStandards.md` v1.7 "案件文件夹命名规范"大段：

- 格式：`{YYMMNN} {原告简称} 与 {被告简称} {案由}`
- **YYMMNN 严格 6 位**（年 2 + 月 2 + NN 2）
- **NN 自然顺序号规则**：本所当月内案件按收案时间递增；每月独立起算；不跳号；一旦使用不回收
- 特殊场景变体：行政诉讼用"诉"、婚姻家事保留双名、多原告/多被告用"等"、仲裁、涉外
- 6 个标准例 + 6 个反例修正示例
- 一键合规检查 bash 脚本

#### 新增 /organize-case 命令

`.claude/commands/organize-case.md`：调起 new-case skill Mode 2 重整理模式

- 触发：用户自然语言"整理这个案件 / 重命名案件 / 案件归一化"，或显式 `/organize-case [folder-path]`
- 自动 dispatch 到 new-case Mode 2
- 完整使用示例（含 260507 重命名实战 demo）

### 🔄 调整 (Changed)

- **`.gitignore` 案件文件夹兜底**（在 v1.10.0 commit 中已含；本 v1.10.1 仅补 patch 说明）：
  - 原 `[0-9][0-9][0-9][0-9]*/` 仅匹配 YYMMNN 数字前缀
  - 新增 14 个兜底模式：`*纠纷/` / `*离婚/` / `*案件/` / `*合同案/` / `*诉*/` / `* 与 */` / `* 对 */` / `* V. */` 等
  - 验证 4 案件文件夹（260507 / 260508 / 260509 / 王荣 劳动纠纷）全部 ignored，vscode-extension/ 无误伤
- `OutputStandards.md` v1.7：新增"案件文件夹命名规范"大段（约 100 行）+ v1.7 变更历史
- `CHANGELOG.md` v1.10.1

### 📋 仓库外配套（tmp/ 不入仓）

- `tmp/cn-litigation-case-folder-organizer_patch_v2.md`：外置 organizer skill 的 Rename 阶段 patch（8-stage workflow，新增 Stage 3.5 Rename）
- `tmp/rename-existing-cases.md`：4 个现有案件的一对一重命名建议（含 mv 命令模板 + 字段空位）

### 📐 设计要点

- **dual-mode skill 设计**：new-case 一个 skill 处理两种相关但不同的场景（创建 + 重整理），避免新建独立 skill 造成的功能重复（与外置 organizer skill 既配套又区隔）
- **NN 自然编号实施**：扫描项目根目录 YYMMNN 文件夹，取当月最大 + 1；即使中间跳号也按最大值递增（不回填）
- **保密硬约束**：4 案件的重命名建议在 tmp/（不入仓）；mv 命令模板含字段空位（`[对方简称]`）由用户本机手工填入
- **/organize-case 命令与 new-case Mode 2 的关系**：命令是触发入口，skill 是方法论实现；用户既可以直接说"整理这个案件"触发，也可以显式 `/organize-case <path>` 触发

### 🚫 不在 v1.10.1

- 外置 organizer skill 的自动同步（在 read-only mount，用户本机手动 patch）
- 4 个现有案件的自动 mv（必须人 in the loop + 字段空位需用户填）
- NN 编号的全局唯一性强制（仍允许跳号；多月间编号独立）

---

## [v1.10.0] - 2026-05-10 — PRC 实战工具实化（G + F + L 合并 phase）

### ➕ 新增 (Added)

#### 2 个新 agent

- **`.claude/agents/TrialPrep.md`** (gold)：庭审准备编排器（orchestrator 模式）
  - 触发：开庭日期 ≤ 3 周；或用户明示"庭审准备 / 庭前准备 / 庭审提纲 / 出庭准备"
  - 调起 `cn-trial-preparation` skill 输出 4 份庭审实战工具
  - 落盘：`02 - 法律研究/案件分析/庭前准备/`
  - 与 DocAnalyzer（庭审后处理）/ Strategist（策略层）边界明确
- **`.claude/agents/Postmortem.md`** (violet)：案件结案复盘编排器（orchestrator 模式）
  - 触发：matter.yaml 阶段为"已结案"；或用户明示"结案 / 复盘 / 归档"
  - 调起 `cn-case-postmortem` skill 完成 5-stage workflow
  - **关键能力**：Stage 4 Distill 触发**人 in the loop**——memory 沉淀草稿等用户明确确认后才写入对应 skill 的 memory.md（保密硬约束 zero tolerance）
  - 落盘：`99 - 复盘沉淀/`；matter.yaml 字段更新为"已结案归档"

agent 总数：13 → 15

#### 3 个新内置 skill

- **`.claude/skills/cn-trial-preparation/`**（5 文件 / 573 行）：庭审准备方法论
  - 4-stage workflow（Prepare / Build / Discuss / Execute / Learn）
  - references：`trial-procedure-cn.md`（PRC 民事庭审 4 阶段）、`witness-questioning-techniques.md`（主询问 / 反询问 / 重新询问 / 弹劾路径）、`oral-argument-template.md`（争点对抗预演 + 法庭辩论 + 最后陈述）、`evidence-presentation-strategy.md`（证据出示顺序与策略）
- **`.claude/skills/cn-client-communications/`**（5 文件 / 727 行）：律所对客户日常沟通文书
  - 5 类文书：周报 / 月报 / 阶段性总结 / 风险预警（红 / 橙 / 黄三级）/ 决策建议书 / 客户问询回复
  - 与 cn-firm-documents（**正式**文书）边界明确——本 skill 仅做**ongoing 非正式**沟通
  - 决策建议书**不重复 LegalOpinion 法律分析**——仅引用既有结论 + 给执行建议
  - references：`progress-report-template.md` / `stage-summary-template.md` / `risk-alert-template.md` / `decision-recommendation-template.md`
  - 落盘：`10 - 综合报告/客户沟通/`
- **`.claude/skills/cn-case-postmortem/`**（4 文件 / 625 行）：案件结案复盘方法论
  - 5-stage workflow（Prepare / Recap / Analyze / Improve / Distill / Archive）
  - references：`postmortem-template.md`（完整复盘报告模板）、`win-loss-analysis-framework.md`（5 维度胜败原因分析：法律层 / 事实层 / 程序层 / 策略层 / 资源层 + 反事实推演）、`memory-distillation.md`（人 in the loop 沉淀指引 + 脱敏检查清单）

3 个 skill 全部受项目根 LICENSE（GNU AGPL v3）约束。

### 🔄 调整 (Changed)

- **Writer.md** 加第三类文书路由：律所对客户**日常**沟通文书 → `cn-client-communications` skill
- **Workflow.md**：
  - 新增 TrialPrep + Postmortem 触发块（含消歧规则）
  - 新增场景 10（庭审准备）+ 场景 11（结案复盘与归档）
  - skill 桥接表加 3 行（cn-trial-preparation / cn-client-communications / cn-case-postmortem）
  - Writer 触发关键词加客户日常沟通词（周报 / 月报 / 风险预警 / 决策建议书等）
- **AgentMapping.md** v3.5：
  - 输出层加 TrialPrep；支持层加 Postmortem
  - Writer 段加 cn-client-communications 路由说明
  - 反向映射加 `02 - 法律研究/案件分析/庭前准备/`、`10 - 综合报告/客户沟通/`、`99 - 复盘沉淀/`
- **AGENTS.md**：
  - 系统定位 13 → 15 个 agent
  - 项目内置 skill 表加 3 行
  - skill → agent 对照表加 TrialPrep / Postmortem / cn-client-communications 映射
- **SubagentStandards.md** v3.2：color 表扩到 15（+ gold for TrialPrep / + violet for Postmortem）
- **OutputStandards.md** v1.6：标准输出表加 TrialPrep（4 份）+ Postmortem（3 份）行；Writer 行扩展含客户日常沟通
- **README.md**：13 → 15 Subagent；架构图加 TrialPrep + Postmortem；skill 表加 3 行；变更轨迹加 v1.10.0
- **CHANGELOG.md** v1.10.0

### 📐 设计要点

- **TrialPrep 在无开庭实战时仍可用作方法论沉淀**：开庭前不一定每次都触发；当前真实案件（260507 / 260508）尚未开庭，本 skill 暂为方法论占位，等下次开庭实战时校准
- **F 与 cn-firm-documents 的边界**（确认决策）：律师函 / 委托代理协议 / 法律意见书等正式文书归 cn-firm-documents（外置）；周报 / 月报 / 风险预警 / 决策建议书等日常沟通归 cn-client-communications（内置）。决策建议书**仅引用** LegalOpinion 结论，不重复法律分析
- **TrialPrep 落盘路径**（确认决策）：`02 - 法律研究/案件分析/庭前准备/`（作为案件分析子产物，而非 `08 - 庭审笔录/` 同级 sibling）
- **memory 自动沉淀的人 in the loop**（确认决策）：Postmortem agent Stage 4 显式输出"沉淀草稿"等用户明确确认后才写入；用户回复必须明确含"同意 / 全部同意 / 部分同意（指定哪些）/ 重做"，模糊回复（如"看起来还行"）不得据此判定为同意
- **保密硬约束 zero tolerance**：cn-case-postmortem/references/memory-distillation.md 含完整脱敏检查清单（9 项）+ 抽象 4 步法 + 合规 / 不合规示例

### 🚫 不在 v1.10.0

- E（Researcher 工具实化）/ J（跨 matter 知识库）→ v1.11.0+（系统级基础设施，需要前期 case 数据积累后做才有用）
- K（Workflow DAG 重构）→ 仅触发时做（YAGNI）
- H（IP 诉讼）/ I（数据合规）→ 用户当前不办相关案件，非紧急需求
- A/B/C/D（跨境扩展）→ PRC 完善阶段稳定后再做

### 🎯 v1.10.0 后系统状态

- **15 agent**（4 个 v1.10.0+ 新增 / 11 个原有）
- **8 项目内置 skill**（含 v1.10.0+ 新增 3 个：cn-trial-preparation / cn-client-communications / cn-case-postmortem）
- **1 外置必需依赖 skill**（cn-firm-documents）
- **1 外置可选 skill**（cn-litigation-case-folder-organizer）

---

## [v1.9.0] - 2026-05-09 — Phase 6: 4 个 legal skill 内置到项目（含 BREAKING）

### ⚠️ 破坏性变更 (Breaking)

- **4 个核心 legal skill 直接内置到 `.claude/skills/`**：克隆仓库即可用，无需另行从用户全局 skill 库同步
  - `cn-litigation-drafting`（11 模板版，A-K）：从 Phase 1 提案稿迁入
  - `cn-jiubufa-case-analysis`（SKILL.md + 3 references）：从用户全局 skill 库迁入
  - `cn-judgment-analysis`（SKILL.md）：从用户全局 skill 库迁入
  - `cn-contract-review`（v1.8.0+ 统一版，92 个 .md）：从 `tmp/cn-contract-review_unified_proposed/` 迁入
- 仅 `cn-firm-documents` 因含具体律所名/对客户文书规则，**保持外置**
- 用户本机用户级 4 个旧 cn-contract-review-* specialized skill 与 4 个内置 skill 同名同位时，须手动废旧（保留 30 天备份后删）

### 🔧 Skill 内 Chris 标识 scrub

为支持公开 GitHub 发布与同事 fork，`cn-contract-review` 内 86 个含 "Chris" 字面的文件已批量 scrub：
- `personal-preferences.md`：标题"当前审查人：Chris" → "默认审查偏好（项目继承自原作者，可由实际审查人覆盖）"
- 80 个 contract-types/*.md 第 8 行 v1 marker：`chris-patterns` → `personal-preferences`
- 4 个框架文件 v1 marker：`v1.1 待 Chris 复核` → `v1.1 待项目维护者复核`
- LICENSE.txt：Copyright 由 `Chris (lem923/SuitAgent fork)` 改为 `SuitAgent contributors`

**保留**：所有具体方法偏好（背靠背付款 / 通知送达 / 数据安全 / CNAS-CMA / 跨境法域选择等）作为项目默认设置；**默认审查偏好继承自原作者 Chris 的实战经验**，接手者可在 `personal-preferences.md` 覆盖。

### 📋 License 处理

`cn-contract-review` 是**双 license** skill（详见 `.claude/skills/cn-contract-review/NOTICE.md`）：
- 60 个继承自 contract-copilot v1.5.1 的文件：受 **CC BY-NC 4.0** 约束（保留原 LICENSE.txt 作为下位许可）
- 26 个 SuitAgent contributors 原创文件 + NOTICE.md：受**项目根 LICENSE（GNU AGPL v3）**约束

其他 3 内置 skill（`cn-litigation-drafting` / `cn-jiubufa-case-analysis` / `cn-judgment-analysis`）全部受项目根 AGPL v3 约束，各 SKILL.md 末尾加 License 注释指向项目根 LICENSE。

### 🔄 调整 (Changed)

- **AGENTS.md**：
  - "外部 skill 桥接" 必需依赖表 6 行 → 1 行（仅保留 `cn-firm-documents` 外置）
  - "项目内置 skill 关系"表加 4 行（cn-litigation-drafting / cn-contract-review / cn-jiubufa-case-analysis / cn-judgment-analysis）
  - skill 直接调用对照表标注每个 skill 的"项目内置/外置"位置
- **Workflow.md**：skill 桥接表加"位置"列（项目内置 vs 外置）
- **AgentMapping.md** v3.4：新增"项目内置 4 个 legal skill"段
- **4 个 orchestrator agent**（Writer / ContractReviewer / JiubufaAnalyst / JudgmentAnalyzer）：必需依赖描述加项目内置路径标注
- **README.md**：
  - "外部 skill 依赖" 段重写为"skill 依赖"
  - 项目内置 skill 表（5 行：4 内置 legal + docx 系列工具）
  - 外置 skill 表（2 行：cn-firm-documents + cn-litigation-case-folder-organizer）
  - 每行加 License 列
  - 变更轨迹加 v1.9.0
- **CHANGELOG.md** v1.9.0 BREAKING

### 📐 设计要点

- **公开 GitHub 暴露 cn-contract-review 完整方法论**：80+ 个 contract-types/ + 14 类合同审查口径全部进入公开仓库；CC BY-NC 4.0 限制商业再分发，AGPL v3 要求 fork 同样开源
- **同事 fork 易用性**：内置后 clone 即用；`personal-preferences.md` 标题改为"默认审查偏好"便于同事改名为自己的口径
- **License 兼容性**：项目根 AGPL v3 与 contract-copilot 上游 CC BY-NC 4.0 在同一文件不能合并 → 通过 NOTICE.md 显式分组保留各自约束
- **git log author 字段**：本次 commit 仍以 `Chris <hetsong@gmail.com>` 签署（git 历史无法匿名）；skill 文件内不出现"Chris"

### 🚫 不在 Phase 6

- 用户本机废旧 4 个旧 cn-contract-review-* specialized + cn-litigation-drafting / cn-jiubufa-case-analysis / cn-judgment-analysis 全局 skill（用户本机操作；建议先备份再删）
- v1.1 本土化校对（80 个 content 文件中 contract-copilot 继承的 60 个）—— 留待实战触发后逐项校对

---

## [v1.8.0] - 2026-05-09 — Phase 5: 合同审查 skill 统一（4 → 1，含 BREAKING）

### ⚠️ 破坏性变更 (Breaking)

- **`ContractReviewer` agent 的必需依赖从 4 个 cn-contract-review-* specialized skill 改为单一统一的 `cn-contract-review` skill（v1.8.0+）**：
  - 原 4 个 specialized 已弃用：`cn-contract-review-universal` / `cn-contract-review-gov-tech-dev` / `cn-contract-review-gov-tech-licensing` / `cn-contract-review-labor-employment`
  - 新统一 skill 覆盖 14 类合同（增加原 specialized 未覆盖的：买卖 / 租赁 / 服务 / 担保 / 借贷赠与 / 互联网协议 / 婚姻家事 / 房地产 / 建设工程 / 公司投资）
  - 14 类路由由 skill 自身处理；ContractReviewer agent 不再复制路由逻辑

### 🔄 调整 (Changed)

- **`.claude/agents/ContractReviewer.md`**：
  - frontmatter description 改为依赖单一 skill
  - 删除"自动路由判断逻辑"段（路由迁到 skill 内部 `references/orientation-and-dispatch.md`）
  - 改为"调用 skill 的路由信息"段（仅作 14 类摘要参考）
  - 工作流程 Step 2-4 改为调起 `cn-contract-review` skill
  - re-entry 规则 / 重要提醒 / 完成标识中所有 `cn-contract-review-*` 引用改为单一 `cn-contract-review`
- **`.claude/rules/AgentMapping.md` v3.3**：ContractReviewer 段功能描述更新；变更历史加 Phase 5 条目
- **`.claude/rules/Workflow.md`**：
  - ContractReviewer 触发块的内部分流改为调起单一 skill
  - skill 桥接表 4 行 → 1 行
  - 场景 8（合同审查）说明同步更新
- **`AGENTS.md`**：
  - 必需依赖表 4 行 cn-contract-review-* → 1 行 cn-contract-review，附完整 14 类描述
  - skill 直接调用对照表更新
  - 第 5 行系统定位 → 13 个 agent（数量未变，描述微调）
- **CHANGELOG.md** v1.8.0 BREAKING 条目

### 📋 仓库外配套（Phase 5 不入仓）

- **`tmp/cn-contract-review_unified_proposed/`**：完整的统一 skill 提案稿
  - 92 个 .md 文件 / 约 7075 行
  - 14 类 contract-types/ 子目录（80 个 content 文件）：12 类继承自 contract-copilot v1.5.1（已 P0/P1/P2 → REDLINE/ORANGE/YELLOW 转换 + v1.1 校对标记），2 类（01-universal / 14-gov-procurement）+ 各类目专属内容（5/9/16/5）来自 4 个原 specialized skill 注入
  - 10 个框架 references/*.md：orientation-and-dispatch / review-framework / revision-strategy / deliverable-format / playbook（v1 仅骨架）/ negotiation-patterns / presign-checklist / qc-checklist / cross-border-review / personal-preferences
  - 1 个 SKILL.md（202 行，4-stage workflow + 14 类路由 + REDLINE/ORANGE/YELLOW 等级 + fallback 三档）
  - 1 个 memory.md（按 14 类合同分节的统一审查经验库）
  - 1 个 LICENSE.txt（CC BY-NC 4.0，与 contract-copilot 上游一致）
- **用户本机迁移路径**：
  1. 复制 `tmp/cn-contract-review_unified_proposed/` 到 `~/Library/Application Support/Claude/.../skills/cn-contract-review/`
  2. 测试 1-2 份合同跑通
  3. 弃用旧 4 个 specialized skill（保留备份 30 天）
  4. v1.1 阶段对 80 个 content 文件做本土化复核（v1 marker 已加在各文件顶部）

### 📐 设计要点

- **方法论吸收**：contract-copilot v1.5.1 的 12 类 contract-types/ + review-framework + revision-strategy；Claude `legal:review-contract` 的 playbook + fallback positions + Negotiation tier framework + Business Impact Summary
- **Chris 偏好保留**：4-stage workflow（Prepare→Review→Discuss→Execute→Learn）100% 保留；REDLINE/ORANGE/YELLOW + GREEN 体系 100% 保留；7-section deliverable format 保留；docx skill 调用路径保留
- **fallback positions 强约束**：REDLINE / ORANGE 必填三档（目标 / 可签 / 底线）；YELLOW 不强制
- **playbook v1 骨架**：未填字段降级到 checklist 隐式立场，不退化；同事 fork 后填自己的标准立场
- **personal-preferences.md**（重命名自 chris-contract-review-patterns.md）：通用化模板，便于同事拿走改成自己的偏好
- **License CC BY-NC 4.0**：律所同事可任意使用 / fork / 转载，禁止商业再分发

### 🚫 不在 Phase 5

- 真实合同审查 v1 完整实战测试（用户本机执行）
- v1.1 本土化校对（80 个 content 文件中 contract-copilot 继承的 60 个的本土化复核 + 与 4 specialized 注入的 20 个去重）
- 4 个旧 specialized skill 的清理（用户本机操作，保留备份后再删）

---

## [v1.7.0] - 2026-05-09 — Phase 4: 强化触发与路由（4-phase 整合计划收官）

### 🔧 Bucket A · Hard fixes（路由精度修正）

- **Workflow.md 关键词冲突消歧**（5 处）：
  - Writer：`再审申请` / `检察监督` 触发条件加意图限定（仅起草文书时命中；评估可行性走 JudgmentAnalyzer）
  - Writer：`质证意见` 仅"写/起草"前缀触发；裸"质证意见"走 EvidenceAnalyzer
  - EvidenceAnalyzer：`质证意见` 加条件注释（无"写/起草"前缀才命中）+ 我方/对方证据划分说明
  - Strategist：删除裸词 `案件评估`（深度场景走 JiubufaAnalyst）+ 上下游 handoff 边界
  - JudgmentAnalyzer：与 Writer 边界明示（评估可行性 vs 起草文书）
- **DocAnalyzer 触发词收紧**：删 `合同分析` / `协议分析`（应归 ContractReviewer）；裸 `识别` 改为 `OCR识别` / `图片识别` / `扫描件识别`（避免误命中"识别争议焦点"）；加消歧段说明与 ContractReviewer / JudgmentAnalyzer 的上下游关系
- **Reviewer.md 审查范围扩充**：补 ContractReviewer / JiubufaAnalyst / JudgmentAnalyzer 三个 agent 的检查项（共 12 个新检查项）；加"orchestrator 模式审查规则"（审落盘文件，不审 skill 内部）；触发机制补 Phase 2+ 新输出类型
- **ContractReviewer.md re-entry 规则**：4-stage 工作流的 Discuss → Execute 续接路由——用户后续说"继续/执行/出红线版"时直接进 Execute，不重走 Step 1-3；含孤儿文件检测提示（review_complete 但无红线 DOCX）
- **AGENTS.md skill 直接调用原则**：用户显式提及 skill 名时主 agent 仍通过包装 agent 调用，不旁路；含 skill → agent 对照表（6 项）

### 🎨 Bucket B · description 升级

- **Researcher.md** description 加 `覆盖：` 关键词列表：法条/判例/司法解释检索、pkulaw 北大法宝、威科先行、中国裁判文书网、search-first 引用源白名单
- **Scheduler.md** description 加 `覆盖：` 关键词列表：法定期限（含上诉 15 日 / 再审 6 月 / 检察监督 2 年 / 执行异议 15 日）、matter.yaml 关键日期更新、工时记录.md 累加
- 其他 7 个老 agent 不动（关键词密度足够 + Workflow 路由表已覆盖）

### 🚫 不在 Phase 4

- 触发块格式统一（老式引号 vs 新式分类，纯 cosmetic 不影响功能命中）
- 真实案件 260507 / 260508 迁移（仍是用户本机操作）
- skill 内部内容（read-only + 框架良好）
- 进一步重构（4 phase 已足够，未来增量改动通过常规 PR）

### 🎯 4-phase 整合计划全景（收官）

```
v1.7.0 phase4   强化触发与路由（路由精度 + Reviewer 覆盖 + skill 入口标准化）
v1.6.0 phase3!  命名规范清理（JudgmentReviewer → JudgmentAnalyzer BREAKING + 跨引用）
v1.5.0 phase2c  JiubufaAnalyst + JudgmentReviewer (改名 JudgmentAnalyzer) agent
v1.4.0 phase2b  ContractReviewer agent
v1.3.0 phase2a! 案件目录结构统一为 14-slot（11 numbered + matter triplet）
v1.2.0 phase1   Writer 与 cn-litigation-drafting 合并 + advisory
```

最终态：13 agent（4 orchestrator + 9 内嵌方法）+ 8 外部必需依赖 skill + 11-slot 案件目录 + matter triplet + 工时记录.md。

---

## [v1.6.0] - 2026-05-08 — Phase 3: 命名规范与跨引用清理（含 BREAKING）

### ⚠️ 破坏性变更 (Breaking)

- **`JudgmentReviewer` agent 重命名为 `JudgmentAnalyzer`**：套用 DocAnalyzer / EvidenceAnalyzer 的 `-Analyzer` 后缀风格，彻底消除与 SuitAgent 既有 `Reviewer`（QA 质量审查器）的字符重叠
  - 文件：`.claude/agents/JudgmentReviewer.md` → `.claude/agents/JudgmentAnalyzer.md`
  - frontmatter：`name: judgment-reviewer` → `name: judgment-analyzer`
  - 跨文件引用全部同步更新（AgentMapping / Workflow / OutputStandards / AGENTS / DocAnalyzer / Strategist 共 6 个文件）
  - CHANGELOG 历史条目（v1.5.0）保留旧名作为变更记录，正文不动

### 🔧 Bucket A · Hard violations 修正

- `.claude/commands/evidence-review.md` 补 YAML frontmatter（违反 CommandMeta.md 已修）
- `DocAnalyzer.md` / `EvidenceAnalyzer.md` / `Strategist.md` 共 6 处死链修正：`.claude/rules/...` → `../rules/...`（agents/ 内 .md 的相对路径）
- `Writer.md`：'12 层映射' → '11-slot 目录映射'（Phase 2A 残留术语）
- `deepresearch.md`：'12层目录集成' → '11-slot 目录集成'
- `SubagentStandards.md` 第 258、287 行：'12 层结构规范' → '11-slot 目录结构规范'
- `README.md`：'10 个 Subagent' 小节升级为 13 个 agent 表格 + 系统架构图升级（含 JiubufaAnalyst / JudgmentAnalyzer / ContractReviewer 三个 Phase 2 新增 agent）
- `AGENTS.md` 第 5 行系统定位：'10 个专业 Agent' → '13 个专业 agent'

### 🎨 Bucket B · 软整理（cosmetic 一致性）

- **color 字段去重**：13 agent 颜色互不重复
  - 调整：`Scheduler` blue → teal、`Reviewer` purple → gray、`ContractReviewer` orange → magenta、`JiubufaAnalyst` purple → indigo、`JudgmentAnalyzer`（原 JudgmentReviewer）red → brown
- **tools 字段顺序统一**：所有 agent 改为 `Read, Write, Edit, Bash, Grep, Glob`（文件操作优先 → shell → 搜索 → 全局匹配；Researcher 末尾追加 WebSearch / WebFetch）
- `SubagentStandards.md` v3.1：新增 tools 字段顺序约定与 color 字段去重约定（13 agent 配色表纳入规范）

### 🚫 不在 Phase 3

- description 风格升级（老 agent 短语连缀 vs 新 orchestrator 结构化）→ Phase 4 与触发关键词改造一起
- new-case skill 内部 references 文件清理（属 skill 内部，不在本范围）
- CHANGELOG 历史条目（v1.5.0 及更早，保留旧术语作为变更记录）
- `JiubufaAnalyst` 拼音名（对中国诉讼律师反而直觉，不改）
- `Reviewer` agent 命名（Survey 评估混淆风险低，已在 JudgmentAnalyzer 顶部明示边界，不动）

---

## [v1.5.0] - 2026-05-08 — Phase 2C: 建 JiubufaAnalyst + JudgmentReviewer agents

### ➕ 新增 (Added)

- **`.claude/agents/JiubufaAnalyst.md`**：要件审判九步法分析师（orchestrator 模式），调起 `cn-jiubufa-case-analysis` skill 完成 9 步结构化分析（请求权基础穷举 → 构成要件归入 → 举证责任矩阵 → 证据缺口清单 → 胜诉概率区间）
- **`.claude/agents/JudgmentReviewer.md`**：裁判文书深度审查器（orchestrator 模式），调起 `cn-judgment-analysis` skill 完成 5 步评审（文书画像 → 争议反向还原 → 证据认定拆解 → IRAC 重建 → 救济路径概率评估）。**与 SuitAgent 既有 `Reviewer`（QA 质量审查器）功能完全不同**——两者命名相似但职能正交：JudgmentReviewer 是法律层评审；Reviewer 是 cross-agent QA
- **新工作流场景 9（判决书深度评估）**：DocAnalyzer → JudgmentReviewer → Strategist → 可选 Writer（按选定救济路径起草上诉/再审/监督文书）
- **救济路径时效预警**：JudgmentReviewer 必查项，对各路径法定时效（民事 15 日上诉、再审 6 月、检察监督 2 年、执行异议 15 日）主动核查并在响应中预警

### 🔄 调整 (Changed)

- **IssueIdentifier.md / Strategist.md / DocAnalyzer.md** 的 Phase 1 advisory 升级为"hand off to agent"模式：不再直接调 skill，改为按阈值 hand off 给对应 orchestrator agent（JiubufaAnalyst / JudgmentReviewer）
  - `IssueIdentifier`：深度场景 → hand off to **JiubufaAnalyst**
  - `DocAnalyzer`：post-judgment 场景 → hand off to **JudgmentReviewer**
  - `Strategist`：庭前 SWOT 深度场景 → 期待上游 JiubufaAnalyst 底稿；再审/监督场景 → hand off to JudgmentReviewer
- **Workflow.md 工作流场景 1 / 3 / 6 升级**：在 IssueIdentifier 后插入 JiubufaAnalyst 节点（复杂阈值命中时触发）；DocAnalyzer 后插入 JudgmentReviewer 节点（post-judgment 时触发）
- **AgentMapping.md v3.2**：分析层加 JiubufaAnalyst + JudgmentReviewer；反向映射 `02 - 法律研究/案件分析/` 列加两个新 agent
- **OutputStandards.md v1.5**：标准输出表加 JiubufaAnalyst + JudgmentReviewer 行
- **AGENTS.md 必需依赖表**：`cn-jiubufa-case-analysis` 与 `cn-judgment-analysis` 从 Advisory 升级为**必需依赖**（被两个新 agent 必需调起）

### 📐 设计要点

- **JiubufaAnalyst 不替代 IssueIdentifier**：轻量场景下 IssueIdentifier 的 4-6 个争议点列表已足够；只有触发阈值（请求权 ≥3 / 起诉答辩前置 / 再审监督评估前置）命中时才 hand off 到 JiubufaAnalyst
- **JudgmentReviewer 不替代 DocAnalyzer**：纯归档/检索/事实摘录由 DocAnalyzer 完成；post-judgment 法律层评审才 hand off
- **JudgmentReviewer 与 Reviewer 命名说明**：两 agent 文件顶部均明示"职能不同"以避免协作者混淆
- **per-case AGENTS.md 控制开关**：每案 matter.yaml 的 `agent_behavior` 字段可关闭九步法 / 判决书评审等深度功能（默认全开）
- **search-first 强约束**：JudgmentReviewer 涉及法定时效引用必须 web_search 核对现行有效版本（参 CLAUDE.md），不凭训练数据

### 🚫 不在 Phase 2C

- 改 cn-jiubufa-case-analysis / cn-judgment-analysis skill 内容（read-only + 框架良好）
- 实质性 Workflow 路由机制重构（→ Phase 4，本 Phase 仅做加 trigger 关键词与场景节点的浅层调整）
- 命名规范统一（Reviewer / JudgmentReviewer 命名相近但暂不改 → Phase 3 决定）

## [v1.4.0] - 2026-05-08 — Phase 2B: 建 ContractReviewer agent

### ➕ 新增 (Added)

- **`.claude/agents/ContractReviewer.md`**：合同审查编排器（orchestrator 模式），thin wrapper 分流 4 个 cn-contract-review-* skill
  - 政企技术采购 / 委托开发 / 系统集成 → `cn-contract-review-gov-tech-dev`
  - 专利 / 软著 / 技术许可 → `cn-contract-review-gov-tech-licensing`
  - 劳动 / 劳务 / 竞业 / 保密 → `cn-contract-review-labor-employment`
  - 其他商事合同（买卖 / 租赁 / 服务 / 框架 / M&A / 股权等，兜底） → `cn-contract-review-universal`
- **新工作流场景 8（合同审查）**：DocAnalyzer（合同解析）→ ContractReviewer（路由 skill）→ Reviewer（质量把关）→ 可选 Writer（修订重签）
- **`Workflow.md` 触发关键词扩充**：合同审查、合同审阅、红线审查、签署前检查、合同把关、看一下这份合同、合同有没有坑等
- **`AGENTS.md` 必需依赖表**：ContractReviewer 的 4 个 cn-contract-review-* skill 全部纳入

### 🔄 调整 (Changed)

- `AgentMapping.md` v3.1：输出层加 ContractReviewer 段；反向映射 `00 - 客户提供/`（合同输入）与 `02 - 法律研究/案件分析/`（审查产出）补 ContractReviewer
- `OutputStandards.md` v1.4：标准输出表加 ContractReviewer 行（`YYMMDD [合同名] 审查报告.md` + `YYMMDD [合同名] 红线版.docx`）
- `Workflow.md` "外部 skill 桥接" 表加 ContractReviewer 必需依赖

### 📐 设计要点

- **与 Writer 边界划分**：Writer = 我方主动起草新文书；ContractReviewer = 审查既有合同（对方草拟 / 第三方拟 / 我方旧合同复审）。客户要求"重新拟一份"时先 ContractReviewer 审查 → 用户确认 → 再调 Writer
- **多类目命中**：合同同时含技术许可 + 劳动条款时按主类目优先调起，并显式提示次类目可能需要补充审查
- **两种工作模式**：matter 内审查（推荐）+ 独立审查（无诉讼上下文，需先用 new-case 建独立 matter）
- **memory.md 不归 SuitAgent 管辖**：cn-contract-review-* skill 的 Prepare/Learn 阶段读写 `memory.md`（在 skill 路径内，本机），ContractReviewer 不接管
- **execute 阶段输出 redirect**：cn-contract-review-* skill 默认输出到 `/mnt/user-data/outputs/`，ContractReviewer 必须把红线 DOCX 移动到案件 slot

### 🚫 不在 Phase 2B

- JiubufaAnalyst / JudgmentReviewer 两个新 agent → Phase 2C
- 改 4 个 cn-contract-review-* skill 内容（read-only + 框架良好）

## [v1.3.0] - 2026-05-08 — Phase 2A: 案件目录结构统一

### 🏗️ 重构 (Refactored, BREAKING)

- **目录结构由 12 层带 emoji 改为 11 numbered slots（无 emoji）+ 4 root level 文件**：吸收 cn-litigation-case-folder-organizer skill 的 14-slot 方案为基础，叠加 SuitAgent 12 层中"综合报告"slot 与"工时记录"作为补强。SuitAgent 项目内外案件结构统一
- **新结构布局**：
  - root level: `matter.yaml` / `matter_dashboard.md` / `AGENTS.md` / `工时记录.md`
  - numbered slots: `00 - 客户提供` / `01 - 委托材料` / `02 - 法律研究`（含 `案件分析/` 子目录）/ `03 - 我方证据` / `04 - 对方证据` / `05 - 我方法律文书` / `06 - 对方法律文书` / `07 - 法院法律文书` / `08 - 庭审笔录` / `09 - 参考文件` / `10 - 综合报告` / `99 - 复盘沉淀`
- **`AgentMapping.md` 全文重写为 v3.0**：包含完整新结构定义、Agent 输出映射、与旧结构的迁移对照表
- **9 个 agent 文件路径硬编码全部更新**：DocAnalyzer / EvidenceAnalyzer / IssueIdentifier / Researcher / Strategist / Writer / Reporter / Summarizer / Scheduler
- **`Scheduler` 不再使用 numbered slot**：结构化期限合并入 root `matter.yaml`，工时记录提到 root `工时记录.md`
- **`new-case` skill 升级**：生成新结构（matter triplet + 工时记录.md root + 11 numbered + `02 - 法律研究/案件分析/` 子目录），停止生成旧 `[案件编号] 案件信息.md` 与 `[案件编号].yaml`
- **per-case `AGENTS.md` 引入**：每个案件根目录拥有自己的 agent 边界文件，明确 client identifier 保密硬约束、文件操作禁区、索引规约。这是 14-slot 方案吸收来的关键能力，旧 SuitAgent 缺失

### ➕ 新增 (Added)

- `99 - 复盘沉淀/` slot：结案后的复盘笔记、归档心得、工作流改进
- `04 - 对方证据/` 与 `06 - 对方法律文书/` 的拆分：原 `07 - 📥 对方提交` 合并 slot 不利于精确归档
- `AGENTS.md` 顶层文件新增"案件目录结构（v3.0+ 统一方案）"小节
- `OutputStandards.md` 标准输出表更新：Scheduler 持续维护文件指向 root level

### 📋 仓库外配套（Phase 2A 不入仓）

- `tmp/cn-litigation-case-folder-organizer_补丁.md`：organizer skill 的同步补丁（加 slot 10 + 工时记录.md 处理逻辑），用户本机复制到 skill 路径
- `tmp/案件迁移说明.md`：260507 / 260508 真实案件从旧 12 层迁移到新结构的具体指令
- `tmp/per-case_AGENTS.md_template.md`：每个新案件根目录 AGENTS.md 的模板

### ⚠️ 破坏性变更 (Breaking)

- **真实案件 260507 / 260508 当前为旧 12 层结构**：commit 一推，agent 在这两个案子上跑会找不到新路径。**必须先用 organizer skill 跑迁移**（见 `tmp/案件迁移说明.md`）
- **`案件信息.md` 被 matter triplet 替代**：Phase 2A 不主动删除现有案件的 `案件信息.md`（让 organizer 在迁移时同步），但 new-case 不再生成
- **`00 - 📅 日程管理/[案件编号].yaml` 被 root `matter.yaml` 替代**
- **依赖 organizer skill 完成同步迁移**：未跑迁移的案件不应被 SuitAgent agent 写入

### 🚫 不在 Phase 2A 范围（留给后续 phase）

- ContractReviewer / JiubufaAnalyst / JudgmentReviewer 三个新 agent → Phase 2B/2C
- Workflow.md 路由机制深度改造 → Phase 4
- 命名规范统一 → Phase 3

## [v1.2.0] - 2026-05-08 — Phase 1: 合并冗余

### 🔄 重构 (Refactored)

- **Writer agent 重写为 orchestrator 模式**：删除内嵌的 13 类文书模板与质量红线，改为路由分流
  - 诉讼文书 → 调起 `cn-litigation-drafting` skill（必需依赖）
  - 律所对客户文书 → 调起 `cn-firm-documents` skill（必需依赖）
  - Writer 自身只负责上下文承接、文件落盘、命名规范、DOCX 生成
- **方法论 single source of truth 上移到 skill 层**：起草标准、质量红线、模板结构由各 skill 统一维护，agent 不再各自维护一份

### ➕ 新增 (Added)

- **方法论 advisory 引用**：三个 agent 加上"方法论参考"段，明确深度场景下推荐调起的外部 skill
  - `IssueIdentifier.md`：深度争点结构分析 → `cn-jiubufa-case-analysis` skill
  - `Strategist.md`：再审/检察监督可行性研判 → `cn-judgment-analysis` skill
  - `DocAnalyzer.md`：判决书 IRAC 反向还原 → `cn-judgment-analysis` skill
- **AGENTS.md 新增"外部 skill 桥接"段**：登记必需依赖（cn-litigation-drafting / cn-firm-documents）与 advisory 依赖（cn-jiubufa-case-analysis / cn-judgment-analysis），交付者据此知道项目对外部 skill 的依赖关系
- **Workflow.md 新增"外部 skill 桥接"小节**：路由表层补充"哪个 agent 该调哪个 skill"的对照表
- **.gitignore 新增 `cn-firm-documents/`**：律所专用 skill（含具体律所名/对客户文书规则），与 `china_law_firm_template.md` 同性质私有

### 📋 配套补强（Phase 1 仓库外）

- 准备 `tmp/cn-litigation-drafting_SKILL_proposed.md`：cn-litigation-drafting skill 的提案稿，新增 11 项中尚缺的 6 类模板（代理词 F、质证意见书 G、财产保全 H、证据清单 I、仲裁申请书 J、反诉状 K）；用户本机复制到 `~/Library/Application Support/Claude/.../skills/cn-litigation-drafting/SKILL.md` 即生效
- 6 类律所对客户文书（律师函、委托代理协议、授权委托书、谈话笔录、法律意见书、调解协议）已 verbatim 桥接到 `cn-firm-documents` skill 的现有 references；不再在 SuitAgent 内重复维护

### ⚠️ 破坏性变更 (Breaking)

- **Writer agent 不再独立工作**：失去内嵌模板，必须依赖 cn-litigation-drafting + cn-firm-documents skill。两 skill 缺失时退化到兜底手工模式（必须在响应中显式警告）
- **AGENTS.md 中的"外部 skill 桥接"是新合规节点**：交付项目给协作者时必须确保这两个 skill 在他们机器上能解析到

## [v1.1.0] - 2026-04-02

### 🏗️ 重构 (Refactored)

- **📁 项目结构扁平化**：移除 `output/` 中间层，所有案件文件夹直接放在项目根目录（格式：`YYMMNN 当事人 案由/`）
- **📄 AGENTS.md 精简**：重构为 AI 行动指引手册（60行），移除冗余的 Agent 详情和工作流描述，详细信息按需指向 `.claude/` 下具体文件
- **🔗 CLAUDE.md 改为 @include**：根目录 CLAUDE.md 改为 `@include ./AGENTS.md`，与 legal-skills 保持一致
- **🗂️ 清理无用目录**：删除 `downloads/`、`input/`、`temp/`、`test/`、`.claude/config/` 等废弃目录
- **📜 模板内化到 Skill**：case-setup skill 的模板从 `.claude/templates/` 迁移到 `references/`，删除 templates 目录
- **🛠️ 工具脚本集中管理**：`sync-skills.sh` 移至 `.claude/scripts/`，避免污染项目根目录

### 🔧 优化 (Changed)

- **路径引用更新**：所有 `.claude/` 下的配置文件中的 `output/` 路径已批量更新为根目录格式
- **.gitignore 更新**：`output/*` 改为 `2*-*/`，匹配根目录案件文件夹格式
- **文档清理**：更新 `CommandMeta.md` 移除已删除的 `update-paths` 命令示例

### ⚠️ 破坏性变更 (Breaking)

- **案件目录迁移**：`output/` 下的 7 个案件文件夹已迁移到项目根目录，旧 `output/` 目录已删除
- **路径变更**：所有案件相关引用路径从 `output/[案件编号]/` 变为 `[案件编号]/`

## [v1.0.8] - 2026-01-01

### 🔧 优化 (Changed)

- **📋 技能规划颗粒度重构**
  - 对齐官方 Skill 机制，合并为“任务方向级”技能
  - 更新 `docs/skills发展规划.md` 的技能清单与颗粒度原则

## [v1.0.7] - 2026-01-01

### 🔧 优化 (Changed)

- **📋 技能规划文档精简与表格化**
  - 重构 `docs/skills发展规划.md` 为按类别的技能清单表格
  - 移除技术实现、实施路径、集成策略、效果评估等内容

## [v1.0.6] - 2026-01-01

### 🔧 优化 (Changed)

- **📝 中文翻译优化**
  - 将所有Agent配置中的"当被调用时"改为"适用场景"，更符合中文表达习惯
  - 更新SubagentStandards.md标准文档中的对应章节标题

- **🧹 工具配置清理**
  - 从所有10个Agent的tools列表中移除MCP工具（mcp_mineru、mcp_pkulaw）
  - 从DocAnalyzer的tools列表中移除"Skill"工具
  - 修正DocAnalyzer的tools格式从JSON数组改为逗号分隔格式

- **📝 Description详细化优化**
  - 扩展所有Agent的description字段，提供更详细的功能说明
  - 移除description中硬编码的MCP使用要求，改为功能描述
  - 确保description与实际功能一致，便于主Agent自动路由决策

### 📋 配置优化成果

- **中文本地化**: 所有章节标题符合中文表达习惯
- **工具配置精简**: 移除MCP和Skill工具，tools列表更简洁规范
- **描述信息增强**: 10个Agent的description平均长度增加150%
- **格式一致性**: 100%使用逗号分隔格式，符合SubagentStandards.md规范

---

## [v1.0.5] - 2026-01-01

### ✨ 新增 (Added)

- **📋 Subagent配置标准增强**
  - 更新SubagentStandards.md，添加tools和skills字段详细说明
  - 明确tools字段使用逗号分隔格式（不包含Skill）
  - 添加skills字段说明（可选，独立配置）
  - 增加变更历史章节，符合RulesMeta.md要求

### 🔧 优化 (Changed)

- **🤖 全部10个Agent配置标准化**
  - **EvidenceAnalyzer.md**: 删除触发词、添加工作检查清单、工具列表逗号化
  - **Scheduler.md**: 添加适用场景、输出要求章节、统一工具格式
  - **IssueIdentifier.md**: 删除触发词、移除Skill、添加标准章节
  - **Researcher.md**: 删除触发词、MCP规范章节、添加标准章节
  - **Strategist.md**: 删除触发词、添加工作检查清单
  - **Writer.md**: 删除触发词、移除Skill、添加标准章节
  - **Summarizer.md**: 删除触发词、添加适用场景章节
  - **Reporter.md**: 删除触发词、添加工作检查清单
  - **Reviewer.md**: 添加适用场景、工作检查清单、输出要求章节
  - **DocAnalyzer.md**: 确认已符合标准格式

### 📊 标准化成果

- **YAML格式统一**: 所有Agent工具列表使用逗号分隔格式
- **删除冗余内容**: 移除所有"🚨 需求识别触发词"章节
- **统一章节结构**: 采用4标准章节（角色定义、工作流程、输出标准、后续指引）
- **添加必需章节**: 10个Agent全部添加"适用场景"、"工作检查清单"、"输出要求"
- **工具配置清理**: 从所有Agent中移除"Skill"（技能为独立配置）

### 📋 配置规范 (Configuration)

- **配置规范完善**: 所有Agent配置文件现在完全符合SubagentStandards.md规范
- **工作流优化**: 清理触发词，Subagent专注于核心功能描述

### 📈 完成统计 (Statistics)

- **标准化统计**: 10个Agent配置文件全部完成标准化
- **删除内容**: 移除所有触发词定义和Skill工具引用
- **新增章节**: 添加30+个标准章节（每个Agent 3-4个新章节）
- **格式统一**: 100%符合Claude Code官方Subagent配置标准

---

## [v1.0.4] - 2026-01-01

### ✨ 新增 (Added)

- **📋 Subagent配置标准**
  - 创建SubagentStandards.md，制定基于Claude Code官方规范的统一配置标准
  - 定义YAML前置数据格式（name + description）
  - 规范4个标准章节：角色定义、工作流程、输出标准、后续指引

### 🔧 优化 (Changed)

- **📁 路径配置更新**
  - 在paths.yaml中添加subagent_standards规则引用
  - 更新引用模板，包含SubagentStandards.md链接

- **📝 标准格式重构**
  - 删除触发词定义章节（面向主Agent路由，Subagent调用时不可见）
  - 参考官方示例重构：采用"When invoked"、"核心职责"、"工作检查清单"、"输出要求"格式
  - 章节数从7个精简至4个，更实用更直接

### 📝 文档 (Documentation)

- **📋 路线图更新**
  - 更新DECOUPLING_ROADMAP.md v2.4，标记阶段4完成并记录重要修订
  - 添加阶段4详细成果和统计数据
  - 更新版本变更记录

### 📊 统计数据

- **标准制定成果**
  - 新建标准文件: 1个（SubagentStandards.md）
  - 规范配置字段: 2个必需（name, description）
  - 标准章节数: 4个（统一结构，精简后）
  - 更新配置文件: 1个（paths.yaml）
  - 阶段4标记为已完成（100%）

---

## [v1.0.3] - 2026-01-01

### 🔧 优化 (Changed)

- **📁 Rules目录精简与工作流合并**
  - 删除agent-workflows目录（10个工作流文件）
  - 从paths.yaml中移除agent_workflows规则
  - 从CLAUDE.md中删除对agent-workflows的引用
  - 简化Rules结构：8个子目录减至7个

### 📝 文档 (Documentation)

- **📋 路线图更新**
  - 更新DECOUPLING_ROADMAP.md v2.2，标记阶段3完成（Rules目录精简）
  - 添加阶段3完成内容和简化成果统计

### 📊 统计数据

- **解耦成果统计**
  - 删除工作流文件: 10个
  - 简化Rules目录: 从8个子目录减至7个
  - 更新配置文件: 2个（paths.yaml, CLAUDE.md）
  - 阶段3标记为已完成（100%）

---

## [v1.0.2] - 2026-01-01

### 🔧 优化 (Changed)

- **📋 Agent配置精简与规则提取**
  - 创建PDFProcessingRules.md共享PDF处理规范，统一MCP调用优先级规则
  - 精简DocAnalyzer.md：710行 → 227行（-68%，删除483行重复PDF处理规范）
  - 精简Scheduler.md：262行 → 147行（-44%，删除115行重复输出格式定义）
  - 确认OutputStandards.md无需Agent显式引用（规则自动加载机制）

### 📁 配置 (Configuration)

- **📄 路径配置更新**
  - 更新paths.yaml添加pdf_processing规则
  - 保持所有规则文件路径一致性

### 📊 统计数据

- **解耦成果统计**
  - DocAnalyzer.md: 删除483行重复内容
  - Scheduler.md: 删除115行重复内容
  - 新建共享规则: 2个（OutputStandards.md, PDFProcessingRules.md）
  - 阶段2进度：30% → 85%

### 📝 文档 (Documentation)

- **路线图更新**
  - 更新DECOUPLING_ROADMAP.md v2.2，标记阶段2基本完成（85%）
  - 添加精简成果统计和技术说明

---

## [v1.0.1] - 2026-01-01

### 🔧 优化 (Changed)

- **📁 Rules文件重组与路径配置优化**
  - 重命名Rules文件为更清晰的命名规范（AgentMapping.md, WorkflowSystem.md, WorkflowScenarios.md等）
  - 更新paths.yaml配置，添加workflow_scenarios、output_standards、rules_meta等新路径
  - 创建OutputStandards.md统一输出规范，为后续Agent配置精简做准备
  - 完成路径引用一致性验证，确认所有引用与paths.yaml配置匹配

### 📝 文档 (Documentation)

- **路径配置完善**
  - 更新`.claude/config/paths.yaml`添加完整的Rules文件引用
  - 添加reference_templates配置用于标准化Markdown链接格式
  - 验证28+处路径引用一致性

---

## [v1.0.0] - 2026-01-01

### 🚀 重大更新 (Major Updates)

- **🏗️ CLAUDE.md解耦与Claude Code Rules架构迁移**
  - **CLAUDE.md大幅精简**：从942行减少到200行（-79%）
  - **移除重复内容**：删除详细工作流、执行方式、文档管理结构等，改为引用配置文件
  - **架构升级**：迁移到Claude Code最新Rules架构（`.claude/rules/`）
  - **模块化配置**：5个配置文件转换为Markdown规则文件
  - **优化标题层级**：移除case-directories.md中12个不必要的三级标题
  - **统一变更历史**：为所有rules文件添加变更历史表格
  - **清理旧文件**：删除`.claude/config/`目录下所有YAML文件
  - **完整决策记录**：详见`docs/DECISIONS.md#决策-039`

### 📝 文档 (Documentation)

- **核心配置文件创建**
  - `.claude/rules/workflows.md` - 工作流定义（223行）
  - `.claude/rules/scenarios.md` - 场景识别规则（178行）
  - `.claude/rules/time-rules.md` - 时间管理规范（174行）
  - `.claude/rules/case-directories.md` - 案件目录结构（64行）
  - `.claude/rules/agent-mappings.md` - Agent目录映射（95行）

### 🔧 架构 (Architecture)

- **Claude Code Rules系统集成**
  - 采用Markdown + YAML frontmatter格式（description, category, version）
  - 自动加载`.claude/rules/`目录下的所有.md文件作为项目记忆
  - 实现文档模块化和关注点分离
  - 提升可维护性和可扩展性

---

## [v0.9.0] - 2026-01-01

### 📝 文档 (Documentation)

- **ROADMAP.md 精简重构**
  - 从900+行精简到82行（减少91%）
  - 删除过于详细的实现细节，只保留核心方向和简述
  - 恢复路线图的"鸟瞰视角"可读性
  - 版本升级至 v2.5

### 🐛 修复 (Fixed)

- **CHANGELOG.md 版本号修复**
  - 修复版本号顺序混乱问题（v0.1.2-v0.1.6与v0.2.0-v0.3.0交错）
  - 重新排序为递增序列：v0.1.0 → v0.2.0 → ... → v0.8.0

---

## [v0.8.0] - 2026-01-01

### 🔧 优化 (Changed)

- **📂 配置文件重命名和优化**
  - 将 `paths.yaml` 重命名为 `case-directories.yaml`，提高配置文件名称的描述性
  - 新名称更清楚地表明这是案件目录结构的配置文件
  - 更新所有10个文件中的配置引用，确保一致性
  - 与 `agent-mappings.yaml` 形成统一的配置文件命名规范

### 📝 文档 (Documentation)

- **配置解耦完成**
  - 创建 `.claude/config/agent-mappings.yaml` 作为Agent与目录映射的单一真实源
  - 更新9个Agent配置文件，将硬编码的输出目录改为引用配置文件
  - 更新2个Command文件（new-case.md、evidence-review.md）
  - 所有Markdown文档现在只包含配置引用，不包含硬编码
  - 实现"单一真实源"设计原则，提高可维护性

### 🏗️ 架构 (Architecture)

- **配置管理优化**
  - 建立两个核心配置文件的清晰职责划分：
    - `case-directories.yaml`: 定义12个标准目录结构和命名规范
    - `agent-mappings.yaml`: 定义10个Agent与目录的映射关系
  - 支持四层架构（输入层、分析层、输出层、支持层）
  - 提供双向映射（Agent→目录、目录→Agent）

---

## [v0.7.0] - 2025-12-30

### 🚀 新增功能 (Added)

- **🖥️ UI 化方案基线**
  - 新增 `UI化开发方案.md`，定义前端 Next.js + 后端 Node/Fastify + 事件流的两层架构
  - 明确 Claude Code CLI/SDK 双轨适配、Job 队列、长任务可观测性以及 skills/MCP 插件化管理路径
  - 给出 P0-P3 迭代路线和落地建议
  - 补充参考项目调研（Claudian/Obsidian Agent Client/opcode），确认以 ClaudeAdapter 为核心的非 ACP 路线，吸收侧边栏交互与安全模型

### 📝 文档 (Documentation)

- **决策记录**：添加决策 #037，确认 UI 化方案、Claude Adapter 抽象与技能/MCP 管理策略
- **任务清单**：在“目标 2.1: 用户界面优化”中补充并完成“UI 化开发方案”子任务
- **方案补充**：新增 Wireframe 指南（导航/核心转化区/侧栏布局），明确“先定结构后定视觉”的实施步骤

---

## [v0.6.0] - 2025-11-21

### 🚀 新增功能 (Added)

- **🔧 完整MCP配置修复和标准化**

  - 统一为所有10个Agent添加 `mcp_mineru` 工具权限
  - 建立主AgentMCP强制检查机制，防止绕过MCP直接使用技能
  - 创建统一的MCP配置检查器和验证脚本
  - 修复系统架构缺陷，确保MCP优先使用机制
- **📋 MCP配置管理工具**

  - 新增 `.claude/scripts/mcp_checker.py` - 统一MCP配置检查器
  - 新增 `.claude/scripts/validate_agent_permissions.py` - Agent权限验证脚本
  - 提供100%准确的MCP状态检测和权限验证
  - 自动化测试机制，确保配置正确性
- **📚 标准化文档和规范**

  - 新增 `.claude/memory/standards/MCP_USAGE_STANDARD.md` - MCP使用标准
  - 新增 `docs/MCP_CONFIGURATION_REPAIR_REPORT.md` - 完整修复报告
  - 建立详细的操作指南和问题排查检查清单
  - 标准化MCP工作流程和错误处理机制

### 🔧 优化 (Changed)

- **⚙️ Agent权限配置统一化**

  - 修复7个Agent缺少MCP权限的问题 (IssueIdentifier、Reporter、Researcher、Reviewer、Scheduler、Strategist、Summarizer)
  - Researcher增强为同时支持 `mcp_pkulaw` 和 `mcp_mineru`
  - 所有10个Agent现在都具备完整的MCP工具权限
  - 验证结果：100%权限配置正确
- **🚫 主Agent强制规范**

  - 在CLAUDE.md中添加主AgentMCP强制检查规范
  - 明确禁止主Agent直接调用PDF技能或MCP工具
  - 建立违规检测与纠正机制和质量保障清单
  - 确保所有PDF/图片处理必须通过DocAnalyzer Agent

### 🐛 修复 (Fixed)

- **🔗 架构缺陷修复**

  - 修复主Agent绕过MCP直接使用技能的架构问题
  - 解决MCP检查流程过于复杂的问题（从220行简化为标准化检查器）
  - 统一不同Agent的MCP回退机制
  - 建立标准化的错误处理和输出格式
- **📊 配置一致性修复**

  - 修复7个AgentMCP工具权限缺失问题
  - 统一MCP配置检查和验证流程
  - 建立自动化配置验证机制
  - 确保系统配置的一致性和可维护性

### 📝 文档 (Documentation)

- **📖 MCP架构文档完善**
  - 详细记录MCP配置修复过程和架构改进
  - 提供完整的测试验证结果和维护建议
  - 建立长期的配置监控和改进机制
  - 为未来的MCP扩展建立标准基础

---

## [v0.5.0] - 2025-11-20

### 🚀 新增功能 (Added)

- **🆕 完整技能发展规划体系**

  - 新增 `docs/SKILLS_ROADMAP.md` - 完整的技能发展规划文档
  - 规划9个核心专业技能，覆盖文档整理、协作优化、专业增强、自动化办公
  - 详细定义4个优先级发展路径和分阶段实施计划
  - 建立完整的风险评估和应对策略
- **📄 智能文档整理技能规划**

  - `doc-organizer` 智能文档整理技能：自动分类、重命名、归档
  - `content-extractor` 文档内容智能提取：结构化信息提取
  - `case-binder` 案件卷宗自动生成：标准化卷宗制作
  - 预计提升文档整理效率80%，降低错误率90%
- **🤝 智能协作与流程优化技能规划**

  - `workflow-orchestrator` 多Agent协作编排：智能工作流设计
  - `incremental-updater` 增量更新：智能变化检测和更新
  - 支持并行处理和智能调度，提升整体工作效率60%
- **⚖️ 专业领域增强技能规划**

  - `law-matcher` 法条智能匹配：基于案情智能匹配法规
  - `legal-visualizer` 数据可视化：法律关系和数据图表展示
  - 集成法律数据库和知识图谱，提升专业分析能力
- **📧 自动化办公技能规划**

  - `email-processor` 智能邮件处理：自动分类和回复建议
  - `communication-organizer` 沟通记录整理：多渠道沟通档案
  - 扩展应用场景到整个法律服务链条

### 🔧 优化 (Changed)

- **📋 任务清单更新**

  - 新增"目标 2.1: 技能发展规划与实施"
  - 详细的技能开发任务分解和时间估算
  - 建立技能验证标准和集成策略
  - 总开发时间估算40-50小时
- **📊 技能集成策略**

  - 设计技能与Agent的映射关系
  - 建立三种技能调用方式（直接调用、Agent触发、工作流集成）
  - 规划技能配置目录结构和调用规范

### 📝 文档 (Documentation)

- **📖 技能发展文档体系**
  - 完整的技能功能说明和技术实现方案
  - 详细的使用场景和预期输出示例
  - 风险评估与应对策略
  - 技能调用方式和集成规范

---

## [v0.4.0] - 2025-11-17

### 🚀 新增功能 (Added)

- **🆕 全新12目录诉讼模板架构**

  - 新增完整的12层诉讼模板架构（00-11目录）
  - 📅 00 - 日程管理：案件看板、时间线、期限预警、工时统计
  - 🤝 01 - 委托材料：委托合同、授权书、谈话笔录、服务方案
  - 📄 02 - 案件分析：案件分析报告、争议焦点、风险评估、策略方案
  - 🔍 03 - 法律研究：法条检索、判例研究、法律适用【前置核心】
  - 📤 04 - 客户提供：客户文档、需求确认、反馈记录
  - 📎 05 - 证据材料：证据清单、质证意见、鉴定报告
  - 📝 06 - 法律文书：起诉状、答辩状、代理词等各类文书
  - 📥 07 - 对方提交：对方起诉状、证据、答辩书
  - 🏛️ 08 - 法院送达：传票、裁定、判决书
  - 🎯 09 - 庭审笔录：庭审记录、庭后分析
  - 📊 10 - 综合报告：案件摘要、阶段报告
  - 📚 11 - 参考文件：参考案例、法律条文
- **🆕 案件管理文件模板**

  - 新增 `[案件编号].yaml` - 案件管理看板数据（核心文件）
  - 新增 `[案件编号].md` - 案件工作记录看板
  - 支持YAML + MD双版本设计
- **🆕 Agent目录映射优化**

  - 10个Agent工作流与12目录完美对接
  - 每个目录都有专门的README.md说明文件
  - 支持AI工作流和传统实务操作双轨并行

### 🔧 优化 (Changed)

- **📦 文档管理策略优化**

  - 优化.gitignore配置，docs和status目录本地保留
  - 不推送到GitHub，但完整保留本地功能
  - 保持开发灵活性同时保护敏感信息
- **📄 Markdown转Word命令完善**

  - 增强自动化安装脚本功能
  - 支持3种预设格式 + 自定义格式
  - 完全支持中文文档转换
- **📘 Git 管理最佳实践指南更新**

  - 将 `docs/Git-Management-Best-Practices.md` 重命名为 `docs/Git-管理最佳实践指南.md`
  - 增加“Git 功能使用指南”章节：包含日常分支工作流、`git worktree` 用法及 WorkTree 与测试分支的对比，帮助多任务并行和并行测试

### 🐛 修复 (Fixed)

- 解决了文档版本控制冲突问题
- 修复了模板目录结构不完整的问题
- 优化了Git历史记录管理

---

## [v0.3.0] - 2025-11-14

### 🚀 新增功能 (Added)

- **🆕 委托文件生成系统整合**

  - 成功整合独立的委托文件生成系统到SuitAgent工作流
  - 新增 `.claude/tools/placeholder_mapper.py` - 字段映射工具 (348行)
  - 新增 `.claude/tools/DocxProcessor.py` - Word文档处理引擎 (365行)
  - 新增 `.claude/tools/docx_tools.py` - 便捷调用接口 (304行)
  - 支持14个核心字段的自动映射
  - 支持公司和个人两种委托类型
- **🆕 委托确定工作流**

  - 新增完整"委托确定"场景工作流
  - 支持5个核心委托文件自动生成
  - 集成DocAnalyzer → Writer → Summarizer → Reporter自动化流程
  - 可通过自然语言触发："我需要制作委托材料"
- **🆕 模板资源库**

  - 新增 `.claude/templates/公司委托模板/` - 9个Word模板
  - 新增 `.claude/templates/个人委托模板/` - 8个Word模板
  - 支持自动模板选择和批量生成
  - 保持Word文档原始格式（字体、颜色、对齐等）
- **🆕 集成测试套件**

  - 新增 `tests/test_integration.py` - 完整测试套件 (452行)
  - 覆盖字段映射、YAML验证、模板发现、批量生成等场景
  - 提供颜色输出和详细测试报告

### 🔧 改进 (Changed)

- **Writer Agent扩展**

  - 更新 `.claude/agents/Writer.md`，添加委托文件生成功能
  - 新增15种文书类型（包含Word格式委托文件）
  - 集成自动工作流触发机制
  - 支持Markdown和Word双格式输出
- **目录结构优化**

  - 精简 `.claude/memory/` 目录，减少上下文污染60%
  - 将可引用文件移至 `.claude/tools/`、`.claude/templates/`、`docs/`、`output/`
  - memory目录仅保留必要文件：integration/、workflows/
  - 上下文大小从~500KB减少到~200KB
- **文档体系增强**

  - 新增 `docs/委托文件生成系统整合报告.md` - 完整整合报告 (11KB)
  - 新增 `.claude/memory/integration/字段对照表.md` - 字段映射规范
  - 新增 `.claude/memory/integration/工具使用指南.md` - API文档
  - 新增 `.claude/memory/integration/委托确定工作流.md` - 工作流说明
  - 新增 `.claude/memory/integration/目录结构优化.md` - 优化说明
  - 新增 `.claude/memory/integration/整合工作总结.md` - 工作总结

### 🔒 性能优化 (Optimized)

- **上下文优化**

  - memory目录文件减少60%（从~50个到~20个）
  - 上下文大小减少60%（从~500KB到~200KB）
  - 响应速度显著提升
  - 成本显著降低
- **文件组织优化**

  - 按功能分类存放文件：tools/（工具）、templates/（模板）、docs/（文档）
  - 通过路径引用使用，无需加载到上下文
  - 统一的模板管理 `.claude/templates/`
  - 清晰的目录层次结构

### 🐛 修复 (Fixed)

- 修复了工具文件相对导入问题，使用 `.claude/tools/` 内部导入
- 更新了所有文档中的路径引用，从 `.claude/memory/` 迁移到新位置
- 修复了批量生成函数的模板目录选择逻辑

### 📋 技术细节 (Technical)

- **核心技术**: 基于 `word_template_processor.py` 的字符级精确替换
- **算法优化**: Run级别操作保持Word格式，反向迭代避免索引偏移
- **字段映射**: 14个核心字段（委托人、律师、案件、日期信息）
- **模板类型**: 支持公司委托（9种文件）和个人委托（8种文件）
- **输出格式**: Word(.docx) + Markdown(.md) 双格式支持
- **工作流步骤**: 4步自动化（客户确认 → 委托文件 → 清单摘要 → 完整报告）

### 📖 文档更新 (Documentation)

- `docs/委托文件生成系统整合报告.md` - v1.0，完整整合报告
- `status/CHANGELOG.md` - 记录 v0.3.0 所有变更
- `status/JOURNAL.md` - 添加整合工作日志
- `status/TASKS.md` - 更新任务完成状态
- `CLAUDE.md` - 更新场景7说明，新增Word格式委托文件特性

### ⚠️ 破坏性变更 (Breaking Changes)

- **工具路径变更**: 原 `tools/*.py` 移至 `.claude/tools/*.py`
- **模板路径变更**: 原 `.claude/memory/公司委托模板/` 移至 `.claude/templates/公司委托模板/`
- **导入路径变更**: 工具内部导入使用相对路径 `from .xxx import xxx`

### 📌 迁移指南 (Migration)

- 更新所有引用 `tools/*.py` 的代码为 `.claude/tools/*.py`
- 更新模板路径从 `.claude/memory/委托模板/` 到 `.claude/templates/委托模板/`
- 安装新依赖: `pip install python-docx pyyaml`

---

## [v0.2.0] - 2025-11-11

### 🚀 新增功能 (Added)

- **🆕 自动化安装脚本系统**

  - 新增 `install.sh` 交互式安装向导
  - 支持一键安装 Node.js、Claude Code CLI、Zed 编辑器
  - 完整支持 macOS、Linux、Windows (PowerShell/CMD/WSL)
  - 智能系统检测，自动选择最佳安装方式
  - 支持 4 大 AI 模型平台：智谱AI、月之暗面、MiniMax、DeepSeek
  - API 密钥安全输入（密码模式），自动生成配置文件
  - 安装后完整验证，确保所有组件正常工作
- **🆕 完整文档体系**

  - 新增 `INSTALL.md` - 详细安装指南 (11KB)
  - 新增 `QUICKSTART.md` - 一分钟快速开始 (1.2KB)
  - 更新 `README.md` - 集成自动化安装说明

### 🔧 改进 (Changed)

- **更新 ROADMAP.md 至 v2.3**
  - 新增"阶段1.5：安装体验优化"
  - 当前阶段调整为"核心功能增强"
  - 添加自动化安装系统完成状态记录

### 📋 技术细节 (Technical)

- **跨平台支持矩阵**：

  - macOS: Homebrew 自动安装
  - Linux: apt/yum 包管理器支持
  - Windows: PowerShell + Chocolatey + WSL 三种方式
  - WSL: 完整 Linux 体验支持
- **安装流程优化**：

  - 系统检测 → 依赖安装 → CLI 安装 → AI 配置 → Zed 安装 → 项目打开 → 验证完成
  - 全程交互式向导，用户只需按提示操作
  - 错误处理和重试机制，提升安装成功率

### 📖 文档更新 (Documentation)

- `docs/ROADMAP.md` - v2.3，添加安装体验优化阶段
- `status/CHANGELOG.md` - 记录 v0.2.0 所有变更
- `status/TASKS.md` - 已完成安装相关任务
- `README.md` - 新增自动化安装章节

---

## [未发布] - 待定

### 📝 文档优化 (Documentation Optimization)

- **精简CLAUDE.md"常见使用场景"部分**
  - **精简内容**：删除场景8-15（保全申请、上诉材料、法律意见书、律师函、强制执行、调解协议、异议申请等）
  - **保留核心**：只保留7个最常用场景（被告应诉、新证据质证、庭审后分析、诉前沟通、策略优化、原告起诉、制作委托材料）
  - **更新表格**：同步精简自动识别规则表格，移除不常用场景
  - **效果**：显著减少文档长度，提升可读性和维护性

### 🔧 架构修复 (Architecture Fixes)

- **修复PDF处理工作流程绕过DocAnalyzer的关键问题**
  - **问题描述**：用户上传PDF时系统直接使用PDF技能，绕过DocAnalyzer Agent，导致架构违规和错误处理
  - **解决方案**：
    - 修正DocAnalyzer配置，明确PDF处理职责和双格式输出要求
    - 创建文档处理强制工作流程规范 (`DOCUMENT_PROCESSING_ENFORCEMENT.md`)
    - 在CLAUDE.md中添加文档处理强制规范章节
    - 记录架构决策 #020
  - **工作流程**：
    - 错误：用户上传PDF → 直接使用PDF技能 ❌
    - 正确：用户上传PDF → DocAnalyzer Agent → 使用pdf技能 → 返回PDF+MD+JSON ✅
  - **核心要求**：所有PDF文档必须通过DocAnalyzer处理，必须返回带文字层的PDF和Markdown文件，OCR准确率需达95%+

### 重大更新 (Major Updates)

- **Agents架构重大升级 v2.0 - 符合Claude Code官方标准**
  - **新增双层架构设计**：
    - 使用层 (`.claude/agents/`)：标准subagent配置，符合Claude Code官方规范
    - 知识层 (`.claude/memory/workflows/`)：详细工作流程，便于开发优化
  - **标准化所有10个agents**：
    - 添加YAML frontmatter配置（name, description, tools, model, color）
    - 移除非标准tools配置（pdf, docx, xlsx等），避免"Unrecognized"警告
    - 优化模型选择和temperature参数
    - 配置8种颜色区分agents（2个agents共享颜色）
  - **保留完整设计文档**：
    - 10个详细工作流程文件完整迁移至workflows目录
    - 保留所有功能定位、JSON配置、性能指标、质量验证等
    - 创建COLOR-GUIDE.md说明颜色分配原则
  - **创建说明文档**：
    - AGENTS-CONVERSION-SUMMARY.md：转换总结
    - AGENTS-FINAL-COLOR-CONFIG.md：颜色配置完成文档
  - **优势**：完全符合官方标准，可直接使用/agents命令管理，保持专业设计完整性
- **ROADMAP.md重大版本更新 v2.2**
  - **新增目标5：专业化法律技能库建设**
    - 建立案件专业化分析模板库：医疗纠纷、知识产权、建筑工程、合同纠纷、劳动争议等专项模板
    - 构建证据需求清单库：各类案件的标准化证据清单，包括基础证据、关键证据、辅助证据
    - 完善法律要点知识库：核心法条库、争议焦点分析框架、典型案例参考库
  - **新增目标6：法律文书模板系统优化**
    - 建立答辩意见要点模板库：标准答辩框架、不同案件类型答辩策略、律师自定义添加机制
    - 开发委托文档自动生成系统：预设委托合同模板库、基于Skills的自动生成功能
    - 实现Markdown转Doc文档功能：基于docx skill的转换引擎、自动排版格式化、批量转换
  - **新增目标7：命令脚本系统**
    - 构建简单命令执行框架：案件管理、文档处理、工作流、Agent控制等常用命令集合
    - 集成Hooks机制：文件生成后自动转换、自定义脚本、插件式扩展、自动化质量检查
    - 实现脚本化工作流执行和批处理任务支持
  - **完善目标4：案件基础信息与工作记录模板持续优化**
    - 详细规划数据可视化支持：案件进度甘特图、工时统计图表、期限倒计时显示、风险等级可视化
    - **扩展跨案件数据汇总：新增统一数据看板🆕**
      - 自动扫描所有案件YAML文件，生成统一数据索引
      - 支持列表视图、卡片视图、甘特图视图
      - 多维度筛选：案件类型、负责人、状态、时间
      - 智能数据洞察：预测分析、资源优化、效率提升建议
    - 建立模板持续迭代机制：反馈收集、结构优化、新字段扩展、版本管理

### 新增 (Added)

- **案件PDF扫描OCR处理能力**

  - 实现PDF扫描件OCR文字提取，使用tesseract + ImageMagick技术
  - 成功提取起诉状全文内容，生成带文字层的PDF和Markdown文件
  - 建立OCR质量优化流程：图像预处理、拼写检查、格式校正
- **日程管理目录精简优化**

  - 删除不必要的文件：客户沟通记录、费用跟踪、庭审安排、风险预警、README等
  - 保留3个核心文件：工时记录.md、期限管理.md、重要任务清单.md
  - 将[案件编号].yaml移回06_日程管理/目录
  - [案件编号].md保留在案件根目录
  - 贯彻"Less is more"理念，精简高效
- **案件看板位置优化与日程管理内容扩展**

  - 将案件看板（[案件编号].yaml和[案件编号].md）移动到案件根目录，便于查看和使用
  - 从案件看板抽取重要任务清单为独立文档：重要任务清单.md
  - 从案件看板抽取期限管理为独立文档：期限管理.md
  - 新增客户沟通记录.md，记录与客户的日常沟通情况
  - 新增费用跟踪.md，跟踪案件相关费用开支
  - 新增庭审安排.md，记录庭审安排和准备清单
  - 新增风险预警.md，记录和管理案件风险点
  - 扩展日程管理内容从2个文件到9个文件，形成完整的案件管理工具集
- **Agent输出路径标准化实施完成**

  - 完成标准目录结构创建：output/cases/[案件编号]/01-06完整目录
  - 创建Writer 12个法律文书子目录：起诉状、答辩状、代理词等
  - 建立YAML和MD模板文件：案件数据总表、工作记录模板
  - 创建实施指南文档：OUTPUT_PATH_STANDARDIZATION.md
  - 生成示例文档：案件分析报告、法律文书样例、操作日志
  - 建立路径分配工具函数和YAML自动更新机制
- **全面更新项目规划文档**

  - 创建新版roadmap (ROADMAP_NEW.md)，反映9个Agent已完成、当前处于文档管理优化阶段
  - 创建新版task清单 (TASKS_NEW.md)，定义阶段0-3的完整规划，聚焦文档管理优化
  - 重新定位项目现状：9个AI代理已完成，核心功能完整，当前阶段为文档管理优化
- **Agent输出文档管理规范**

  - 创建完整输出路径规范 (AGENT_OUTPUT_MANAGEMENT.md)
  - 定义9个Agent的输出路径映射：01-06目录分配、12种文书分类
  - 建立文档命名规范：[日期]_[类型]_[版本]格式
  - 设计智能路径分配机制：案号识别、案件匹配、冲突处理
  - 实现数据同步机制：YAML自动更新、MD工作记录同步
- **YAML数据总表模板优化**

  - 创建统一数据总表模板 (YAML_DATA_TEMPLATE.md)
  - 整合16个部分：基础信息、当事人、律师、费用、进展、时间线、期限、工时、工作记录、风险、文档、数据分析、项目管理、扩展字段、系统信息
  - 支持双版本设计：YAML用于看板数据，MD用于工作记录
  - 实现数据可视化支持：进度看板、工时统计图表、期限预警
  - 提供完整使用指南和最佳实践
- **智能案件识别系统**

  - 设计智能识别系统 (INTELLIGENT_CASE_RECOGNITION.md)
  - 实现案号识别算法：支持标准/简化/无括号格式，提取年份、地区、类型、序号
  - 建立文件类型识别规则：15种文档类型的关键词和特征匹配
  - 设计案件匹配算法：完全匹配/部分匹配/模糊匹配三级匹配
  - 实现新案件自动创建：目录结构、yaml模板、工作记录模板
  - 建立冲突处理机制：重复案件检测、文件冲突解决
  - 提供完整工作流程图和代码实现
- **Agent输出路径配置指南**

  - 创建Agent输出配置指南 (AGENT_OUTPUT_PATHS_CONFIG.md)
  - 为9个Agent提供具体输出实现：DocAnalyzer、EvidenceAnalyzer、IssueIdentifier、Researcher、Strategist、Writer、Summarizer、Reporter、Scheduler
  - 定义输出内容：每种Agent的文档类型、文件名格式、扩展名
  - 实现完整工具函数：路径生成、文件名创建、YAML更新、日志记录
  - 提供使用示例：每种Agent的调用方式和返回结果
  - 支持Writer 12种子类型：起诉状、答辩状、代理词等分类存储
- **创建长期优化规划文档**

  - 创建长期优化规划 (LONG_TERM_OPTIMIZATION_PLAN.md)
  - 定义四大优化方向：Claude Code高级功能、技能库完善、插件生态扩展、MCP工具集成
  - 详细规划Memories系统：案件上下文记忆、律师个人偏好学习、知识库记忆与智能推荐
  - 设计MCP工具集成方案：法院系统MCP、律所系统MCP、法律知识库MCP、云存储MCP
  - 制定插件生态扩展计划：法律专用插件、可视化插件、数据导入插件、第三方集成插件
  - 建立实施路线图：分阶段实施、技术架构设计、风险评估与应对、预算估算
- **Claude Code高级功能深度研究**

  - 创建高级功能研究报告 (CLAUDE_CODE_ADVANCED_FEATURES_RESEARCH.md)
  - 深度研究Memories上下文记忆系统：记忆存储架构、记忆分类标记、案件记忆空间设计
  - 全面分析MCP工具集成方案：法院系统MCP服务器实现、律所管理MCP集成、法律知识库MCP设计
  - 详细设计高级指令模板系统：完整案件分析模板、模板执行引擎、模板使用示例
  - 探索多模态处理能力：文本OCR增强、图像证据分析、音频庭审处理、视频庭审分析
  - 提供完整代码实现：MCP服务器开发、客户端集成模式、错误处理与重试机制
  - 制定实施建议与总结：优先级建议、技术路线图、风险控制、成功指标
- **项目路线图整合与优化**

  - 将长期优化规划全面整合到新版roadmap中，包含Claude Code高级功能、Skills知识库、插件生态、MCP工具四大方向
  - 按照用户指示调整MCP工具集成描述，留空作为未来展望展示，不涉及具体实现方案
  - 删除老版本ROADMAP.md文件，重命名ROADMAP_NEW.md为ROADMAP.md
  - 完善长期规划内容：Memories上下文记忆、高级指令模板、多模态处理、API集成等
- **记忆分层系统架构设计**

  - 在ARCHITECTURE.md中添加"记忆分层系统"章节，详细描述基于Claude Code架构的三层记忆设计
  - 定义案件级记忆、个人级记忆、系统级记忆的位置、内容、访问方式和更新机制
  - 在ROADMAP.md中新增"优化方向0：记忆分层系统建设（基础架构）"，将记忆分层系统列入长期规划
  - 建立记忆分层系统的理论基础，为后续Agent智能化协作提供基础支持
- **文档结构优化与整合**

  - 删除已整合的LONG_TERM_OPTIMIZATION_PLAN.md文档
  - 合并AGENT_OUTPUT_PATHS_CONFIG.md到AGENT_OUTPUT_MANAGEMENT.md，添加技术实现详解章节
  - 合并YAML_DATA_TEMPLATE.md到INTELLIGENT_CASE_RECOGNITION.md，添加YAML数据模板章节
  - 创建docs/research/文件夹，将CLAUDE_CODE_ADVANCED_FEATURES_RESEARCH.md移动到专门的研究文档夹
  - docs文件夹文档数量从10个减少到6个，大大简化文档结构
  - 保持文档层次结构：核心文档在根目录，详细研究在research子目录
- **优化案件文档管理结构（目录整合+双版本设计+简洁命名）**

  - 重新整合目录结构：9个目录精简为6个，按工作流相关性合并
    - 争议焦点+法律研究 → "02_法律研究"
    - 案件分析+策略规划 → "01_案件分析"
    - 报告摘要+综合报告 → "05_综合报告"
    - 期限工时管理 → "06_日程管理"
  - 创建日程管理双版本设计（Scheduler输出增强）
    - YAML版本：[案件编号].yaml 支持案件管理看板和数据分析
    - MD版本：[案件编号].md 支持工作记录和日常查看
    - 工作记录模板.md 提供标准化记录格式
  - 添加新材料自动分类指引（DocAnalyzer增强功能）
    - 材料归属识别（案号提取、案件匹配、新案件创建）
    - 材料类型分析（6种材料类型的识别特征和放置位置）
    - 自动放置流程（input → DocAnalyzer → 自动分类 → 放置）
    - 批量处理支持（ZIP、多文件、OCR识别）
    - 命名规范应用（自动重命名、版本管理、冲突处理）
    - 上下文更新（yaml/md文件更新、进度标记）
    - 完整示例操作流程演示
  - 添加PDF扫描处理指引（DocAnalyzer OCR增强）
    - PDF扫描与OCR识别（文字层提取、OCR识别、准确率95%+、多语言支持）
    - 双格式输出设计（带文字层的PDF + Markdown文件）
    - 质量优化机制（预处理、后处理、人工校对、质量报告）
    - 支持法律文档场景（扫描合同、法院传票、证据材料、判决书）
    - 完整示例操作流程演示
  - 编写完整的目录使用说明（README.md）
  - 更新所有相关文档反映新的目录结构
- **规范化案件文档管理结构**

  - 创建output/cases/[案件编号]/目录结构，为9个agents建立专门输出文件夹
  - 细化Writer输出为12种文书类型（起诉状、答辩状、代理词、申请书等）
  - 创建Scheduler期限工时管理（日程安排、工时统计、期限提醒）
  - 建立模板库支持标准化文档生成
  - 编写各目录使用说明和输出格式规范
  - 定义统一命名规范（案件编号、文档命名、版本管理）
- **AI代理协作规范**

  - 重写docs/DEVELOPMENT.md为AI代理协作规范
  - 定义Agent配置、知识库、官方技能集成标准
  - 明确协作流程和工作流编排机制

### 变更 (Changed)

- **项目配置纠正**
  - 清理不必要Python项目配置（setup.py、requirements.txt等）
  - 更新README.md为AI代理协作使用说明
  - 重写DEVELOPMENT.md为AI代理协作规范
  - 重新聚焦Claude Code协作系统核心功能
  - 创建 `docs/` 和 `status/` 目录结构
  - 编写项目路线图 (`docs/ROADMAP.md`)
  - 编写决策记录文档 (`docs/DECISIONS.md`)
  - 编写架构文档 (`docs/ARCHITECTURE.md`)
  - 编写任务清单 (`status/TASKS.md`)
  - 编写变更记录 (`status/CHANGELOG.md`)
  - 编写工作日志 (`status/JOURNAL.md`)
- 建立AI代理协作与文档协议 (`CLAUDE.md`)
- 定义8个核心工作流模块（XXXer命名模式）
  - DocAnalyzer - 起诉文档解析器
  - EvidenceAnalyzer - 证据分析器
  - IssueIdentifier - 争议焦点识别器
  - Researcher - 法律研究者
  - Strategist - 法律策略规划器
  - Writer - 法律文书起草器
  - Summarizer - 报告摘要生成器
  - Reporter - 报告整合器
- 完善SubAgent配置规范
  - 统一配置结构和格式标准
  - 添加详细的概述、功能描述和步骤定义
  - 完善输入输出规范和错误处理机制

### 变更 (Changed)

- 重构项目文档结构，引入模块化设计理念
- 优化架构设计，采用Sub-Workflows设计模式
- **统一Agent命名为XXXer模式** - 所有8个agent采用简洁、功能明确的新命名，提升可读性和易用性
- **实现主Agent自动化工作流编排** - 添加自动化编排功能，支持文件类型自动识别和场景智能匹配，实现一键启动工作流
- **扩展支持沟通记录场景** - 新增"诉前沟通"和"诉讼中沟通"两个自动化场景，分别用于法律服务方案出具和策略优化
- **扩展支持原告起诉场景** - 新增"原告起诉"完整流程，从案件材料到完整起诉材料包（起诉状+证据目录等）
- **扩展支持委托材料制作场景** - 新增"制作委托材料"场景，自动生成委托合同、授权委托书、谈话笔录等
- **大规模扩展核心场景覆盖** - 新增7个重要法律场景（保全申请、上诉材料、律师函、法律意见书、强制执行申请、调解协议、异议申请），场景数量从8个增加到15个，专注于诉讼全流程
- **架构优化：整合模板到Memory知识库** - 将 `.claude/templates/` 中的文书模板整合到标准Memory组件 `.claude/memory/legal_templates.md`，提升管理效率和协作规范性
- **架构优化：移除不符合标准的skills目录** - 删除 `.claude/skills/` 目录，该目录包含的是API文档描述而非真正的Skills，不符合Claude Code标准
- **集成Claude Code官方Skills库** - 从Anthropic官方仓库抓取xlsx、pdf、docx三个官方技能，显著增强项目的文档处理能力
- **DocAnalyzer增强：支持ZIP处理和OCR智能重命名** - 新增压缩包解压、OCR识别、文档类型判断、智能重命名、文件整理等功能，支持法院文件批量处理
- **新增Scheduler工作流：法律期限管理和工时统计** - 实现法定期限自动计算、案件时间线管理、工作记录和工时统计分析，支持律师执业全流程的期限管理和工时管理

### 修复 (Fixed)

- 填补项目缺乏系统化文档的空白

### 移除 (Removed)

- N/A

### 安全 (Security)

- N/A

---

## [v0.1.0] - 2025-10-31

### 新增 (Added)

- 初始项目架构
  - 创建项目README.md
  - 定义核心功能和特性
  - 建立8个工作流的设计理念
  - 设计模块化架构
  - 制定上下文最小化策略
- 基础目录结构
  - `.claude/` 配置目录
  - `data/` 输入数据目录
  - `output/` 输出报告目录
- 基础配置文件
  - `requirements.txt` 依赖管理
  - `CLAUDE.md` 协作协议

### 变更 (Changed)

- N/A

### 修复 (Fixed)

- N/A

### 移除 (Removed)

- N/A

### 安全 (Security)

- N/A

---

## 版本说明

### 版本号规则

- 主版本号：API不兼容的重大变更
- 次版本号：向后兼容的功能增加
- 修订号：向后兼容的问题修复

### 变更类型

- **新增 (Added)**: 新功能、新特性
- **变更 (Changed)**: 对现有功能的修改
- **修复 (Fixed)**: 问题修复
- **移除 (Removed)**: 已移除的功能
- **安全 (Security)**: 安全相关的修复

### 变更记录规范

每次发布新版本时，请按以下格式记录：

```markdown
## [版本号] - 发布日期

### 新增 (Added)
- 简短描述
- 详细说明...

### 变更 (Changed)
- 变更内容描述
- 变更原因和影响...

### 修复 (Fixed)
- 问题描述
- 修复方案...

### 移除 (Removed)
- 已移除功能描述
- 移除原因...

### 安全 (Security)
- 安全漏洞描述
- 修复方案...
```

---

## 重要变更

### [0.1.0] - 2025-10-31

这是项目的首个正式版本，标志着项目启动和基础架构的建立。

**主要变更：**

- 项目初始化完成
- 定义了8个核心工作流
- 建立了文档协作协议
- 创建了基础项目结构

**影响范围：**

- 整个项目
- 所有协作者
