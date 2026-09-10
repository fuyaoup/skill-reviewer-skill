# Review Output Format

Use this structure for completed reviews.

## Review Result

One of:

- `PASS`
- `PASS WITH FOLLOW-UP`
- `CHANGES REQUIRED`
- `DESIGN DECISION REQUIRED`

## Executive Summary

State the current maturity of the reviewed skill and the most important conclusion.

Keep this concise and evidence-based.

## Review Identity

When available, record enough information to uniquely identify the reviewed state:

- repository;
- PR number;
- base revision;
- reviewed head revision;
- artifact path/version.

## Findings

For each material finding, use:

### [SEVERITY] FINDING-ID — Title

**Location**

Exact section, rule, file, or artifact location.

**Problem**

What is wrong or missing.

**Why it matters**

The operational consequence.

**Failure scenario**

At least one realistic scenario showing how the defect can fail in practice.

**Required correction**

The specific change required to resolve the finding.

Do not create findings for pure style preferences unless they materially affect execution.

## Missing Scenarios

List important scenarios the skill does not currently cover.

Omit this section if there are no meaningful missing scenarios.

## Overengineering / Simplification

Identify unnecessary states, artifacts, approvals, repeated operations, or rules that can be simplified without reducing reliability.

Omit this section when there is no meaningful simplification opportunity.

## Test Recommendations

List tests that would materially increase confidence in the reviewed rules.

Prioritize tests for blocking findings, failure paths, handoff, stale state, permissions, and interruption recovery.

## Final Gate

State explicitly:

- whether the skill may proceed to the next stage;
- which findings are blocking, if any;
- what must happen next;
- whether the complete revised artifact must be re-reviewed.

# Re-review Output

For a revision, also state:

- which previous findings are resolved;
- which remain unresolved;
- any regressions or new findings;
- the identity of the newly reviewed version.

Never inherit a previous PASS after the artifact changes without reviewing the new state.
