# Feature Specification: Roles, Permissions & Menus

**FD ID**: `FD-IAM-002`

**Domain**: Administration

**Subdomain**: Identity & Access

**Owner**: GASI:One Identity & Access

**Affected Systems**: GASI:One applications that use shared authorization and navigation

**Source References**: GASI:One wiki migration sources, IAM brainstorming, `FD-IAM-001`, and the prior authentication ERD as a technical reference only

**Feature Branch**: `N/A`

**Created**: 2026-09-25

**Status**: Draft

**Input**: Define role, permission, menu, user-role assignment, multiple active-role, and segregation-of-duties behavior for GASI:One.

## Purpose and Scope

Roles, Permissions & Menus gives authorized administrators a consistent way to
define reusable access roles, associate them with allowed actions and navigation,
assign multiple roles to a global user, and determine the user's effective access
from one or more compatible active roles.

The capability separates authorization from data visibility. A permission answers
whether an action is allowed; record rules in `FD-IAM-003` determine which records
the action may affect. A visible menu helps a user discover an allowed capability
but never grants permission by itself.

### In Scope

- Create, review, update, activate, and deactivate roles.
- Maintain a catalog of permissions representing allowed business actions.
- Associate permissions and menus with roles.
- Maintain hierarchical menu visibility for roles.
- Assign one or more roles to a global user account.
- Activate multiple compatible assigned roles in the same session context.
- Calculate effective permissions and menus as unions across active roles.
- Prevent prohibited role assignments or active-role combinations through segregation-of-duties constraints.
- Explain which roles contribute to a user's effective permissions and menus.
- Preserve history for security-relevant role and assignment changes.

### Out of Scope

- User-account creation, profile maintenance, activation, or deactivation.
- Company and record-level access filtering or allow/deny record-rule evaluation.
- Authentication method, AppClient, identity-provider, or login-channel policy.
- MFA strength, session lifecycle, token issuance, trusted devices, or PAT scopes.
- Application-specific workflow approvals unrelated to role compatibility.
- Technical discovery or registration of application resources and routes.
- Rendering application navigation or implementing protected business operations.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Manage Reusable Roles (Priority: P1)

As an authorized access administrator, I want to create and maintain reusable
roles so that business responsibilities can be assigned consistently instead of
configuring access separately for each user.

**Why this priority**: Roles are the foundation for permission, menu, user-role,
and active-role behavior.

**Independent Test**: Create a uniquely identified inactive role, update its
business description, activate it, and verify a duplicate role identity is
rejected.

**Acceptance Scenarios**:

1. **AC-US1-01** — **Given** an authorized administrator and a unique role code and name, **When** the administrator creates a role, **Then** the role is stored as `Inactive` with a clear business purpose.
2. **AC-US1-02** — **Given** an existing role code, **When** an administrator attempts to create or rename another role to the same normalized code, **Then** the operation is rejected without changing either role.
3. **AC-US1-03** — **Given** an existing role, **When** an authorized administrator updates its name or description, **Then** the new definition is visible while the role's stable identity and history are preserved.
4. **AC-US1-04** — **Given** an inactive role with a valid definition, **When** an authorized administrator activates it, **Then** it becomes available for assignment and activation.
5. **AC-US1-05** — **Given** an active role is assigned to users, **When** an authorized administrator deactivates it, **Then** it can no longer contribute permissions or menus and affected access contexts are reevaluated.
6. **AC-US1-06** — **Given** an actor without role-management authority, **When** the actor attempts to create, update, activate, or deactivate a role, **Then** the operation is denied and role state remains unchanged.

---

### User Story 2 - Manage Permissions, Menus, and Role Associations (Priority: P1)

As an authorized access administrator, I want to maintain permission and menu
definitions and associate them with roles so that users receive coherent access
appropriate to each responsibility.

**Why this priority**: A role has no useful effect until its permissions and
navigation are defined.

**Independent Test**: Create active permission and hierarchical menu definitions,
assign them to an active role, verify its effective definition, then remove or
deactivate one item and confirm it no longer contributes access.

**Acceptance Scenarios**:

