# Proposal：OpenAI 官方 Skill Guidance 作为 Skill Review 基线

## 状态

讨论中（仅用于设计与决策跟踪，当前 PR 不修改 Skill Reviewer 的运行时行为）。

## 背景

Skill Reviewer 当前已经有一套独立的通用评审标准，覆盖目标正确性、指令完整性、歧义、执行确定性、状态管理、跨 Agent / 跨 Session handoff、权限模型、review loop、版本身份、工具与环境假设、失败处理、安全、可测试性、可观察性、文档持久性、复杂度、内部一致性以及未受信任内容等。

近期提出一个新问题：是否应把 **OpenAI 当前官方 Skill guidance** 纳入 Skill Reviewer 的评审基线，用于约束或辅助所有 Skill review。

直接把官方资料设为“每轮 review 都必须重新完整读取的强制基线”虽然能够提高与官方资料的一致性，但也会带来明显成本和设计问题：

- 每次 review / re-review 都加载多份官方文档，会显著增加上下文与 token 消耗；
- Help Center、Academy、Developers/API Reference 面向的 product surface 不完全相同，不能机械混用；
- 官方资料中存在 requirement、product constraint、recommendation、design guidance、example 等不同强度，不能全部当成 blocking rule；
- 官方页面会更新，需要定义 freshness 和 stale detection；
- Skill Reviewer 本身已经有通用 correctness / reliability 基线，需要明确官方 guidance 与现有 criteria 的关系，避免重复和冲突；
- 一些 Skill 并不依赖 OpenAI-specific product behavior，强制加载官方资料可能没有实际收益。

因此，本议题先独立设计，不与 PR #4 的 comment marker / session continuity 修改混在一起。

## 目标

确定一套成本可控、证据可追溯、不会误用官方资料的规则，使 Skill Reviewer 在需要时能够正确使用 OpenAI 官方 Skill guidance，同时不让每轮 re-review 无条件加载大量重复上下文。

## 非目标

本 Proposal 当前不决定具体实现，也不立即：

- 修改 `SKILL.md`；
- 修改 `review-criteria.md`；
- 增加强制 `OpenAI Official Guidance Status` 输出；
- 设置固定 freshness TTL；
- 把任何单一 OpenAI 页面内容固化为永久规范；
- 规定所有 Skill 都必须符合某一个 OpenAI product surface 的行为。

这些应在本 Proposal 决策收敛后再进入实现 PR。

## 当前默认评审基线

在没有额外 OpenAI 官方 guidance 规则时，Skill Reviewer 的默认 review baseline 应继续由三类 evidence 构成：

1. **Skill Reviewer 自身的通用 review criteria**：判断 artifact 是否正确、完整、确定、可恢复、可测试、可审计。
2. **被评审 Skill 自身声明的 contract**：检查其 trigger、输入、输出、workflow、权限、失败处理、state、review gate 等是否内部一致并真实可执行。
3. **当前 repository / runtime / PR evidence**：检查规范与实际实现、工具能力、PR head、supporting artifacts、tests 和 review history 是否一致。

OpenAI 官方 guidance 应被设计成额外的权威 evidence layer，而不是取代以上三层。

## 需要解决的核心设计问题

### 1. OpenAI 官方 guidance 是强制全局 baseline，还是 conditional baseline？

候选方案：

**方案 A：所有 Skill review 都强制读取官方资料。**

优点：
- 最保守；
- 能持续发现与官方产品 guidance 的偏差。

缺点：
- 上下文和 token 成本高；
- 对与 OpenAI-specific product behavior 无关的 Skill 收益有限；
- re-review 重复读取问题明显。

**方案 B：只在存在 OpenAI-specific dependency / compatibility claim 时强制读取。**

例如：
- Skill 声称符合 OpenAI Skills 规范；
- 依赖 ChatGPT Skills 的安装、触发、共享或产品行为；
- 依赖 Codex Skill 的特定运行行为；
- 使用 OpenAI Skills API；
- 用户明确要求“按 OpenAI 官方规范评审”。

优点：
- 上下文成本低；
- 官方资料只在真正有约束力时加载。

缺点：
- 需要可靠判断“何时适用”；
- 可能漏掉作者未声明但实际存在的 OpenAI-specific assumption。

**方案 C：轻量官方 baseline 始终存在，深度官方核验按条件触发。**

例如 repository 内维护一个很短的、可验证 freshness 的 guidance index；普通 review 先使用 index 判断是否需要加载官方正文。

优点：
- 在成本和覆盖率之间折中；
- 可减少每轮完整网页读取。

缺点：
- 增加 baseline 维护与 freshness state；
- 需要避免 repository 摘要变成比官方来源更“权威”。

当前倾向：**优先研究 B 或 C，不采用无条件 A。**

### 2. 首次 review 与 re-review 的读取策略是否应不同？

需要决定：

