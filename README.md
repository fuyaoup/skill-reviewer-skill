# skill-reviewer-skill

用于独立、专业地评审 AI Skill、Agent Skill、Workflow Skill 和 Prompt Skill，重点检查正确性、执行确定性、状态管理、失败处理、权限边界和可评审性。

## 触发条件

只有当用户明确要求对 Skill 或其设计 / 实现进行 **review、audit、inspect、evaluate、评审或审计** 时，才应触发本 Skill。

仅仅因为当前任务涉及 Skill，不应自动触发 Skill Reviewer。

典型触发方式：

```text
review this skill
review-skill <PR>
audit this SKILL.md
review this skill proposal
review the implementation of this skill
```

中文也可以直接使用：

```text
review 这个 skill
审查这个 SKILL.md
评审这个 skill proposal
review PR #3 里的 skill 改动
审计这个 workflow skill
检查这个 skill 是否可以正式采用
```

推荐统一使用：

```text
/review-skill <PR / 文件 / Proposal / Skill>
```

例如：

```text
/review-skill https://github.com/owner/repo/pull/123
```

或：

```text
/review-skill SKILL.md
```

`/review-skill` 是推荐的统一触发写法，用于让 AI 明确进入 Skill Reviewer 的评审流程；它本身不是由 CLI 或程序注册的命令。

## 如何确认 Skill 已触发

当 Skill Reviewer 被触发并开始执行时，AI 必须在首次实质性评审输出中显示：

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

如果没有触发本 Skill，则不应输出 `Skill activated: skill-reviewer`。

## PR 多轮 review

传入 Pull Request 时，Skill Reviewer 不只检查当前 head，还会读取与 review、实现修订和设计决策有关的 PR history。

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
