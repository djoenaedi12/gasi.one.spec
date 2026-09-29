# Feature Specification: Record-Level Access Control

**FD ID**: `FD-IAM-003`

**Domain**: Administration

**Subdomain**: Identity & Access

**Owner**: GASI:One Identity & Access

**Affected Systems**: GASI:One capabilities that read or change protected business records

**Source References**: GASI:One wiki migration sources, IAM brainstorming, `FD-IAM-001`, `FD-IAM-002`, and the prior authentication ERD as a technical reference only

**Feature Branch**: `N/A`

**Created**: 2026-09-25

**Status**: Draft

**Input**: Define role-based record rules that restrict which GASI:One records a permitted action may read or change.

## Purpose and Scope

Record-Level Access Control restricts which business records a user may access
after role-derived action permission has been granted. It enables a global user
to work across GASI:One while limiting data by company, ownership, organization,
workflow responsibility, or other approved record attributes.

The capability does not grant actions. `FD-IAM-002` first determines whether the
active roles permit an action; this FD then evaluates the record scope for that
action. Allow rules contribute accessible scopes, while any matching explicit
deny rule takes precedence.

### In Scope

- Create, review, update, activate, and deactivate record rules.
- Define rule effect, protected resource, applicable actions, and business conditions.
- Associate record rules with roles.
- Evaluate rules from multiple active roles.
- Apply consistent scope to lists, searches, details, counts, exports, reports, and background processing.
- Enforce scope for create, update, delete, and bulk changes.
- Preview and explain effective record scope for authorized reviewers.
- Preserve history for security-relevant rule and assignment changes.

### Out of Scope

- Granting action permissions or controlling menu visibility.
- Defining roles, user-role assignments, or active-role compatibility.
- Creating company membership for a global user.
- Authentication, MFA, session, token, trusted-device, or PAT lifecycle.
- Field-level masking or field-level editability.
- Application-specific approval workflow decisions.
- Technical query language, persistence strategy, or policy engine selection.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Define a Record Rule (Priority: P1)

As an authorized access administrator, I want to define an allow or deny rule for
a protected resource and action so that data scope reflects an explicit business
policy.

**Why this priority**: All record-level enforcement depends on valid, reviewable
rule definitions.

**Independent Test**: Create an inactive allow rule with a unique code, target
resource, actions, and condition; activate it; and verify an invalid or duplicate
definition is rejected.

**Acceptance Scenarios**:

1. **AC-US1-01** — **Given** an authorized administrator and a unique rule code, **When** a complete allow or deny rule is created, **Then** it is stored as `Inactive` with its business purpose, target resource, actions, effect, and condition.
2. **AC-US1-02** — **Given** an existing normalized rule code, **When** another rule is created or renamed to that code, **Then** the operation is rejected without changing either rule.
3. **AC-US1-03** — **Given** a rule condition references an unsupported resource attribute or context value, **When** the administrator attempts to activate it, **Then** activation is rejected with an actionable validation reason.
4. **AC-US1-04** — **Given** a complete inactive rule, **When** an authorized administrator activates it, **Then** it becomes eligible for role association and evaluation.
5. **AC-US1-05** — **Given** an active rule is associated with roles, **When** it is deactivated, **Then** it stops affecting new authorization evaluations while its history and associations remain traceable.
6. **AC-US1-06** — **Given** an actor without rule-management authority, **When** the actor attempts to create, update, activate, or deactivate a rule, **Then** the operation is denied and rule state remains unchanged.

---

### User Story 2 - Associate Record Rules with Roles (Priority: P1)

As an authorized access administrator, I want to associate record rules with
roles so that each responsibility receives an appropriate data scope.

**Why this priority**: Role association connects reusable rules to the active-role
model established in `FD-IAM-002`.

**Independent Test**: Associate multiple active rules with a role, remove one
association, and verify only current active associations participate in future
scope evaluation.

**Acceptance Scenarios**:

1. **AC-US2-01** — **Given** an active role and active record rule, **When** an authorized administrator associates them, **Then** the rule becomes applicable when that role contributes the targeted action permission.
2. **AC-US2-02** — **Given** an existing role-rule association, **When** the same association is submitted again, **Then** no duplicate association is created.
3. **AC-US2-03** — **Given** an inactive role or inactive rule, **When** an administrator attempts to create an association, **Then** the operation is rejected.
4. **AC-US2-04** — **Given** a role has several associated rules, **When** one association is removed, **Then** only that rule stops participating through that role.
5. **AC-US2-05** — **Given** a role has a record rule for an action the role does not permit, **When** effective access is evaluated, **Then** the rule does not grant the missing action permission.
6. **AC-US2-06** — **Given** an actor without role-rule authority, **When** the actor attempts to add or remove an association, **Then** the operation is denied and associations remain unchanged.

---

### User Story 3 - Restrict Record Reading (Priority: P1)

As a user with permission to read a resource, I want to see only records allowed
by my effective record scope so that protected data is not disclosed.

**Why this priority**: Preventing unauthorized data disclosure is the primary
purpose of record rules.

**Independent Test**: Read, search, count, export, and directly request records
inside and outside an allow scope, including a matching deny rule, and verify all
channels expose the same permitted set.

**Acceptance Scenarios**:

1. **AC-US3-01** — **Given** an active role grants read permission and has no applicable active record rule, **When** the user reads that resource, **Then** the record-rule layer does not further restrict the role's permission path.
2. **AC-US3-02** — **Given** one or more applicable allow rules, **When** a list or search is performed, **Then** records matching at least one allow rule and no deny rule are returned.
3. **AC-US3-03** — **Given** a record matches both an allow rule and a deny rule, **When** it is requested, **Then** access is denied because explicit deny takes precedence.
4. **AC-US3-04** — **Given** a record is outside effective scope, **When** it is requested directly, **Then** its data and existence are not disclosed.
5. **AC-US3-05** — **Given** a filtered result set, **When** totals, pagination, aggregation, export, or report output is produced, **Then** those outputs are derived only from records within effective scope.
6. **AC-US3-06** — **Given** the active roles do not grant read permission, **When** a record rule would otherwise match, **Then** access remains denied because a record rule cannot grant an action.
7. **AC-US3-07** — **Given** a service identity or background process accesses protected records, **When** it performs work, **Then** explicit roles, permissions, and record rules produce the same effective scope as an equivalent user context without an implicit bypass.

---

### User Story 4 - Restrict Record Changes (Priority: P1)

As a user with permission to change a resource, I want create, update, delete, and
bulk actions to honor the same record policy so that write access cannot bypass
read-time restrictions.

**Why this priority**: Protecting reads without protecting mutations would leave
the underlying business data exposed to unauthorized changes.

**Independent Test**: Attempt create, update, delete, and bulk changes inside and
outside effective scope, including a change that moves a record out of scope.

**Acceptance Scenarios**:

1. **AC-US4-01** — **Given** the user has create permission, **When** a proposed new record satisfies at least one applicable allow rule and no deny rule, **Then** creation is allowed.
2. **AC-US4-02** — **Given** a proposed new record matches a deny rule or fails all applicable allow rules, **When** creation is attempted, **Then** creation is rejected without persisting the record.
3. **AC-US4-03** — **Given** the user has update permission and may access the current record, **When** proposed changes remain within allow scope and match no deny rule, **Then** the update is allowed.
4. **AC-US4-04** — **Given** proposed changes would move a record outside effective scope or into a denied scope, **When** the update is attempted, **Then** the update is rejected and the original record remains unchanged.
5. **AC-US4-05** — **Given** the user has delete permission and the existing record is within effective scope, **When** deletion is requested, **Then** deletion is allowed; otherwise it is denied without disclosing hidden record details.
6. **AC-US4-06** — **Given** a bulk change includes at least one record outside effective scope, **When** the operation is submitted, **Then** the entire bulk operation is rejected and no targeted record is changed.

---

### User Story 5 - Combine Rules from Multiple Active Roles (Priority: P1)

As a user with multiple active roles, I want their compatible record scopes
combined predictably so that I receive the intended union of responsibilities
without bypassing explicit restrictions.

**Why this priority**: Multiple roles are active together by default, so record
scope composition must be deterministic and secure.

**Independent Test**: Activate roles with overlapping allow scopes, one
unrestricted permission path, and a deny rule; verify the resulting scope uses
allow union and global deny precedence.

**Acceptance Scenarios**:

1. **AC-US5-01** — **Given** several active permission-contributing roles have applicable allow rules, **When** scope is calculated, **Then** the allowed scope is the deduplicated union of records matched by their allow rules.
2. **AC-US5-02** — **Given** any applicable active-role rule denies a record, **When** scope is calculated, **Then** that record is excluded even when another active role allows it or has an unrestricted path.
3. **AC-US5-03** — **Given** a permission-contributing active role has no applicable rule for a resource and action, **When** scope is calculated, **Then** that role contributes an unrestricted record scope before applicable deny rules are subtracted.
4. **AC-US5-04** — **Given** a role does not contribute the required action permission, **When** its associated record rules are evaluated, **Then** those rules do not contribute an allow scope for that action.
5. **AC-US5-05** — **Given** an applicable role has allow rules but a record matches none of them, **When** the record is requested through that role path, **Then** that role contributes no access to the record.
6. **AC-US5-06** — **Given** a role is removed from the active-role set, **When** effective scope is recalculated, **Then** its allow and deny contributions are removed while contributions from remaining active roles continue.
7. **AC-US5-07** — **Given** rule evaluation lacks mandatory context or encounters unavailable policy state, **When** access is requested, **Then** the request is denied rather than evaluated with a broader fallback scope.
8. **AC-US5-08** — **Given** a policy change removes record access while an older authorization context is still active, **When** that context requests the removed access, **Then** access fails closed until reevaluation is complete.

---

### User Story 6 - Preview and Explain Effective Scope (Priority: P2)

As an authorized administrator or reviewer, I want to preview and explain a
user's effective record scope so that policies can be validated before and after
changes without exposing protected record content.

**Why this priority**: Composed allow and deny rules are safer to operate when
administrators can understand their outcome and source.

**Independent Test**: Preview a user's scope for a resource and action, confirm
the result identifies contributing roles and rules, and verify an unauthorized
reviewer cannot access the explanation.

**Acceptance Scenarios**:

1. **AC-US6-01** — **Given** a user, active-role set, resource, and action, **When** an authorized reviewer requests a scope preview, **Then** the result identifies whether action permission exists and summarizes the effective allow and deny scope.
2. **AC-US6-02** — **Given** a record is included or excluded, **When** an authorized reviewer requests an explanation, **Then** the contributing roles and matching allow or deny rules are identified without revealing unrelated protected records.
3. **AC-US6-03** — **Given** a proposed rule or association change, **When** it is previewed before activation, **Then** the reviewer can compare current and proposed scope without granting the proposed access.
4. **AC-US6-04** — **Given** an actor lacks scope-review authority, **When** the actor requests a preview or explanation, **Then** the information is not disclosed.
5. **AC-US6-05** — **Given** rule or role-rule changes have occurred, **When** an authorized reviewer opens the policy history, **Then** each event identifies the actor, time, action, and relevant before-and-after values.
6. **AC-US6-06** — **Given** two administrators edit the same rule or association state, **When** the second administrator saves after the first change succeeds, **Then** the stale update is rejected and the latest state remains intact.
7. **AC-US6-07** — **Given** emergency or administrative unrestricted access is required, **When** it is used, **Then** the access follows an explicit governed policy path and its actor, purpose, time, and affected scope are auditable.

### Edge Cases