- 首次触发 `/review-skill` 时是否完整获取 applicable official guidance；
- 同一 target 的 re-review 是否只做 freshness check；
- 什么情况下必须重新读取正文；
- 新会话是否可以复用 durable verified baseline；
- head 变化是否自动导致官方 guidance 重新加载，还是只有 scope / source 变化才触发。

建议方向：**re-review 不应因为 PR head 改变就自动重新完整加载官方 guidance。** PR head freshness 和 official-source freshness 是两个不同状态。

### 3. freshness 如何定义？

候选机制：

- 固定 TTL，例如 24 小时或 7 天；
- HTTP metadata / ETag / Last-Modified / content hash（如果工具可可靠获得）；
- 每轮只检查官方 source identity / update signal；
- 新会话直接重新核验；
- 仅当 source change、scope change、TTL 过期或 evidence 不完整时重读正文。

需要评估工具是否能稳定提供足够的 freshness evidence，避免设计无法执行的规则。

### 4. 官方资料的 authority 如何分层？

至少需要区分：

- OpenAI 官方明确 requirement / product constraint；
- OpenAI 官方 recommendation / design guidance；
- OpenAI 官方 example；
- OpenAI 官方页面链接或引用的开放标准；
- 第三方资料。

建议原则：

- requirement / product constraint 可以形成 compatibility / correctness finding；
- recommendation 不能因为“没照做”就自动成为 blocker；
- example 不能当作唯一合法实现；
- 当 OpenAI 当前 product guidance 与外部 open standard 的解释冲突时，需要明确适用 surface 后再决定 authority，而不是简单全局覆盖。

### 5. 不同 product surface 如何隔离？

ChatGPT、Codex、OpenAI API 中的 Skills 能力和 lifecycle 可能不同。

需要规定：

- reviewer 必须先识别 target surface；
- 只读取与该 surface 有关的官方 guidance；
- API-specific create/version/default-version 等 contract 不应强加给只面向 ChatGPT 的 Skill；
- ChatGPT UI / installation / sync 行为也不能未经验证推广到 API 或 Codex。

### 6. 输出中是否需要强制显示官方 guidance 状态？

候选方案：

- 每次 review 都输出完整 `OpenAI Official Guidance Status`；
- 仅在官方 guidance 实际适用时输出；
- 只在有 official-guidance finding / conflict / evidence limitation 时输出；
- 在 `Review Identity` 或 `Evidence` 中以更紧凑方式记录。

需要权衡 auditability 与输出噪音。

### 7. 官方资料不可访问时应如何处理？

需要区分：

- 官方资料对当前 target 不适用；
- 官方资料适用，但 retrieval 失败；
- reviewer 已有 durable verified baseline，但 freshness 未过期；
- 没有任何当前可验证 evidence。

建议原则：只有当缺失的官方 evidence **可能改变当前 compatibility / correctness / approval 判断** 时，才应关闭 gate 为 `REVIEW INCOMPLETE`。不能因为任何一个官方页面暂时不可读，就机械阻塞所有 Skill review。

## 上下文 / Token 成本约束

设计必须把上下文成本作为一等约束。

完成方案至少应满足：

- 普通 re-review 不重复加载未变化的大段官方正文；
- 不把多个 product surface 的无关文档全部装入上下文；
- repository 内如果维护摘要，只加载适用部分；
- 只有需要验证具体 finding 时才拉取对应官方证据；
- 不因为追求“官方对齐”让 Skill Reviewer 自身变成上下文过重的 Skill。

## 推荐的下一步研究

在进入实现前，建议完成以下验证：

1. 列出当前真正与 Skill Reviewer 相关的 OpenAI 官方 Skill 来源，并按 ChatGPT / Codex / API surface 分类。
2. 对每个来源标注可机器判断的 freshness signal：是否有更新时间、ETag、Last-Modified、稳定 URL、content hash 能力等。
3. 用三个真实 review 场景估算上下文成本：
   - 普通 Skill review；
   - 同一 PR 连续 3 次 re-review；
   - OpenAI API Skill compatibility review。
4. 比较方案 B 与 C 的可靠性和 token 成本。
5. 决定是否需要 durable guidance cache / index，以及它的 source-of-truth 地位。

## 建议验收标准

本 Proposal 可以进入实现阶段前，至少应明确：

- 官方 guidance 何时是 mandatory；
- 首次 review 与 re-review 的读取差异；
- freshness 判定；
- source authority 层级；
- product-surface isolation；
- evidence retrieval failure 行为；
- 输出中如何记录官方 evidence；
- 上下文 / token 成本上限或控制机制；
- 哪些规则属于 correctness requirement，哪些只是 reviewer guidance。

## 暂不采纳的方案

暂不采纳：**“每次完整 review / re-review 都重新完整读取所有 OpenAI 官方 Skill 页面，并把它们统一作为 blocking baseline。”**

原因：该方案会引入明显重复上下文，混淆不同 product surface，而且没有解决 freshness、recommendation vs requirement、scope applicability 等问题。
