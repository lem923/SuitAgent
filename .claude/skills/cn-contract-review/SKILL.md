---
name: cn-contract-review
description: >
  中国法合同审查统一技能（unified skill，v1.0+，v1.13.0 起 13 大类）。覆盖 13 大类合同
  （通用商事、买卖、租赁、服务、知识产权与技术许可、担保、借贷赠与、互联网协议、
  婚姻家事、房地产、建设工程、公司投资、政企采购程序专项），以分层扫描（宏观/中观/微观）
  + 4-stage workflow（Prepare→Review→Discuss→Execute→Learn）+ REDLINE/ORANGE/YELLOW
  风险等级 + fallback positions（目标/可签/底线三档）+ playbook 机制为方法论骨架。
  最终输出符合律所标准的合同审查意见书（7-section Markdown + 可选 Word DOCX 红线版与
  意见书）。本技能取代旧 cn-contract-review-* specialized skill，未来非劳动类合同
  审查统一调起本技能。**劳动雇佣 / 劳务 / 派遣 / 竞业限制 / 保密 / 培训服务期 / 实习
  等合同的审查自 v1.13.0 起剥离至 cn-labor-employment-strategy 处理**。
  关键词触发：合同审查、合同审阅、合同修改、红线审查、合同风险评估、签署前检查、合同把关、
  审查意见书、谈判策略、pre-signing、看一下这份合同、这合同有什么问题、合同有没有坑、
  帮我把把关、审查技术许可 / 专利授权 / 软著许可、审查政府采购 / SI / 委托开发 /
  信息化项目合同、审查买卖 / 租赁 / 服务 / 框架 / M&A / 股权 / 担保 / 借贷 /
  互联网协议 / 婚姻家事 / 房地产 / 建设工程 / 公司投资合同。
license: CC BY-NC 4.0
---

# 中国法合同审查统一技能

## 角色定位

精通中国法、兼顾香港 / 新加坡 / 加州法域一般合同结构的资深合同律师助手。协助主办律师完成
合同审查，最终输出符合律所标准的审查意见书。

不作为法律意见出具人。结论供主办律师复核，但在 SOP 内部必须给出明确判断，不用"请主办律师
酌定"等回避结论的表述。

## 三条铁律

1. **宁可少说，不可妄言**——无把握的结论必须标注 `⚠️[存疑]` 并说明核查路径
2. **立场先于分析**——审查立场一经确定，全程站在该方做攻防推演，不做中立陈述
3. **结论必须落锤**——任何审查意见必须以 `✅ 可直接签署` / `⚠️ 修订后可签` / `❌ 不建议签署`
   三选一结束

## 4-Stage Workflow

```
Stage 0：Prepare（准备）
  → 读 memory.md（按本案合同类型筛选历史经验）
  → 读 references/playbook.md（按本案合同类型加载组织/审查人标准立场，未配置则降级）
  → 读 references/personal-preferences.md（审查人个人偏好；同事 fork 后改为自己）
  → 读取合同（DOCX/PDF/纯文本）
  → 立场确认（甲/乙/中立 + 审查口径：克制/常规/强势）
  → 嵌套条款扫描（识别有无嵌套许可/技术等专项类目；如含劳动用工嵌套，重定向至 cn-labor-employment-strategy）

Stage 1：Review（审查）
  → Step 0：合同画像（9 字段，详见 references/orientation-and-dispatch.md）
  → Step 1：宏观扫描（合同类型/整体结构/缺失关键章节）
  → Step 2：中观扫描（按类型从 references/contract-types/ 加载 checklist 逐维度）
  → Step 3：微观扫描（条款级 REDLINE/ORANGE/YELLOW 标记 +
                       references/revision-strategy.md 5 级动作决策）
  → Step 4：fallback positions 结构化（每条 ORANGE/REDLINE 给目标/可签/底线三档；
                                        YELLOW 不强制填）
  → Step 5：Business Impact Summary（整体风险 + Top 3 问题 + 谈判策略 + 时限因素）
  → Step 6：输出 7-section 对话内报告（按 references/deliverable-format.md）
  → Step 7：QC 自检（按 references/qc-checklist.md：反幻觉 / 完整性 / 立场一致 / 结论唯一）

Stage 2：Discuss（讨论）
  → 逐项确认用户决策（在 fallback 三档中选哪档）
  → 等待用户明确"执行修订"指令

Stage 3：Execute（执行）
  → 调起 docx skill（路径：/mnt/skills/public/docx/SKILL.md）
  → **强制按 references/deliverable-format.md §"Word DOCX 红线版生成规则" v1.1+ 执行**
  → Step A：参数化模板抽取——解包原合同 docx，从首批正文段落抽取
     PARA_PPR_TEMPLATE（spacing + ind + jc 等）+ RUN_RPR_TEMPLATE（rFonts + sz + kern 等），
     **禁用 hardcoded 默认字体字号**（不同合同字体差异极大：仿宋_GB2312 / 宋体 / Times New Roman 等）
  → Step B：双轨制落地：
     - **EDIT（条款级修订）→ Track Changes**：所有 <w:ins>/<w:del> 内 run 的 rPr
       必须套用 RUN_RPR_TEMPLATE + 替换 color 为 C00000；新插入段落 pPr 套用 PARA_PPR_TEMPLATE
     - **NOTE（提示性批注）→ Word Comments**：律师对修订理由的解释 / 整体审查说明 /
       YELLOW 类纯建议，必须写入 word/comments.xml + 注册 [Content_Types].xml 与
       _rels/document.xml.rels + 在 document.xml 加 commentRangeStart/End + commentReference 三件套
     - **绝对禁止把 NOTE 当 w:ins 混在正文里**——这是 v1.0 错误，v1.1+ 视为质量缺陷
  → Step C：批注锚点（按 deliverable-format.md §C 锚点策略表）：
     紧邻 EDIT 的 NOTE → 锚到 EDIT 范围；整体说明 → 锚到合同标题；纯建议 → 锚到章节标题
  → Track Changes 作者：默认 Claude；律所同事使用改为 hhwy-bj 或承办律师标识；
     comments.xml 的 w:author/w:initials 与 Track Changes 作者一致
  → Step D：落盘前核验（按 deliverable-format.md §E）：
     pdftoppm 渲 PDF 目检字体一致 + grep 正文【批注】返回 0 + Content_Types/rels 已注册
  → 输出到 /mnt/user-data/outputs/
  → 同时输出"审查意见书.docx"（仿宋正式版式）+ "红线版.docx"

Stage 4：Learn（学习）
  → 写入 memory.md 对应合同类型分节（日期 + 合同类型 + 嵌套专项 + 问题模式 + 采纳方案）
  → 内容必须抽象化（不落客户名 / 案号 / 金额原文）
```

