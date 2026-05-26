# 合同审查意见书输出格式

默认以 Markdown 在对话中输出。报告正文超过约 2000 字时，主动询问是否需要 Word 文件。

## 固定 7-section 结构（v1.0+ 统一）

```markdown
# 合同审查意见书

## 一、合同概要

| 字段 | 内容 |
|---|---|
| 合同名称 | |
| 合同类型 | |
| 主要领域 | |
| 适用法律 | |
| 争议解决 | |
| 客户身份 | |
| 对方主体 | |
| 合同金额 | |
| 期限 | |

## 二、审查立场与范围

- 立场：[甲方 / 乙方 / 中立]
- 审查口径：[克制 / 常规 / 强势]
- 适用法律：[中国法 / 香港 / 新加坡 / 加州 / 其他]
- 调用 contract-types：[加载的子类目，如 02-sale/movables-sale + 14-gov-procurement]
- 嵌套专项：[如适用，列出]
- 未覆盖事项：[扫描盲区，如附件未提供]

## 三、风险矩阵

| 等级 | 数量 | 必填 fallback | 说明 |
|------|------|------------|------|
| REDLINE（不改不签） | n | ✅ 必填 | Deal Breaker / Tier 1 |
| ORANGE（强烈建议争取） | n | ✅ 必填 | Strong Preference / Tier 2 |
| YELLOW（优化项） | n | ⚪ 可选 | Concession Candidate / Tier 3 |
| GREEN（提示/有利条款） | n | — | 仅记录 |

## 四、条款级修订清单（按严重程度降序）

### REDLINE（不改不签）

| # | 条款位置 | 问题描述 | 法律依据 | 建议改法 | 目标条款 | 可签底线 | 绝对底线 |
|---|---|---|---|---|---|---|---|
| REDLINE-01 | 第 X 条第 Y 款 | ... | 《XX法》第 X 条 | ... | （你最理想的措辞） | （你能让步到的措辞） | （再让一步就不签） |

### ORANGE（强烈建议争取）

| # | 条款位置 | 问题描述 | 法律依据 | 建议改法 | 目标条款 | 可签底线 | 绝对底线 |
|---|---|---|---|---|---|---|---|

### YELLOW（优化项，fallback 可选填）

| # | 条款位置 | 问题描述 | 法律依据 | 建议改法 |
|---|---|---|---|---|

### GREEN（提示 / 有利条款 / 不进谈判清单）

| # | 条款位置 | 内容 | 提示原因 |
|---|---|---|---|

## 五、谈判策略建议

- **必守底线（Tier 1）**：[列 REDLINE 项的"绝对底线"汇总]
- **可让步事项（Tier 2-3）**：[列 ORANGE / YELLOW 项中可作为筹码交换的]
- **筹码与反制**：[对方可能的让步空间 + 我方可投放的对应让步]
- **谈判顺序建议**：[REDLINE → ORANGE，YELLOW 视进度补]

## 六、商业影响摘要（Business Impact Summary）

- **整体风险评估**：[基于 REDLINE / ORANGE / YELLOW 数量与权重]
- **Top 3 关键问题**：
  1. ...
  2. ...
  3. ...
- **签约时限因素**：[如有时间压力，说明]
- **预估谈判轮次**：[1-2 轮 / 3+ 轮]
- **是否需要外部协查**：[如涉及税务 / 外汇 / 监管审批，注明]

## 七、签署结论

```text
✅ 可直接签署   （可选，仅在 REDLINE = 0 且 ORANGE 已全部接受 fallback 可签底线时）
⚠️ 修订后可签   （ORANGE / REDLINE 存在但有可执行 fallback）
❌ 不建议签署   （REDLINE 存在且对方不接受可签底线 / 不可签底线）
```

任何模糊结论不得过线。信息不足时，先要求用户补充必要信息再出结论。

## （附）八、待确认事项

- 附件：[未提供 / 提供不完整]
- 主体资料：[需补 / 已确认]
- 前序协议：[需补 / 不适用]
- 其他：[列出]

```

## 语言与法条规范

- 法条引用格式：`《XX法》第X条第X款（第X项）`；引用前先核对现行有效版本（search-first）
- 司法解释：法释〔YYYY〕XX 号
- HK 判例：`[YYYY] reporter cite`
- 加州 / 联邦：默认 Bluebook
- 引用他人原文（法条/判决/合同条款）保留原文；译文另起一段标注 `[译文]`

