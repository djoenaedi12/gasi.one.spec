---
name: speckit-testcases
description: "Generate or update traceable QA/SIT and automation-ready test-case suites for the active Spec Kit feature from spec.md and plan.md. Use after planning and before task generation; do not use it to execute tests."
metadata:
  author: "GASI:One"
  source: "project"
---

# Spec Kit Test Cases

Generate maintainable test-case definitions under the active feature's
`test-cases/` directory. Treat `spec.md` as the authority for expected
business behavior and `plan.md` as supporting context for technical,
integration, and security scenarios.

## User Input

```text
$ARGUMENTS
```

Use non-empty user input to narrow categories, priorities, or scenarios. Do not
ask the user to repeat context already present in the active feature artifacts.

## Scope Guard

This skill may create or update only files under `FEATURE_DIR/test-cases/`.
It must not:

- modify `spec.md`, `plan.md`, `tasks.md`, application code, or test code;
- invent business behavior that is absent from the specification;
- record execution results such as Passed or Failed as permanent case status;
- delete or renumber existing test cases automatically.

When an expected result cannot be derived from the artifacts, report a
specification gap with its source location instead of guessing.

## Setup

1. From the Spec Kit repository root, run:

   ```bash
   SPECIFY_FEATURE_NO_PERSIST=1 .specify/scripts/bash/check-prerequisites.sh --json --require-spec --template test-suite-template
   ```

   Parse `FEATURE_DIR`, `AVAILABLE_DOCS`, and `TEMPLATE_CONTENT`. The
   prerequisite script also requires `plan.md`; stop with its actionable error
   when the active feature, specification, or plan is missing.

2. Resolve the per-case scaffold:

   ```bash
   .specify/scripts/bash/resolve-template.sh test-case-template --json
   ```

   Parse `TEMPLATE_CONTENT`. Stop if either template cannot be resolved.

3. Load the minimum relevant context:

   - required: `spec.md` and `plan.md`;
   - when present: `research.md`, `data-model.md`, `contracts/`,
     `quickstart.md`, and the project constitution;
   - existing files under `test-cases/`, including their IDs and traceability.

## Test Categories

Create only relevant files:

- `acceptance.md`: business journeys, acceptance scenarios, business
  validation, and observable outcomes. This is normally required.
- `authorization.md`: roles, permissions, menus, record rules, access scopes,
  and allow/deny behavior.
- `security.md`: authentication, MFA, credentials, secrets, sessions, tokens,
  abuse cases, and security controls.
- `integration.md`: boundaries between repositories, services, providers,
  APIs, events, files, or other external interfaces.
- `regression.md`: existing critical behavior that must remain unchanged.
  Create it only when the feature changes existing behavior or the user asks
  for regression coverage.

A case belongs to the category matching its primary purpose. Use `Related`
metadata for secondary concerns instead of duplicating the case.

## Generation Rules

1. Build a traceability inventory from:
   - user stories and acceptance scenarios;
   - functional requirements and buildable success criteria;
   - edge cases and explicit assumptions;
   - plan decisions, contracts, and security or integration obligations.

2. Derive the FD test-case prefix from the FD identifier:
   - `FD-IAM-007` becomes `TC-IAM-007`;
   - assign case IDs as `TC-IAM-007-001`, `TC-IAM-007-002`, and so on;
   - if the FD identifier is unavailable, stop and ask for it rather than
     generating unstable IDs.

3. Preserve stable IDs:
   - scan every existing category file before assigning an ID;
   - update an existing case when its traceability and intent match;
   - allocate new IDs after the highest existing sequence;
   - never reuse, renumber, or silently delete IDs;
   - flag apparently obsolete cases for review and mark them `Deprecated`
     only with user approval.

4. Use `test-suite-template` for each category file and repeat
   `test-case-template` for its cases.

5. Every case must include:
   - one stable ID and a concise title;
   - type, priority, definition status, automation status, and test level;
   - an automation target naming the component, contract, flow, or planned test
     file path when automation is planned or already implemented;
   - references to applicable `US/AC`, `FR`, `SC`, contract, or plan item;
   - objective, preconditions, test data, ordered steps, expected results,
     postconditions, and related cases;
   - observable outcomes without implementation assumptions unless the case is
     explicitly integration or security focused.

6. Use these controlled values:

   ```text
   Type: Acceptance | Authorization | Security | Integration | Regression
   Priority: P1 | P2 | P3
   Case Status: Draft | Reviewed | Approved | Deprecated
   Automation: Manual | Planned | Automated | Not Applicable
   Test Level: Unit | Component | Contract | Integration | E2E | Manual
   ```

7. Select the narrowest test level that proves the case objective:
   - `Unit`: one rule, policy, calculation, or state transition in isolation;
   - `Component`: one module or service with controlled dependencies;
   - `Contract`: an interface schema, protocol, or compatibility boundary;
   - `Integration`: collaboration across repositories, services, databases,
     providers, or other real boundaries;
   - `E2E`: a critical user or system journey through deployed interfaces;
   - `Manual`: behavior requiring human judgment or intentionally excluded from
     automated execution.

   Category and level are independent: for example, an `Authorization` case
   can target either a unit-level policy or an integration-level API boundary.
   Use a separate case only when another level validates a materially different
   risk; do not duplicate the same intent merely to cover every level.

8. Keep automation metadata internally consistent:
   - `Planned` or `Automated` requires a non-empty automation target and an
     automated level (`Unit`, `Component`, `Contract`, `Integration`, or `E2E`);
   - `Manual` or `Not Applicable` uses test level `Manual` and automation target
     `None`;
   - use `Automated` only when the implementation already exists and identify
     its test file when known;
   - use `Planned` when `$speckit-tasks` should create implementation work for
     the automated test.

9. Preserve human-authored material when updating. Merge traceability and
   scenario changes narrowly; do not rewrite unrelated cases or replace a suite
   wholesale.

## Coverage Validation

Before completion, verify:

- each acceptance scenario has at least one test case;
- each functional requirement is covered by at least one applicable case;
- relevant authorization, security, integration, and regression obligations
  are represented;
- all case IDs are unique and match the FD prefix;
- all traceability references resolve to loaded artifacts;
- test level, automation status, and automation target are mutually consistent;
- every step has an observable expected result;
- no unresolved template placeholders remain;
- no duplicate cases express the same intent.

Report uncovered or ambiguous requirements as gaps. Do not modify their source
artifacts automatically.

## Completion Report

Report:

- active FD and `test-cases/` path;
- files created and updated;
- case count by category and priority;
- case count by test level and automation status;
- coverage by user story, acceptance scenario, and functional requirement;
- specification gaps or cases requiring review;
- whether the feature is ready for `$speckit-tasks`.

## Done When

- [ ] Relevant test-case suites exist under the active FD.
- [ ] IDs and traceability are stable and validated.
- [ ] Existing human-authored content is preserved.
- [ ] Coverage and remaining gaps are reported.
