# Resource JSON Contract

Baseline source: normalizer, validator, README, and tests from `gasi.one.cli`
commit `fcc740631b838aed241d66619ebc9a69e853e263`.

Input may be an object with a `resources` array, one resource object, or an
array of resource objects. The recommended form is:

```json
{
  "resources": [
    {
      "name": "Example",
      "pluginName": "example",
      "package": "gasi.one.plugins.example.resource",
      "mode": "crud",
      "table": "example_resources",
      "endpoint": "/examples",
      "fields": [
        { "name": "code", "type": "string", "required": true }
      ]
    }
  ]
}
```

## Resource properties

| Property | Required | Contract |
|---|---:|---|
| `name` | Yes | Resource name; alias `entityName`; normalized to PascalCase |
| `package` | Yes | Must start with `gasi.one` and use lowercase Java segments |
| `fields` | Yes | At least one field |
| `pluginName` | No | Lowercase kebab-case; explicit value recommended for plugins |
| `mode` | No | `crud`, `read`, or `embed`; default `crud` |
| `table` | No | Defaults to plural snake_case of the name |
| `endpoint` | No | Defaults to plural kebab-case of the name |
| `lookup` | No | Default `false`; unsupported for `embed`/nested routes |
| `reference` | No | Exposes a cross-plugin validation/label provider |
| `parent` | No | Parent metadata for nested/embed resources |
| `ui` | No | Style, list, lookup, breadcrumb, confirm, create/edit/detail |
| `i18n` | No | Field and validation text by locale |

`mode: read` generates no API/web mutations. `mode: embed` generates child
model/DTO/entity/migration parts managed by a parent, not independent CRUD.

## Field types

| JSON type | Java | TypeScript | Default input |
|---|---|---|---|
| `string` | `String` | `string` | text |
| `text` | `String` | `string` | textarea |
| `integer` | `Integer` | `number` | number |
| `long` | `Long` | `number` | number |
| `decimal` | `BigDecimal` | `number` | number |
| `double` | `Double` | `number` | number |
| `boolean` | `Boolean` | `boolean` | checkbox |
| `date` | `LocalDate` | `string` | date |
| `datetime` | `LocalDateTime` | `string` | datetime-local |
| `instant` | `Instant` | `string` | datetime-local |
| `uuid` | `UUID` | `string` | text |
| `enum` | Java enum | `string` | select |

## Field properties

| Property | Description |
|---|---|
| `name`, `type` | Required; name normalized to camelCase |
| `column` | Column name; defaults to snake_case |
| `length` | String/JPA length; defaults to maxLength or 255 |
| `precision`, `scale` | Decimal; SQL defaults are 19 and 4 |
| `required` | Required validation and non-null column |
| `unique` | Per-field unique hook, DB constraint, and JPA metadata |
| `defaultValue` | Boolean/number/string SQL default |
| `validation` | Supported Bean Validation and Zod rules |
| `dto` | Inclusion in create/update/summary/detail |
| `projection` | Included in default projection; requires summary DTO |
| `sortable` | Enables a public sort path |
| `relation` | Local JPA relation; currently many-to-one |
| `reference` | Cross-plugin scalar reference through a resolver |
| `enum` | Enum definition/use for an enum field |

Supported validations:

```text
email, minLength, maxLength, pattern,
positive, positiveOrZero, negative, negativeOrZero,
past, pastOrPresent, future, futureOrPresent
```

All DTO flags default to `true`:

```json
"dto": {
  "create": true,
  "update": true,
  "summary": true,
  "detail": true
}
```

## Enum

```json
{
  "name": "status",
  "type": "enum",
  "enum": {
    "name": "AccountStatus",
    "type": "string",
    "values": ["ACTIVE", "INACTIVE"]
  }
}
```

`enum.type` accepts `ordinal` or `string` and defaults to `ordinal`. Use
`string` when database/wire values must remain stable if enum ordering
changes. If `enum.package` points to an existing enum, the generator imports
it instead of creating a new enum file.

## Supported UI properties

| Property | Purpose |
|---|---|
| `ui.resource` | `standard` or `simple`; default `standard` |
| `ui.list.pageSize` | Default page size; default 10 |
| `ui.list.searchFields` | Public fields used by global search |
| `ui.list.defaultSort` | Array of `{field, direction: ASC|DESC}` |
| `ui.lookup.*` | Page size, display/search/label fields, default sort |
| `ui.breadcrumb.labelFields` | Breadcrumb label from the detail response |
| `ui.confirm.labelFields` | Record label in confirmation dialogs |
| `ui.create/edit/detail` | Layout `rows`, `sections`, `children`, or `inherit` |

A row layout has 1–4 `columns`; the total field `span` must not exceed the
column count. The validator ensures fields are available in the relevant page
DTO. A child layout must reference a valid embed or nested resource.

`projection: true` selects columns visible by default. Other summary fields
remain available as hidden columns. The web sends visible fields through
`QueryRequest.fields`.

## Lookup, reference, and relation

- `lookup: true`: provides a choice endpoint/preset for the UI.
- Resource `reference.expose: true`: provides a
  `ResourceReferenceProvider` for existence checks and label lookup.
- Field `reference`: stores an internal scalar `Long` and exposes an encoded
  public `String`, without a cross-plugin foreign key.
- `relation`: for JPA relations safe within the same persistence boundary.

Do not use a cross-plugin relation solely to obtain a label. Use a reference so
the consumer does not import the owner's model, entity, repository, or service.

## Verified limitations

- No configuration switch disables only DELETE.
- No domain-action endpoint declaration such as activate/approve.
- Unknown keys are dropped by the normalizer.
- At the baseline commit, `ui.customizationModule` is neither normalized nor
  generated. It is a gap, not an available contract.
