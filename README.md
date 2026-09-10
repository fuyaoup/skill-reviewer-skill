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

详细触发规则和完整评审流程见 [`SKILL.md`](./SKILL.md)。
