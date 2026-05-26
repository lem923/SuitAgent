# 劳动 / 劳务类合同条款审查

> **本目录用途**：劳动 / 劳务 / 劳务派遣 / 服务期 / 竞业限制等合同的**签约前条款审查**——
> 提供条款级风险点（REDLINE / ORANGE / YELLOW）+ 推荐措辞 + 整改建议。
>
> 本目录原属于 cn-contract-review/references/contract-types/10-employment/，
> 自 v1.13.0 起劳动雇佣类合同审查从 cn-contract-review 剥离，统一归入 cn-labor-employment-strategy。
>
> **与 playbook 子目录的协同**：
> - `contract-clauses/`（本目录）：**签约前**条款级审查（"这一条该不该签 / 这一条怎么改"）
> - `*-playbook.md`（上级目录）：**签约后**用工策略决策（"已经签了，怎么操作"）
> - `scenarios/`：低频场景速查
> - 同一案件可能两侧都用：先用 clauses 看条款风险，再用 playbook 设计履行策略

---

## 文件清单

### 劳动关系合同（须 §2 主体资格 + 三要素齐备）

| 文件 | 适用合同 | 与 playbook 的协同 |
|---|---|---|
| `employment-contract.md` | 标准劳动合同 | + `contract-renewal-playbook` + 整个 P0 |
| `confidentiality.md` | 保密协议（独立 / 劳动合同附件）| + `non-compete-and-confidentiality-playbook` |
| `non-compete.md` | 竞业限制协议（独立 / 劳动合同附件）| + `non-compete-and-confidentiality-playbook` |
| `training-service.md` | 培训服务期协议 | + `scenarios/service-period.md` |
| `dispatch.md` | 劳务派遣（含派遣协议 + 用工协议）| + `cn-labor-relationship-determination/labor-dispatch-playbook` |
| `part-time-employment.md` | 非全日制劳动合同 | + `cn-labor-relationship-determination/special-subjects.md` |
| `internship.md` | 实习协议（在校学生 / 毕业实习）| + `cn-labor-relationship-determination/special-subjects.md` |

### 民事关系合同（劳动关系外的服务安排）

| 文件 | 适用合同 | 与 playbook 的协同 |
|---|---|---|
| `personal-service.md` | 个人劳务协议（民事）| + `cn-labor-relationship-determination/employment-vs-civil-playbook`（防止被穿透认定）|
| `reemployment.md` | 退休返聘协议 | + `cn-labor-relationship-determination/special-subjects.md` |
| `business-outsourcing.md` | 业务外包协议 | + `cn-labor-relationship-determination/employment-vs-civil-playbook` |

---

## 调用时机

启动 skill 时按时点判断进入哪个子目录：

```
合同还没签 / 待签 / 改稿 / 模板设计   → contract-clauses/（本目录）
       ↓ 签约后
履行中 / 出口 / 协商解除               → playbook（上级目录）
关系性质有疑义                         → cn-labor-relationship-determination
工伤社保特殊领域                       → cn-labor-insurance-and-injury
```

---

## 使用要点

1. **每个文件采用统一三层结构**：宏观（合同整体）→ 中观（条款类型）→ 微观（具体条款）
2. **风险等级**：REDLINE（不改不签）/ ORANGE（强烈建议争取）/ YELLOW（优化项）
3. **fallback positions**：每条 ORANGE/REDLINE 应给出目标 / 可签 / 底线三档（详见 playbook）
4. **双视角**：用人单位版 vs 劳动者版 → 对应 stance 参数切换
5. **联动其他类目**：
   - 涉及股权激励 → 联动 cn-contract-review/13-corporate-investment/
   - 涉及知识产权许可 → 联动 cn-contract-review/05-ip/
   - 涉及政企采购 → 联动 cn-contract-review/14-gov-procurement/

---

## 历史脉络

- 早期：cn-contract-review-labor-employment（specialized skill，独立存在）
- v1.0：合并到 cn-contract-review/references/contract-types/10-employment/（与 contract-copilot 继承内容混合）
- v1.13.0（2026-05）：劳动域整体迁移至 cn-labor-employment-strategy/references/contract-clauses/，cn-contract-review 14 大类缩为 13 大类
