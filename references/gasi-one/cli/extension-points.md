# Generator Boundaries and Extension Points

Baseline source: `gasi.one.cli` commit
`fcc740631b838aed241d66619ebc9a69e853e263`, with runtime contracts from the
API and web commits recorded in the [baseline](../baseline.md).

## Approach selection order

1. **Generator configuration** when the requirement can be represented in JSON.
2. **Framework hook/extension** when the requirement is local to a resource and
   a suitable extension point exists.
3. **Custom implementation** only when neither of the mechanisms above can
   express or orchestrate the behavior.
4. **Dependency on another FD** when another domain owns the data or behavior.

## Capability matrix

| Requirement | Baseline approach |
|---|---|
| Standard CRUD | Resource JSON and `resource sync` |
| Required/length/pattern/email/time/numeric ranges | Field `validation` |
| Per-field uniqueness | `unique: true`; generated hook plus DB constraint |
| DTO inclusion | `dto.create/update/summary/detail` |
| Default projection/columns | `projection` |
| Search, sorting, paging, layout, lookup | Resource/field `ui` metadata |
| Local many-to-one | `relation` |
| Cross-plugin reference | `reference` plus provider/resolver |
| Domain normalization before validation/save | Ordered custom service hook |
| Reject DELETE for one resource | Custom service hook and remove the UI action |
| Change web columns/filters/row actions | `registerResourceCustom` |
| Domain-action endpoint | Narrow custom controller/service |
| Cross-plugin capability | Contract module plus `CapabilityRegistry` |
| Basic CRUD audit | `@AuditResource` and the audit owner |
| Custom-action audit | `@AuditAction` and the audit owner |

## Targetable backend extension points

- Service hooks for normalization/validation before create/update, response
  enrichment, or delete rejection.
- Controller hooks for concerns genuinely located at the CRUD HTTP boundary.
- Mapper hooks for mapping adjustments supported by the mapper contract.
- Repository hooks for repository concerns that do not replace the persistence
  model.
- `CapabilityProvider`/`CapabilityRegistry` for cross-plugin business
  interaction.
- `ResourceReferenceProvider` for lightweight cross-plugin existence/label
  lookup.
- PF4J `AppExtension`, `FlywayMigrationExtension`, and `I18nExtension`.

Backend signatures are documented in [CRUD hooks](../api/crud-hooks.md).

## Targetable frontend extension points

- `registerResourceCustom` for page replacement, columns, row/header/toolbar/
  bulk/form/detail actions, and filters.
- A local service wrapper for additional domain endpoints.
- Generated `queryKeys` for cache invalidation after custom actions.
- Plugin `onStart` or a module import actually reached by the plugin entry
  point for custom registration.

See [resource customization](../web/resource-customization.md).

## Baseline gaps

### Disable one CRUD operation per resource

The only modes are `crud`, `read`, and `embed`; there is no per-operation
toggle. If create and update remain available but delete is prohibited, enforce
the rule with a backend hook and remove the UI action.

### Domain-action endpoints

CRUD definitions do not generate actions such as activate, approve, submit, or
revoke. Use a small custom implementation while reusing the framework model,
repository port, ID codec, permission, response envelope, versioning, and audit.

### Web custom-module bootstrap

`registerResourceCustom` exists in the web runtime, but the baseline CLI
commit has no `ui.customizationModule` key. The normalizer drops unknown keys,
and the reachable plugin entry/route files are generator output. Until the
generator provides a declarative import or the plugin has a safe non-generated
bootstrap, do not claim that custom registration loads automatically.

### Changed fields in audit

Audit annotations exist, but the baseline audit event does not carry the set of
changed field names. This is an audit-contract gap, not a reason to place
before/after values in the description or create feature-local audit storage.

## Acceptable rationale for custom implementation

A plan must state:

- the generator and hook capabilities that were inspected;
- the exact gap preventing the requirement from being fulfilled;
- that custom files stay outside manifest ownership;
- which framework contracts remain in use;
- which tests prove integration and regeneration safety.
