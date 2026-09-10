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
- tool assumptions cause the workflow to silently take an unintended path.

## MEDIUM

A reliability or maintainability defect that does not normally break the workflow immediately but materially increases ambiguity, operational cost, or future failure probability.

Typical examples:

- duplicated state with weak synchronization rules;
- important but non-critical behavior left underspecified;
- missing regression scenario for a meaningful edge case;
- avoidable workflow complexity with ongoing maintenance cost.

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
- A blocking issue requiring an owner choice between materially different designs -> `DESIGN DECISION REQUIRED` rather than pretending there is one mechanical fix.