1. **AC-US2-01** — **Given** an active permission and role, **When** an authorized administrator associates the permission with the role, **Then** the permission becomes part of that role's allowed actions.
2. **AC-US2-02** — **Given** a permission is removed from a role, **When** effective access is reevaluated, **Then** that role no longer contributes the removed permission.
3. **AC-US2-03** — **Given** an active menu and role, **When** an authorized administrator associates the menu with the role, **Then** the menu becomes eligible for that role's navigation set.
4. **AC-US2-04** — **Given** a role has a menu association but lacks the permission required by the menu's target capability, **When** effective navigation is calculated, **Then** the menu is not actionable and the association does not grant the missing permission.
5. **AC-US2-05** — **Given** a user can access at least one visible child menu, **When** navigation is calculated, **Then** its required parent containers are also visible for navigation.
6. **AC-US2-06** — **Given** an inactive permission or menu, **When** an administrator attempts to add it to a role, **Then** the association is rejected.
7. **AC-US2-07** — **Given** an authorized administrator and a unique permission code describing a business action, **When** the permission is created and activated, **Then** it becomes available for role association without being granted to any role automatically.
8. **AC-US2-08** — **Given** an authorized administrator and valid menu details, **When** a menu is created under an active parent, **Then** its hierarchy, display identity, target capability, and required permission are retained.
9. **AC-US2-09** — **Given** a permission or menu is associated with roles, **When** an authorized administrator deactivates it, **Then** it stops contributing effective access or navigation while its historical associations remain traceable.

---

### User Story 3 - Assign Multiple Roles to a User (Priority: P1)

As an authorized access administrator, I want to assign multiple roles to one
global user account so that the account can perform several compatible
responsibilities without needing separate logins.

**Why this priority**: Multiple role assignment is the agreed operating model for
users who perform more than one responsibility.

**Independent Test**: Assign two compatible active roles to an active user,
remove one assignment, and verify the remaining role continues to be assigned
without changing the user's global identity.

**Acceptance Scenarios**:

1. **AC-US3-01** — **Given** an active global user and two compatible active roles, **When** an authorized administrator assigns both roles, **Then** both assignments are retained for the same user identity.
2. **AC-US3-02** — **Given** a user has multiple assigned roles, **When** one role is removed, **Then** only that assignment stops contributing to future active-role sets.
3. **AC-US3-03** — **Given** a role is inactive, **When** an administrator attempts to assign it to a user, **Then** the assignment is rejected.
4. **AC-US3-04** — **Given** an existing user-role assignment, **When** the same assignment is submitted again, **Then** no duplicate assignment is created.
5. **AC-US3-05** — **Given** an actor without user-role authority, **When** the actor attempts to add or remove an assignment, **Then** the operation is denied and assignments remain unchanged.
6. **AC-US3-06** — **Given** a user has different responsibilities in different company data, **When** roles are assigned, **Then** the assignments remain attached to the global user while company visibility is delegated to applicable record rules.

---

### User Story 4 - Use Multiple Active Roles Together (Priority: P1)

As a user with several compatible assigned roles, I want them to be active in the
same access context so that I can complete my responsibilities without repeatedly
switching roles.

**Why this priority**: This is the central usability decision for GASI:One's
multi-role model.

**Independent Test**: Establish an access context using two compatible assigned
roles and verify the effective permissions and eligible menus are the union of
both roles without adding access from unassigned or inactive roles.

**Acceptance Scenarios**:

1. **AC-US4-01** — **Given** a user has multiple compatible active assigned roles, **When** an access context is established without a narrower role selection, **Then** all compatible assigned roles are active together.
2. **AC-US4-02** — **Given** a user explicitly selects a compatible subset allowed by the login policy, **When** the access context is established, **Then** only the selected assigned roles contribute access.
3. **AC-US4-03** — **Given** multiple roles are active, **When** effective permissions are calculated, **Then** the result is the union of permissions granted by those roles.
4. **AC-US4-04** — **Given** multiple roles are active, **When** effective navigation is calculated, **Then** the result is the union of eligible menus from those roles after permission and menu-state checks.
5. **AC-US4-05** — **Given** a role is not assigned, inactive, or no longer valid for the user, **When** an active-role set is requested, **Then** that role is excluded and contributes no access.
6. **AC-US4-06** — **Given** a permission or menu is contributed by more than one active role, **When** effective access is calculated, **Then** it appears once and remains effective while at least one active role still contributes it.

---

### User Story 5 - Enforce Segregation of Duties (Priority: P1)

