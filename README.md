# GASI:One Specification & Architecture Repository

This repository is the **source of truth for GASI:One Solution Architecture,
Functional Design (FD), and executable specifications**.

The documents are intended for **Business Analysts, System Analysts, Developers,
QA, and AI agents**. The main principle is **human-first, AI-consumable**:
documentation must remain easy for people to review while being structured enough
for AI to support analysis, planning, task breakdown, and implementation.

---

## 1. Repository Purpose

`gasi.one.spec` is used to store and manage:

- Solution Architecture and Architecture Decision Records (ADR)
- Functional Specification / Functional Design
- Business Rules
- Technical Plan / Technical Design
- Data Model
- API, event, file, and integration contracts
- Non-functional requirements
- Development Tasks
- Requirement Checklists
- Test / SIT documentation when needed

Specifications are grouped by **domain / subdomain / FD**, not by source code
repository. One FD may affect multiple implementation repositories.

Example:

```text
specs/
└── administration/
    └── identity-access/
        ├── FD-ADM-001-user-management/
        ├── FD-ADM-002-role-management/
        └── FD-ADM-003-access-control/
```

---

## 2. Repository Structure

```text
gasi.one.spec/
├── .agents/
│   └── skills/                         # Spec Kit skills for AI agents
├── .specify/
│   ├── memory/
│   │   └── constitution.md             # Global governance
│   ├── scripts/
│   ├── templates/
│   │   └── overrides/                  # GASI:One template customizations
│   └── workflows/
├── architecture/                       # Cross-domain architecture and ADRs
├── references/
│   └── gasi-one/                       # Verified API/web/CLI framework baseline
├── specs/
│   └── <domain>/
│       └── <subdomain>/
│           ├── README.md               # Subdomain overview and glossary
│           └── <FD-ID>-<feature-name>/
│               ├── spec.md
│               ├── plan.md
│               ├── research.md
│               ├── data-model.md
│               ├── quickstart.md
│               ├── contracts/
│               ├── checklists/
│               ├── test-cases/
│               │   ├── acceptance.md
│               │   ├── authorization.md
│               │   ├── security.md
│               │   ├── integration.md
│               │   └── regression.md
│               └── tasks.md
└── README.md
```

Directories shown above are the target structure. They are added incrementally as
the constitution, templates, and pilot FD are established.

### Main directories

| Path | Purpose |
|---|---|
| `.agents/skills/` | Spec Kit skills used by AI agents such as Codex |
| `.specify/` | Core Spec Kit configuration, templates, scripts, and workflows |
| `.specify/memory/constitution.md` | Core principles and governance for GASI:One specifications |
| `.specify/templates/overrides/` | Project-specific templates without modifying Spec Kit core templates |
| `architecture/` | Cross-domain system context, standards, integration landscape, and ADRs |
| `references/gasi-one/` | Verified API, web, and CLI framework baseline for planning when implementation repositories are unavailable |
| `specs/` | Domain, subdomain, and FD documentation |

### FD identity and naming

FD IDs must be stable and unique across GASI:One:

```text
FD-<DOMAIN-CODE>-<SEQUENCE>
```

Examples: `FD-ADM-001`, `FD-HCM-001`, and `FD-PAY-001`.

FD directories use:

```text
specs/<domain>/<subdomain>/<FD-ID>-<kebab-case-feature-name>/
```

Each `spec.md` must identify at least the FD ID, domain, subdomain, status,
owner, affected systems, and source wiki/document references. Folder placement
provides navigation; metadata preserves context when a document is moved or
referenced outside this repository.

---

## 3. Core Principles

### 3.1 Human-first, AI-consumable

Specifications must remain easy for people to read and understand.

AI may help create, refine, analyze, or implement specifications, but documentation must not be written only for AI consumption.

### 3.2 Specification is the source of truth for product behavior

`spec.md` defines **what the product must do**.

If implementation differs from the specification, do not automatically change the specification to match the code. First determine whether:

- the implementation is incorrect, or
- the business requirement has actually changed.

### 3.3 Separate WHAT from HOW

```text
spec.md   = WHAT / WHY
plan.md   = HOW
```

