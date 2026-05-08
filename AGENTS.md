# SuitAgent 法律智能分析系统

## 系统定位

SuitAgent 是面向律师的诉讼辅助系统，通过10个专业Agent处理诉讼全生命周期的分析工作。

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

## 外部 skill 桥接

SuitAgent 的部分 Agent 不内嵌方法论，而是调起外部 skill 作为 single source of truth。安装/同步这些 skill 到本机才能让对应 Agent 完整工作。

### 必需依赖（缺失则 Writer 退化为兜底手工模式）

| Skill | 被依赖方 | 用途 | 来源 |
|-------|---------|------|------|
| `cn-litigation-drafting` | Writer | 诉讼文书起草（起诉状/答辩状/上诉状/再审/检察监督/代理词/质证意见书/财产保全/证据清单/仲裁/反诉等 11 类） | 用户全局 skill 库 |
| `cn-firm-documents` | Writer | 律所对外/对客户文书（律师函/委托代理协议/授权委托书/法律意见书/谈话笔录/调解协议/离婚协议审阅/刑事格式文书等） | 用户全局 skill 库 |

### Advisory 依赖（推荐使用，缺失不阻塞）

| Skill | 推荐被调起方 | 触发场景 |
|-------|------------|---------|
| `cn-jiubufa-case-analysis` | IssueIdentifier、Strategist | 深度争点结构分析（请求权基础穷举 / 要件归入 / 九步法底稿） |
| `cn-judgment-analysis` | DocAnalyzer、Strategist | 判决书 IRAC 反向还原 / 程序瑕疵审查 / 再审与检察监督可行性研判 |

### 与项目内置 skill 的关系

| 项目内置 skill (`.claude/skills/*`) | 用途 |
|-----------|------|
| `docx` / `pptx` / `xlsx` / `pdf` / `mineru-ocr` / `md2word` | 文件格式处理工具层（Writer / DocAnalyzer / Reporter 调起） |
| `new-case` | 案件目录脚手架生成（待 Phase 2 与 cn-litigation-case-folder-organizer 协调） |

外部 skill 的引用路径在各 Agent 文件的"方法论参考"段中显式声明，不要硬编码本机绝对路径。

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