As a security administrator, I want to define incompatible role combinations so
that users cannot hold or activate combinations that violate internal controls.

**Why this priority**: Combining access from multiple roles increases risk unless
known conflicts are enforced before access is granted.

**Independent Test**: Configure one static and one dynamic conflict, verify the
static conflict blocks assignment, and verify the dynamic conflict permits
assignment but blocks simultaneous activation.

**Acceptance Scenarios**:

1. **AC-US5-01** — **Given** two roles have a static segregation-of-duties conflict, **When** an administrator attempts to assign the prohibited combination to one user, **Then** the new assignment is rejected with the conflicting roles identified.
2. **AC-US5-02** — **Given** two assigned roles have a dynamic segregation-of-duties conflict, **When** an access context attempts to activate them together, **Then** simultaneous activation is rejected and no conflicting access is granted.
3. **AC-US5-03** — **Given** assigned roles include a dynamic conflict, **When** the user selects a compatible subset, **Then** the access context can be established using that subset.
4. **AC-US5-04** — **Given** a new static constraint would conflict with existing assignments, **When** an administrator attempts to activate the constraint, **Then** activation is blocked until the conflicting assignments are resolved or an approved exception policy applies.
5. **AC-US5-05** — **Given** a new dynamic constraint affects active access contexts, **When** the constraint becomes active, **Then** those contexts are reevaluated and the conflicting role combination stops granting access.

---

### User Story 6 - Explain Effective Access (Priority: P2)

As an authorized administrator or reviewer, I want to understand which roles
produce a user's effective permissions and menus so that I can resolve access
questions and review least-privilege alignment.

**Why this priority**: Union-based access is easier to operate when its sources
are visible and traceable.

**Independent Test**: Review a user with several assigned and active roles and
verify each effective permission and menu identifies its contributing roles,
excluded roles, and relevant compatibility result.

**Acceptance Scenarios**:

1. **AC-US6-01** — **Given** a user has multiple assigned roles, **When** an authorized reviewer opens the access summary, **Then** assigned, inactive, excluded, and active roles are clearly distinguished.
2. **AC-US6-02** — **Given** several active roles contribute access, **When** effective permissions and menus are reviewed, **Then** each item identifies all contributing roles.
3. **AC-US6-03** — **Given** a role is excluded because of state, assignment, login policy, or segregation of duties, **When** the access summary is reviewed, **Then** the exclusion reason is shown without exposing secret authentication data.
4. **AC-US6-04** — **Given** an actor lacks access-review authority, **When** the actor requests another user's access summary, **Then** the summary is not disclosed.
5. **AC-US6-05** — **Given** role, catalog, assignment, or compatibility changes have occurred, **When** an authorized reviewer opens the authorization history, **Then** each event identifies the actor, time, action, and relevant before-and-after values.
6. **AC-US6-06** — **Given** two administrators edit the same role or assignment state, **When** the second administrator saves after the first change succeeds, **Then** the stale update is rejected and the latest state remains intact.

### Edge Cases