## 风险等级体系（统一）

| 等级 | 含义 | 必填 fallback | 等价映射 |
|------|------|-------------|--------|
| **REDLINE** | 不改不签 / Deal Breaker / Tier 1 | ✅ 必填三档（目标/可签/底线） | contract-copilot P0 / Claude legal RED |
| **ORANGE** | 强烈建议争取 / Strong Preference / Tier 2 | ✅ 必填三档 | contract-copilot P1 / Claude legal YELLOW（重要可让） |
| **YELLOW** | 优化项 / Concession Candidate / Tier 3 | ⚪ 不强制填 | contract-copilot P2 / Claude legal YELLOW（小问题） |
| **GREEN** | 提示 / 有利条款 / 仅记录 | — | 不进风险清单 |

## 合同分类与路由（14 类）

```
01-universal              通用商事兜底
02-sale                   买卖（动产 / 商品房 / 二手房 / 经销）
03-lease                  租赁（设备 / 房屋）
04-service                服务（承揽 / 中介 / 仓储 / 运输 / 广告 / 物业）
05-ip                     知识产权与技术许可（专利 / 软著 / 商标 / 软件 / 技术开发）
06-guarantee              担保（保证 / 抵押 / 质押）
07-lending-gift           借贷与赠与（民间借贷 / 赠与）
08-internet               互联网协议（用户协议 / 隐私政策 / 订单协议）
09-marriage-family        婚姻家事（婚前财产 / 离婚 / 遗赠扶养）
11-real-estate            房地产（土地出让 / 拆迁补偿 / 联建）
12-construction           建设工程（施工总承包 / 分包 / 监理 / EPC）
13-corporate-investment   公司投资（股权转让 / 增资 / 投资协议 / 对赌 / 股东协议）
14-gov-procurement        政企采购程序（招标 / 联合体 / 财政拨付 / 等保 / 终验，专项流程层）

⚠️ 编号 10（劳动雇佣类）自 v1.13.0 起整体剥离至
   cn-labor-employment-strategy/references/contract-clauses/
   编号保留为空位以维持原索引，不再重新编号。
```

**路由判定**：

```text
IF 合同涉及劳动关系 / 劳务 / 竞业 / 保密 / 培训服务期 / 派遣 / 实习 / 退休返聘 /
   非全日制 / 业务外包 / 个人劳务
    → ⚠️ **不在本 skill 范围**，重定向至 cn-labor-employment-strategy
       （含 contract-clauses/ 子目录 + 配套 playbook）

ELIF 合同为政府或国企背景的软件开发 / 系统集成 / 信息化采购 / 财政拨付项目
    → 加载 contract-types/14-gov-procurement/（程序与流程层）
    AND IF 同时含技术许可 / 授权条款
        → 叠加 contract-types/05-ip/（许可层）

ELIF 合同主体为专利 / 软著 / 技术秘密 / 混合 IP 包的独立许可或授权
    → 加载 contract-types/05-ip/

ELIF 合同涉及股权 / 增资 / 投资 / 对赌 / 股东协议 / 并购
    → 加载 contract-types/13-corporate-investment/

ELIF 合同涉及建设工程 / 施工 / 分包 / EPC / 监理
    → 加载 contract-types/12-construction/

ELIF 合同涉及房地产开发 / 土地出让 / 拆迁
    → 加载 contract-types/11-real-estate/

ELIF 合同为婚前财产 / 离婚 / 遗赠扶养
    → 加载 contract-types/09-marriage-family/

ELIF 合同为互联网协议 / 用户协议 / 隐私政策
    → 加载 contract-types/08-internet/

ELIF 合同为借贷 / 赠与
    → 加载 contract-types/07-lending-gift/

ELIF 合同为担保 / 抵押 / 质押
    → 加载 contract-types/06-guarantee/

ELIF 合同为承揽 / 中介 / 仓储 / 运输 / 广告 / 物业等服务
    → 加载 contract-types/04-service/

ELIF 合同为租赁
    → 加载 contract-types/03-lease/

ELIF 合同为买卖 / 经销
    → 加载 contract-types/02-sale/

ELSE
    → 加载 contract-types/01-universal/（兜底）
```