- An allow condition matches no records.
- A record matches several allow rules and several deny rules.
- One active role contributes an unrestricted path while another contributes a deny rule.
- A record changes company, owner, organization, or workflow state during an update.
- A parent record is visible while a related child record is outside scope, or vice versa.
- Sorting, pagination, totals, aggregates, reports, and exports could reveal hidden records if scope is applied too late.
- A direct record identifier refers to a hidden or nonexistent record.
- A background job or service identity processes records on behalf of a user.
- Rule evaluation encounters missing context, an unsupported attribute, or an evaluation error.
- A policy change removes access while active sessions or long-running work still use an older authorization state.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Only authorized actors MUST be able to create, update, activate, or deactivate record rules.
- **FR-002**: Each record rule MUST have a stable identity, globally unique normalized code, business purpose, lifecycle status, `Allow` or `Deny` effect, target resource, applicable actions, and a valid business condition.
- **FR-003**: A new rule MUST start as `Inactive` and MUST require successful validation and explicit activation before evaluation.
- **FR-004**: Rule conditions MAY reference approved record attributes and trusted evaluation context but MUST NOT reference unsupported or secret values.
- **FR-005**: Invalid or duplicate rule definitions MUST be rejected without changing active policy.
- **FR-006**: Deactivating a rule MUST stop it from participating in new evaluations without deleting its history or role associations.
- **FR-007**: Only authorized actors MUST be able to add or remove role-rule associations.
- **FR-008**: A role-rule association MUST require an active role and active rule and MUST be unique.
- **FR-009**: An associated record rule MUST participate only when its role is active and contributes the targeted action permission.
- **FR-010**: Record rules MUST NOT grant an action permission that is absent from the effective permissions calculated by `FD-IAM-002`.
- **FR-011**: For each permission-contributing active role, no applicable active rule for a resource and action MUST mean that the record-rule layer adds no restriction to that role's permission path.
- **FR-012**: When applicable allow rules exist for a role path, a record MUST match at least one of those allow rules for that path to contribute access.
- **FR-013**: Allow scopes contributed by all permission-contributing active roles MUST be combined using union semantics.
- **FR-014**: A matching applicable deny rule from any active role MUST override all allow and unrestricted contributions for that record.
- **FR-015**: Effective record scope MUST be applied consistently to lists, searches, direct details, counts, pagination, sorting, aggregation, reports, exports, and background processing.
- **FR-016**: A request outside effective scope MUST NOT disclose protected record data or confirm whether a hidden record exists.
- **FR-017**: Create evaluation MUST use the proposed record state and MUST reject a record that fails all applicable allow rules or matches any applicable deny rule.
- **FR-018**: Update evaluation MUST authorize both the current record and proposed resulting state and MUST reject changes that move the record outside effective scope.
- **FR-019**: Delete evaluation MUST use the existing record state and MUST deny deletion outside effective scope.
- **FR-020**: A bulk mutation containing any target outside effective scope MUST be rejected atomically without changing any target.
- **FR-021**: Company-specific visibility MUST be expressed through record-rule conditions rather than user-company membership.
- **FR-022**: Changing the active-role set MUST add or remove the corresponding rule contributions in the same authorization context.
- **FR-023**: Rule evaluation errors, missing mandatory context, or unavailable policy state MUST fail closed.
- **FR-024**: Removed record access MUST fail closed while active authorization contexts are being reevaluated.
- **FR-025**: Authorized reviewers MUST be able to preview effective scope for a user, active-role set, resource, and action without activating proposed policy changes.
- **FR-026**: Authorized reviewers MUST be able to identify which roles and rules contribute to a record being allowed or denied without revealing unrelated protected data.
- **FR-027**: Preview and explanation capabilities MUST themselves enforce authorization and applicable record scope.
- **FR-028**: Every rule lifecycle and role-rule association change MUST record the actor, event time, action, and relevant before-and-after values.
- **FR-029**: Concurrent administrative updates MUST detect stale state and MUST NOT silently overwrite newer rule or association changes.
- **FR-030**: Rejected administrative or data operations MUST leave policy and business records unchanged and provide an actionable reason without exposing protected data.
- **FR-031**: Service identities and background work MUST use explicit roles, permissions, and record rules; they MUST NOT receive an implicit bypass.
- **FR-032**: Any emergency or administrative unrestricted access MUST be represented by an explicit governed policy path and MUST be fully auditable.

### Requirement-to-Acceptance Traceability

| Requirement | Acceptance Coverage |
|---|---|
| FR-001–FR-006 | AC-US1-01–AC-US1-06 |
| FR-007–FR-010 | AC-US2-01–AC-US2-06 |
| FR-011–FR-014 | AC-US5-01–AC-US5-05 |
| FR-015–FR-016 | AC-US3-02–AC-US3-05 |
| FR-017–FR-020 | AC-US4-01–AC-US4-06 |
| FR-021–FR-022 | AC-US2-01, AC-US2-05, AC-US3-01, AC-US5-06 |
| FR-023–FR-024 | AC-US5-07, AC-US5-08 |
| FR-025–FR-027 | AC-US6-01–AC-US6-04 |
| FR-028–FR-029 | AC-US6-05, AC-US6-06 |
| FR-030 | Negative scenarios across US1–US6 |
| FR-031 | AC-US3-07 |
| FR-032 | AC-US6-07 |