- Two roles grant the same permission or menu and one role is later removed.
- A role is deactivated while it is part of active access contexts.
- A permission or menu is deactivated while assigned to several roles.
- A menu is assigned but its required permission is not effective.
- A child menu is visible while its parent is not directly assigned.
- A user has no assigned role or no compatible role available for activation.
- A user explicitly selects roles that are not assigned, inactive, or mutually incompatible.
- A new segregation-of-duties rule conflicts with existing assignments or active contexts.
- Two administrators concurrently change the same role or user-role assignment.
- A role change is accepted but downstream access contexts have not yet confirmed reevaluation; access must fail closed for removed privileges.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Only authorized actors MUST be able to create, update, activate, or deactivate roles.
- **FR-002**: Each role MUST have a stable identity, a globally unique normalized code, a name, a business-purpose description, and a lifecycle status.
- **FR-003**: A new role MUST start as `Inactive` and MUST require explicit activation before assignment or use.
- **FR-004**: A role code conflict MUST be rejected without changing either role.
- **FR-005**: Deactivating a role MUST prevent it from being newly assigned or activated and MUST stop it from contributing effective permissions and menus.
- **FR-006**: Authorized administrators MUST be able to create, update, activate, and deactivate permission catalog entries that identify allowed business actions independently of roles and menus.
- **FR-007**: Permissions MUST be additive grants; absence of a permission from all active roles means the action is not permitted.
- **FR-008**: Authorized administrators MUST be able to add or remove active permissions from a role.
- **FR-009**: Authorized administrators MUST be able to create, update, activate, and deactivate hierarchical menu catalog entries that identify parent-child relationships and any permission required by a target capability.
- **FR-010**: Authorized administrators MUST be able to add or remove active menus from a role.
- **FR-011**: A menu association MUST NOT grant a permission and MUST NOT make a target action available when its required permission is absent.
- **FR-012**: Parent menu containers required to reach at least one eligible child MUST be visible even when only the child is contributed by active roles.
- **FR-013**: Only authorized actors MUST be able to add or remove user-role assignments.
- **FR-014**: One global user account MUST support multiple simultaneous role assignments.
- **FR-015**: Duplicate user-role assignments MUST be prevented.
- **FR-016**: Inactive roles MUST NOT be newly assigned or included in an active-role set.
- **FR-017**: User-role assignments MUST NOT establish company membership; company and record visibility remain governed by record rules.
- **FR-018**: An active-role set MUST contain only active roles currently assigned to the user and permitted by the applicable login policy.
- **FR-019**: Unless a narrower compatible subset is explicitly selected or required by login policy, all compatible active assigned roles MUST be active together.
- **FR-020**: Effective permissions MUST be the deduplicated union of permissions granted by all active roles.
- **FR-021**: Effective menus MUST be the deduplicated union of eligible menus contributed by all active roles after menu state, hierarchy, and required-permission checks.
- **FR-022**: Removing one contributing role MUST NOT remove an effective permission or menu that another active role still contributes.
- **FR-023**: The system MUST support static segregation-of-duties constraints that prohibit specified roles from being assigned to the same user.
- **FR-024**: The system MUST support dynamic segregation-of-duties constraints that allow specified roles to be assigned but prohibit them from being active together.
- **FR-025**: A segregation-of-duties violation MUST fail closed and MUST identify the conflicting roles and applicable constraint to an authorized actor.
- **FR-026**: Activating a new static constraint MUST be blocked while unresolved existing assignments violate it unless an explicitly governed exception exists.
- **FR-027**: Role, permission, menu, assignment, or compatibility changes that remove access MUST trigger timely reevaluation of affected active access contexts.
- **FR-028**: Authorized reviewers MUST be able to distinguish assigned, active, inactive, and excluded roles and see which roles contribute each effective permission and menu.
- **FR-029**: Every role lifecycle change, role-permission change, role-menu change, user-role assignment change, and segregation-of-duties change MUST record the actor, event time, action, and relevant before-and-after values.
- **FR-030**: Concurrent administrative updates MUST detect stale state and MUST NOT silently overwrite a newer role or assignment change.
- **FR-031**: Rejected operations MUST leave authorization state unchanged and provide an actionable reason without disclosing credential, token, or other secret values.
- **FR-032**: Removed access MUST fail closed while downstream access contexts are being reevaluated.
- **FR-033**: Permission and menu entries MUST have stable identities and globally unique normalized codes within their respective catalogs.
- **FR-034**: Deactivating a permission or menu MUST stop it from contributing effective access or navigation without deleting its historical associations.

### Requirement-to-Acceptance Traceability

| Requirement | Acceptance Coverage |
|---|---|
| FR-001–FR-005 | AC-US1-01–AC-US1-06 |
| FR-006–FR-008 | AC-US2-01, AC-US2-02, AC-US2-06, AC-US2-07, AC-US2-09 |
| FR-009–FR-012 | AC-US2-03–AC-US2-06, AC-US2-08, AC-US2-09 |
| FR-013–FR-017 | AC-US3-01–AC-US3-06 |
| FR-018–FR-022 | AC-US4-01–AC-US4-06 |
| FR-023–FR-026 | AC-US5-01–AC-US5-04 |
| FR-027 | AC-US1-05, AC-US5-05 |
| FR-028 | AC-US6-01–AC-US6-03 |
| FR-029 | AC-US6-05 |
| FR-030 | AC-US6-06 |
| FR-031 | Negative scenarios across US1–US6 |
| FR-032 | AC-US1-05, AC-US5-05 |
| FR-033–FR-034 | AC-US2-07–AC-US2-09 |

### Business Rules