## Word DOCX 红线版生成规则（v1.1+ 强制）

红线版是直接发给对方法务的对外文书，格式不一致 / 批注混在正文都会暴露"AI 生成"痕迹并影响专业度。两条硬规则：

### A. 参数化字体段落模板（非全局固定值）

**不要用预设的"仿宋三号 / 30pt / 首行 2 字"硬参数**——必须在 Stage 3 Execute 启动时**从原合同自身抽取**模板，再套用到红线版的修订插入和新增段落。

**抽取步骤**（Stage 3 启动时强制执行）：

```
1. 解包原合同 docx → word/document.xml
2. 取首批 3-5 个正文段落（含"乙方/承租人/支付/履行"等正文身份词）的：
   - pPr 全文（spacing line/lineRule, ind firstLine/firstLineChars, jc, widowControl 等）
   - run rPr 全文（rFonts hint/ascii/hAnsi/eastAsia/cs, sz, szCs, kern, color）
3. 取 docDefaults 的 rPrDefault 作为兜底
4. 形成本次红线版的"原合同正文样式模板"（PARA_PPR_TEMPLATE + RUN_RPR_TEMPLATE）
```

**应用规则**：
- 所有 `<w:ins>` 内的 `<w:r>` 的 `<w:rPr>` 必须含 RUN_RPR_TEMPLATE 全部字段，**仅将 color 替换为 `C00000` 作为 Track Changes 视觉标识**（其他字段——字体/字号/字距/hint——严禁省略）
- 新插入的段落 `<w:p>` 必须含 PARA_PPR_TEMPLATE（spacing + ind + jc 三件套不可省）
- 替换原文段落不动其 pPr，仅修订 run 内容
- 不同合同字体字号可能差异极大（仿宋_GB2312 16pt / 宋体 11pt / Times New Roman 12pt），**绝不使用 hardcoded 默认值**

**反例（v1.0 错误，禁止）**：

```xml
<!-- 错：仅 color，无字体字号，渲染时回退到 docDefaults 宋体 11pt -->
<w:ins ...><w:r><w:rPr><w:color w:val="C00000"/></w:rPr><w:t>...</w:t></w:r></w:ins>
```

**正例（v1.1+ 强制）**：

```xml
<w:ins ...><w:r><w:rPr>
  <w:rFonts w:hint="eastAsia" w:ascii="仿宋_GB2312" w:hAnsi="宋体"
            w:eastAsia="仿宋_GB2312" w:cs="宋体"/>
  <w:color w:val="C00000"/>
  <w:kern w:val="0"/>
  <w:sz w:val="32"/><w:szCs w:val="32"/>
</w:rPr><w:t>...</w:t></w:r></w:ins>
```

### B. NOTE / EDIT 双轨制（修订与批注分离）

红线版含两类内容，**绝对禁止混在 w:ins 里**：

| 类型 | 内容 | 落地方式 |
|---|---|---|
| **EDIT** 条款级修订 | 实际改动条款文字（押金扣除条件、违约金率、催告期日数等）| `<w:ins>` / `<w:del>` 在 document.xml 正文 |
| **NOTE** 提示性批注 | 律师对修订理由的解释（"REDLINE-01 押金回归实际损失原则"、"ORANGE-05 催告期 7→15 日"等）；整体审查标记说明；建议但未实际修订的事项（如 YELLOW 提示） | Word Comments：独立 `word/comments.xml` 文件 + `commentRangeStart/End` + `commentReference` 锚点 |

**反例（v1.0 错误，禁止）**：

```xml
<!-- 错：把"【hhwy-bj 批注 YYYY-MM-DD】REDLINE-01 ..."当作 w:ins 插入正文 -->
<w:ins ...><w:r><w:t>【hhwy-bj 批注 2026-05-26】REDLINE-01 押金扣除回归...</w:t></w:r></w:ins>
```

**正例（v1.1+ 强制）**：

1. **comments.xml** 文件（`word/comments.xml`）：

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<w:comments xmlns:w="http://schemas.openxmlformats.org/wordprocessingml/2006/main">
  <w:comment w:id="0" w:author="hhwy-bj" w:date="2026-05-26T12:00:00Z" w:initials="HC">
    <w:p><w:r><w:rPr>...RUN_RPR_TEMPLATE (sz 可缩小到 21 节省边栏空间)...</w:rPr>
    <w:t>REDLINE-01 押金扣除回归实际损失原则。原条款【违约即全扣】与押金担保性质相悖…</w:t>
    </w:r></w:p>
  </w:comment>
  ...
