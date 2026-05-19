# Agent 目录映射与案件结构

**版本**: v3.7
**最后更新**: 2026-05-10
**说明**: 定义统一的案件目录结构（11 numbered slots + 5 root level files）与 Agent 输出映射关系

## 🎯 文档职责说明

本文档专注于**目录结构与映射关系**：
- ✅ 案件根目录 5 件套文件
- ✅ 11 个 numbered slots 的标准化结构
- ✅ Agent → 目录映射
- ✅ Agent 分层架构

**文件命名与输出格式标准**详见 [`OutputStandards.md`](./OutputStandards.md)

## 📁 案件目录架构

SuitAgent 项目内外统一采用以下结构（继承自 `cn-litigation-case-folder-organizer` skill 的 14-slot 方案，吸收 SuitAgent 12 层中"综合报告"slot 与"工时记录"作为补强）。

### 案件根目录布局

```
[案件文件夹]/
├── matter.yaml                ← 结构化操作数据：当事人/案号/阶段/关键日期/文件夹规约
├── matter_dashboard.md        ← 人读案件看板（替代旧 案件信息.md）
├── AGENTS.md                  ← per-case agent 边界与保密硬约束
├── 工时记录.md                ← 工时与费用核算（律师/财务读）
├── handoff_ledger.md          ← agent 间结构化简报滚动账本（v1.13.0；下游 ledger-first lazy-load）
├── 00 - 客户提供/             ← 客户递交的原始材料
├── 01 - 委托材料/             ← 委托代理协议、授权委托书、谈话笔录、监督卡
├── 02 - 法律研究/             ← 法条、判例、研究报告
│   └── 案件分析/              ← 子目录：争议焦点、SWOT、策略、案件摘要、综合分析
├── 03 - 我方证据/             ← 我方提交的证据原件、扫描件、证据目录
├── 04 - 对方证据/             ← 对方提交的证据材料
├── 05 - 我方法律文书/         ← 我方提交的起诉状、答辩状、上诉状、代理词等
├── 06 - 对方法律文书/         ← 对方提交的法律文书
├── 07 - 法院法律文书/         ← 法院送达的传票、裁定、判决、调解书
├── 08 - 庭审笔录/             ← 庭审记录、笔录、录音转录
├── 09 - 参考文件/             ← 参考法条、参考判例、参考模板（不属于本案直接证据/文书）
├── 10 - 综合报告/             ← Reporter / Summarizer 的产物
└── 99 - 复盘沉淀/             ← 结案后复盘笔记、归档心得、工作流改进
```

### 5 个 root level 文件职责

| 文件 | 主要使用者 | 更新频率 | 职责 |
|------|---------|--------|------|
| **matter.yaml** | 所有 Agent、Scheduler | 案件状态变化时 | 结构化操作数据：matter_id、案号、案由、阶段、关键日期、当事人、代理律师、文件夹规约（禁动后缀） |
| **matter_dashboard.md** | 承办律师、客户汇报 | 关键节点 | 人读看板：时间线、待办、风险提示、阶段总结。**替代旧 `[案件编号]案件信息.md`** |
| **AGENTS.md** | 所有 agent | 极少改 | per-case 的保密硬约束 / 文件操作禁区 / 索引规约。继承根目录 CLAUDE.md，只写 case-specific 内容 |
| **工时记录.md** | 承办律师、财务 | 每天/每周 | 工时记录、费用预算、月度统计 |
| **handoff_ledger.md** (v1.13.0) | 所有 orchestrator agent（读侧 ledger-first / 写侧落盘后追加 briefing）；Reviewer 不简化（读 full 产物）| 每 agent 完成落盘后追加 | agent 间结构化简报滚动账本（倒序，最新在顶）：下游据 briefing 指针 lazy-load 仅需 full 产物，砍长 context 膨胀。schema/软约束/硬边界见 [`HandoffProtocol.md`](./HandoffProtocol.md)。缺失→回退全读 gated |

## 🔄 文件职责分工（取代旧 12 层时代的"三件套"）

| 文件 | 主要消费方 | 取代了 |
|------|----------|------|
| `matter.yaml` | Agent 上下文加载 + Scheduler 期限计算 | 旧 `00 - 📅 日程管理/[案件编号].yaml` + `案件信息.md` 的结构化字段 |
| `matter_dashboard.md` | 人读看板 | 旧 `[案件编号]案件信息.md` |
| `AGENTS.md` (per-case) | agent 行为约束 | 无（新增层） |
| `工时记录.md` | 律师/财务 | 旧 `00 - 📅 日程管理/[案件编号]工时记录.md` |
| `handoff_ledger.md` (v1.13.0) | orchestrator agent context 经济（ledger-first lazy-load）| 无（新增层；取代"下游全量读上游 .md"）|

