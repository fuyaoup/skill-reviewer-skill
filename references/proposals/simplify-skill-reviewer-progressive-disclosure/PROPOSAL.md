# Proposal — Skill Reviewer 精简与渐进披露重构

## 1. 背景

当前 `skill-reviewer-skill` 的核心目标很小而明确：

> 对 AI / Agent / Workflow / Prompt Skill 进行独立、可重复、可审计的专业评审。

但当前 `SKILL.md` 已膨胀为 500+ 行级别，并同时承担：

- 首次触发规则；
- 同一会话 continuation；
- PR comment identity；
- PR comment queue / marker protocol；
- review independence；
- review scope；
- immutable identity；
- evidence completeness；
- 四遍 review method；
- review criteria 摘要；
- severity / result mapping；
- version recheck；
- re-review；
- modification boundary；
- output contract。

与此同时，以下内容又已经存在独立 reference：

```text
references/review-criteria.md
references/severity-model.md
references/review-output-format.md
```

因此当前结构存在明显的重复 ownership 和 progressive-disclosure 不足。

此外，PR #4 已尝试加强多轮 PR review 的 durable marker，但真实执行仍出现：

```text
没有 reviewer: skillpro
没有 reviewed-comment-id
没有 reviewed-comment-version
没有 reviewed-at-head
```

说明当前问题不是“缺少更多格式规则”，而是核心 execution contract 没有把“写入 GitHub”定义成不可跳过的 side effect。

## 2. OpenAI 官方 Skill guidance 对本 Proposal 的设计启示

本 Proposal 使用当前 OpenAI 官方 Skill guidance 作为**设计参考**，不把它变成本 Skill 每次运行都必须重新读取的 runtime baseline。

当前 OpenAI 官方资料把 Skill 描述为 reusable/shareable workflow，并说明典型 `SKILL.md` 主要定义：

- Skill 做什么；
- required inputs；
- step-by-step instructions；
- required output format；
- final checks before completion。

官方同时明确支持 supporting resources，例如 templates、examples、schemas、tool access、instructions、supporting files 和 code，并建议复杂 workflow 优先考虑较小、可组合的 building blocks，而不是一个巨大的 end-to-end Skill。

因此本 Proposal 的结构目标是：

```text
concise SKILL.md core/router
+ single normative owner per concept
+ progressive disclosure
+ PR-specific protocol only when target is PR
```

说明：

- OpenAI 没有规定 `SKILL.md` 最大行数，因此“500+ 行”本身不是官方格式违规；
- 当前仅允许精确 `/review-skill` 首次触发，是本项目的自定义 workflow policy，不是 OpenAI 官方 invocation contract；
- 是否把 OpenAI 官方 guidance 设为 Skill Reviewer 的 runtime review baseline，继续由独立 PR #5 讨论，本 Proposal 不做决定。

## 3. 问题定义

### 3.1 `SKILL.md` 同时承担 router 和 encyclopedia

当前主文件重复描述了已经由 references 持有的 criteria、severity、output 语义。

后果：

- 每次 Skill 激活都加载大量与当前 target 无关的规则；
- 修改一项规则可能要同步多处；
- 容易产生 summary drift；
- PR-specific 逻辑污染 standalone Skill / Proposal review；
- 小功能修复不断向主文件追加规则，复杂度持续增长。

### 3.2 PR durable review 的真实 invariant 没有被直接表达

当前规则重点描述 marker 格式和 queue counters，但真正需要保证的是：

```text
PR review complete
=
需要处理的 comment marker 已持久化
+ current-head overall review 已持久化
```

如果只生成 review 文本但没有写回 PR，不应报告 durable PR review 已完成。

### 3.3 `Comment Queue Gate` 过度建模

当前 `discovered / already-reviewed / processed-this-round / pending / marker-write-failures` 五个计数器对于 observability 有价值，但不是 correctness 的必要条件。

把这些计数器作为核心 gate 会增加：

- 状态计算；
- context；
- re-read 成本；
- transient failure 语义复杂度；
- 实现与 review burden。