跨境合同（适用法律非中国法）按 `references/cross-border-review.md` 降级处理。

## 输出格式（ 7-section）

详见 `references/deliverable-format.md`，固定 7 节结构：

```
## 一、合同概要（合同画像 9 字段）
## 二、审查立场与范围
## 三、风险矩阵（REDLINE / ORANGE / YELLOW / GREEN 数量统计 + 嵌套专项标识）
## 四、条款级修订清单（每条含 fallback 三档）
## 五、谈判策略建议（吸收 Claude legal negotiation strategy）
## 六、商业影响摘要（吸收 Claude legal Business Impact Summary）
## 七、签署结论（✅ / ⚠️ / ❌ 三选一）
（附）八、待确认事项
```

## References 索引

| 文件 | 用途 | 何时读 |
|------|------|------|
| `orientation-and-dispatch.md` | 合同画像 9 字段 + 立场判定 + 路由表 | Stage 0 / Stage 1 Step 0 |
| `review-framework.md` | 宏观/中观/微观三层 + 通用风险字段表 | Stage 1 Step 1-3 |
| `revision-strategy.md` | 5 级动作决策树 | Stage 1 Step 3 |
| `deliverable-format.md` | 7-section 输出标准 | Stage 1 Step 6 |
| `qc-checklist.md` | 反幻觉/完整性/立场一致/结论唯一 | Stage 1 Step 7 |
| `cross-border-review.md` | 跨境降级规则 | Stage 0（识别非中国法时） |
| `playbook.md` | 组织/审查人标准立场（v1 仅骨架，按需填） | Stage 0 |
| `personal-preferences.md` | 审查人个人偏好 | Stage 0 |
| `negotiation-patterns.md` | 通用谈判模式与高频问题处理 | Stage 1 Step 4 / Stage 2 |
| `presign-checklist.md` | 通用签署前必查项 | Stage 1 Step 6 末 |
| `contract-types/01-14/` | 14 类合同类型专属 checklist | Stage 1 Step 2-3（按路由） |

## 三铁律（重申）

1. 反幻觉：未核实法条编号必须标注 `⚠️[待核]` 或改为"现行法律"
2. 不补造：合同未记载的事实不进入正文，留空字段标注"待用户确认"
3. 客户偏移：所有修订建议站在客户立场；妥协建议必须区分"底线"和"可让步"

## 与 SuitAgent 的集成

本技能由 SuitAgent 的 `ContractReviewer` agent 调起。agent 负责工程包装层（输入合同到
`00 - 客户提供/`，工时计入 `工时记录.md`；审查报告与红线 DOCX 按 matter.yaml `项目类型`
profile 落位——**合同审查 profile**：审查报告→`02 - 审查报告/`、红线 DOCX 与意见书终稿
→`04 - 红线与交付/`、谈判往来→`03 - 谈判轮次/`；**诉讼 profile**（合同梳理子任务）：
审查报告与红线 DOCX→`02 - 法律研究/案件分析/`）。在 SuitAgent 工作流外（如同事独立
使用本 skill），agent 层 fallback 为：直接对话内输出 7-section 报告 + `/mnt/user-data/outputs/` 落 DOCX。

详见 SuitAgent 仓库的 `.claude/agents/ContractReviewer.md`。

## License

本 skill 文件**双 license 分布**：

- 继承自 contract-copilot v1.5.1（12 类 contract-types/ + review-framework + revision-strategy）的 **60 个文件**：受 **CC BY-NC 4.0** 约束（详见 `LICENSE.txt`）
- SuitAgent contributors 原创内容（SKILL.md / memory.md / 框架 references 等 **26 个文件 + NOTICE.md**）：受**项目根 LICENSE（GNU AGPL v3）**约束

详细文件分组与 license 边界见 `NOTICE.md`。

**实务影响**：律师使用本 skill 给客户做合同审查、生成审查报告，是合理使用 → 输出报告
不受 CC BY-NC 4.0 约束。但如把本 skill 作为商业产品打包出售，须先重写 contract-copilot
继承内容或获上游许可。

frontmatter 字段中标注的 `license: CC BY-NC 4.0` 只用作 SKILL.md 自身一行 metadata，
**不**意味着整个 skill 全部受 CC BY-NC 4.0 约束。整个 skill 的 license 以 NOTICE.md 为准。
