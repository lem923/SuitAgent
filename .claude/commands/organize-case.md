---
name: organize-case
description: 对已有案件文件夹做合规检查 + 重命名 + 内部结构整理。触发 new-case skill 的重整理模式（Mode 2）。
---

# /organize-case · 案件文件夹重整理

## 用法

```
/organize-case [folder-path]
```

- `folder-path` 可选；如未提供，对当前对话所在案件文件夹操作
- 也可直接自然语言触发："整理这个案件文件夹"、"重命名案件"、"案件归一化"、"这个文件夹乱"

## 功能

调起 [new-case skill 的 Mode 2 重整理模式](../skills/new-case/SKILL.md#mode-2重整理模式v1101)，对已有案件文件夹做：

1. **现状扫描**：解析当前文件夹名 + 内部结构
2. **命名合规检查**（5 项）：
   - YYMMNN 6 位前缀
   - 字段间单空格分隔
   - 含"与"分隔符（行政诉讼用"诉"）
   - 原告 / 被告简称清晰
   - 案由为标准案由名
3. **缺字段询问**（人 in the loop）：缺什么问什么，不臆造
4. **NN 自然编号建议**：扫描项目根已有 YYMMNN，建议下一个未用编号
5. **生成 mv 命令**：显式输出，等待用户明确确认；不自动跑
6. **用户确认后执行 mv + 内部结构整理**：
   - 缺 matter triplet 生成
   - 缺 11 numbered slots 创建
   - 旧 12 层 emoji 目录迁移
   - 散落根目录的客户材料移到对应 slot
7. **matter.yaml 更新**：matter_id 字段更新为新合规名

## 命名规范

严格遵循：`{YYMMNN} {原告简称} 与 {被告简称} {案由}`

详细规范见 [OutputStandards.md · 案件文件夹命名规范](../rules/OutputStandards.md#案件文件夹命名规范)。

## 输出

- 命名合规报告（5 项检查通过/不通过）
- mv 命令（用户确认后执行）
- 内部结构整理结果（11 slot 完整度 / matter triplet 状态）
- 待用户配合事项清单

## 禁忌（重申）

- ❌ 不自动执行 mv（用户必须明确确认）
- ❌ 不删除任何原始文件
- ❌ 不在含 `_FINAL` / `_SIGNED` / `_盖章` 后缀的文件上 mv
- ❌ 不跨 matter 移动文件
- ❌ 不臆造缺失字段

## 与外置 cn-litigation-case-folder-organizer skill 的关系

- 本命令 + new-case Mode 2 = **项目内置**，处理 SuitAgent 内的案件文件夹
- 外置 organizer skill 是用户全局工具，适用于 SuitAgent 外的杂乱文件夹
- 两者结构方案一致（11 numbered + matter triplet + 工时记录）
- 外置 organizer 若未升级 rename 步，可通过 `tmp/cn-litigation-case-folder-organizer_patch_v2.md` 同步

## 使用示例

```
> 用户：整理 "260507 华雄公司 土地使用权转让纠纷" 这个案件

[organize-case 命令触发 new-case Mode 2]

→ 现状扫描：
    文件夹名：260507 华雄公司 土地使用权转让纠纷
    路径：/Volumes/EXTERNAL/cowork/SuitAgent/260507 华雄公司 土地使用权转让纠纷

→ 命名合规检查：
    [a] ✅ YYMMNN 6 位前缀（260507）
    [b] ❌ 字段间分隔：缺 "与 被告" 段
    [c] ❌ 含"与"分隔符：缺
    [d] ❌ 被告简称：缺
    [e] ✅ 案由："土地使用权转让纠纷" 为标准案由

→ 询问用户：
    本案被告简称是什么？

> 用户：北京 XX 公司

→ 建议新名：
    260507 华雄公司 与 北京 XX 公司 土地使用权转让纠纷

→ 生成 mv 命令：
    mv "/Volumes/EXTERNAL/cowork/SuitAgent/260507 华雄公司 土地使用权转让纠纷" \
       "/Volumes/EXTERNAL/cowork/SuitAgent/260507 华雄公司 与 北京 XX 公司 土地使用权转让纠纷"

→ 请用户确认执行 [Y/N]

> 用户：Y

→ 执行 mv ✅
→ 内部结构整理：[列详细操作]
→ matter.yaml matter_id 字段已更新
```

## 变更历史

| 版本 | 日期 | 说明 |
|------|------|------|
| v1.0 | 2026-05-11 | 初始版本（v1.10.1+ 新增） |