真正需要的 correctness state 只有：

```text
哪些 current comment versions 仍 pending
哪些 write 未确认成功
```

## 4. 设计目标

本重构必须：

1. 保留 Skill Reviewer 的核心 review quality、independence、evidence、identity、severity 和 result semantics；
2. 保留 `/review-skill` 首次显式触发和 same-session same-target continuation；
3. 将 PR-specific 行为从核心 `SKILL.md` 下沉到按需 reference；
4. 修复 PR review 未持久化 marker / overall review 的实际缺口；
5. 删除重复 normative content；
6. 每个 normative concept 只有一个 authoritative owner；
7. 不引入新的复杂状态机；
8. 不把 PR comment history 当作通用 review core；
9. 不与 PR #5 的 OpenAI runtime-baseline 设计耦合；
10. 不改变用户明确要求的“未经授权不得 merge”边界。

## 5. Target structure

建议结构：

```text
SKILL.md
references/
├── review-method.md
├── review-criteria.md
├── severity-model.md
├── review-output-format.md
└── pr-review-protocol.md
```

### 5.1 `SKILL.md` — core/router

只负责：

- purpose；
- first activation policy；
- same-session continuation；
- target classification；
- required inputs / capability check；
- top-level review steps；
- reference loading rules；
- modification boundary；
- final completion check。

目标规模：

```text
约 100–180 行
```

这不是 OpenAI 官方 hard limit，只是本 repo 的工程目标。

如果最终超过约 200 行，应说明为什么相关规则不能下沉到单一 authoritative reference。

### 5.2 `references/review-method.md`

唯一拥有：

- independence；
- review scope；
- evidence completeness；
- immutable review identity；
- four-pass method；
- version recheck；
- re-review semantics；
- reviewed-content isolation。

`SKILL.md` 不再复制这些规则正文，只负责路由到该 owner。

### 5.3 `references/review-criteria.md`

继续唯一拥有详细评审维度。

`SKILL.md` 只写：

```text
apply references/review-criteria.md
```

不得再复制 A–R criteria 摘要。

### 5.4 `references/severity-model.md`

唯一拥有：

- severity definitions；
- result vocabulary；
- severity → result mapping；
- `DESIGN DECISION REQUIRED` / `REVIEW INCOMPLETE` 判定规则。

`SKILL.md` 不再重复完整 mapping。

### 5.5 `references/review-output-format.md`

唯一拥有：

- overall review format；
- finding format；
- conditional sections；
- PR comment prefix 格式。

### 5.6 `references/pr-review-protocol.md`

只在 target 是 Pull Request 时加载。

唯一拥有：

- PR history 读取范围；
- comment event identity；
- edited comment detection；
- marker persistence；
- overall review persistence；
- write failure semantics；
- PR-specific re-review behavior。

Standalone file / Proposal / non-PR Skill review 不加载该文件。

## 6. 简化后的 PR persistence contract

### 6.1 默认行为

当 target 是 Pull Request，并且 reviewer 有可用的 GitHub write capability 时，完整 PR review 默认是 durable review。

执行顺序：

```text
1. 读取 current PR head + relevant review history
2. 找出未处理或已编辑的 review-relevant comment versions
3. 对每个 pending event 做判断
4. 将 marker-bearing reply 写回 PR
5. 完成 current-head full review
6. 将 overall review 写回 PR
7. 确认必要写入成功
8. 才报告 durable PR review complete
```

### 6.2 comment marker

每个真正需要处理的 pending comment/reply，在完成判断后写入：

```text
reviewer: skillpro
reviewed-comment-id: <GitHub comment ID>
reviewed-comment-version: <updated_at 或稳定 body hash>
reviewed-at-head: <reviewed PR head SHA>

<判断与建议>
```

其中：

- `reviewed-comment-id + reviewed-comment-version` 是去重 identity；
- `reviewed-at-head` 记录判断时的代码状态，不作为 comment 去重键；
- edited comment version 会重新进入 pending；
- head 改变但 comment version 未变，不重复处理 comment，本轮 current-head full review 负责验证代码状态。

