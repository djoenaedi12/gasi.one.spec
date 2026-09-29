# API CRUD Extension Points

Baseline source: public hook contracts and registry at `gasi.one.api` commit
`88417135daeeb2b10c2df3fb8c23d345b734849c`.

## Registration and ordering

A hook is annotated with:

```java
@ResourceHook(value = "ResourceType", layer = HookLayer.SERVICE)
```

`value` is a resource name or `ResourceHook.ANY_RESOURCE`. The layer follows
`HookLayer`. Hooks for the same resource/layer are ordered using Spring
`@Order` semantics; lower order values run first.

If custom normalization must run before a generated uniqueness hook, set the
order explicitly and prove it with a test. Do not rely on component-scan order.

## Service hook

`ResourceServiceHook<D, CRQ, URQ, SRS, DRS>` provides:

```text
beforeFindById(resourceType, id)
afterFindByIdResponse(resourceType, response, domain)
beforeFindBy(resourceType, filter)
afterFindByResponse(resourceType, response, domain)
beforeFindAll(resourceType, filter, orders)
afterFindAllResponse(resourceType, response, domains)
beforeFindAllPaged(resourceType, page, size, filter, orders)
afterFindAllPagedResponse(resourceType, response, domains)

beforeCreateRequest(resourceType, request)
beforeCreate(resourceType, domain, request)
afterCreate(resourceType, saved, request)
afterCreateResponse(resourceType, response, saved, request)

beforeUpdateRequest(resourceType, id, request)
beforeUpdate(resourceType, domain, request)
afterUpdate(resourceType, saved, request)
afterUpdateResponse(resourceType, response, saved, request)

beforeDeleteRequest(resourceType, id)
beforeDelete(resourceType, id)
afterDelete(resourceType, id)
```

Appropriate uses include:

- normalizing a request before later mapping/validation;
- validating resource business rules;
- enriching a response;
- rejecting deletion with a business exception;
- lightweight side effects with a clearly defined transaction boundary.

Limitations: a service hook cannot add routes, has no return contract for
short-circuiting a save, and cannot produce a no-op update without following
the base-service flow. An idempotent domain action normally requires a narrow
custom service/controller.

## Controller hook

`ResourceControllerHook<CRQ, URQ, SRS, DRS>` provides:

```text
beforeFindByIdRequest / afterFindByIdResponse
beforeFindByRequest / afterFindByResponse
beforeFindAllRequest / afterFindAllResponse
beforeFindAllPagedRequest / afterFindAllPagedResponse
beforeCreateRequest / afterCreateResponse
beforeUpdateRequest / afterUpdateResponse
beforeDeleteRequest / afterDeleteResponse
```

Callbacks receive the encoded ID, request, `ApiResponse`, and
`ResourceRequestContext` as appropriate for the operation. Use them for
resource HTTP-boundary concerns, not to replace a business service or backend
authorization.

Generated lookup endpoints do not run controller hooks in this baseline.

## Mapper hook

`ResourceMapperHook<D, CRQ, URQ, SRS, DRS>` provides:

```text
afterToCreateDomain(domain, request)
afterToUpdateDomain(domain, request)
afterUpdateDomain(domain, request)
afterToSummary(response, domain)
afterToDetail(response, domain)
```

Use it only for mapping/enrichment that cannot be configured. Do not perform
expensive cross-plugin access per row; list/reference enrichment must be
batched.

## Repository hook

`ResourceRepositoryHook<D>` provides transformations/callbacks for:

```text
beforeSave / afterSave
beforeSaveAll / afterSaveAll
beforeFindById / afterFindById
beforeFindBy / afterFindBy
beforeFindAll / afterFindAll
beforeFindAllPaged / afterFindAllPaged
applyRecordRules
beforeDelete / afterDelete
beforeDeleteAllByIds / afterDeleteAllByIds
beforeDeleteAllBy / afterDeleteAllBy
```

Some `before*` methods return a transformed model, ID list, or filter.
`applyRecordRules` is the record-level restriction point, but its permission
and filter design must remain consistent with the policy owner.

## Choosing a layer

| Requirement | Narrowest layer |
|---|---|
| Trim/lowercase before uniqueness check | Service `beforeCreateRequest`/`beforeUpdateRequest` |
| Domain validation after mapping | Service `beforeCreate`/`beforeUpdate` |
| Reject permanent deletion | Service `beforeDeleteRequest` or `beforeDelete` |
| Add a response field available from the domain | Mapper/service response hook |
| Record-level filter | Repository `applyRecordRules` |
| Add an action endpoint | Not a hook; narrow custom controller/service |

## Minimum hook tests

- The hook activates only for its target resource/layer.
- Its `@Order` relative to generated hooks is proven.
- Normalization occurs before uniqueness checking and persistence.
- Exceptions produce the expected HTTP status, code, and field.
- Retries/no-ops cause no double save, duplicate event, or side effect.
- Regeneration does not overwrite custom hook source.
