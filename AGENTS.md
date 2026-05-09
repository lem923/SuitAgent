# SuitAgent 法律智能分析系统

## 系统定位

SuitAgent 是面向律师的诉讼辅助系统，通过 13 个专业 agent 处理诉讼全生命周期的分析工作（含合同审查、要件审判九步法、判决书深度评审三个方法论入口）。

## 核心工作模式

**两种入口，任选其一：**

1. **文档入口**：上传法律文档 → 自动识别类型 → 触发对应工作流
2. **场景入口**：描述需求（如"我收到了起诉状"）→ 自动匹配工作流

> 详细工作流定义见 [`.claude/rules/Workflow.md`](.claude/rules/Workflow.md)

## 协作核心原则

1. **理解用户意图优先** — 在执行任何操作前，先理解用户想要什么结果
2. **遵循工作流规范** — 使用预设工作流或自由组合Agent，减少手动操作
3. **保持上下文连贯** — 案件信息在Agent间传递时，确保关键信息不丢失
4. **输出到正确位置** — 每个Agent有固定的输出目录，遵循 [`.claude/rules/AgentMapping.md`](.claude/rules/AgentMapping.md)
5. **文档边界清晰** — 项目建设文档（`docs/`、`status/`）与案件档案（根目录案件文件夹）严格分开

## 重要指令

- Do what has been asked; nothing more, nothing less.
- NEVER create files unless they're absolutely necessary.
- ALWAYS prefer editing an existing file to creating a new one.
- NEVER proactively create documentation files (*.md) unless requested.

## 律所文件起草规范（对客户文书）

涉及律所对客户的代理方案、法律意见书、风险评估等正式文书：

- **措辞克制**：避免胜负宣告、不堆砌强断言、派生金额用范围词
- **封面排版**：参考律所样本（仿宋三号 + 宋体 22pt 标题），阶段+文书类型合并为单行
- **生成 docx**：套用 `.claude/skills/docx/china_law_firm_template.md` 的格式参数
- **措辞规则细节**：详见 `cn-firm-documents` skill 的 `references/client-doc-style-rules.md`

## 案件目录结构（v3.0+ 统一方案）

每个案件文件夹的标准布局：

```
[案件文件夹]/
├── matter.yaml              ← 结构化操作数据（当事人/案号/阶段/关键日期/文件夹规约）
├── matter_dashboard.md      ← 人读案件看板（取代旧 案件信息.md）
├── AGENTS.md                ← per-case agent 边界与保密硬约束
├── 工时记录.md              ← 工时与费用核算
├── 00 - 客户提供/
├── 01 - 委托材料/
├── 02 - 法律研究/
│   └── 案件分析/            ← 子目录（DocAnalyzer/IssueIdentifier/Strategist 落盘）
├── 03 - 我方证据/           ← EvidenceAnalyzer 默认落盘
├── 04 - 对方证据/
├── 05 - 我方法律文书/       ← Writer 主要落盘
├── 06 - 对方法律文书/
├── 07 - 法院法律文书/
├── 08 - 庭审笔录/
├── 09 - 参考文件/
├── 10 - 综合报告/           ← Reporter / Summarizer 落盘
└── 99 - 复盘沉淀/
```

**权威定义**：[`.claude/rules/AgentMapping.md`](.claude/rules/AgentMapping.md)
**新案件搭建**：调起 `new-case` skill
**已存在案件迁移**（旧 12 层 → 新统一方案）：用户在本机用 `cn-litigation-case-folder-organizer` skill 跑迁移（见 `tmp/案件迁移说明.md`）

## 外部 skill 桥接

SuitAgent 的 4 个 orchestrator agent 不内嵌方法论，而是调起 skill 作为 single source of truth。v1.9.0+ 起，4 个核心 legal skill **直接内置于 `.claude/skills/`**（克隆仓库即可用），仅 `cn-firm-documents` 因含律所专用模板保持外置。

### 必需依赖（外置；缺失则对应 agent 退化为兜底手工模式）

