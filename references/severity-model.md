# Severity Model

Use severity to describe impact, not reviewer preference.

## BLOCKER

A defect that breaks workflow correctness, safety, authority boundaries, or the ability to identify what was approved.

The skill should not be adopted or progressed until the issue is resolved.

Typical examples:

- irreversible action can occur without required approval;
- author can silently self-approve where independent review is required;
- approval is not bound to a concrete version and later changes may inherit it;
- state transitions can produce contradictory authoritative states;
- the workflow can continue after a failed mandatory gate.

## HIGH

A defect likely to cause incorrect execution, incorrect state, or incorrect decisions in realistic scenarios.

It should normally be fixed before release.

Typical examples:

- incomplete failure recovery for a common failure mode;
- ambiguous rule controlling a major workflow transition;
- fresh sessions cannot reliably reconstruct required workflow state;
- tool assumptions cause the workflow to silently take an unintended path;
- review can combine evidence from different artifact versions without detecting drift;
- the reviewer can be controlled by adversarial instructions embedded in the reviewed content.

## MEDIUM

A reliability or maintainability defect that does not normally break the workflow immediately but materially increases ambiguity, operational cost, or future failure probability.

Typical examples:

- duplicated state with weak synchronization rules;
- important but non-critical behavior left underspecified;
- missing regression scenario for a meaningful edge case;
- avoidable workflow complexity with ongoing maintenance cost;
- an unclear boundary between `CHANGES REQUIRED` and `DESIGN DECISION REQUIRED` when both would keep the gate closed.

## LOW

A minor clarity, consistency, maintainability, or robustness issue with limited operational impact.

## NIT

A purely editorial or very small quality issue that does not materially affect execution.

Do not escalate style preference into a higher severity.

# Result Mapping

Severity informs the result but does not replace reviewer judgment.

Use these defaults:

- Any unresolved `BLOCKER` -> `CHANGES REQUIRED` or `DESIGN DECISION REQUIRED`.
- Any unresolved `HIGH` that affects normal operation -> normally `CHANGES REQUIRED`.
- Only `MEDIUM`, `LOW`, or `NIT` findings -> may be `PASS WITH FOLLOW-UP` when they are genuinely non-blocking.
- No material findings -> `PASS`.
- A blocking issue requiring an owner choice between two or more materially different valid designs -> `DESIGN DECISION REQUIRED`.
- Insufficient reviewer evidence to complete the requested review -> `REVIEW INCOMPLETE`.

`REVIEW INCOMPLETE` is not a severity and does not assert that the reviewed artifact is defective. It means the reviewer lacks enough reliable evidence to issue a complete verdict. Its final gate is always closed until the missing evidence is obtained and the required review is completed.

# DESIGN DECISION REQUIRED Boundary

Use `CHANGES REQUIRED` when a defect has one required correction that can be stated without selecting product, workflow, governance, or authority policy.

Use `DESIGN DECISION REQUIRED` when all of the following are true:

1. the issue is blocking;
2. two or more materially different valid designs remain;
3. choosing among them changes product, workflow, governance, or authority policy;
4. the reviewer does not have authority to make that policy choice.

Examples:

- Approval lacks immutable version binding -> `CHANGES REQUIRED`.
- A required failure branch has no defined behavior -> `CHANGES REQUIRED`.
- The system must choose between author-merge and independent-maintainer-merge governance models -> `DESIGN DECISION REQUIRED`.
- The owner must choose whether reviewer approval is advisory or a mandatory release gate -> `DESIGN DECISION REQUIRED`.
