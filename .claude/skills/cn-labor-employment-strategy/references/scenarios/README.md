# Scenarios 子目录

> 本目录存放 cn-labor-employment-strategy 覆盖的**低频场景**速查。
> **高频场景**（违纪、不胜任、调岗、被迫离职、离职协议、竞业限制、续签）已独立为 playbook，见 `references/` 根目录的 `*-playbook.md` 文件。
>
> 本目录场景按主题文件组织，每个文件结构相对简化：
> 法律依据 → 关键认定标准 → 典型成本路径 → 双视角策略 → 代表性判例 → 文书模板调用。

## 场景文件索引

| 文件 | 主题 | 涉及汇编节 | 案例数 |
|---|---|---|---|
| `probation-period.md` | 试用期 | 3.4 | 2 |
| `double-wage.md` | 未签书面合同二倍工资 | 3.3 | 22 |
| `service-period.md` | 培训服务期 / 违约金 | 3.5 + 7.1 | 9+ |
| `wages-and-bonuses.md` | 工资、奖金、同工同酬 | 4.1+4.2 | 37 |
| `overtime-and-leave.md` | 加班费、特殊工时、年休假 | 4.3+4.4 | 23 |
| `performance-of-duty-damage.md` | 履职过错损害赔偿 | 3.9 | 12 |
| `special-protection-periods.md` | 三期 / 医疗期 / 工伤 / 老员工 / 工会代表 | 5.5 | 19 |
| `contract-formation.md` | 合同期限、效力（含欺诈）| 3.1+3.2 | 15 |
| `subsequent-obligations.md` | 附随义务（离职证明、社保转移、保密、不诋毁）| 5.9 | 11 |
| `bankruptcy-claims.md` | 职工破产债权 | 5.10 | 4 |
| `public-institution.md` | 事业单位人事争议 | 7.1 | 7 |
| `covid-related.md` | 涉疫情案件 | 4.5 | 17 |
| `procedure-and-evidence.md` | 裁审程序与虚假诉讼（附录）| 第1章 | 18 |

## 调用规范

涉及多场景叠加的，按顺序加载相关文件。例如：
- 调岗 + 三期 → 同时加载 `contract-modification-playbook` + `special-protection-periods.md`
- PIP + 加班费追索 → 加载 `dismissal-incompetence-playbook` + `overtime-and-leave.md`
