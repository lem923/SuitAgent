# NOTICE — License & Attribution

This skill is part of the SuitAgent project. The repository root is licensed
under **GNU AGPL v3** (see project root `LICENSE`). Within this skill folder
（`.claude/skills/cn-contract-review/`），files have **dual licensing**
depending on origin:

## 文件分组与对应 License

### 继承自 contract-copilot v1.5.1（License: CC BY-NC 4.0）

以下文件继承自上游 contract-copilot v1.5.1 的内容，**保留 CC BY-NC 4.0 许可证**
（详见 `LICENSE.txt`）。这些文件不能被合并为 AGPL v3 — 项目维护者使用与再分发
时必须遵守 CC BY-NC 4.0 的约束（**禁止商业使用 / 必须保留原作者归属 / 衍生作品
保留 NC 限制**）。

- `references/review-framework.md`（已转换 P0/P1/P2 → REDLINE/ORANGE/YELLOW + v1 marker）
- `references/revision-strategy.md`（已加 v1 marker）
- `references/contract-types/01-universal/*.md`（**例外**：本目录下 `universal-checklist.md` 与 `negotiation-patterns-universal.md` 来自 SuitAgent contributors 原创工作，受 AGPL v3 约束）
- `references/contract-types/02-sale/*.md`（4 个文件全部继承）
- `references/contract-types/03-lease/*.md`（2 个文件全部继承）
- `references/contract-types/04-service/*.md`（8 个文件全部继承）
- `references/contract-types/05-ip/copyright.md / patent.md / software-license.md / technology-development.md / trademark-assignment.md / trademark-license.md`（6 个文件继承）
- `references/contract-types/06-guarantee/*.md`（3 个文件全部继承）
- `references/contract-types/07-lending-gift/*.md`（2 个文件全部继承）
- `references/contract-types/08-internet/*.md`（3 个文件全部继承）
- `references/contract-types/09-marriage-family/*.md`（3 个文件全部继承）
- `references/contract-types/10-employment/business-outsourcing.md / confidentiality.md / employment-contract.md / internship.md / labor-dispatch.md / non-compete.md / part-time-employment.md / personal-service.md / reemployment.md / training-service.md`（10 个文件继承）
- `references/contract-types/11-real-estate/*.md`（7 个文件全部继承）
- `references/contract-types/12-construction/*.md`（6 个文件全部继承）
- `references/contract-types/13-corporate-investment/*.md`（10 个文件全部继承）

合计 **60 个文件** under CC BY-NC 4.0。

### SuitAgent contributors 原创工作（License: AGPL v3）

以下文件是项目内 SuitAgent contributors 的原创工作，受**项目根 LICENSE（AGPL v3）**
约束：

- `SKILL.md`（统一 skill 入口；4-stage workflow + 14 类路由 + REDLINE/ORANGE/YELLOW + fallback positions 三档）
- `memory.md`（按 14 类合同分节的统一审查经验库）
- `references/orientation-and-dispatch.md`（合同画像 9 字段 + 14 类路由判定）
- `references/deliverable-format.md`（7-section 输出标准 + REDLINE/ORANGE/YELLOW + fallback 三档）
- `references/playbook.md`（组织/审查人标准立场骨架）
- `references/qc-checklist.md`（反幻觉/完整性/立场一致/结论唯一）
- `references/cross-border-review.md`（跨境降级规则）
- `references/personal-preferences.md`（项目内置默认审查偏好）
- `references/negotiation-patterns.md`（通用谈判模式 10 个高频问题）
- `references/presign-checklist.md`（通用签署前必查清单）
- `references/contract-types/01-universal/universal-checklist.md`
- `references/contract-types/01-universal/negotiation-patterns-universal.md`
- `references/contract-types/05-ip/licensing-dimensions-a-to-j.md`
- `references/contract-types/05-ip/licensing-clause-pack.md`
- `references/contract-types/05-ip/licensing-memory-patterns.md`
- `references/contract-types/10-employment/labor-employment-main-checklist.md`
- `references/contract-types/10-employment/labor-service-checklist.md`
- `references/contract-types/10-employment/labor-noncompete-ip-checklist.md`
- `references/contract-types/10-employment/labor-termination-checklist.md`
- `references/contract-types/10-employment/labor-evidence-compliance-checklist.md`
- `references/contract-types/10-employment/labor-clause-pack.md`
- `references/contract-types/14-gov-procurement/dev-dimensions-a-to-i.md`
- `references/contract-types/14-gov-procurement/dev-licensing-j-dimension.md`
- `references/contract-types/14-gov-procurement/gov-procurement-clause-pack.md`
- `references/contract-types/14-gov-procurement/gov-procurement-presign-checklist.md`
- `references/contract-types/14-gov-procurement/gov-procurement-memory-patterns.md`

合计 **26 个文件 + 1 NOTICE.md** under AGPL v3。

## 总计

92 个 `.md` 文件 = 60（CC BY-NC 4.0）+ 26（AGPL v3）+ 6（其他类目继承自 contract-copilot 已转换文件不另列）。
（注：分类口径以 SuitAgent contributors 原创工作 vs contract-copilot 直接继承为准；少量边界情况按"含原创修改 ≥ 50%" 划归 AGPL v3。）

## 上游归属说明

继承自 contract-copilot v1.5.1 的内容来源：
- 项目仓库：contract-copilot（独立第三方/社区项目，非 Anthropic 官方）
- 上游 License：CC BY-NC 4.0
- 已做的修改：① 风险等级符号 P0/P1/P2 → REDLINE/ORANGE/YELLOW；② 各文件顶部加 v1 状态 marker；③ 嵌入 SuitAgent 4-stage workflow 与 14 类路由

## 默认审查偏好继承

`references/personal-preferences.md` 中的默认审查偏好（背靠背付款 / 通知送达 /
数据安全 / 跨境法域选择等具体处理策略）继承自原作者实践经验，作为项目开箱即用
的默认设置。接手者可在该文件覆盖默认偏好。

## License 分类的实务影响

- **接手者使用 SuitAgent 项目用于商业合同审查（含律所收费业务）**：项目根
  AGPL v3 不限制商业使用。但 cn-contract-review 内继承自 contract-copilot 的 60
  个文件受 **CC BY-NC 4.0** 约束，禁止用于商业用途（包括但不限于：作为商业 SaaS
  产品的核心引擎、向客户收费提供合同审查服务时直接复用这些文件）。**实务上**，
  律师使用 skill 内嵌的方法论给客户做合同审查后，**输出的审查报告（衍生作品的
  最终交付物）不受 CC BY-NC 4.0 约束**——因为审查报告是律师服务成果，不是合同
  审查 skill 的复制或再分发。但如果你想把 cn-contract-review skill 作为商业产品
  打包出售，需要先重写或获得 contract-copilot 上游许可。

- **接手者 fork 本项目修改后再分发**：项目根 AGPL v3 要求 fork 也以 AGPL v3 公开
  源代码；继承自 contract-copilot 的 60 个文件还需保留 CC BY-NC 4.0。

## 修改本 NOTICE.md

如未来文件归属调整（如把 contract-copilot 继承内容完全重写为原创）→ 更新本 NOTICE.md
对应文件清单 + 通知 contract-copilot 上游（如可联系到）。

修改本 NOTICE.md 不需要 contract-copilot 上游同意，但**修改"继承自 contract-copilot
v1.5.1"清单中的文件内容必须保留 CC BY-NC 4.0**。
