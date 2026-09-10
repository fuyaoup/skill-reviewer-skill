# Skill Reviewer

## Purpose

Use this skill to perform an independent, professional review of an AI Skill, Agent Skill, Workflow Skill, Prompt Skill, or a pull request that changes such a skill.

The reviewer must evaluate whether another AI, without relying on author explanations or hidden chat context, can execute the reviewed skill correctly, consistently, safely, and repeatably.

The reviewer is an auditor, not a co-author by default.

## Trigger

Use this skill when the user asks to review, audit, inspect, or evaluate a skill, skill proposal, skill implementation, or pull request containing skill changes.

Typical requests include:

- `review this skill`
- `review-skill <PR>`
- `audit this SKILL.md`
- `review this skill proposal`
- `review the implementation of this skill`

Do not use this skill merely because a task happens to involve a skill. The user must be asking for review or audit work.

## Core Reviewer Principle

Review what the specification actually causes an AI to do, not what the author says they intended.

Never repair missing rules by silently relying on chat history, author explanations, model intuition, or unstated conventions.

If a critical behavior is not specified, treat it as undefined behavior.

## Reviewer Independence

The reviewer must:

- remain independent from the authoring perspective;
- challenge incorrect assumptions and unnecessary complexity;
- distinguish correctness defects from style preferences;
- avoid lowering standards because an implementation already exists;
- avoid approving its own unreviewed modifications;
- avoid treating a working happy-path example as proof that the skill is correct.

## Review Scope

Before reviewing, identify the actual review subject and its version identity.

When reviewing a pull request, inspect the current PR state and the complete relevant skill, not only the diff. Read supporting references, tests, workflow files, and linked specifications when they materially affect behavior.

When reviewing a standalone file or proposal, review the complete supplied artifact.

The reviewer should establish, when available:

- repository and path;
- pull request number;
- base revision;
- reviewed head revision;
- artifact or document revision;
- related authoritative references.

Approval must apply to a specific reviewed state, not an unspecified moving target.

## Required Review Method

Perform the review in four passes.

### Pass 1 — Architecture

Understand:

- stated goal;
- inputs and outputs;
- roles;
- authority boundaries;
- workflow phases;
- state model;
- artifacts;
- gates;
- terminal conditions.

### Pass 2 — Rule Audit

Inspect the full specification for:

- ambiguity;
- missing conditions;
- undefined behavior;
- conflicting instructions;
- unsafe defaults;
- hidden assumptions;
- incomplete state transitions;
- incomplete failure handling.

### Pass 3 — Adversarial Scenarios

Simulate realistic failures and handoffs, including where relevant:

- interrupted execution;
- a fresh session taking over;
- stale state;
- changed branch or artifact after approval;
- authentication failure;
- missing files;
- tool unavailability;
- merge conflicts;
- partial execution;
- tests passing against the wrong artifact;
- reviewer or author role confusion.

### Pass 4 — Simplification

Ask whether the same reliability can be achieved with fewer rules, artifacts, states, approvals, or tool operations.

Flag accidental complexity and overengineering separately from correctness defects.

## Review Criteria

Apply the detailed criteria in `references/review-criteria.md`.

At minimum, review:

- goal correctness;
- instruction completeness;
- ambiguity;
- execution determinism;
- state management;
- cross-agent and cross-session handoff;
- authority model;
- review loops;
- version identity;
- tool and environment assumptions;
- failure handling;
- irreversible actions and safety gates;
- testability;
- observability;
- documentation durability;
- complexity and overengineering;
- internal consistency;
- instruction hierarchy and untrusted content handling.

## Evidence Rules

Do not make a blocking finding without identifying the concrete rule, omission, contradiction, or failure path that supports it.

For each material finding, explain:

1. what is wrong;
2. why it matters;
3. a realistic failure scenario;
4. the required correction.

Do not manufacture issues to appear strict.

Do not classify editorial preference as a correctness failure.

## Severity

Use the severity model in `references/severity-model.md`.

Allowed severities:

- `BLOCKER`
- `HIGH`
- `MEDIUM`
- `LOW`
- `NIT`

## Final Result

The final review result must be exactly one of:

- `PASS`
- `PASS WITH FOLLOW-UP`
- `CHANGES REQUIRED`
- `DESIGN DECISION REQUIRED`

Use `CHANGES REQUIRED` when concrete defects must be corrected before adoption or progression.

Use `DESIGN DECISION REQUIRED` when the blocker cannot be resolved as a straightforward correction because the owner must choose between materially different design directions.

Use `PASS WITH FOLLOW-UP` only when all remaining items are explicitly non-blocking and durably trackable.

## Re-review Rules

When reviewing a revision after `CHANGES REQUIRED` or `DESIGN DECISION REQUIRED`:

- review the entire current artifact, not only previously reported findings;
- verify that old findings were actually resolved;
- look for regressions introduced by the revision;
- bind the new result to the new reviewed version;
- do not inherit a previous approval after the reviewed artifact changes.

## Modification Boundary

Unless the user explicitly asks for corrections, implementation, or a revised artifact, perform review only and do not modify the reviewed skill.

If the user asks for both review and correction, keep the review result logically separate from the changes. The corrected version is not automatically approved; it requires review as a new state.

## Output

Use the structure defined in `references/review-output-format.md`.

The output must clearly state:

- final result;
- executive summary;
- material findings with severity;
- missing scenarios;
- simplification opportunities;
- recommended tests;
- final gate and next action.

If there are no material findings, do not invent sections full of filler. State why the artifact passes and note only meaningful non-blocking observations.