### 6.3 overall review persistence

完整 current-head review 完成后，必须写入 PR overall review，正文第一行：

```text
reviewer: skillpro
```

Chat 中展示的 review 不能替代该 durable artifact。

### 6.4 persistence success

不强制每次都重新扫描完整 PR history。

写入成功可由以下任一方式确认：

```text
A. write action/tool 的返回结果明确确认已持久化；
或
B. targeted re-fetch/readback 确认。
```

只有当工具返回不能证明 persistence 时，才要求 targeted readback。

这避免把“完整 history 二次扫描”变成每个 comment 的固定成本。

### 6.5 write failure

任何必需 marker reply 或 overall review 无法确认持久化：

```text
→ REVIEW INCOMPLETE
→ 明确说明未持久化的内容
→ 不得声称 durable PR review complete
```

不需要额外构造复杂的 marker-write-failure 状态机。

### 6.6 chat-only diagnostic mode

如果：

- GitHub write capability 不可用；或
- 用户明确要求只在聊天中 review；

可以执行 diagnostic chat-only review，但必须明确标记：

```text
mode: diagnostic-chat-only
```

它不能被当作 durable PR approval/review state。

## 7. 删除 / 简化的现有机制

### 7.1 删除五计数器作为 hard gate

不再强制：

```text
discovered
already-reviewed
processed-this-round
pending
marker-write-failures
```

作为 normative completion gate。

实现可以继续输出 telemetry，但它不是 correctness 的 source of truth。

最低必须知道：

```text
pending current comment versions = 0
required writes confirmed = yes
```

### 7.2 删除重复的 criteria / severity / output summaries

从 `SKILL.md` 删除 references 已经唯一拥有的详细正文。

### 7.3 不新增第二套 PR state artifact

不创建：

```text
PR_REVIEW_STATE.md
comment ledger file
database/cache
```

PR comment/review 本身继续承担 durable marker storage。

## 8. Reference loading policy

### 所有 review

加载：

```text
SKILL.md
references/review-method.md
references/review-criteria.md
references/severity-model.md
references/review-output-format.md
```

### PR target

额外加载：

```text
references/pr-review-protocol.md
```

### 非 PR target

不得仅因为 Skill Reviewer 支持 PR 而加载 `pr-review-protocol.md`。

### OpenAI official guidance

本 Proposal 不定义 runtime loading policy。

由 PR #5 决定：

- 是否 conditional；
- freshness；
- 是否需要 index/cache；
- 哪些 surface 适用。

## 9. OpenAI Skill alignment 检查

实现后至少验证：

### Job-to-be-done

Skill 仍清楚表达“独立 review Skill”这一 repeatable task。

### Inputs

清楚定义 target 与必要 evidence。

### Workflow

主 `SKILL.md` 以简洁 numbered steps 表达 top-level process，不把所有 exception 展开在主文件。

### Output

由 `review-output-format.md` 单一拥有。

### Final checks

主 workflow 最后明确：

- reviewed identity 是否 current；
- required evidence 是否完整；
- 如果是 durable PR review，必需写入是否已经确认。

### Supporting resources

references 按需加载，且每个 normative concept 只有一个 owner。

### Invocation

文档必须明确：精确 `/review-skill` 是本 repo 的自定义 activation policy，而不是 OpenAI 产品原生 Skill invocation 机制。

## 10. Behavior preservation / intentional changes

### 必须保持

- `/review-skill` 首次显式触发；
- same-session same-target continuation；
- independent reviewer requirement；
- reviewed-content isolation；
- immutable review identity；
- complete current-state re-review；
- severity / result vocabulary；
- PR comment event dedup；
- edited comment re-review；
- current-head full review；
- no self-approval after material modification。

### 有意改变