Functional specifications should not contain Spring Boot classes, React components, database columns, or other implementation details.

### 3.4 Organize by domain, subdomain, and FD

Use the canonical hierarchy:

```text
specs/
├── administration/
│   └── identity-access/
│       ├── FD-ADM-001-user-management/
│       ├── FD-ADM-002-role-management/
│       └── FD-ADM-003-access-control/
├── people/
│   └── employee-management/
├── time-management/
├── payroll/
└── recruitment/
```

Do not organize specifications primarily by implementation repository such as
`api/`, `web/`, or `cli/`. A single FD may affect multiple repositories.

### 3.5 Separate documentation intent

Documents must distinguish:

- **As-is**: current GASI:One behavior and architecture.
- **To-be**: behavior or architecture introduced by the FD.
- **Decision record**: decision, rationale, alternatives, and consequences.
- **Execution artifact**: plan and tasks used to realize the change.

This prevents current-state documentation from being interpreted as new work by
people or AI agents.

---

## 4. New FD Workflow

A typical workflow is:

```text
Project governance
    ↓
$speckit-constitution
    ↓
Requirement → $speckit-specify → spec.md
    ↓
$speckit-clarify
    ↓
$speckit-plan → plan.md / research.md / data-model.md / contracts/ / quickstart.md
    ↓
$speckit-testcases → test-cases/
    ↓
$speckit-tasks → tasks.md
    ↓
$speckit-analyze
    ↓
Human review and approval
    ↓
$speckit-implement
    ↓
$speckit-converge
```

Not every supporting command must always be used, but `spec.md`, `plan.md`, and
`tasks.md` are the core traceability chain before implementation.
`speckit-analyze` is read-only. `speckit-converge` assesses the implemented
state and appends remaining work to `tasks.md` without rewriting the intent.

---

## 5. Creating a New Specification

Example: creating **User Management**.

Open the `gasi.one.spec` repository in VS Code, then open **Codex Chat**.

Use:

```text
$speckit-specify

SPECIFY_FEATURE_DIRECTORY:
specs/administration/identity-access/FD-ADM-001-user-management

Create a functional specification for User Management in GASI:One.

Domain: Administration
Subdomain: Identity and Access
FD ID: FD-ADM-001

Purpose:
User Management allows authorized users to manage user accounts that can access GASI:One.

Initial scope:
- Authorized users can view the user list.
- Authorized users can create users.
- Authorized users can view user details.
- Authorized users can update users.
- Authorized users can activate or deactivate users.
- A user may be associated with an employee.
- A user can have one or more roles.
- Username must be unique.
- Inactive users cannot log in.
- Access to User Management is controlled by authorization.

Focus on functional behavior and business rules.
Do not include implementation details.
If a business rule is unclear, identify it as requiring clarification instead of inventing behavior.
```

Expected initial result:

```text
specs/
└── administration/
    └── identity-access/
        └── FD-ADM-001-user-management/
            ├── spec.md
            └── checklists/
                └── requirements.md
```

The explicit path is required because default Spec Kit auto-numbering scans feature
directories immediately below `specs/`. Nested domain/subdomain paths are supported
through `SPECIFY_FEATURE_DIRECTORY`. The selected path is persisted in
`.specify/feature.json` for subsequent plan, tasks, analysis, and implementation steps.

---

## 7. Reviewing `spec.md`

After `spec.md` is created, use clarification to find material ambiguity in
scope, actors, business rules, validation, edge cases, and success criteria.
The command asks focused questions one at a time and records accepted answers
back into the specification.

```text
$speckit-clarify
```

### 7.1 Acceptance Criteria

Acceptance criteria remain in `spec.md`, inside each user story as independently
testable Given/When/Then scenarios. They define the expected business behavior
and become the source for detailed test cases after planning.

If an acceptance criterion is unclear or missing, update and review `spec.md`
before generating test cases.

---

## 8. Creating a Technical Plan

Once `spec.md` has been reviewed, create the technical design that explains how
the approved requirements will be delivered. Planning resolves technical
decisions and produces the data, interface, and validation contracts needed by
later phases.

```text
$speckit-plan
```

Generated artifacts:

