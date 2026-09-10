# Review Criteria

Use these criteria to audit the reviewed skill as an executable specification.

## A. Goal Correctness

Check whether the skill actually solves the problem it claims to solve.

Review:

- clarity of goal;
- defined inputs;
- defined outputs;
- success criteria;
- scope boundaries;
- mismatch between stated purpose and actual workflow;
- incorrect premises or proxy solutions that miss the root problem.

## B. Instruction Completeness

Check whether all behavior required to complete the workflow is specified.

Look for missing:

- preconditions;
- trigger conditions;
- forbidden actions;
- branch logic;
- exception handling;
- fallback behavior;
- retry behavior;
- stop conditions;
- success and failure conditions;
- state updates;
- handoff rules;
- review gates.

A critical behavior that depends on model guessing is a defect.

## C. Ambiguity

Identify wording with multiple reasonable operational interpretations.

Examples include phrases such as:

- when necessary;
- when appropriate;
- confirm completion;
- inspect the latest state;
- update relevant files;
- synchronize code.

For each ambiguous rule, determine who decides, using what evidence, and what exact condition triggers the behavior.

## D. Execution Determinism

Check whether the same input produces materially consistent execution.

Review:

- ordering of actions;
- gate conditions;
- precedence rules;
- stopping rules;
- whether important actions are left to free-form model choice;
- whether the agent may execute beyond authorized scope.

Pay particular attention to workflows where the agent may do too much or cannot tell when to stop.

## E. State Management

For multi-stage workflows, inspect:

- where current state is stored;
- the authoritative source of truth;
- auxiliary versus authoritative artifacts;
- stale-state detection;
- conflicting state files;
- interruption recovery;
- cross-session restoration;
- hidden or implicit state.

Chat history should not silently become the only authoritative workflow state when durable continuation is required.

## F. Cross-Agent and Cross-Session Handoff

Check whether a fresh agent can continue the workflow from durable artifacts.

Review:

- context acquisition;
- required handoff artifacts;
- dependence on user relaying messages;
- information loss between agents;
- next-action markers;
- dependence on inaccessible prior chat history.

Prefer workflows where a new agent can reconstruct state from durable artifacts alone.

## G. Authority Model

Check whether the skill clearly defines:

- who may author;
- who may modify;
- who may review;
- who may approve;
- who may merge or publish;
- who may override a decision;
- who owns the final decision.

Look specifically for self-approval and role-confusion risks.

## H. Review Loop

If review is part of the workflow, verify a complete loop:

1. artifact creation;
2. review;
3. review result;
4. revision;
5. re-review;
6. approval;
7. transition.

Check what happens after `CHANGES REQUIRED`, whether re-review covers the full current state, whether approval is durable, and whether the reviewed version is identifiable.

## I. Version Identity

Check whether the reviewed subject can be uniquely identified.

Possible identifiers include:

- commit SHA;
- PR number;
- branch;
- document revision;
- artifact version;
- base SHA;
- head SHA.

Approval that is not bound to a specific version is unreliable.

## J. Tool and Environment Assumptions

Identify unverified assumptions such as:

- network access;
- authenticated CLI;
- browser availability;
- file existence;
- current working directory;
- active branch;
- synchronized remote state;
- installed dependencies.

The skill should distinguish verified facts, assumptions, and fallback behavior.

## K. Failure Handling

Review realistic failure paths, including:

- command failure;
- authentication failure;
- network failure;
- merge conflict;
- stale branch;
- missing file;
- malformed artifact;
- partial execution;
- process interruption;
- tool unavailable;
- test failure;
- rejected review;
- inconsistent state.

The skill must define what the agent does after failure, not only how the happy path works.

## L. Safety and Irreversible Actions

Inspect destructive or hard-to-reverse actions such as:

- merge;
- delete;
- overwrite;
- force push;
- production deploy;
- database mutation;
- secret change;
- destructive migration.

Check for appropriate approval, preconditions, verification, rollback, and stop gates.

Higher irreversibility should require stronger gates.

## M. Testability

Determine whether critical rules can be proven through tests.

Consider:

- positive tests;
- negative tests;
- regression tests;
- scenario tests;
- interruption tests;
- stale-state tests;
- permission tests;
- failure-path tests.

If a rule cannot be tested, ask how compliance can be demonstrated.

## N. Observability

Check whether execution is auditable and failures are visible.

Review:

- visible errors;
- stack traces where appropriate;
- progress for long-running work;
- current phase visibility;
- clear final result;
- silent failure risk;
- partial success incorrectly reported as success.

Silent failure is a reliability defect.

## O. Documentation Durability

Identify critical workflow knowledge that exists only in transient locations such as:

- chat;
- one-off prompts;
- model memory;
- temporary review comments.

If future execution depends on that knowledge, determine whether it belongs in a durable artifact.

## P. Complexity and Overengineering

Look for unnecessary:

- state layers;
- duplicate artifacts;
- approvals;
- git operations;
- documents without consumers;
- theoretical completeness that creates maintenance cost without meaningful reliability gains.

Distinguish necessary complexity from accidental complexity.

Recommend simpler alternatives when they preserve correctness and reliability.

## Q. Internal Consistency

Check for contradictions between:

- different sections;
- examples and normative rules;
- role permissions;
- stop rules and next-step rules;
- state transitions;
- tool-priority rules;
- referenced specifications.

## R. Instruction Hierarchy and Untrusted Content

If the skill reads repository files, issues, PR comments, external documents, or user-generated content, inspect whether untrusted content can be mistaken for higher-priority workflow instructions.

The skill should distinguish:

- workflow instructions;
- task data;
- untrusted embedded instructions.
