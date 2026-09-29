# Web Query and Data-Hook Contract

Baseline sources: `api.types.ts`, `createBaseService`, `createBaseHooks`, and
resource documentation at `gasi.one.web` commit
`cce0c54cdc1f7e8a0fd2e4b42123a034720bf6b8`.

## Request and response types

```ts
type SearchRequest = {
  filter?: GenericFilter;
  sorts?: SortOrder[];
  fields?: string[];
  page?: number;
  size?: number;
};
```

```ts
type PageResult<T> = {
  content: T[];
  page: number;
  size: number;
  totalElements: number;
  totalPages: number;
};
```

The web `ApiResponse<T>` mirrors `success`, optional `code`, `message`, `data`,
optional `errors`, and optional `timestamp`. An error item contains optional
`code`/`field` and a `message`.

## Filters

```ts
type SimpleFilter = {
  type: 'simple';
  field: string;
  operator: FilterOperator;
  value?: unknown;
};

type AndFilter = { type: 'and'; filters: GenericFilter[] };
type OrFilter = { type: 'or'; filters: GenericFilter[] };
```

Operators must match the API:

```text
EQUALS, NOT_EQUALS,
GREATER_THAN, GREATER_THAN_OR_EQUALS,
LESS_THAN, LESS_THAN_OR_EQUALS,
LIKE, IN, IS_NULL, IS_NOT_NULL
```

Sorting uses `{field, direction: 'ASC' | 'DESC'}`.

## Base service

`createBaseService(basePath)` maps:

| Method | HTTP |
|---|---|
| `list(request)` | `POST <basePath>/query/list` |
| `page(request)` | `POST <basePath>/query/page` |
| `lookupPage(request)` | `POST <basePath>/lookup/query/page` |
| `detail(id)` | `GET <basePath>/{id}` |
| `create(data)` | `POST <basePath>` |
| `update(id, data)` | `PUT <basePath>/{id}` |
| `delete(id)` | `DELETE <basePath>/{id}` |

The shared Axios client adds a base URL that defaults to `/platform-app`.
Extend a local service for additional domain endpoints; do not change the
framework base service for one feature.

## Base hooks and query keys

`createBaseHooks(entityKey, service)` produces:

```text
queryKeys
useList
usePage
useLookupPage
useDetail
useCreate
useUpdate
useDelete
```

Stable keys:

```text
[entityKey]
[entityKey, 'list', request]
[entityKey, 'page', request]
[entityKey, 'lookup', 'page', request]
[entityKey, 'detail', id]
```

Create/delete invalidates the entire resource key. Update invalidates the
entire resource key and the related detail ID. Custom actions must perform
equivalent invalidation through `queryKeys`, especially after state changes.

## Permission helper

`useResourcePermissions(resource)` provides:

```text
canRead, canCreate, canUpdate, canDelete, canDownload, canUpload, hasAction
```

The helper creates `${resource}:${action}`. It controls only the UI experience;
the server must still reject unauthorized requests.

## Consumer rules

- Use server-side paging for large administrative lists.
- Send `fields` according to the required projection/columns; do not retrieve
  full detail for every row.
- Do not store raw secrets in the TanStack Query cache.
- Handle 409 with reload/review, not silent overwrite.
- Distinguish loading, authoritative empty state, unavailable dependency, and
  error.
- For an idempotent action, a no-op response must not appear as a second
  transition or cause misleading invalidation/events.