| Skill | 被依赖方 | 用途 | 来源 |
|-------|---------|------|------|
| `cn-firm-documents` | Writer | 律所对外/对客户文书（律师函/委托代理协议/授权委托书/法律意见书/谈话笔录/调解协议/离婚协议审阅/刑事格式文书等）。**外置** —— 含具体律所名 / 对客户文书规则，律所专用，不入仓 | 用户全局 skill 库（外置） |

### Advisory 依赖（推荐使用，缺失不阻塞）

（Phase 2C 之后 jiubufa / judgment-analysis 已升级为必需依赖；目前无 advisory 依赖项。本表保留作未来扩展位。）

### 与项目内置 skill 的关系

| 项目内置 skill (`.claude/skills/*`) | 用途 |
|-----------|------|
| `docx` / `pptx` / `xlsx` / `pdf` / `mineru-ocr` / `md2word` | 文件格式处理工具层（Writer / DocAnalyzer / Reporter 调起） |
| `new-case` | 案件目录脚手架生成（待 Phase 2 与 cn-litigation-case-folder-organizer 协调） |
| **`cn-litigation-drafting`**（v1.9.0+ 内置） | Writer 调起；诉讼文书起草 11 类模板（起诉状 / 答辩状 / 上诉状 / 再审 / 检察监督 / 代理词 / 质证意见书 / 财产保全 / 证据清单 / 仲裁 / 反诉） |
| **`cn-contract-review`**（v1.9.0+ 内置） | ContractReviewer 调起；统一合同审查 skill 覆盖 14 类（通用 / 买卖 / 租赁 / 服务 / 知识产权 / 担保 / 借贷赠与 / 互联网 / 婚姻家事 / 劳动 / 房地产 / 建设工程 / 公司投资 / 政企采购）；4-stage workflow + REDLINE/ORANGE/YELLOW + fallback 三档 + playbook 机制。**双 license**：详见 `.claude/skills/cn-contract-review/NOTICE.md` |
| **`cn-jiubufa-case-analysis`**（v1.9.0+ 内置） | JiubufaAnalyst 调起；要件审判九步法（请求权基础穷举 / 构成要件归入 / 举证责任矩阵 / 证据缺口 / 胜诉概率区间） |
| **`cn-judgment-analysis`**（v1.9.0+ 内置） | JudgmentAnalyzer 调起；判决书 IRAC 反向还原 + 程序瑕疵审查 + 救济路径概率对比 |

外部 skill 的引用路径在各 Agent 文件的"方法论参考"段中显式声明，不要硬编码本机绝对路径。

### Skill 直接调用原则（用户显式提及 skill 名时）

用户显式说"用 cn-jiubufa-case-analysis skill 分析"、"调起 cn-contract-review"、"按 cn-litigation-drafting 起草" 等场景：

- **不旁路 agent 直接触发 skill**——主 agent 仍调起对应包装 agent，由 agent 完成上下文承接、落盘、命名规范、handoff
- **包装 agent 对照表**：

| 用户提及的 skill | 调起的 agent |
|------------------|------------|
| `cn-litigation-drafting`（项目内置）/ `cn-firm-documents`（外置） | Writer |
| `cn-contract-review`（项目内置） | ContractReviewer |
| `cn-jiubufa-case-analysis`（项目内置） | JiubufaAnalyst |
| `cn-judgment-analysis`（项目内置） | JudgmentAnalyzer |
| `cn-litigation-case-folder-organizer` | 不经 agent，由用户本机手动跑（保留人工确认） |
| `new-case` | 不经 agent，由 Scheduler 在新案件场景中调起，或用户主动触发 |

理由：旁路 agent 会丢失 SuitAgent 的工程层（matter context、case slot 落盘、命名规范、与下游 agent 的 handoff），违反"matter 隔离是合规底线"原则。

## 规范文件索引（按需查阅）

| 目录 | 何时查阅 |
|------|---------|
| `.claude/agents/` | 定义某个Agent的详细职责和输出格式 |
| `.claude/rules/` | 工作流、输出标准、时间规范等核心规则 |
| `.claude/skills/` | Skill的使用说明 |
| `.claude/commands/` | /new-case、/deepresearch 等命令用法 |
| `.claude/scripts/` | sync-skills.sh 等工具脚本 |
| `docs/` | 架构文档、路线图 |
| `status/` | 当前任务、变更记录、工作日志 |
