---
name: new-case
description: 创建新案件——按 SuitAgent 统一案件目录结构（11 numbered slots + matter triplet + 工时记录.md root level）建立案件框架。生成 matter.yaml（结构化数据）/ matter_dashboard.md（人读看板）/ AGENTS.md（per-case 边界）/ 工时记录.md，搭建 11 个 numbered 目录骨架，按客户提供的材料分类入位。不要用于：单独生成法律文书、进行法律研究、证据分析等非案件初始化任务。
license: CC BY-NC-SA 4.0 - 详见 LICENSE.txt
---

# New Case - 创建新案件

按 SuitAgent v3+ 案件目录结构建立新案件框架，生成 matter triplet（matter.yaml + matter_dashboard.md + AGENTS.md）+ 工时记录.md（均在案件根目录）+ 11 个 numbered slot 目录。

> 目录结构权威定义见 `.claude/rules/AgentMapping.md`。本 skill 是该规范的执行者。

## 适用场景

1. 新建案件档案，需要建立标准化目录结构
2. 已有案件材料，需要整理成统一格式
3. 接收新案件，需要快速搭建案件框架

## 触发方式

### 自然语言触发
- "整理这个案件材料：/path/to/case-folder"
- "帮我建立案件结构"
- "创建新案件"
- "新建案件"

### 参数化触发

| 参数 | 必需 | 说明 | 示例 |
|------|------|------|------|
| `--case-id` | 是 | 案件编号 | `[2025]京0105民初1234号` 或 `260507 华雄公司 土地使用权转让纠纷` |
| `--client-name` | 否 | 委托人姓名/公司名 | `北京科技有限公司` |
| `--case-type` | 否 | 案件类型 | `民事`/`刑事`/`行政` |
| `--case-cause` | 否 | 案由 | `合同纠纷` |
| `--opposite-party` | 否 | 对方当事人 | `上海某某公司` |
| `--input-dir` | 否 | 案件材料目录 | `/path/to/materials` |

## 案件根目录结构（生成目标）

```
[案件文件夹]/
├── matter.yaml              ← 结构化操作数据（必生成）
├── matter_dashboard.md      ← 人读案件看板（必生成）
├── AGENTS.md                ← per-case agent 边界（必生成）
├── 工时记录.md              ← 工时与费用（必生成）
├── 00 - 客户提供/
├── 01 - 委托材料/
├── 02 - 法律研究/
│   └── 案件分析/
├── 03 - 我方证据/
├── 04 - 对方证据/
├── 05 - 我方法律文书/
├── 06 - 对方法律文书/
├── 07 - 法院法律文书/
├── 08 - 庭审笔录/
├── 09 - 参考文件/
├── 10 - 综合报告/
└── 99 - 复盘沉淀/
```

## 工作流程

### 第一步：分析输入材料

1. **扫描案件文件夹**，列出所有文件
2. **识别材料类型**：法律服务方案（.md/.docx）、沟通记录、证据材料（图片/PDF）、委托材料、身份证明文件、其他文档
3. **提取关键信息**：当事人（原告/被告/第三人）、案件类型/案由、涉案金额、对方当事人

### 第二步：创建案件根目录与 11 个 numbered slot

按上图结构 `mkdir`，创建 11 个 numbered 目录与 `02 - 法律研究/案件分析/` 子目录。

> 各 slot 的内容定义、Agent 输出关联见 `.claude/rules/AgentMapping.md`。

### 第三步：生成 matter triplet + 工时记录.md

#### 3.1 matter.yaml（结构化操作数据）

> 字段 schema 见 [references/yaml-schema.md](references/yaml-schema.md)

核心字段：
- `matter_id` / `matter_name` / `short_code`
- `案号`（如已立案）/ `案由` / `案件类型` / `当前阶段`
- `当事人`：原告 / 被告 / 第三人
- `代理律师` / `律所`
- `争议焦点` / `诉讼请求金额`
- `关键日期`：律所立案日期 / 立案日期 / 应诉期限 / 举证期限 / 上诉期限 / 开庭日期
- `文件夹规约`：禁动后缀（_FINAL / _SIGNED / _盖章 / _终审）

#### 3.2 matter_dashboard.md（人读看板）

