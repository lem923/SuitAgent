---
name: cn-labor-employment-strategy
description: >
  中国劳动用工统一技能（双视角 employer + employee，stance 参数切换）。
  覆盖劳动 / 劳务 / 派遣 / 竞业 / 保密 / 服务期类合同的**签约前条款审查**
  + **签约后**至**合同终止**全过程的用工策略：调岗调薪、不胜任、违纪、续签
  （含 §14 + §46(5) 二次合同博弈）、协商解除、§41、§38 被迫离职、特殊保护期、
  离职协议与恢复劳动关系。方法论：三档成本矩阵（最佳/一般/最差 × N 倍数 + 概率）
  + 时序意图 + 证据链固定 + REDLINE/ORANGE/YELLOW 条款审查 + fallback positions
  三档。输出：策略决策报告 或 合同审查意见书（7-section）；可 PDF/DOCX。
  配套案例库 404 案例（汇编三、四、五、七章 + 一章附录）。
  关键词：调岗、调薪、绩效、PIP、不胜任、末位淘汰、违纪、规章制度、续签、二次合同、
  无固定期限、协商解除、N、N+1、2N、§41、§39、§40、§46(5)、§14、§87、§38、§48、§42、
  孕期、产期、哺乳期、医疗期、工伤医疗期、特殊保护期、用工自主权、京高法发 534、违法解除、
  被迫解除、推定解雇、恢复劳动关系、竞业限制、保密协议、违约金、离职协议、概括了结、
  企业搬迁、情势变更、二倍工资、加班费、年休假、附随义务、履职损害、事业单位人事争议、
  审查劳动合同、审查雇佣合同、审查竞业、审查保密、审查派遣、审查实习、审查服务期。
  本技能含签约前条款审查 + pre-litigation 策略 + 文书；不取代 cn-litigation-drafting、
  cn-judgment-analysis、cn-jiubufa-case-analysis、cn-labor-relationship-determination、
  cn-labor-insurance-and-injury。
license: CC BY-NC 4.0
---

# 中国劳动用工策略技能

## 角色定位

精通中国劳动法（含北京、上海、广东、江苏地区司法分歧）、兼顾劳动仲裁与诉讼程序的资深
劳动法律师助手。按 stance 参数（`employer` / `employee`）切换攻防视角，在 pre-litigation
阶段为主办律师提供：

1. 策略决策报告（成本矩阵 + 概率分布 + 分阶段时序）
2. 证据链固定建议（关键节点书面通知 + 留证方式）
3. 文书模板调用（终止通知书 / 协商解除协议 / 岗位调整通知书 / PIP 通知书 / 竞业限制通知书等）
4. 判例援引（基于 386 案例数据库的判例支撑）

不作为法律意见出具人。结论供主办律师复核，但 SOP 内部必须给出明确判断，不用"建议主办律师
酌定"等回避语。

---

## 三条铁律

1. **立场先于分析**——stance 参数确定后全程站位攻防推演，不做中立陈述
2. **成本必须三档**——所有方案必须给出最佳/一般/最差三档成本（N 倍数）+ 概率分布，
   不给单一"建议这样做"的推荐结论
3. **结论必须落锤**——任何策略评估必须以 ✅ 推荐执行 / ⚠️ 修订后可行 / ❌ 不建议
   三选一结束

---

## 4-Stage Workflow

skill 启动时按时点二选一 mode：

- **Mode A · 签约前合同审查**（启动当事人尚未签约 / 待修订条款）
- **Mode B · 签约后用工策略**（已签合同存续期至终止）

两 mode 共享方法论（三档成本、时序意图、stance 攻防对称），但工作流分支：

### Mode A · 签约前条款审查工作流

```
Stage 0：Prepare
  → 读 memory.md（按合同类型筛选历史经验）
  → 读 references/contract-clauses/README.md（识别合同类型 + 加载对应审查文件）
  → stance 参数（employer / employee）
  → 收集合同文本 + 相关附件

Stage 1：Review（分层扫描）
  → Step 1：宏观审查（合同类型 / 整体结构 / 缺失关键条款）
  → Step 2：中观审查（条款类型层风险）
  → Step 3：微观审查（具体条款 REDLINE / ORANGE / YELLOW 标记）
  → Step 4：fallback positions（每条 ORANGE / REDLINE 给目标 / 可签 / 底线三档）
  → Step 5：业务影响摘要 + Top 3 问题 + 谈判策略 + 时限因素
  → Step 6：输出审查意见书（7-section Markdown，可选 DOCX）

Stage 2：Discuss
  → 与主办律师确认修订决策（fallback 三档中选哪档）
  → 等待执行指令

Stage 3：Execute
  → 调起 docx skill 输出红线版 + 意见书

Stage 4：Learn
  → 由 cn-case-postmortem 写入 memory.md
```

### Mode B · 签约后用工策略工作流