1. PR target 默认 durable：如果 write capability 可用，不能只在 chat 中输出后声称 PR review 已完成。
2. 必须持久化 overall `reviewer: skillpro` review。
3. 必须持久化每个 pending comment version 的 marker reply。
4. marker/overall write 失败 → `REVIEW INCOMPLETE`。
5. 五个 Comment Queue counters 从 correctness hard gate 降为 optional telemetry。
6. 不再要求无条件完整 history readback 来证明每一次 write；明确 write-success result 或 targeted readback 即可。

## 11. Tests / acceptance scenarios

至少覆盖：

### Activation

- `/review-skill <target>` 首次触发；
- 自然语言不能首次触发；
- same-session same-target continuation；
- new session / switched target 不继承 activation。

### Generic review

- standalone Skill 不加载 PR protocol；
- reviewer authoring current state 后不能 independent PASS；
- immutable identity 变化使旧 approval stale；
- missing required evidence → `REVIEW INCOMPLETE`。

### PR durable review

- 无 pending comment：current-head review 后写 overall `reviewer: skillpro`；
- 有 pending top-level comment：marker reply 成功后再 overall review；
- inline comment/reply 正确标记实际 event ID；
- edited comment 重新进入 pending；
- head-only change 不重复写旧 comment marker；
- marker write 失败 → `REVIEW INCOMPLETE`；
- overall review write 失败 → `REVIEW INCOMPLETE`；
- write tool 已明确返回 persisted result 时，不要求全量 history 再扫描；
- write result 不确定时执行 targeted readback；
- chat-only mode 明确标记 diagnostic，不形成 durable approval。

### Progressive disclosure

- 非 PR target 不读取 `pr-review-protocol.md`；
- criteria / severity / output normative rule 不在 `SKILL.md` 重复定义；
- 每个 normative concept exactly one authoritative owner。

## 12. Size / complexity acceptance

这是工程约束，不是 OpenAI 官方限制：

```text
SKILL.md target: 100–180 lines
soft ceiling: ~200 lines
```

如果实现超过该范围，implementation review 必须检查：

- 是否存在重复规则；
- 是否有 PR-only 内容可下沉；
- 是否有 examples / scenarios / format 可移到 reference；
- 是否出现第二 authoritative owner。

最终判断依据不是行数本身，而是：

```text
主文件是否只保留 core workflow + routing + final checks
```

## 13. 实现顺序

Proposal 批准后建议一个 implementation PR 完成：

```text
1. 建立 target owner map
2. 新建 review-method.md
3. 新建 pr-review-protocol.md
4. 将现有规则移动到唯一 owner
5. 精简 SKILL.md 为 core/router
6. 删除重复 summaries
7. 用简化 persistence contract 替换现有 Comment Queue Gate 复杂逻辑
8. 更新 README
9. 增加/更新 scenario tests（若 repo 有自动测试框架则自动化，否则形成 deterministic manual scenarios）
10. 对行为保持项与 intentional changes 做完整 review
```

不要采用“先继续向现有 500+ 行 `SKILL.md` 追加 hotfix，之后再重构”的顺序。

原因：这会继续制造重复 owner，重构时还需要再次拆除。

## 14. Non-goals

本 Proposal 不决定：

- OpenAI 官方 guidance 是否成为每次 review 的 runtime baseline；
- guidance freshness/cache；
- OpenAI API Skill lifecycle；
- 新的 review result vocabulary；
- 新的 database/state store；
- 自动 merge；
- PR #5 的最终设计。

## 15. Definition of Done

Implementation 完成后必须满足：

```text
SKILL.md 是 concise core/router，而不是完整 encyclopedia
PR-only protocol 仅在 PR target 加载
criteria / severity / output / PR protocol 各有单一 normative owner
无 material duplicate normative rules
review quality / independence / identity semantics 保持
PR durable review 实际写入 reviewer: skillpro overall review
pending comment 实际写入三字段 marker
write failure fail-closed 为 REVIEW INCOMPLETE
chat-only review 不伪装成 durable PR review
无新增复杂 state artifact
PR #5 仍保持独立设计范围
```