> 模板见 [references/case-info-template.md](references/case-info-template.md)（其中"案件信息"模板适配为 dashboard 格式）

包含：基本信息总览、时间线、近期任务、风险提示、阶段进展。**不再生成旧 `[案件编号] 案件信息.md`**——本 dashboard 替代。

#### 3.3 AGENTS.md（per-case agent 边界）

模板：

```markdown
# [案件编号] - Agent 协作边界

> 本文件继承根目录 CLAUDE.md / AGENTS.md，仅写 case-specific 约束。

## 保密硬约束（per-case）
- 当事人姓名 / 案号 / 对方代理 / 客户内部代号 等 client identifier **永不出现于 web_search / web_fetch / 公开外发**
- 客户文件原文片段不进入公开 web 上下文
- 涉港跨境敏感事项不调用境内不可用的境外服务做内容处理

## 文件操作禁区
- 文件名含 `_FINAL` / `_EXECUTED` / `_SIGNED` / `_盖章` / `_用印` / `_终审` 的不动
- 不跨 matter 移动文件
- 删除/覆盖前必须明确确认

## 索引规约
- 案件入口：matter.yaml + matter_dashboard.md
- 期限权威源：matter.yaml 的 `关键日期` 字段
- 工时权威源：工时记录.md（root level）
```

#### 3.4 工时记录.md（root level）

> 模板见 [references/timesheet-template.md](references/timesheet-template.md)

### 第四步：材料分类入位

按下表把客户提供的材料移到对应 slot：

| 材料类型 | 目标 slot |
|---------|----------|
| 客户递交的原始材料 | `00 - 客户提供/` |
| 委托代理协议 / 授权委托书 / 谈话笔录 | `01 - 委托材料/` |
| 我方手头的证据 | `03 - 我方证据/` |
| 对方提交的证据（如已有） | `04 - 对方证据/` |
| 法院送达的传票/裁定/判决/调解书 | `07 - 法院法律文书/` |
| 庭审记录 | `08 - 庭审笔录/` |
| 参考法条/判例/模板 | `09 - 参考文件/` |

### 可选第五步：生成法律服务方案

如材料中包含初步沟通记录，可调起 `cn-firm-documents` skill 起草法律服务方案到 `01 - 委托材料/`（视为律所对客户文书）。

## 关键规则

### 案件编号规则

格式：`{YYMMNN} {原告简称} 与 {被告简称} {案由}`

示例：
- `260221 张家宁 与 pokebone 模型 著作权`
- `260315 甲公司 与 乙公司 合同纠纷`

### 文件命名规则

- matter.yaml / matter_dashboard.md / AGENTS.md / 工时记录.md：固定名（不加日期前缀，root level）
- 各 slot 内一次性输出文件：`YYMMDD [文书类型].md`（详见 OutputStandards.md）
- 文件名一律 UTF-8，不改 GBK / 拼音 ASCII

## 时间要求

使用系统当前时间 `date "+%Y-%m-%d"`：
- matter.yaml 的 `创建日期` / `更新日期` 字段为当前日期
- matter_dashboard.md 的"更新历史"首条为当前日期
- 时间线逻辑正确（过去→现在→未来）
- 关键日期剩余天数计算准确

## 输出验证

完成案件整理后，确认：

- [ ] 案件根目录已创建
- [ ] matter.yaml / matter_dashboard.md / AGENTS.md / 工时记录.md 全部生成（root level）
- [ ] 11 个 numbered slot 目录全部创建（含 `02 - 法律研究/案件分析/` 子目录）
- [ ] 客户提供的材料已分类移到对应 slot
- [ ] 文件命名符合规范（YYMMDD 前缀 + UTF-8 中文）
- [ ] 时间逻辑正确

## 禁止事项

- 禁止生成旧 `[案件编号] 案件信息.md`（已被 matter_dashboard.md 替代）
- 禁止生成旧 `00 - 📅 日程管理/[案件编号].yaml`（已被 root 级 matter.yaml 替代）
- 禁止使用旧 12 层带 emoji 目录命名
- 禁止在案件档案中记录项目自身的 SuitAgent 系统信息
- 禁止创建额外的说明文档或 README
- 禁止遗漏任何已有材料
- 禁止虚构案件信息