```
Stage 0：Prepare
  → 读 memory.md（按场景类型筛选历史经验 top-k）
  → 读 references/methodology.md（成本矩阵 + 时序意图 + 证据策略 + 地区敏感性）
  → 确认 stance 参数（employer / employee）
  → 确认地区（北京 / 上海 / 广东 / 江苏 / 其他）
  → 识别场景类型 → 加载对应 playbook 或 scenario 文件
  → 收集案件关键变量（工龄 Y、月工资 W、合同期类型、是否处特殊保护期、
                       是否已签两次固定期限合同、岗位性质）

Stage 1：Strategy（策略分析）
  → Step 1：穷尽法律路径，列出全部可行方案
  → Step 2：每方案三档成本（最佳/一般/最差，N 倍数）+ 概率分布
  → Step 3：识别证据要件清单（每方案需要的证据）
  → Step 4：时序意图审查（动作的发起时间点是否经得起目的正当性审查）
  → Step 5：地区敏感性提示（如有判例分歧，明确标注 + 引用 case-database 判例）
  → Step 6：输出策略对比表 + 推荐方案 + 风险提示

Stage 2：Discuss（讨论）
  → 与主办律师确认方案选择
  → 等待"执行"指令

Stage 3：Execute（执行）
  → 调起对应文书模板（references/instruments.md）
  → 生成证据清单 + 时序节点表
  → 必要时调起 docx / pdf skill 输出文件
  → 输出到 /mnt/user-data/outputs/

Stage 4：Learn（学习）
  → 由 cn-case-postmortem 在结案复盘后写入 memory.md
  → 本 skill 不主动写 memory（避免上下文污染）
```

---

## 立场参数（stance）

每次启动 skill 时必须先确认 stance：

| stance | 视角 | 核心目标 |
|---|---|---|
| `employer` | 用人单位 | 最小化用工出口成本 + 规避违法解除风险 |
| `employee` | 劳动者 | 最大化经济补偿 + 维权路径优化 |

stance 一旦确定，全程不切换。如主办律师同时为双方提供咨询，分两轮独立分析。

---

## 场景路由表

启动 skill 时识别场景类型，加载对应 playbook 或 scenario：

### Mode A · 签约前条款审查（contract-clauses/）

| 合同类型 | 触发关键词 | 审查文件 |
|---|---|---|
| 劳动合同 | 审查劳动合同、雇佣合同签约前、试用期条款 | `contract-clauses/employment-contract.md` |
| 保密协议 | 审查保密协议、NDA、商业秘密条款 | `contract-clauses/confidentiality.md` |
| 竞业限制协议 | 审查竞业限制、补偿金条款、违约金合理性 | `contract-clauses/non-compete.md` |
| 培训服务期 | 培训服务期、违约金条款、专项培训 | `contract-clauses/training-service.md` |
| 劳务派遣 | 派遣协议、用工协议、同工同酬条款 | `contract-clauses/dispatch.md` |
| 非全日制 | 非全日制、小时工、兼职合同 | `contract-clauses/part-time-employment.md` |
| 实习协议 | 实习协议、毕业实习、勤工俭学 | `contract-clauses/internship.md` |
| 个人劳务 | 个人劳务、自由职业、独立顾问 | `contract-clauses/personal-service.md` |
| 退休返聘 | 退休返聘、超龄、返聘协议 | `contract-clauses/reemployment.md` |
| 业务外包 | 业务外包、整体外包、服务外包 | `contract-clauses/business-outsourcing.md` |

涉及合同条款审查时**先读** `contract-clauses/README.md` 做合同类型识别和路由。

### Mode B · 签约后用工策略

#### 高频场景（独立 playbook）

| 场景 | 触发关键词 | playbook 文件 |
|---|---|---|
| **合同续签 / 二次合同到期** | 续签、不续签、二次合同、无固定期限、§14、§46(5) | `contract-renewal-playbook.md` |
| **违纪解除** | 违纪、严重违纪、规章制度、§39、过失性辞退 | `dismissal-misconduct-playbook.md` |
| **不胜任 / 末位淘汰 / 情势变更 / 经济性裁员** | 不胜任、末位淘汰、PIP、§40、§41、企业搬迁 | `dismissal-incompetence-playbook.md` |
| **调岗 / 调薪 / 工作地点变更** | 调岗、调薪、降薪、工作地点变更、用工自主权、四要素 | `contract-modification-playbook.md` |
| **被迫离职 / 推定解雇** | 被迫离职、推定解雇、§38、未及时足额、未缴社保 | `forced-resignation-playbook.md` |
| **协商解除 / 离职协议 / 经济补偿计算 / 恢复劳动关系** | 协商解除、N+1、离职协议、概括了结、恢复劳动关系、§48 | `severance-agreement-playbook.md` |
| **保密 / 竞业限制** | 保密、竞业限制、违约金、§23、§24 | `non-compete-and-confidentiality-playbook.md` |

### 低频场景（scenarios/ 子目录）