</w:comments>
```

2. **document.xml 锚点三件套**（围着被批注的文本）：

```xml
<w:commentRangeStart w:id="0"/>
<w:ins ...>...EDIT 内容...</w:ins>
<w:commentRangeEnd w:id="0"/>
<w:r><w:rPr><w:rStyle w:val="CommentReference"/></w:rPr>
  <w:commentReference w:id="0"/>
</w:r>
```

3. **`[Content_Types].xml`** 注册（必须）：

```xml
<Override PartName="/word/comments.xml"
  ContentType="application/vnd.openxmlformats-officedocument.wordprocessingml.comments+xml"/>
```

4. **`word/_rels/document.xml.rels`** 注册（必须）：

```xml
<Relationship Id="rIdN"
  Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/comments"
  Target="comments.xml"/>
```

### C. 批注锚点策略（NOTE → 锚点位置匹配）

| NOTE 类型 | 锚点策略 |
|---|---|
| 紧邻一个 EDIT（前 / 后 1-2 个 ins 块内） | 锚到该 EDIT 的文本范围（`commentRangeStart/End` 包裹 `<w:ins>`） |
| 多个 EDIT 同源（如 REDLINE-01 押金扣除 1-3 三条都对应同一 NOTE） | 该 NOTE 锚到首条 EDIT；其余 EDIT 可加各自独立 NOTE 或共享 |
| 整体审查标记说明（"本红线版由 XX 律所制作…"） | 锚到合同标题（如"房屋租赁合同"/"股权转让合同"）首次出现处 |
| 纯建议（YELLOW 类，未实际修订正文） | 锚到对应章节标题（如"声明及保证"/"附则"/"违约责任"），气泡形式提醒承办律师 |

### D. Track Changes 作者（personalization）

- 默认 `Claude`；律所同事使用时改为 `hhwy-bj`（合弘威宇北京）或承办律师标识（见用户 memory `feedback_revision_author_default`）
- `comments.xml` 的 `w:author` / `w:initials` 与 Track Changes 作者保持一致

### E. Stage 3 Execute 落盘前必做核验

| 核验项 | 通过标准 |
|---|---|
| 字体一致性 | 用 `pdftoppm -r 110` 渲 PDF → PNG，目检修订插入文本与原文字体字号一致 |
| 批注分离完整性 | `grep -c '【.*批注'` 正文 .txt 应返回 **0** |
| comments.xml 注册 | `[Content_Types].xml` 与 `_rels/document.xml.rels` 已含 comments part |
| Word/WPS 实际打开 | 边栏正确显示气泡批注；正文修订呈红色下划线 |

### F. 套用既有 docx skill

- 套用 `.claude/skills/docx/china_law_firm_template.md` 时**仅作为兜底默认**——若原合同有自己的字体段落体系（如政府类常用方正小标宋），抽取后覆盖该默认
- 不要把"仿宋三号 / 30pt / 首行 2 字"作为永远的硬模板

## 不允许的输出形式

- ❌ 中立陈述 / "对双方均有利"稀释立场
- ❌ "请主办律师酌定"等回避结论
- ❌ 未标注 `⚠️[待核]` 的不确定法条
- ❌ 直接搬用《民法典》具体条文作为非中国法合同的结论（应走 cross-border-review 降级）
- ❌ ORANGE / REDLINE 缺 fallback 三档（YELLOW 可选填）
- ❌ 红线版 DOCX 中 w:ins 内 run rPr 仅含 color 而无字体字号（v1.1+ 视为质量缺陷）
- ❌ 红线版 DOCX 中把"【XX 批注 YYYY-MM-DD】..."律师批注当 w:ins 混在正文（v1.1+ 视为质量缺陷，必须分离到 word/comments.xml）

## 变更历史

| 版本 | 日期 | 更新内容 |
| :--- | :--- | :--- |
| v1.1 | 2026-05-27 | Word DOCX 红线版生成规则升级：参数化字体段落模板（从原合同抽取，禁用 hardcoded 默认）+ NOTE/EDIT 双轨制（条款修订走 Track Changes / 提示性批注走 word/comments.xml）+ 锚点策略 + Stage 3 落盘前核验 4 项 |
| v1.0 | 2026-01-01 | 初始 7-section 输出格式 |
