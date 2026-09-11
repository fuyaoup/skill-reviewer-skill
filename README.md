# skill-reviewer-skill

用于独立、专业地评审 AI Skill、Agent Skill、Workflow Skill 和 Prompt Skill，重点检查正确性、执行确定性、状态管理、失败处理、权限边界、OpenAI 官方 Skill guidance alignment 和可评审性。

## 触发条件

本 Skill **只通过一个显式命令首次触发**：

```text
/review-skill <PR / 文件 / Proposal / Skill>
```

例如：

```text
/review-skill https://github.com/owner/repo/pull/123
/review-skill SKILL.md
/review-skill <Skill Proposal>
```

除此之外，其他方式都不得首次触发 Skill Reviewer，包括：

```text
review this skill
review-skill <PR>
audit this SKILL.md
review this skill proposal
review the implementation of this skill
review 这个 skill
审查这个 SKILL.md
评审这个 skill proposal
review PR #3 里的 skill 改动
审计这个 workflow skill
/review-plan
/review-implementation
/review
```

也就是说：

- 没有前导 `/` 的 `review-skill` 不触发；
- 自然语言中的 review / audit / inspect / evaluate / 评审 / 审计 不能首次触发；
- 即使对象明确是 Skill、SKILL.md、Skill Proposal 或包含 Skill 改动的 PR，也不能通过语义推断自动首次触发；
- `/review-plan`、`/review-implementation` 和其他项目级 review workflow 与本 Skill 的首次触发完全分离。

`/review-skill` 用于让 AI 明确进入 Skill Reviewer 评审流程；它本身不是由 CLI 或程序注册的命令。

## 同一会话连续 review

`/review-skill <target>` 已在当前会话显式触发后，如果用户明确要求继续评审**同一个 target**，不需要再次输入 `/review-skill`。

例如以下表达可视为同一 review 的 continuation：

```text
继续 review
re-review 最新改动
PR 更新了，再检查
检查新的 review comments
验证之前的 findings 是否解决
```

以下情况不能自动延续：

- 开启新会话；
- 切换到另一个 PR / Skill / Proposal；
- 用户只是报告状态，例如“已经更新好了”，但没有要求继续 review；
- 用户明确结束当前 review。

新会话或切换 target 时，必须重新使用 `/review-skill <target>` 显式触发。

## OpenAI 官方 Skill guidance

每次完整 Skill review 都必须重新读取当前可访问、与 target 相关的 OpenAI 官方 Skill 资料，并把它们作为 review baseline，而不是依赖模型记忆或历史摘要。

至少包括适用的：

- OpenAI Help Center — Skills in ChatGPT；
- OpenAI Academy — Using skills；
- OpenAI Developers / API Reference — 当 target 涉及 API Skills 时的当前 Skills contract；
- 其他由 OpenAI 官方当前发布且与 target 直接相关的 Skills / Agent Skills guidance。

Reviewer 必须区分官方 requirement / product constraint、recommendation / design guidance 和 example。官方 recommendation 或 example 不能被机械提升为 blocking requirement。

如果与结论相关的 OpenAI 官方 guidance 无法获取，且缺失 evidence 可能改变 compatibility / correctness / approval 判断，本轮必须使用 `REVIEW INCOMPLETE`。

详细规则见 [`references/openai-official-skill-guidance.md`](./references/openai-official-skill-guidance.md)。

## 如何确认 Skill 已触发

当 Skill Reviewer 被 `/review-skill` 显式触发并开始执行时，AI 必须在首次实质性评审输出中显示：

```text
Skill activated: skill-reviewer
Review mode: independent
Review target: <当前评审对象>
```

例如评审 PR #123 时：

```text
Skill activated: skill-reviewer
Review mode: independent
Review target: PR #123
```

看到 `Skill activated: skill-reviewer`，即可确认 AI 已进入本 Skill 的评审流程。

这三个标记只表示 Skill 已触发并开始执行，不代表评审完成，也不代表评审对象已经通过。

如果没有显式 `/review-skill` 首次触发，且当前会话不存在有效的同-target review continuation，则不应输出 `Skill activated: skill-reviewer`。

## PR 多轮 review

通过 `/review-skill <Pull Request>` 触发 PR review 时，Skill Reviewer 不只检查当前 head，还会读取与 review、实现修订和设计决策有关的 PR history。

多轮 review 以单条 comment / reply event 为状态单位，而不是以整个 thread 为单位。

对于已经由 skillpro 处理过、且内容版本没有变化的 comment，后续轮次不会重复 review；新的 reply 或被编辑后的 comment 会重新进入待评审队列。

skillpro 针对具体 comment 的回复会写入持久化 marker：

```text
reviewer: skillpro
reviewed-comment-id: <GitHub comment ID>
reviewed-comment-version: <updated_at 或稳定 body hash>
reviewed-at-head: <PR head SHA>
```

Quote / 引用原 comment 仅用于可读性，不作为已处理状态依据。

即使没有新的 pending comments，Skill Reviewer 仍会独立 review 当前完整 PR head；历史 comments 不替代最终 PR review。

详细规则见 [`SKILL.md`](./SKILL.md)。