```text
<FD>/
├── plan.md
├── research.md
├── data-model.md
├── contracts/
└── quickstart.md
```

This command does not generate `tasks.md` or implementation code.

---

## 9. Creating Test Cases

After `plan.md` has been reviewed, derive traceable QA/SIT scenarios from the
specification and plan. Test cases cover applicable acceptance, authorization,
security, integration, and regression concerns. Missing expected behavior is
reported as a specification gap rather than invented.

```text
$speckit-testcases
```

Generated artifacts, when relevant:

```text
<FD>/
└── test-cases/
    ├── acceptance.md
    ├── authorization.md
    ├── security.md
    ├── integration.md
    └── regression.md
```

This command generates test-case documentation only. It does not execute tests,
generate automated test code, or create `tasks.md`.

---

## 10. Creating Development Tasks

After `plan.md` and applicable test cases have been reviewed, convert the
approved design into an ordered implementation checklist. Tasks retain
traceability to user stories and test cases, identify concrete target paths,
and mark work that can be executed in parallel.

```text
$speckit-tasks
```

Generated artifact:

```text
<FD>/
└── tasks.md
```

This command creates the implementation task breakdown only. It does not
execute tasks or modify implementation code.

---

## 11. Analyze Before Implementation

To check consistency between specification, plan, and tasks:

```text
$speckit-analyze
```

Use this to identify:

- requirements not covered by the plan
- plan items without a supporting requirement
- tasks without a supporting specification
- ambiguous requirements
- contradictions between artifacts

---

## 12. Implement

Once the specification, plan, and tasks have been reviewed:

```text
$speckit-implement
```

Implementation must continue to follow:

```text
spec.md
   ↓
plan.md
   ↓
test-cases/
   ↓
tasks.md
   ↓
code and automated tests
```

Not the other way around.

---

## 13. Existing Features / Requirement Changes

Do not create a new FD for every change. Update the existing feature when the
requested behavior still belongs to the same business capability and owner.

Choose the command based on the type of change:

| Situation | Command |
|---|---|
| The business change is already known | `$speckit-specify` |
| A rule is ambiguous and requires a decision | `$speckit-clarify` |
| Only the technical design changes | `$speckit-plan` |
| The implementation is incomplete against existing artifacts | `$speckit-converge` |

### Updating a known requirement

Use the existing feature directory and describe only the requested change:

```text
$speckit-specify

SPECIFY_FEATURE_DIRECTORY:
specs/<domain>/<subdomain>/<FD-ID>-<feature-name>

Update the existing specification with this requirement:
- One employee can have multiple users.
- Only one user can be the primary user.

Preserve unaffected requirements, user stories, and stable IDs.
Do not create a new feature.
```

After reviewing the updated `spec.md`, continue with:

```text
$speckit-clarify
$speckit-plan
$speckit-testcases
$speckit-tasks
$speckit-analyze
```

### Clarifying an uncertain rule

Use `$speckit-clarify` when the requirement is not yet a decision. For example,
the business has allowed multiple users for one employee but has not decided
whether those users may all be active at the same time.

```text
$speckit-clarify
```

The command asks focused questions and records accepted answers in `spec.md`.
It must not be used as a substitute for describing a known requirement change.

### Technical-only and implementation changes

- If observable behavior remains unchanged, keep `spec.md` unchanged and update
  the technical design with `$speckit-plan`.
- If the approved specification, plan, and tasks are correct but implementation
  work remains, use `$speckit-converge` to append the missing work to `tasks.md`.
- If an implementation conflicts with the approved specification, fix the
  implementation; do not rewrite the specification merely to match the defect.

Create a new FD only when the request represents a distinct business
capability, owner, lifecycle, or independently deliverable scope.

---

## 14. When to Use Each Spec Kit Command

| Command | Use it when |
|---|---|
| `$speckit-specify` | Creating a functional specification for a new FD |
| `$speckit-clarify` | Requirements are ambiguous or additional clarification is needed |
| `$speckit-plan` | The functional spec is sufficiently clear and technical design is needed |
| `$speckit-testcases` | The plan is ready and traceable QA/SIT or automation-ready cases are needed before task generation |
| `$speckit-tasks` | The technical plan needs to be broken down into implementation tasks |
| `$speckit-analyze` | Checking consistency between spec, plan, and tasks |
| `$speckit-checklist` | Creating a review or quality checklist |
| `$speckit-implement` | Implementing based on the approved artifacts |
| `$speckit-converge` | Checking and aligning implementation against the specification |

