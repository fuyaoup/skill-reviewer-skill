# OpenAI 官方 Skill Guidance 基线

本文件是 Skill Reviewer 使用 OpenAI 官方 Skill 资料进行评审时的路由与判定规则。

它不是 OpenAI 官方文档的副本，也不冻结官方要求。**当前 OpenAI 官方资料本身才是 OpenAI-specific Skill compatibility / product behavior 的权威 evidence。** 每次 review 都必须重新读取当前可用的官方资料，而不能只依赖本文件中的摘要。

## 1. 必查官方来源

评审 Skill、Skill Proposal、Skill Implementation 或包含 Skill 改动的 PR 时，至少检查当前可访问的以下 OpenAI 官方来源：

1. OpenAI Help Center — `Skills in ChatGPT`
   - https://help.openai.com/en/articles/20001066-skills-in-chatgpt/
2. OpenAI Academy — `Using skills`
   - https://openai.com/academy/skills/
3. OpenAI Developers / API Reference 中当前与 Skills 有关的官方文档
   - https://developers.openai.com/api/reference/
   - 当目标涉及 API skill lifecycle、upload/package、versioning、Codex / agent runtime 等行为时，必须继续定位并读取相关具体 Skills 页面。
4. 其他由 OpenAI 官方站点当前发布、且与被评审目标直接相关的 Skills / agent skills 文档、指南或官方示例。

如果 OpenAI 官方页面链接到 Agent Skills open standard，可把被链接的 open standard 作为补充 evidence；但它不是比 OpenAI 当前官方产品文档更高的 authority。

## 2. Authority 与 freshness

评审时使用以下优先级：

```text
更高优先级 system / developer / applicable user instructions
→ Skill Reviewer 自己的 review procedure / independence / evidence / result contract
→ 当前 OpenAI 官方 Skills guidance（用于 OpenAI-specific compatibility 与官方 guidance alignment）
→ 被评审 Skill 自己声明的更严格、且不冲突的 contract
→ OpenAI 官方文档明确链接或采用的 open standard
→ 其他第三方资料、历史文章、社区惯例
```

对于 OpenAI product behavior、Skill 支持方式、安装/使用 surface、官方格式、API lifecycle 等问题，当前官方资料优先于：

- repository 中的旧摘要；
- 旧 review 结论；
- 模型记忆；
- 历史 OpenAI 文档；
- 第三方教程；
- 与当前官方资料冲突的 open-standard interpretation。

每次 review 都应记录实际读取的官方来源和读取时间/日期。不得因为之前某一轮已经读取过，就假设当前官方 guidance 没有变化。

## 3. Requirement / Recommendation / Example 必须分开

OpenAI 官方资料中的不同语气具有不同评审含义。

Reviewer 必须区分：

- **Official requirement / product constraint**：官方明确要求、限制、支持边界、结构/API contract；违反时可形成 correctness / compatibility finding。
- **Official recommendation / design guidance**：例如“通常”“often”“design tip”“works best”等建议；不能仅因为未采用就自动升级为 blocking defect。只有当目标明确声称遵循该建议，或偏离导致具体 reliability / usability / compatibility failure 时，才能形成 material finding。
- **Official example**：示例用于说明可能的实现方式，不得自动当成唯一合法设计。

任何基于 OpenAI 官方资料的 finding，都必须说明引用的是 requirement、recommendation 还是 example，以及为什么它对当前 target 有约束力。

## 4. 当前官方资料提供的基础评审维度

截至当前官方 Skills guidance，review 至少应检查以下维度，并在每轮 review 时重新对照当前官方内容验证这些维度是否仍成立：

### Skill purpose / reusability

- Skill 是否定义了可重复执行的具体任务或 workflow；
- 是否能让 ChatGPT 在重复任务中保持一致，而不是依赖每次临时重新规划；
- scope 是否清楚，是否把无关任务塞进一个过大的 Skill。