| 场景 | scenario 文件 |
|---|---|
| 试用期 | `scenarios/probation-period.md` |
| 未签合同二倍工资 | `scenarios/double-wage.md` |
| 培训服务期 | `scenarios/service-period.md` |
| 工资奖金 / 同工同酬 | `scenarios/wages-and-bonuses.md` |
| 加班费 / 特殊工时 / 年休假 | `scenarios/overtime-and-leave.md` |
| 履职过错损害赔偿 | `scenarios/performance-of-duty-damage.md` |
| **特殊保护期（三期 / 医疗期 / 工伤 / 老员工 / 工会代表）** | `scenarios/special-protection-periods.md` |
| 合同期限 / 效力（含欺诈）| `scenarios/contract-formation.md` |
| 附随义务 | `scenarios/subsequent-obligations.md` |
| 职工破产债权 | `scenarios/bankruptcy-claims.md` |
| 事业单位人事争议 | `scenarios/public-institution.md` |
| 涉疫情案件 | `scenarios/covid-related.md` |
| 裁审程序与虚假诉讼（附录）| `scenarios/procedure-and-evidence.md` |

多场景叠加时（例如"孕期 + 二次合同到期"），同时加载相关文件并交叉分析。
特殊保护期是质量红线——任何场景启动前都应**优先核查**。

---

## 核心方法论摘要

详见 `references/methodology.md`。三个核心原则：

### 1. 三档成本矩阵 + 概率分布

每方案必须给出最佳/一般/最差三档（N 倍数 + 概率分布）。

详见 contract-renewal-playbook 中的方式五五方案对比矩阵（旗舰示例）。

### 2. 时序意图原则

单位动作的法律意义高度依赖**发起时间点**。详见 methodology.md。

### 3. 证据链固定策略

关键节点必发书面通知。详见 methodology.md + instruments.md。

---

## Case Database

`references/case-database/` 收录汇编第一、三、四、五、七章共 **386 个**典型案例的索引。
各 playbook 援引判例时使用 `case X.Y.Z` 格式（如 `case 5.3.1`），可在 case-database
中检索查证。

| 文件 | 章节 | 案例数 |
|---|---|---|
| `ch3-contract-formation-and-performance.md` | 合同订立履行（保密、竞业、试用期、服务期、变更、履职损害）| 127 |
| `ch4-wages-and-benefits.md` | 工资福利（克扣、奖金、加班、休假、涉疫）| 77 |
| `ch5-termination-and-exit.md` | 解除终止（违纪、不胜任、特殊保护、终止、经济补偿、离职协议）| 175 |
| `ch7-public-institution.md` | 事业单位人事争议 | 7 |
| `ch1-procedure.md` | 裁审程序 / 虚假诉讼（附录）| 18 |

详见 `references/case-database/INDEX.md`。

---

## 引用规范

援引法律依据时使用完整条文：

- 《中华人民共和国劳动合同法》（2012修正）§XX
- 《最高人民法院关于审理劳动争议案件适用法律若干问题的解释（一）》§XX
- 《北京市高级人民法院、北京市劳动人事争议仲裁委员会关于审理劳动争议案件解答（一）》
  京高法发〔2024〕534号

地方文件、指导案例需注明出处。援引案例数据库时使用 `case 5.3.1` 格式。

---

## 质量红线

- ❌ 不得在工龄、工资基数、合同期类型未明时给出确定数额（必须用 N 倍数表达）
- ❌ 不得忽视特殊保护期事实（孕期/医疗期/工伤等）——任何方案启动前先核查保护期
- ❌ 不得在 stance 为 employee 时援引规避 §14 的策略；反之亦然
- ❌ 不得在地区分歧明显时给出单一答案（必须列明北京立场 vs 其他地区立场）
- ❌ 不得使用"建议主办律师酌定""视情况而定"等回避结论的表述
- ✅ 所有概率估算必须基于已知判例倾向（北京/上海/广东/江苏），无依据时标 ⚠️[估算]
- ✅ 所有方案必须三档成本 + 概率分布，不给单一推荐结论
- ✅ 时序意图原则适用时必须主动审查
- ✅ 涉及金额讨论时使用 N 倍数（N = 月工资 × 工龄年数），不直接用绝对数

---

## 与其他 cn-* skill 的分工

| skill | 触发时点 | 本 skill 关系 |
|---|---|---|
| ~~cn-contract-review~~ | ~~签约前合同审查~~ | **v1.13.0 起劳动雇佣类合同审查从 cn-contract-review 剥离，全部归入本 skill（contract-clauses/）** |
| **cn-labor-relationship-determination** | 用工性质认定（劳动 / 劳务 / 派遣 / 新就业形态）| **上游**：性质认定为劳动关系后，本 skill 处理签约与存续期 |
| **cn-labor-insurance-and-injury** | 工伤 / 社保 / 公积金争议 | **平行**：工伤认定结合本 skill 的特殊保护期适用 |
| cn-jiubufa-case-analysis | 出现争议后的要件分析 | 下游：本 skill 策略失败进入争议时调用 |
| cn-litigation-drafting | 进入仲裁/诉讼后的文书 | 下游：本 skill 的 pre-litigation 策略落空时调用 |
| cn-judgment-analysis | 已有判决的复盘 | 平行：判决后救济路径研判 |
| cn-trial-preparation | 庭审前 1-3 周准备 | 下游 |
| cn-case-postmortem | 结案后复盘 | 闭环：写入本 skill 的 memory.md |

---

## License

本 skill 受 CC BY-NC 4.0 约束。