## 📊 Agent 分层架构与目录映射

SuitAgent 采用四层架构：

### 📥 输入层 — 文档数据采集

#### DocAnalyzer
- **主要输出**: `02 - 法律研究/案件分析/`（解析结果）
- **次要输出**: `00 - 客户提供/`（客户材料归档）、`04 - 对方证据/` + `06 - 对方法律文书/`（对方提交分流）、`07 - 法院法律文书/`（法院送达）、`08 - 庭审笔录/`
- **功能**：解析法律文档，提取结构化信息

#### EvidenceAnalyzer
- **主要输出**: `03 - 我方证据/`（默认）或 `04 - 对方证据/`（按证据归属判定）
- **功能**：证据三性质证、证据缺口分析、补充证据建议

### 🔍 分析层 — 智能分析与研究

#### IssueIdentifier
- **主要输出**: `02 - 法律研究/案件分析/`
- **功能**：争议焦点识别与归类

#### Researcher
- **主要输出**: `02 - 法律研究/`（root of slot，存放检索原始材料与研究报告）
- **次要输出**: `09 - 参考文件/`（参考判例、法条原文存档）
- **功能**：法条解读、判例检索、法律适用研究

#### Strategist
- **主要输出**: `02 - 法律研究/案件分析/`
- **功能**：SWOT、策略制定、风险评估

#### JiubufaAnalyst
- **主要输出**: `02 - 法律研究/案件分析/`（九步法分析底稿、要件归入对照表、证据缺口清单）
- **触发阈值**: 复杂案件（请求权基础 ≥ 3）/ 起诉答辩前置 / 再审监督评估前置
- **功能**：调起 cn-jiubufa-case-analysis skill 完成 9 步结构化分析；不替代 IssueIdentifier 的轻量提取

#### JudgmentAnalyzer
- **主要输出**: `02 - 法律研究/案件分析/`（判决书审查报告、救济路径对比表、程序瑕疵清单）
- **触发阈值**: post-judgment 阶段（用户问"能不能再审/上诉/监督/异议"）
- **功能**：调起 cn-judgment-analysis skill 完成 IRAC 反向还原与救济路径概率评估；不与 Reviewer（QA agent）混淆

### 📝 输出层 — 文书生成与报告

#### Writer
- **主要输出**: `05 - 我方法律文书/`
- **次要输出**: `01 - 委托材料/`（委托代理协议、授权委托书、谈话笔录）
- **特殊场景**: 法律意见书 / 代理方案 / 风险评估 → `02 - 法律研究/案件分析/` 或 `10 - 综合报告/`
- **客户日常沟通文书** (v1.10.0+): 周报 / 月报 / 进度通报 / 阶段总结 / 风险预警 / 决策建议书 / 客户问询回复 → `10 - 综合报告/客户沟通/`
- **功能**：调起 cn-litigation-drafting（诉讼文书）/ cn-firm-documents（律所对客户**正式**文书）/ cn-client-communications（律所对客户**日常**沟通文书）skill

#### ContractReviewer
- **落位按 matter.yaml `项目类型` profile 分流**：
  - **合同审查 profile**（7-slot）：审查报告 → `02 - 审查报告/`；红线 DOCX + 审查意见书终稿 → `04 - 红线与交付/`；谈判往来 → `03 - 谈判轮次/`
  - **诉讼 profile**（11-slot；合同审查作为诉讼前合同梳理的子任务）：审查报告 + 红线 DOCX → `02 - 法律研究/案件分析/`（诉讼结构无审查报告专属 slot）
- **输入接收位**: `00 - 客户提供/`（待审合同原件，两 profile 一致）
- **特殊场景**: 客户要求"代理方案 / 法律意见书"形式 → 调起 cn-firm-documents skill；合同审查 profile 落 `04 - 红线与交付/`，诉讼 profile 落 `02/案件分析/` 或 `10 - 综合报告/`
- **功能**：调起统一的 `cn-contract-review` skill（v1.8.0+ 统一版本）；skill 内部按 14 类合同自动路由；本 agent 不复制路由逻辑，仅做输入接收 + skill 调起 + 按 profile 落盘工程层