---

## 15. Multi-Agent Usage

This repository is not locked to a single AI agent.

Spec Kit is currently initialized for **Codex**, therefore the repository contains:

```text
.agents/skills/
```

However, `specs/` remains agent-neutral.

In the future, the same repository can be used with another integration such as Claude without recreating the functional specifications.

```text
                     specs/
                        │
                 Source of Truth
                        │
             ┌──────────┴──────────┐
             ▼                     ▼
           Codex                 Claude
```

AI integration is tooling. The specification belongs to the GASI:One project.

---

## 16. AI-Executable Documentation

A document is ready for AI-assisted delivery only when:

- Requirements use stable IDs such as `FR-001`.
- User stories have objective, independently testable acceptance scenarios.
- Scope, out-of-scope behavior, assumptions, and dependencies are explicit.
- Entities and fields include validation and lifecycle constraints.
- Interface contracts are machine-readable where practical.
- Affected repositories, services, and components are identified.
- Security, performance, availability, audit, and other NFRs are measurable.
- Tasks trace back to requirements or user stories and name concrete file paths.
- Important architecture decisions are recorded in the plan or an ADR.
- Test cases trace to acceptance criteria and functional requirements, and
  planned automation declares a test level and actionable target.

AI may draft and execute work, but human approval remains required for scope,
architecture decisions, cross-domain impact, and high-risk implementation results.

---

## 17. Templates and Spec Kit Compatibility

Core templates live in `.specify/templates/`. GASI:One customizations belong in
`.specify/templates/overrides/` so Spec Kit upgrades do not overwrite the local
documentation standard.

Template resolution follows:

```text
project overrides → presets → extensions → core templates
```

Project test-case generation uses:

- `.agents/skills/speckit-testcases/SKILL.md`;
- `.specify/templates/overrides/test-suite-template.md`;
- `.specify/templates/overrides/test-case-template.md`.

The installed Spec Kit version supports nested FD paths when
`SPECIFY_FEATURE_DIRECTORY` is explicit. Default sequential numbering still
assumes feature directories immediately below `specs/`; domain-aware FD number
allocation therefore remains a future automation task.

Spec Kit scripts resolve `gasi.one.spec` as their repository root; they do not
automatically switch to sibling implementation repositories. Planning uses the
verified `references/gasi-one/` baseline and may identify logical target
repositories and paths without those repositories being present. Write access
to an implementation repository is required only when that repository is
actually modified. Build, test, and Git validation remain the responsibility of
each affected implementation repository.

---

## 19. Do / Don't

### Do

- Write specifications based on business behavior.
- Use language that BA, SA, Developers, and QA can understand.
- Review AI-generated content before treating it as valid.
- Group documentation by domain / subdomain / FD.
- Reuse the existing GASI:One framework and generator.
- Use hooks for customization when available.
- Update the specification when business behavior changes.

### Don't

- Do not create specifications only for AI consumption.
- Do not put implementation details in the functional specification.
- Do not build CRUD manually when the generator already supports it.
- Do not modify generated code directly when a hook or extension point is available.
- Do not change a specification only to make it match an incorrect implementation.
- Do not maintain duplicate documentation for humans and AI.

---

## 20. Quick Start

To create a new FD:

```text
1. Open gasi.one.spec in VS Code
2. Open Codex Chat
3. Provide SPECIFY_FEATURE_DIRECTORY and run $speckit-specify
4. Review spec.md
5. Run $speckit-clarify if needed
6. Run $speckit-plan (no repeated implementation context required)
7. Review the technical plan
8. Run $speckit-testcases; review statuses, test levels, and automation targets
9. Run $speckit-tasks
10. Run $speckit-analyze
11. Start implementation only after the artifacts have been reviewed
```

Example FD directory:

```text
specs/administration/identity-access/FD-ADM-001-user-management
```