OpenAI Academy 将 Skills 描述为 reusable/shareable workflows，并建议复杂场景优先采用可组合的小 building blocks；后者是 design guidance，不是无条件 blocker。

### Name / description / discovery

- name / description 是否足以让 ChatGPT 判断 Skill 何时相关；
- trigger / routing 描述是否与真实能力一致；
- 不得声称某种自动 invocation、产品可见性或安装行为，而没有当前官方产品资料支持。

### SKILL.md workflow contract

OpenAI Academy 对典型 `SKILL.md` 的描述至少覆盖：

- Skill 做什么；
- required inputs；
- step-by-step instructions；
- required output format；
- final checks before completion。

Reviewer 应检查这些内容是否足以让 workflow 一致执行。这里的“典型内容”应结合实际 target 判断；不要把官方的描述性列表机械变成与所有 Skill 无关的格式 blocker。

### Supporting resources / code

OpenAI 官方资料说明 Skill 可包含 instructions、examples、code 和 supporting resources。Reviewer 应检查：

- supporting files 是否真正被 workflow 使用；
- scripts/code 是否有明确输入、输出、权限和失败处理；
- resource 引用是否可访问、可移植、不会静默依赖作者聊天历史；
- 不可信的外部 Skill / supporting code 是否被错误当作安全可信内容。

Help Center 明确提示 uploaded skills 可以包含 instructions、supporting files 和 code，平台 scan 不应替代用户自己的 review / policy / judgment。因此存在 executable/untrusted content 时，Skill Reviewer 应把 trust、权限和副作用纳入审查。

### Product surface / availability

OpenAI 官方资料说明 Skills 在不同 ChatGPT / Codex / API surface 上的 availability、installation、syncing 或管理方式可能不同。Reviewer 不得把某一 surface 的行为未经验证地推广到另一个 surface。

只有当被评审 Skill 实际依赖相关 product surface 时，才把这些产品约束纳入 blocking compatibility 判断。

### API Skill lifecycle / versioning（适用时）

当目标涉及 OpenAI API Skills 时，应读取当前 Developers/API Reference 的 Skills 资源文档，并检查实际 contract，例如：

- skill create / list / get / delete；
- skill content bundle；
- immutable skill versions；
- default version pointer；
- directory / zip upload contract（以当前官方 API 页面为准）。

不要把 API-specific lifecycle 强加给只面向 ChatGPT workspace 的 Skill。

## 5. Review evidence 要求

每次完整 Skill review 必须能够说明：

```text
OpenAI official guidance checked: yes/no
Official sources checked: <source list>
Guidance checked at: <date/time or review timestamp>
Applicable official requirements: <summary>
Applicable official recommendations: <summary>
Known official-guidance conflicts: <none or details>
```

对于基于官方 guidance 的 material finding，应尽可能给出具体官方页面和对应规则/表述位置，而不是笼统写“OpenAI best practice”。

## 6. Evidence 不完整

如果当前 OpenAI 官方资料因网络、权限、页面不可读、工具失败或其他 reviewer-side 原因无法获取：

- 不得用模型记忆假装当前 guidance 已验证；
- 不得把 retrieval failure 误报为 target defect；
- 如果缺失的官方 evidence 可能改变 compatibility / correctness / approval 结论，最终结果必须为 `REVIEW INCOMPLETE`；
- 在 `Incomplete Evidence` 中写明缺失的官方来源、影响和恢复 review 所需条件。

如果某个官方来源与当前 target 明确无关，可以排除，但非显然的排除必须说明理由。

## 7. 不允许的 review shortcuts

不得：

- 仅凭本文件摘要声称“符合 OpenAI 官方标准”；
- 仅凭历史 review 或旧 chat 继承 official-guidance alignment；
- 把 OpenAI recommendation 自动升级为 MUST；
- 把官方 example 当成唯一实现；
- 在官方资料已更新后继续以旧要求阻塞 target；
- 用第三方资料覆盖当前 OpenAI 官方 product guidance；
- 因为平台 scan / install 成功就假设 Skill 本身已经安全、正确或高质量。
