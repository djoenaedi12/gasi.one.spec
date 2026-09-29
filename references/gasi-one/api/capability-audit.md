# API Capability and Audit

Baseline source: `core-api` contracts at `gasi.one.api` commit
`88417135daeeb2b10c2df3fb8c23d345b734849c`.

## Cross-plugin capabilities

A provider implements:

```java
interface CapabilityProvider<T> {
    String capabilityId();
    Class<T> capabilityType();
    T capability();
}
```

A consumer uses:

```java
interface CapabilityRegistry {
    <T> Optional<T> find(String capabilityId, Class<T> type);
    <T> T require(String capabilityId, Class<T> type);
}
```

`require` throws `CapabilityNotFoundException` when no provider is registered.
Use `require` for mandatory dependencies and `find` only when the specification
defines an optional degradation path.

## Capability design rules

- Stable interfaces, records, enums, and value objects belong in a contract
  module.
- The provider plugin implements the contract and registers its provider.
- Consumers depend on the contract, not the provider implementation JAR.
- Capability IDs must be stable, namespaced, and documented.
- Payloads must not expose JPA entities, repositories, internal Spring beans,
  or secrets.
- Define behavior for absent providers, unavailability, timeouts, and partial
  failures; security decisions must fail closed.
- Provide a batch contract when per-row calls would cause N+1 behavior.

Use `ResourceReferenceProvider` for lightweight existence/label lookups. Use a
capability contract for business operations such as eligibility, revocation, or
policy evaluation.

## Audit annotations

`@AuditResource` on a service class accepts:

```text
module
resourceType (optional)
auditActions (default CREATE, UPDATE, DELETE)
alwaysLog (default false)
```

`@AuditAction` on a method accepts:

```text
action
module (optional)
description (optional)
```

`AuditLogService.log(AuditLogEntry)` is available for explicit events that are
not covered by CRUD interception.

## Baseline audit payload

`AuditLogEntry` contains only:

```text
action, module, resourceType, resourceId, description
```

`AuditLogExtension` can only select a module and resolve a description:

```text
supportedModule()
resolveDescription(action, resourceType, resourceId)
```

Automatic interception, persistence, query/review APIs, and audit-storage
lifecycle are owned by the audit plugin; `core-api` only provides contracts.

## Gaps a plan must record

The baseline payload does not include:

- a stable actor ID;
- an explicit occurred-at value in the entry contract;
- the set of changed field names;
- an idempotency/event identity.

When a specification requires this metadata, the audit owner must extend the
contract/interceptor. Do not work around the gap by placing before/after values
or sensitive data in `description`, and do not create a feature-local audit
table.

## Minimum integration tests

- The capability provider is registered with the correct ID and type.
- A mandatory dependency fails startup/operation explicitly when absent.
- A security capability fails closed when unavailable.
- One successful operation emits exactly one audit event.
- Rejected operations and no-ops are not recorded as successful transitions.
- Audit data contains no credentials, tokens, secrets, or prohibited change
  values.
