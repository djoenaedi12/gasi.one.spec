<!--
Sync Impact Report
- Version change: placeholder scaffold → 1.0.0
- Modified principles:
  - Placeholder principles → I. Human-First Specification Authority
  - Placeholder principles → II. WHAT/HOW Separation and Traceability
  - Placeholder principles → III. Generator-First Delivery
  - Placeholder principles → IV. Automatic Framework Baseline
  - Placeholder principles → V. Explicit Boundaries and Verifiable Quality
- Added sections:
  - Repository and Architecture Constraints
  - Delivery Workflow and Quality Gates
- Removed sections: none
- Follow-up TODOs: none
-->
# GASI:One Specification Constitution

## Core Principles

### I. Human-First Specification Authority

`spec.md` MUST be the source of truth for observable product behavior, business
rules, scope, assumptions, and dependencies. Specifications MUST be readable by
business analysts, system analysts, developers, and QA without requiring source
code knowledge. AI-generated content MUST remain reviewable by humans and MUST
not introduce behavior absent from an approved requirement.

If implementation and specification differ, the discrepancy MUST be assessed
before either is changed. An implementation defect MUST NOT be converted into a
business requirement merely to match existing code.

### II. WHAT/HOW Separation and Traceability

Functional specifications MUST describe WHAT users need and WHY. They MUST NOT
contain implementation choices such as framework classes, database columns,
component structures, or generated-file paths. Technical plans MUST describe
HOW the approved behavior will be delivered.

Requirements MUST use stable identifiers and remain traceable through plans,
contracts, test cases, tasks, and implementation. Planning artifacts MUST use
the primary language of `spec.md` unless the user explicitly requests another
language; identifiers, commands, field names, and code contracts retain their
technical spelling.

### III. Generator-First Delivery

Standard GASI:One CRUD MUST be planned and delivered through `gasi.one.cli`.
The CRUD/resource definition JSON MUST be the source of truth for generated
CRUD. Generated files owned by `.gasi-one/manifest.json` MUST NOT be edited
directly.

Implementation approaches MUST be selected in this order:

1. generator configuration;
2. framework hook or extension point;
3. narrowly scoped custom implementation;
4. dependency on another FD or plugin when that domain owns the behavior.

A custom implementation MUST identify the exact generator or extension-point
gap, remain outside generator ownership, reuse applicable framework contracts,
and include regeneration-safety verification.

### IV. Automatic Framework Baseline

`$speckit-plan` MUST load the feature's `spec.md`, this constitution, and
`references/gasi-one/README.md` without requiring the user to repeat framework
rules in the prompt. It MUST then load the relevant CLI, API, web, and
cross-repository reference documents routed by that README.

`references/gasi-one/baseline.md` is the verified historical baseline for
planning. Planning MUST NOT depend on sibling implementation repositories being
available. If a required capability is absent from the baseline, the plan MUST
label it as a framework gap or an implementation-time verification item rather
than inventing a command, schema key, hook, or extension point.

Before generator `sync`, custom implementation, or other high-risk delivery,
the implementing team MUST verify the baseline against the framework
repository or release that will actually be used. Planning may name logical
target repositories and paths from the baseline without accessing them.

### V. Explicit Boundaries and Verifiable Quality

Each feature MUST state its in-scope behavior, out-of-scope behavior,
dependencies, affected actors, validation rules, lifecycle constraints,
permissions, audit expectations, and measurable success criteria. Behavior
owned by another FD or plugin MUST be represented as a dependency and MUST NOT
be reimplemented locally.

Plans MUST cover applicable data models, migrations, API and web/UI contracts,
error mapping, authorization, audit, integrations, generated-file ownership,
testing strategy, and regeneration safety. Every acceptance scenario MUST be
objectively testable. Security decisions MUST fail closed when a mandatory
dependency is unavailable.

## Repository and Architecture Constraints

- Specifications MUST be organized by domain, subdomain, and stable FD ID under
  `specs/`; they MUST NOT be organized primarily by implementation repository.
- `architecture/` is reserved for cross-domain architecture, standards,
  integration landscapes, and Architecture Decision Records.
- `references/gasi-one/` records verified framework behavior and MUST remain
  separate from Spec Kit's internal configuration under `.specify/`.
- Stable cross-plugin business interfaces MUST use contract modules and
  capability contracts. Consumers MUST NOT import another plugin's entity,
  repository, service implementation, or implementation JAR.
- Public API IDs MUST follow the verified encoded-ID contract; internal storage
  details MUST NOT leak into public contracts.
- Backend authorization remains authoritative. UI visibility MUST NOT be
  treated as authorization.
- Password credentials, authentication attempts, lockout, sessions, and token
  policy belong to the Authentication domain unless an approved specification
  assigns ownership differently.

## Delivery Workflow and Quality Gates

The normal delivery sequence is:

```text
constitution
  → specify
  → clarify when needed
  → plan
  → test cases when applicable
  → tasks
  → analyze
  → human approval
  → implement
  → converge
```

For `$speckit-plan`, the user only needs to select the feature context, normally
through `SPECIFY_FEATURE_DIRECTORY` or the persisted `.specify/feature.json`,
and invoke the command. The plan workflow MUST derive business context from
`spec.md` and framework context from `references/gasi-one/`.

Every plan MUST:

- record the baseline commits it uses;
- classify each material requirement as generator configuration, framework
  hook/extension, custom implementation, or dependency on another FD/plugin;
- use only CLI commands and schema keys verified by the baseline;
- distinguish Bean Validation failures from business-rule failures according
  to the verified error contract;
- identify generated versus custom file ownership;
- record assumptions, framework gaps, and implementation-time verifications;
- stop after design artifacts and MUST NOT create implementation code or
  `tasks.md`.

Before implementation, the specification, plan, contracts, test cases when
required, and tasks MUST be reviewed for traceability and contradictions.
Architecture decisions and cross-domain impacts require explicit human review.

## Governance

This constitution governs all GASI:One specification and planning artifacts.
When another document conflicts with it, this constitution takes precedence
unless a formally approved amendment states otherwise.

Amendments MUST:

1. document the changed rule and rationale;
2. assess effects on templates, skills, references, and existing features;
3. use semantic versioning:
   - MAJOR for incompatible governance changes or principle removal;
   - MINOR for new principles or materially expanded requirements;
   - PATCH for clarifications without changed obligations;
4. update the amendment date and include a sync-impact report for review.

Compliance MUST be checked during planning and again after design. Any
exception MUST be explicit in the plan's Complexity Tracking section with its
rationale and rejected simpler alternative.

**Version**: 1.0.0 | **Ratified**: 2026-09-29 | **Last Amended**: 2026-09-29
