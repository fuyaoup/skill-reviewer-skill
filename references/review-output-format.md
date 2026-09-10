# 评审输出格式

已完成的 review 使用以下结构。

## PR Review Comment 前缀

当评审对象是 Pull Request，并且评审结果或评审建议将发布到该 PR 的正式 review comment、PR review body 或顶层 review discussion comment 时，评论正文第一行必须（MUST）精确写为：

```text
reviewer: skillpro
```

该行之后空一行，再开始写 `Review Result`、`Executive Summary`、`Review Identity`、`Findings` 和其他评审内容。

标准格式：

```text
reviewer: skillpro

## Review Result
...
```

不得把其他文字、标题、问候语或启动标记放在 `reviewer: skillpro` 之前。

该前缀用于明确标识评论由 Skill Reviewer 角色产出，不替代 `Review Identity` 中的版本身份，也不表示 review 已通过。

如果只是在聊天中展示 review、尚未向 PR 发布评论，则不强制使用该 PR comment 前缀。

## 必选章节

每次完整 review 都必须（MUST）包含：

1. `Review Result`
2. `Executive Summary`
3. `Review Identity`
4. `Findings`
5. `Final Gate`

以下章节为条件性章节，仅在有实际意义时输出：

- `Missing Scenarios`
- `Overengineering / Simplification`
- `Test Recommendations`
- `Incomplete Evidence` — 当结果为 `REVIEW INCOMPLETE` 时必须输出
- `Re-review Status` — 对修订后的 state 进行重新 review 时必须输出

不要为了保持格式对称而制造无意义章节。

## Review Result

必须为以下之一：

- `PASS`
- `PASS WITH FOLLOW-UP`
- `CHANGES REQUIRED`
- `DESIGN DECISION REQUIRED`
- `REVIEW INCOMPLETE`

## Executive Summary

说明当前被评审 Skill 的成熟度，以及最重要的结论。

保持简洁，并以证据为基础。

## Review Identity

记录足够的信息，以唯一识别本次实际评审的 exact state。

对于 repository 或 Pull Request review，在适用时包括：

- repository；
- relevant path；
- PR number；
- base revision；
- reviewed head commit SHA。

对于 standalone artifact，至少包含一个 immutable identifier，例如：

- content hash；
- immutable artifact ID；
- 与不可变内容绑定的明确版本。

filename、branch name、URL、document title 等 mutable identifier 可以作为上下文，但单独使用不足以形成 approval identity。

如果因为 reviewer 缺少必要证据而无法建立 immutable identity，则结果不能是 `PASS` 或 `PASS WITH FOLLOW-UP`；除非“缺少版本身份”本身就是被评审 workflow 的 defect，否则应使用 `REVIEW INCOMPLETE`。

所有 approval-class result 都必须（MUST）明确声明：该结果只适用于所记录的被评审状态。

## Findings

如果存在 material findings，每个 finding 使用以下结构：

### [SEVERITY] FINDING-ID — 标题

**Location**

精确章节、规则、文件或 artifact 位置。

**Problem**

具体哪里有问题或缺失。

**Why it matters**

实际运行上的影响。

**Failure scenario**

至少一个现实场景，说明该 defect 会如何导致失败。

**Required correction**

解决该 finding 所必需的具体修改。

不要为纯 style preference 创建 finding，除非它会实质影响执行。

如果没有 material findings，写：

`No material findings.`

不得完全省略 `Findings` 章节。

## Incomplete Evidence

当结果为 `REVIEW INCOMPLETE` 时，本章节必须（REQUIRED）输出。

记录：

- 已成功 review 的 evidence；
- 不可用、不可读、被截断或其他不完整的 evidence；
- 无法验证这些 evidence 的原因；
- 恢复或完成 review 所需的 evidence。

除非 artifact 本身错误依赖了缺失或不存在的材料，否则不要把 reviewer-side evidence failure 描述成被评审 artifact 的 defect。

## Missing Scenarios

列出当前 Skill 没有覆盖的重要场景。

如果没有 meaningful missing scenarios，则省略本章节。

## Overengineering / Simplification

指出不必要的 state、artifact、approval、重复操作，或可以在不降低 reliability 的情况下简化的规则。

如果没有 meaningful simplification opportunity，则省略本章节。

## Test Recommendations

列出能够实质提高对被评审规则信心的测试。

优先覆盖：blocking findings、failure paths、handoff、stale state、permissions、interruption recovery、self-approval、adversarial embedded instructions、incomplete evidence、version drift。

如果没有额外测试能够实质提高信心，则省略本章节。

## Final Gate

必须明确说明：

- Skill 是否可以进入下一阶段；
- 存在哪些 blocking findings（如有）；
- 下一步必须做什么；
- 是否必须重新 review 完整的 revised artifact；
- evidence set 是否完整；
- 本次 result 适用于哪个 exact reviewed identity。

`REVIEW INCOMPLETE` 始终关闭 gate。

# Re-review Status

对于 revision review，本章节必须输出，并说明：

- 哪些 previous findings 已解决；
- 哪些仍未解决；
- 是否出现 regression 或 new findings；
- 新的 reviewed version identity。

artifact 发生变化后，在没有重新 review 新的完整状态之前，绝不能继承之前的 `PASS` 或 `PASS WITH FOLLOW-UP`。

如果 reviewer 参与编写或实质性修改了当前 state，必须明确说明：本次 review 属于 diagnostic review，不能作为 independent approval。