#### TrialPrep（v1.10.0+）
- **主要输出**: `02 - 法律研究/案件分析/庭前准备/`（4 份庭审实战工具：庭审提纲 + 争点对抗预演 + 证人询问问题清单 + 证据出示策略）
- **触发阈值**: 已收到法院开庭传票 + 开庭日期 ≤ 3 周；或用户明示"庭审准备"
- **输入接收**: 上游 IssueIdentifier / Researcher / EvidenceAnalyzer / Strategist / JiubufaAnalyst / JudgmentAnalyzer 已完成产物
- **功能**：调起项目内置 `cn-trial-preparation` skill 完成 4-stage workflow；本 agent 不复制方法论

#### Reporter
- **主要输出**: `10 - 综合报告/`
- **功能**：整合所有内容生成综合报告

#### Summarizer
- **主要输出**: `10 - 综合报告/`
- **次要输出**: `08 - 庭审笔录/`（庭审摘要）
- **功能**：生成各类摘要

### ⚙️ 支持层 — 期限与质量

#### Scheduler
- **主要输出**: `matter.yaml`（关键日期字段更新）+ `工时记录.md`（root level，工时追加）
- **不再使用 numbered slot**——结构化期限与工时记录均迁出 numbered slot 系统
- **功能**：期限管理、工时统计

#### Reviewer
- **主要输出**: 无（不写文件，作 cross-agent 质量审查）
- **功能**：质量把关

#### Postmortem（v1.10.0+）
- **主要输出**: `99 - 复盘沉淀/`（案件复盘报告 + 工作流改进建议 + memory 沉淀清单）
- **触发阈值**: 案件已实际结案（判决送达 / 调解履行完毕 / 撤诉 / 终本）；或用户明示"结案 / 复盘"
- **特色能力**: 人 in the loop 触发 memory 沉淀（用户明确确认后才写入对应 skill 的 memory.md）；matter.yaml 字段更新为"已结案归档"
- **功能**：调起项目内置 `cn-case-postmortem` skill 完成 5-stage workflow（事实回顾 / 5 维度胜败分析 / 工作流改进 / 人 in the loop 沉淀 / 归档）

## 🔄 反向映射（目录 → Agent）

| 目录 | 主要输出 Agent |
|------|-------------|
| `matter.yaml` (root) | Scheduler（写）/ 所有 Agent（读） |
| `matter_dashboard.md` (root) | 人读 / Scheduler（关键日期更新可同步） |
| `AGENTS.md` (root) | 案件创建时定稿 / 所有 agent（读） |
| `工时记录.md` (root) | Scheduler、承办律师 |
| `handoff_ledger.md` (root, v1.13.0) | 写侧：所有 producing agent 落盘后追加 briefing；读侧：orchestrator Explore ledger-first lazy-load（Reviewer 不简化，读 full 产物）|
| `00 - 客户提供/` | DocAnalyzer（解析）、ContractReviewer（合同输入接收）、人工归档 |
| `01 - 委托材料/` | Writer（生成委托文件） |
| `02 - 法律研究/` (root of slot) | Researcher |
| `02 - 法律研究/案件分析/` | DocAnalyzer、IssueIdentifier、Strategist、ContractReviewer、JiubufaAnalyst、JudgmentAnalyzer |
| `02 - 法律研究/案件分析/庭前准备/` (v1.10.0+) | TrialPrep |
| `03 - 我方证据/` | EvidenceAnalyzer（我方部分） |
| `04 - 对方证据/` | DocAnalyzer（解析对方证据）、EvidenceAnalyzer（对方部分） |
| `05 - 我方法律文书/` | Writer |
| `06 - 对方法律文书/` | DocAnalyzer（解析对方文书） |
| `07 - 法院法律文书/` | DocAnalyzer（解析法院送达） |
| `08 - 庭审笔录/` | DocAnalyzer、Summarizer |
| `09 - 参考文件/` | Researcher |
| `10 - 综合报告/` | Reporter、Summarizer |
| `10 - 综合报告/客户沟通/` (v1.10.0+) | Writer（调 cn-client-communications skill）|
| `99 - 复盘沉淀/` (v1.10.0+) | Postmortem（自动复盘 + 人 in the loop memory 沉淀）|

> 上表为**诉讼 profile**（11-slot）。**合同审查 profile**（7-slot，matter.yaml `项目类型: 合同审查`）反向映射：

