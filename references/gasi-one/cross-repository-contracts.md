# Cross-Repository Contracts

This document summarizes mappings that must remain consistent across the JSON
definition, API, web application, and database.

## Primary mapping

| Concern | CLI definition | API/database | Web |
|---|---|---|---|
| Resource name | `name` | Model/DTO/controller name and permission resource | Resource key, type, route, i18n |
| Plugin owner | `pluginName` | Migration/i18n folder and plugin owner | Plugin route/lookup owner |
| Endpoint | `endpoint` | Base `@RequestMapping` | `createBaseService` base path |
| ID | Not configured as a field | Internal TSID `BIGINT`, public encoded `String` | Always a `string` in public requests/responses |
| Version | Base field | `Integer version`, optimistic locking, versioned update request | Sent on update; conflicts must not overwrite |
| Required | `required: true` | Bean Validation and non-null column | Zod/form required rule |
| Length | `length`/`validation.maxLength` | JPA/SQL and Bean Validation | Zod/input limit |
| Unique | `unique: true` | Generated service hook plus database constraint | API field error; not a client-side guarantee |
| Enum | `type: enum` and `enum.type` | Enum class and ordinal/string persistence | Value union/options based on stable wire constants |
| Projection | `projection: true` | Default list/page projection | Visible default columns and request `fields` |
| Query | `ui.list`/field metadata | `QueryRequest`, filters, sorting, maximum page size 100 | `SearchRequest`, server table, query hooks |
| Permission | Generated resource/action | `hasPermission(this, 'ACTION')` | `{resource}:{action}` for routes/buttons |
| Error | Validation/i18n JSON | `ApiResponse.errors[]` | `ApiErrorDetail`, field error, and message |
| File ownership | Generator manifest | Tracked Java/SQL/i18n | Tracked TS/TSX/i18n |

## Source-of-truth flow

```text
spec.md
  → plan.md and contracts
  → plugin.json/resources.json
  → gasi.one.cli validate
  → gasi.one.cli plan --target api|web
  → human review
  → gasi.one.cli sync --target api|web
  → hook/extension/custom file outside manifest ownership
```

The JSON definition is the source of truth for generated CRUD. The
specification remains the source of truth for product behavior. They are not
interchangeable: a generator change does not automatically change a business
requirement, and a business requirement does not justify editing generated
output directly.

## Standard CRUD endpoints

With a base path of `/api/v1/<resources>`, the standard contract is:

| Method/path | Purpose |
|---|---|
| `POST /` | Create |
| `GET /{id}` | Detail |
| `PUT /{id}` | Update with optimistic locking when the DTO is versioned |
| `DELETE /{id}` | Delete in CRUD mode; business rules may reject it through a hook |
| `POST /query/one` | One record selected by a filter |
| `POST /query/list` | Non-paginated list |
| `POST /query/page` | Server-side page |
| `POST /lookup/query/page` | Lookup when `lookup: true` |

Domain-action endpoints such as `/activate` or `/approve` are not standard
CRUD. Use a narrow custom service/controller when hooks cannot implement the
action semantics, while retaining the framework's envelope, encoded IDs,
permissions, versioning, audit, and query invalidation.

## Dependency rules

- Stable public contracts belong in `core-api` or
  `gasi.one.api/contracts/*`.
- Runtime implementation belongs in `core-starter`, `platform-app`, or the
  owning plugin.
- Cross-plugin business interaction uses a contract module and
  `CapabilityRegistry`.
- Lightweight cross-plugin existence/lookup uses
  `ResourceReferenceProvider`/`ResourceReferenceResolver`.
- A consumer must not import another plugin's entity, repository, service
  implementation, or implementation JAR.
- Consumer contract dependencies use Maven `provided` scope; a provider may
  use `compile` when it owns and packages the contract.

## Plan consistency checklist

- The baseline version and commits are recorded.
- Every field has consistent JSON, DTO, database, and web mappings.
- Public IDs are never documented as raw `BIGINT` values.
- Mutable updates carry `version` when optimistic locking is required.
- HTTP errors and codes follow the actual handler.
- API and web permissions use semantically identical resources/actions.
- Custom files remain outside generator-manifest ownership.
- Generator gaps state their reason and use the narrowest extension point or
  custom implementation.
- Dependencies owned by another FD are not duplicated locally.