### Business Rules

- **BR-001**: Action permission is evaluated before record scope; a record rule can restrict but never grant an action.
- **BR-002**: Company is a record attribute or trusted business scope, not ownership of the global user identity.
- **BR-003**: Applicable allow rules combine with logical OR; a matching explicit deny rule always wins.
- **BR-004**: For a permission-contributing role path, no applicable rule means unrestricted scope at the record-rule layer; applicable deny rules can still subtract records.
- **BR-005**: If allow rules apply to a role path, records that match none of them are not contributed by that path.
- **BR-006**: Rule effects from roles that do not grant the relevant action permission do not create an allow path.
- **BR-007**: Record scope must be enforced before hidden records influence results, counts, aggregates, exports, or messages.
- **BR-008**: Evaluation uncertainty fails closed rather than falling back to broader access.

### Key Entities

- **Record Rule**: A reusable allow or deny policy with a stable identity, lifecycle, target resource, applicable actions, condition, and business purpose.
- **Rule Condition**: The business expression that compares approved record attributes with trusted evaluation context.
- **Role Record Rule**: The association that makes a record rule applicable through a role that contributes the targeted action permission.
- **Evaluation Context**: Trusted information about the current global user, active roles, action, and approved business scope used during evaluation.
- **Effective Record Scope**: The resulting set of accessible records after permission, allow union, unrestricted contributions, and deny precedence are evaluated.
- **Record-Scope Decision**: An allow or deny outcome with contributing role and rule references available for authorized explanation.
- **Policy Change Event**: A traceable record of rule lifecycle or role-rule association changes.

### Dependencies

- **FD-IAM-001 — User Management**: Supplies the global user identity and account lifecycle state.
- **FD-IAM-002 — Roles, Permissions & Menus**: Supplies active roles and the action permission that must exist before record scope is evaluated.
- **FD-IAM-004 — App Clients & Authentication Policies**: May constrain the trusted business context available for a login channel.
- **FD-IAM-006 — Sessions, Tokens & Trusted Devices**: Carries active-role and authorization state and reevaluates it when policies change.
- **FD-IAM-007 — Personal Access Token**: Requires PAT scope, role-derived permission, and record rules to all permit an operation.
- **Protected business capabilities**: Supply approved resource attributes and enforce effective scope consistently across user-facing and background operations.
- **Cross-domain audit capability**: Persists and exposes security-relevant policy changes to authorized reviewers.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In usability validation, at least 95% of authorized administrators can define, validate, and activate a common company-, owner-, or responsibility-based rule on the first attempt in under five minutes.
- **SC-002**: All tested list, search, detail, count, aggregate, report, export, and background paths expose exactly the same authorized record population for an equivalent context.
- **SC-003**: All tested records matching an explicit deny rule remain inaccessible even when another active role allows them or contributes an unrestricted path.
- **SC-004**: All tested create, update, delete, and bulk operations outside effective scope are rejected without changing protected records or disclosing hidden record details.
- **SC-005**: In access-review validation, at least 95% of authorized reviewers can explain why a sample record is allowed or denied in under one minute.
- **SC-006**: Preview results for approved test datasets match actual enforcement outcomes in 100% of tested cases.
- **SC-007**: Policy changes that remove access stop granting that access within the approved security window, targeted at no more than one minute.
- **SC-008**: All tested rule lifecycle and role-rule changes can be reconstructed with actor, time, action, and relevant before-and-after values.

## Assumptions

- Record rules are associated with roles rather than directly with users; user-specific scope is expressed through approved context attributes evaluated by reusable rules.
- Permission evaluation occurs before record-rule evaluation.
- A permission-contributing role with no applicable active record rule remains unrestricted by this layer, reflecting that record rules are added when narrower data visibility is needed.
- Allow rules use union semantics across active roles, while a matching explicit deny rule has global precedence.
- Create checks evaluate proposed state; update checks evaluate current and proposed state; delete checks evaluate current state.
- Bulk mutations are atomic when any target is outside effective scope.
- Field-level security is a separate concern and is not inferred from record visibility.
- The project constitution is still a scaffold, so no additional ratified governance rules were applied to this draft.