- **BR-001**: Roles and permissions are global authorization definitions; company-specific visibility is not encoded as user membership in this FD.
- **BR-002**: Permission grants are additive across active roles. Explicit record-level deny behavior belongs to `FD-IAM-003` and is evaluated after action permission.
- **BR-003**: Menu visibility supports navigation only. It is never a security boundary and never grants an action permission.
- **BR-004**: Multiple compatible assigned roles may be active together so users do not need to switch repeatedly between responsibilities.
- **BR-005**: Static segregation of duties constrains assignment; dynamic segregation of duties constrains simultaneous activation.
- **BR-006**: A user with no valid active role has no role-derived permissions or actionable menus.
- **BR-007**: A role may aggregate many permissions and menus, and the same permission or menu may be contributed by many roles.

### Key Entities

- **Role**: A reusable business responsibility with a stable identity, unique code, purpose, status, permissions, menus, and compatibility constraints.
- **Permission**: An allowed business action that can be contributed by one or more active roles.
- **Menu**: A hierarchical navigation item that may point to a capability and may require an effective permission before it is actionable.
- **Role Permission**: The association that makes a permission part of a role's authorization definition.
- **Role Menu**: The association that makes a menu eligible for a role's navigation definition.
- **User Role Assignment**: The association between a global user account and a role; a user may have several simultaneous assignments.
- **Active Role Set**: The compatible subset of currently assigned active roles used to calculate one access context.
- **Segregation-of-Duties Constraint**: A static or dynamic rule identifying a prohibited role combination and the point at which it is enforced.
- **Authorization Change Event**: A traceable record of security-relevant definition, assignment, compatibility, or lifecycle changes.

### Dependencies

- **FD-IAM-001 — User Management**: Supplies the global account and lifecycle status required for assignment and activation.
- **FD-IAM-003 — Record-Level Access Control**: Restricts which records a permitted action may view or change and applies explicit deny behavior.
- **FD-IAM-004 — App Clients & Authentication Policies**: May restrict which assigned roles are eligible or selectable in a particular login context.
- **FD-IAM-005 — MFA & Account Recovery**: Applies the strongest required authentication assurance among the active roles.
- **FD-IAM-006 — Sessions, Tokens & Trusted Devices**: Carries and reevaluates the active-role set when authorization changes.
- **FD-IAM-007 — Personal Access Token**: Intersects PAT scope with the user's currently valid role-derived permissions.
- **Cross-domain audit capability**: Persists and exposes security-relevant authorization changes to authorized reviewers.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In usability validation, at least 95% of authorized administrators can create and activate a role with its initial permissions and menus on the first attempt in under five minutes.
- **SC-002**: All tested users with multiple compatible assigned roles receive the deduplicated union of the expected permissions and eligible menus without switching roles.
- **SC-003**: All tested unassigned, inactive, or incompatible roles are excluded from active-role sets and contribute no access.
- **SC-004**: All tested static segregation-of-duties conflicts are blocked at assignment, and all tested dynamic conflicts are blocked at simultaneous activation.
- **SC-005**: All tested menu paths require the appropriate effective permission; menu association alone never enables a protected action.
- **SC-006**: In access-review validation, at least 95% of authorized reviewers can identify the role source of a permission or menu and explain an excluded role in under one minute.
- **SC-007**: Authorization changes that remove access stop granting that access within the approved security window, targeted at no more than one minute.
- **SC-008**: All tested role, association, assignment, and compatibility changes can be reconstructed with actor, time, action, and relevant before-and-after values.

## Assumptions

- Roles, permissions, and menu definitions are global and reusable across GASI:One; record rules provide company and data scope.
- Permission semantics are allow-only in this FD: an action is denied when no active role grants it. Explicit deny semantics are reserved for record rules.
- All compatible active assigned roles are selected by default to avoid repeated role switching; an AppClient policy or the user may select a narrower compatible subset.
- Direct user-permission grants are excluded; exceptional access is represented by a dedicated role so it remains reviewable and reusable.
- Static and dynamic segregation-of-duties constraints are both required. Exception approval workflow is outside this FD, but any accepted exception must be explicitly governed and auditable.
- Menu catalog content and permission catalog content are supplied by the affected GASI:One capabilities; this FD governs their authorization relationships rather than technical discovery.
- The project constitution is still a scaffold, so no additional ratified governance rules were applied to this draft.