| 目录（合同审查 profile）| 主要输出 Agent |
|------|-------------|
| 5 件 root 文件 | 同诉讼 profile（含 handoff_ledger.md）|
| `00 - 客户提供/` | DocAnalyzer（合同 OCR/解析）、ContractReviewer（待审合同接收）|
| `01 - 委托材料/` | Writer（委托书/服务协议，独立委托时）|
| `02 - 审查报告/` | ContractReviewer（7-section 报告 + RED/ORANGE/YELLOW + fallback三档 + 签署前清单）|
| `03 - 谈判轮次/` | ContractReviewer / 人工（对方回应 + 逐轮修改 + 版本对比）|
| `04 - 红线与交付/` | ContractReviewer（红线版 DOCX + 审查意见书终稿 _FINAL/_SIGNED）|
| `09 - 参考与playbook/` | Researcher（playbook + 同类范本 + 法规监管）|
| `99 - 复盘沉淀/` | Postmortem（复盘；可复用经验另沉淀 cn-contract-review/memory.md）|

## 💡 使用说明

### 新案件创建
- 由 `new-case` skill 按 matter.yaml `项目类型` 生成对应 profile 结构（matter triplet + 工时记录.md + handoff_ledger.md + 诉讼 11 numbered slots **或** 合同审查 7 numbered slots）
- 案件根目录命名规范见 OutputStandards.md（诉讼 `{原告} 与 {被告} {案由}` / 合同审查 `{客户简称} {合同类型}审查`）

### Agent 上下文加载
- 优先读取 root 级的 `matter.yaml`（结构化 fast path）和 `matter_dashboard.md`（叙事 context）
- 避免遍历 numbered slots——按 Agent 输出表查具体目录

### 与外部 organizer skill 的关系
- `cn-litigation-case-folder-organizer` 默认 14-slot 方案与本规范一致（00-09 + 99）
- 本规范新增 `10 - 综合报告/` slot；外部 organizer skill 的同步补丁由用户在本机维护，不纳入本仓（避免悬空引用）

### 项目内置 4 个 legal skill（v1.9.0+）
- `cn-litigation-drafting` / `cn-contract-review` / `cn-jiubufa-case-analysis` / `cn-judgment-analysis` 均位于 `.claude/skills/` 内，由对应 orchestrator agent 调起；克隆仓库即可用
- 仅 `cn-firm-documents` 因含律所专用模板保持外置

### 与旧 12 层结构的迁移
- 当前真实案件文件夹按既定结构组织，迁移由用户在本机用 organizer skill 跑（迁移说明由用户本机维护，不纳入本仓）
- agent 在迁移前**不要在旧结构案件上执行写操作**——会产生混乱

## 🔄 变更历史

| 版本 | 日期 | 更新内容 |
| :--- | :--- | :--- |
| v3.7 | 2026-05-16 | 合同审查 profile WP3：新增合同审查 profile（7-slot）反向映射 + ContractReviewer 落位按 matter.yaml 项目类型分流（合同审查→02-审查报告/04-红线与交付；诉讼→02-法律研究/案件分析）；使用说明同步双 profile |
| v3.6 | 2026-05-16 | v1.13.0 WP2：新增第 5 个 root level 文件 `handoff_ledger.md`（agent 间结构化简报滚动账本）；布局/职责表/反向映射/new-case 完整结构/说明 同步 4→5 件套 |
| v3.5 | 2026-05-16 | v1.11.1 工程债清偿：变更历史与头部版本号对齐（v3.3–v3.5 TrialPrep/Postmortem 落盘映射 + 庭前准备/复盘沉淀 slot 等修订散见正文与 commit 历史，未单列）；本行起强制头部版本=变更表顶行 |
| v3.2 | 2026-05-08 | Phase 2C：新增 JiubufaAnalyst（九步法分析）+ JudgmentAnalyzer（判决书评审）两个方法论 agent，均输出到 `02 - 法律研究/案件分析/` |
| v3.1 | 2026-05-08 | Phase 2B：新增 ContractReviewer agent（合同审查编排器），输入 `00 - 客户提供/`，输出 `02 - 法律研究/案件分析/` |
| v3.0 | 2026-05-08 | Phase 2A 重构：12 层带 emoji → 11 numbered slots（无 emoji） + 4 root level 文件（matter triplet + 工时记录），新增 99 复盘沉淀，引入 per-case AGENTS.md |
| v2.3 | 2026-01-01 | 明确文档职责边界，目录结构与映射关系 |
| v2.2 | 2026-01-01 | 整合 v2.1 与案件模板使用指南 |
| v2.1 | 2025-11-19 | 详细说明 3 个核心文件的使用 |
| v1.0 | 2025-11-17 | 初始版本 |

---

*本文档定义 SuitAgent 案件结构的核心规范，是 Agent 输出落盘的权威映射。*
