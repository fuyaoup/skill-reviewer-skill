# Review Output Format

Use this structure for completed reviews.

## Required Sections

Every completed review MUST include:

1. `Review Result`
2. `Executive Summary`
3. `Review Identity`
4. `Findings`
5. `Final Gate`

The following sections are conditional and should be included only when meaningful:

- `Missing Scenarios`
- `Overengineering / Simplification`
- `Test Recommendations`
- `Incomplete Evidence` — required when result is `REVIEW INCOMPLETE`
- `Re-review Status` — required for re-review of a revised state

Do not create filler sections merely to preserve visual symmetry.

## Review Result

One of:

- `PASS`
- `PASS WITH FOLLOW-UP`
- `CHANGES REQUIRED`
- `DESIGN DECISION REQUIRED`
- `REVIEW INCOMPLETE`

## Executive Summary

State the current maturity of the reviewed skill and the most important conclusion.

Keep this concise and evidence-based.

## Review Identity

Record enough information to identify the exact reviewed state.

For repository or pull-request reviews, include when applicable:

- repository;
- relevant path;
- PR number;
- base revision;
- reviewed head commit SHA.

For standalone artifacts, include at least one immutable identifier such as:

- content hash;
- immutable artifact ID;
- explicit version tied to immutable content.

Mutable identifiers such as filenames, branch names, URLs, or document titles may be included for context but are not sufficient approval identity by themselves.

If immutable identity cannot be established because the reviewer lacks necessary evidence, the result cannot be `PASS` or `PASS WITH FOLLOW-UP`; use `REVIEW INCOMPLETE` unless the lack of version identity is itself a defect in the reviewed workflow.

Every approval-class result MUST state that it applies only to the identified reviewed state.

## Findings

If material findings exist, use the following structure for each finding:

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

If there are no material findings, write:

`No material findings.`

Do not omit the `Findings` section entirely.

## Incomplete Evidence

This section is REQUIRED when the result is `REVIEW INCOMPLETE`.

Record:

- evidence successfully reviewed;
- evidence unavailable, unreadable, truncated, or otherwise incomplete;
- why the evidence could not be verified;
- what evidence is required to resume or complete the review.

Do not present reviewer-side evidence failure as a defect in the reviewed artifact unless the artifact itself incorrectly depends on missing or nonexistent material.

## Missing Scenarios

List important scenarios the skill does not currently cover.

Omit this section if there are no meaningful missing scenarios.

## Overengineering / Simplification

Identify unnecessary states, artifacts, approvals, repeated operations, or rules that can be simplified without reducing reliability.

Omit this section when there is no meaningful simplification opportunity.

## Test Recommendations

List tests that would materially increase confidence in the reviewed rules.

Prioritize tests for blocking findings, failure paths, handoff, stale state, permissions, interruption recovery, self-approval, adversarial embedded instructions, incomplete evidence, and version drift.

Omit this section when no additional test would materially improve confidence.

## Final Gate

State explicitly:

- whether the skill may proceed to the next stage;
- which findings are blocking, if any;
- what must happen next;
- whether the complete revised artifact must be re-reviewed;
- whether the evidence set was complete;
- the exact reviewed identity to which the result applies.

`REVIEW INCOMPLETE` always closes the gate.

# Re-review Status

For a revision, this section is REQUIRED and must state:

- which previous findings are resolved;
- which remain unresolved;
- any regressions or new findings;
- the identity of the newly reviewed version.

Never inherit a previous `PASS` or `PASS WITH FOLLOW-UP` after the artifact changes without reviewing the new complete state.

If the reviewer authored or materially modified the current state, clearly state that the review is diagnostic and cannot serve as independent approval.
