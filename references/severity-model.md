# 严重度模型

使用 severity 描述影响程度，而不是 reviewer 的个人偏好。

## BLOCKER

会破坏 workflow correctness、安全性、authority boundary，或使“到底批准了哪个状态”无法可靠识别的 defect。

在该问题解决前，不应采用该 Skill，也不应继续推进。

典型示例：

- 不可逆动作可以在缺少必需 approval 的情况下发生；
- 在要求独立 review 的场景中，author 可以静默 self-approve；
- approval 没有绑定到具体版本，后续修改可能错误继承该 approval；
- state transition 可能产生相互矛盾的 authoritative states；
- mandatory gate 失败后 workflow 仍可继续。

## HIGH

在现实场景中，很可能导致错误执行、错误状态或错误决策的 defect。

通常应在 release 前修复。

典型示例：

- 对常见 failure mode 的恢复处理不完整；
- 控制主要 workflow transition 的规则存在歧义；
- fresh session 无法可靠重建必需 workflow state；
- tool assumption 使 workflow 静默进入非预期路径；
- review 可以在没有检测 drift 的情况下混合来自不同 artifact version 的证据；
- reviewer 可能被被评审内容中嵌入的对抗性指令控制。

## MEDIUM

通常不会立即破坏 workflow，但会实质增加歧义、运营成本或未来失败概率的 reliability / maintainability defect。

典型示例：

- duplicated state 缺少可靠同步规则；
- 重要但非关键行为定义不足；
- 有意义的 edge case 缺少 regression scenario；
- 可避免的 workflow complexity 带来持续维护成本；
- 当 `CHANGES REQUIRED` 与 `DESIGN DECISION REQUIRED` 都会关闭 gate 时，两者边界仍不清楚。

## LOW

对执行影响有限的轻微 clarity、consistency、maintainability 或 robustness 问题。

## NIT

纯编辑性或极小的质量问题，不会实质影响执行。

不要把 style preference 升级为更高 severity。

# 结果映射

Severity 会影响最终结果，但不能取代 reviewer judgment。

默认规则：

- 存在任何未解决的 `BLOCKER` -> `CHANGES REQUIRED` 或 `DESIGN DECISION REQUIRED`。
- 存在任何影响正常运行的未解决 `HIGH` -> 通常为 `CHANGES REQUIRED`。
- 只剩 `MEDIUM`、`LOW` 或 `NIT` -> 如果它们确实非阻塞，可使用 `PASS WITH FOLLOW-UP`。
- 没有 material findings -> `PASS`。
- blocking issue 需要 owner 在两个或更多实质不同、都有效的设计之间作出选择 -> `DESIGN DECISION REQUIRED`。
- reviewer 缺少足够证据完成请求的 review -> `REVIEW INCOMPLETE`。

`REVIEW INCOMPLETE` 不是 severity，也不表示被评审 artifact 一定存在 defect。它表示 reviewer 缺少足够可靠证据，无法给出完整 verdict。其 Final Gate 始终关闭，直到缺失证据被取得并完成所需 review。

# `DESIGN DECISION REQUIRED` 边界

当某个 defect 可以通过一个明确的 required correction 解决，而无需选择 product、workflow、governance 或 authority policy 时，使用 `CHANGES REQUIRED`。

仅当以下条件全部满足时，使用 `DESIGN DECISION REQUIRED`：

1. 该问题是 blocking；
2. 仍存在两个或更多实质不同、都有效的设计；
3. 在这些设计之间选择会改变 product、workflow、governance 或 authority policy；
4. reviewer 无权作出该 policy choice。

示例：

- Approval 缺少 immutable version binding -> `CHANGES REQUIRED`。
- 必需的 failure branch 没有定义行为 -> `CHANGES REQUIRED`。
- 系统必须在“author 可 merge”和“必须由 independent maintainer merge”两种 governance model 中选择 -> `DESIGN DECISION REQUIRED`。
- owner 必须决定 reviewer approval 是 advisory，还是 mandatory release gate -> `DESIGN DECISION REQUIRED`。
