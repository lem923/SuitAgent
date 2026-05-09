# SuitAgent - 诉讼法律服务智能分析系统

> 本仓库 fork 自 [`cat-xierluo/SuitAgent`](https://github.com/cat-xierluo/SuitAgent)。在原项目基础上做了系统性的重构与扩展（增设和修改包括合同审查在内agents、修改案件目录、修改路由设置等）。当前形态相对原项目已有较大差异。

---

## 目录

- [Fork 来源与鸣谢](#fork-来源与鸣谢)
- [项目概述](#项目概述)
- [核心特性](#核心特性)
- [系统架构](#系统架构)
- [13 个 Subagent](#13-个-subagent)
- [案件目录结构](#案件目录结构)
- [skill 依赖](#skill-依赖)
- [变更历史与开发轨迹](#变更历史与开发轨迹)

---

## Fork 来源与鸣谢

**原项目**：[`cat-xierluo/SuitAgent`](https://github.com/cat-xierluo/SuitAgent)（作者 maoking）

原项目定位：基于 Claude Code 架构的诉讼法律服务智能分析系统，10 个专业 AI 代理（agent）协作处理诉讼工作流，落盘到 12 层带 emoji 的案件目录结构（如 `00 - 📅 日程管理`、`02 - 📄 案件分析`）。原项目把案件管理元数据收纳在单一 `[案件编号]案件信息.md` 中。原项目对外部法律 skill（合同审查、要件审判九步法、判决书评审等）的支持限于 advisory 引用，无对应 agent 入口。

**本 fork 与原项目的关系**：保留原项目的整体定位、Subagent 架构思路、与 Claude Code 的集成方式；在此之上进行的实质性改造见下文"相对原项目的修改"。

---

## 项目概述

本 fork 在原项目基础上分 6 个 phase 进行系统性重构与扩展（详见 [`CHANGELOG.md`](CHANGELOG.md)）：

- **Phase 1（Writer 改为 orchestrator）**：`Writer` agent 重写为 orchestrator 模式，删除内嵌 13 类文书模板与质量红线，改为分流到诉讼文书 skill（`cn-litigation-drafting`）与律所对客户文书 skill（`cn-firm-documents`）作为 single source of truth。
- **Phase 2A（案件目录统一）**：12 层带 emoji 的案件目录改为 11 numbered slots（无 emoji，编号 00-10 + 99）+ root level 4 件套（`matter.yaml` / `matter_dashboard.md` / `AGENTS.md` / `工时记录.md`）。原 `案件信息.md` 由 matter triplet 取代；引入 per-case AGENTS.md 加 client identifier 保密硬约束。
- **Phase 2B（新增 ContractReviewer）**：合同审查编排器 agent，按合同类型分流到合同审查 skill。
- **Phase 2C（新增 JiubufaAnalyst + JudgmentAnalyzer）**：要件审判九步法 agent + 裁判文书深度评审 agent。
- **Phase 3（命名规范清理）**：`JudgmentReviewer` 重命名为 `JudgmentAnalyzer`（避免与 `Reviewer` QA agent 字符重叠）；13 个 agent `color` 字段去重；`tools` 字段顺序统一；跨文件死链与旧术语清理。
- **Phase 4（强化触发与路由）**：路由关键词冲突消歧；`DocAnalyzer` 触发词收紧；`Reviewer` 审查范围扩充到全部 13 agent + orchestrator 模式审查规则；`ContractReviewer` 加 Discuss → Execute re-entry 路由；新增"用户显式提及 skill 名时仍走包装 agent"原则。
- **Phase 5（合同审查 skill 统一）**：4 个 cn-contract-review-* specialized skill 合并为统一的 `cn-contract-review` skill（v1.8.0+），覆盖 14 类合同（通用商事 / 买卖 / 租赁 / 服务 / 知识产权与技术许可 / 担保 / 借贷赠与 / 互联网 / 婚姻家事 / 劳动 / 房地产 / 建设工程 / 公司投资 / 政企采购）；REDLINE/ORANGE/YELLOW + fallback 三档（目标/可签/底线）+ playbook + personal-preferences 机制。
- **Phase 6（4 个 legal skill 内置）**：`cn-litigation-drafting` / `cn-contract-review` / `cn-jiubufa-case-analysis` / `cn-judgment-analysis` 全部内置到 `.claude/skills/`，**克隆仓库即可用**；仅 `cn-firm-documents` 因含具体律所名 / 对客户文书规则保持外置。

### 改动对比

| 维度 | 原项目（cat-xierluo） | 本 fork（lem923） |
|------|---------------------|-----------------|
| Subagent 数量 | 10 | **13**（+ ContractReviewer / JiubufaAnalyst / JudgmentAnalyzer） |
| 案件目录结构 | 12 层带 emoji | **11 numbered + 99 + matter triplet（无 emoji）** |
| 案件元数据 | 单文件 `案件信息.md` | **matter triplet：matter.yaml + matter_dashboard.md + AGENTS.md** |
| per-case 保密边界 | 无 | **每案 root 级 AGENTS.md（client identifier 红线 + 文件操作禁区）** |
| 法律 skill 集成 | advisory 引用 | **5 个核心 legal skill（4 内置于 `.claude/skills/` + 1 外置 cn-firm-documents），由 4 个 orchestrator agent 包装调起** |
| 合同审查 | 无 | **ContractReviewer + 统一 cn-contract-review skill（14 类内置路由）** |
| 要件审判九步法 | 无 | **JiubufaAnalyst + 内置 cn-jiubufa-case-analysis skill** |
| 判决书深度评审 | 无 | **JudgmentAnalyzer + 内置 cn-judgment-analysis skill（含救济路径时效预警）** |
| skill 部署 | 用户全局 skill 库（外置） | **4 个核心 legal skill 直接内置；clone 即用** |

---

## 核心特性

- **诉讼全周期覆盖**：从诉前案件结构分析（九步法）→ 起诉/答辩 → 庭审 → 判决评审（IRAC 反推 + 救济路径概率）→ 再审/检察监督文书起草，单一项目内贯通
- **合同审查打通**：合同审查作为 SuitAgent 工作流的一等公民，统一 `cn-contract-review` skill 覆盖 14 类合同（通用商事 / 买卖 / 租赁 / 服务 / 知识产权 / 担保 / 借贷赠与 / 互联网 / 婚姻家事 / 劳动 / 房地产 / 建设工程 / 公司投资 / 政企采购），按类型自动路由
- **clone 即用**：4 个核心 legal skill（`cn-litigation-drafting` / `cn-contract-review` / `cn-jiubufa-case-analysis` / `cn-judgment-analysis`）直接内置于 `.claude/skills/`，无需另行从用户全局 skill 库同步；同事 fork 仓库后立即可跑
- **方法论的 single source of truth**：起草、合同审查、九步法、判决评审等方法论均承载于 skill，agent 只做工程包装（context 承接、文件落盘、命名规范、handoff），方法论改动只需改一处
- **per-case 保密硬约束**：每个案件根目录有自己的 `AGENTS.md`，明确 client identifier 红线（永不进入 web_search/web_fetch query）、文件操作禁区（`_FINAL` / `_SIGNED` / `_盖章` 等后缀的不动）、索引规约
- **结构化期限管理**：`matter.yaml` 的 `关键日期` 字段集中管理上诉 15 日、再审 6 月、检察监督 2 年、执行异议 15 日等法定时效；`JudgmentAnalyzer` 主动核查并对 ≤30 天时效红色加粗预警
- **fallback positions 谈判结构化**：合同审查的 REDLINE / ORANGE 风险条款必填"目标 / 可签底线 / 绝对底线"三档，配合 playbook（组织/审查人标准立场）支持谈判节奏
- **路由精度**：13 个 agent 触发关键词无冲突；多类目命中场景按意图分流（如"再审申请"按动词前缀分到 Writer 起草 vs JudgmentAnalyzer 评估）


---

## 系统架构

```text
┌──────────────────────────────────────────────────────────────┐
│              输入层 (Input Layer)                              │  文档数据采集与解析
│  ┌─────────────────┐ ┌─────────────────┐                     │
│  │  DocAnalyzer    │ │ EvidenceAnalyzer│                     │
│  │   文档分析      │ │   证据分析       │                     │
│  └─────────────────┘ └─────────────────┘                     │
└────────────────┬─────────────────────────────────────────────┘
                 ▼
┌──────────────────────────────────────────────────────────────┐
│              分析层 (Analysis Layer)                           │  智能分析与研究
│  ┌──────────────────┐ ┌──────────────┐ ┌──────────────────┐ │
│  │ IssueIdentifier  │ │  Researcher  │ │    Strategist    │ │
│  │   争议识别（轻量）│ │   法律研究    │ │    诉讼策略      │ │
│  └──────────────────┘ └──────────────┘ └──────────────────┘ │
│  ┌──────────────────────┐ ┌──────────────────────────────┐  │
│  │  JiubufaAnalyst      │ │     JudgmentAnalyzer          │  │
│  │  要件审判九步法（深度）│ │   裁判文书深度评审（IRAC 反推）│  │
│  └──────────────────────┘ └──────────────────────────────┘  │
└────────────────┬─────────────────────────────────────────────┘
                 ▼
┌──────────────────────────────────────────────────────────────┐
│              输出层 (Output Layer)                             │  文书生成与报告
│  ┌──────────────┐ ┌──────────────────┐ ┌─────────────────┐  │
│  │    Writer    │ │ ContractReviewer │ │     Reporter     │  │
│  │  文书起草    │ │    合同审查      │ │    报告整合      │  │
│  └──────────────┘ └──────────────────┘ └─────────────────┘  │
│  ┌──────────────────────────────────────────────────────┐    │
│  │              Summarizer    摘要生成                  │    │
│  └──────────────────────────────────────────────────────┘    │
└────────────────┬─────────────────────────────────────────────┘
                 ▼
┌──────────────────────────────────────────────────────────────┐
│            支持层 (Support Layer)                              │  期限与质量保证
│  ┌──────────────┐ ┌──────────────────────────────────────┐  │
│  │  Scheduler   │ │            Reviewer                   │  │
│  │  日程规划    │ │      跨 agent 质量审查（QA）          │  │
│  └──────────────┘ └──────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
```

---

## 13 个 Subagent

| Agent | 层级 | 职责 | 核心能力 |
| :---- | :--- | :--- | :------- |
| **DocAnalyzer** | 输入层 | 文档分析 | PDF / Word / 图片 OCR 解析、结构化信息提取；判决书 post-judgment 时 hand off 至 JudgmentAnalyzer |
| **EvidenceAnalyzer** | 输入层 | 证据分析 | 三性质证、证明力评估、证据链分析；按归属落 `03 - 我方证据/` 或 `04 - 对方证据/` |
| **IssueIdentifier** | 分析层 | 争议识别（轻量） | 争点提取、法条归类、法律关系梳理；复杂案件（请求权 ≥3）hand off 至 JiubufaAnalyst |
| **Researcher** | 分析层 | 法律研究 | 法条 / 判例 / 司法解释检索（pkulaw / 北大法宝 / 威科 / 裁判文书网），search-first 引用源白名单合规 |
| **Strategist** | 分析层 | 诉讼策略 | SWOT、风险评估、策略方案；上游接 JiubufaAnalyst 底稿（深度场景）或 JudgmentAnalyzer 救济路径表（再审/监督场景） |
| **JiubufaAnalyst** | 分析层 | 要件审判九步法（深度） | 请求权基础穷举、构成要件归入、举证责任矩阵、证据缺口、胜诉概率区间（调起 `cn-jiubufa-case-analysis` skill） |
| **JudgmentAnalyzer** | 分析层 | 裁判文书深度评审 | 判决书 IRAC 反向还原、程序瑕疵审查、上诉/再审/检察监督/执行异议救济路径概率评估 + 时效预警（调起 `cn-judgment-analysis` skill） |
| **Writer** | 输出层 | 文书起草编排器（orchestrator） | 诉讼文书 → `cn-litigation-drafting` skill；律所对客户文书 → `cn-firm-documents` skill |
| **ContractReviewer** | 输出层 | 合同审查编排器（orchestrator） | 调起统一 `cn-contract-review` skill（v1.8.0+），skill 内部按 14 类合同自动路由（通用 / 买卖 / 租赁 / 服务 / 知识产权 / 担保 / 借贷赠与 / 互联网 / 婚姻家事 / 劳动 / 房地产 / 建设工程 / 公司投资 / 政企采购）；输出 REDLINE/ORANGE/YELLOW 报告 + fallback 三档 + 红线 DOCX；含 Discuss → Execute re-entry 路由 |
| **Summarizer** | 输出层 | 摘要生成 | 多层次摘要（详细 / 简洁 / 要点），落 `10 - 综合报告/` |
| **Reporter** | 输出层 | 案件报告 | 整合多 agent 输出生成综合报告，落 `10 - 综合报告/` |
| **Scheduler** | 支持层 | 期限与工时管理 | 法定期限计算（上诉 15 日 / 再审 6 月 / 检察监督 2 年 / 执行异议 15 日）；维护 root 级 `matter.yaml` 关键日期与 `工时记录.md` |
| **Reviewer** | 支持层 | 跨 agent 质量审查（QA） | A/B/C/D 四级评分；orchestrator 模式下审落盘文件不审 skill 内部产物。**与 JudgmentAnalyzer 不同——本 agent 是 QA 层，JudgmentAnalyzer 是法律评审层** |

---

## 案件目录结构

每个案件文件夹的标准布局：

```
[案件文件夹]/
├── matter.yaml              ← 结构化操作数据（当事人 / 案号 / 阶段 / 关键日期 / 文件夹规约）
├── matter_dashboard.md      ← 人读案件看板
├── AGENTS.md                ← per-case agent 边界与保密硬约束
├── 工时记录.md              ← 工时与费用核算
├── 00 - 客户提供/           ← 客户递交的原始材料
├── 01 - 委托材料/           ← 委托代理协议、授权委托书、谈话笔录
├── 02 - 法律研究/           ← 法条、判例、研究报告
│   └── 案件分析/            ← DocAnalyzer / IssueIdentifier / Strategist / JiubufaAnalyst / JudgmentAnalyzer / ContractReviewer 落盘
├── 03 - 我方证据/           ← EvidenceAnalyzer 默认落盘
├── 04 - 对方证据/           ← 对方提交的证据
├── 05 - 我方法律文书/       ← Writer 主要落盘
├── 06 - 对方法律文书/
├── 07 - 法院法律文书/       ← 法院送达的传票、裁定、判决、调解书
├── 08 - 庭审笔录/
├── 09 - 参考文件/           ← 参考法条、参考判例、参考模板
├── 10 - 综合报告/           ← Reporter / Summarizer 落盘
└── 99 - 复盘沉淀/           ← 结案后复盘、归档心得、工作流改进
```

**权威定义**：[`.claude/rules/AgentMapping.md`](.claude/rules/AgentMapping.md)
**新案件搭建**：调起 `new-case` skill

---

## skill 依赖

本项目的 4 个 orchestrator agent 调起 skill 作为方法论 single source of truth。**v1.9.0+ 起，4 个核心 legal skill 直接内置于 `.claude/skills/`，克隆仓库即可用**；仅 `cn-firm-documents` 因含律所专用模板保持外置。

### 项目内置 skill（克隆仓库即可用）

| Skill | 路径 | 被依赖方 | 用途 | License |
|-------|------|---------|------|---------|
| `cn-litigation-drafting` | `.claude/skills/cn-litigation-drafting/` | Writer | 诉讼文书起草 11 类模板（起诉状 / 答辩状 / 上诉状 / 再审 / 检察监督 / 代理词 / 质证意见书 / 财产保全 / 证据清单 / 仲裁 / 反诉） | AGPL v3 |
| `cn-contract-review` | `.claude/skills/cn-contract-review/` | ContractReviewer | 统一合同审查 skill 覆盖 14 类（通用 / 买卖 / 租赁 / 服务 / 知识产权 / 担保 / 借贷赠与 / 互联网 / 婚姻家事 / 劳动 / 房地产 / 建设工程 / 公司投资 / 政企采购）；4-stage workflow + REDLINE/ORANGE/YELLOW + fallback 三档 + playbook 机制 | **双 license**：60 个继承自 contract-copilot v1.5.1 的文件受 CC BY-NC 4.0；其余 26 个原创文件受项目根 AGPL v3。详见 `.claude/skills/cn-contract-review/NOTICE.md` |
| `cn-jiubufa-case-analysis` | `.claude/skills/cn-jiubufa-case-analysis/` | JiubufaAnalyst | 要件审判九步法（请求权基础穷举 / 构成要件归入 / 举证责任矩阵 / 证据缺口 / 胜诉概率区间） | AGPL v3 |
| `cn-judgment-analysis` | `.claude/skills/cn-judgment-analysis/` | JudgmentAnalyzer | 判决书 IRAC 反向还原 + 程序瑕疵审查 + 救济路径概率对比 | AGPL v3 |
| `docx` / `pptx` / `xlsx` / `pdf` / `mineru-ocr` / `md2word` / `new-case` | `.claude/skills/<name>/` | 多 agent 调起 | 文件格式处理工具层 + 案件目录脚手架 | 项目根 LICENSE |

### 外置 skill（用户全局 skill 库；克隆仓库后另行同步）

| Skill | 被依赖方 | 用途 |
|-------|---------|------|
| `cn-firm-documents` | Writer | 律所对外/对客户文书（律师函 / 委托代理协议 / 授权委托书 / 法律意见书 / 谈话笔录 / 调解协议 / 离婚协议审阅 / 刑事格式文书）。**外置** —— 含具体律所名 / 对客户文书规则，律所专用，不入仓 |
| `cn-litigation-case-folder-organizer` | 用户手动触发（非 agent 流程） | 把任意杂乱案件文件夹整理为本项目标准结构 |

详见 [`AGENTS.md`](AGENTS.md) 的"外部 skill 桥接"段。


---

## 变更历史与开发轨迹

完整变更见 [`CHANGELOG.md`](CHANGELOG.md)。本 fork 在原项目基础上的迭代轨迹：

```
v1.9.0 phase6!  4 个 legal skill 内置到 .claude/skills/（仅 cn-firm-documents 外置）
v1.8.0 phase5!  合同审查 skill 统一（4 个 cn-contract-review-* → 1 个 cn-contract-review，14 类合同全覆盖）
v1.7.0 phase4   强化触发与路由（路由精度 + Reviewer 覆盖 + skill 入口标准化）
v1.6.0 phase3!  命名规范清理（JudgmentReviewer → JudgmentAnalyzer BREAKING）
v1.5.0 phase2c  JiubufaAnalyst + JudgmentReviewer agent
v1.4.0 phase2b  ContractReviewer agent
v1.3.0 phase2a! 案件目录结构统一为 11 numbered slots + matter triplet（去 emoji）
v1.2.0 phase1   Writer ↔ cn-litigation-drafting 合并 + advisory 接入
v1.1.0          原项目最后一个继承版本
```
