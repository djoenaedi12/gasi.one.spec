# API Plugin Contract

Baseline sources: `gasi.one.api` commit
`88417135daeeb2b10c2df3fb8c23d345b734849c` and the plugin template at
`gasi.one.cli` commit `fcc740631b838aed241d66619ebc9a69e853e263`.

## Module boundaries

```text
core-api       public contracts, annotations, base DTO/model, ports, extension points
core-starter   CRUD runtime, JPA, mappers, registries, handlers
contracts/*    stable cross-plugin domain contracts
platform-app   Spring Boot host and plugin bootstrap
plugins/*      business features
```

A feature plugin may depend on `core-api`, `core-starter`, and contract modules.
It must not import another business plugin's implementation.

## Plugin descriptor

Plugin JSON produces the following API metadata:

```text
plugin.id
plugin.class
plugin.version
plugin.provider
plugin.description
plugin.dependencies
plugin.requires
plugin.startup-policy
```

Values are also written to the JAR manifest as `Plugin-Id`, `Plugin-Version`,
`Plugin-Class`, `Plugin-Provider`, `Plugin-Description`, `Plugin-Dependencies`,
and `Plugin-Requires`.

`startupPolicy` accepts `mandatory` or `optional`. Failure of a mandatory plugin
stops platform startup; an optional plugin may be logged and skipped. Reverify
the API runtime default for missing or invalid metadata before relying on it.
New descriptors must state the policy explicitly.

## Dependency contract

- Generated `core-api` and `core-starter` dependencies use `provided` scope.
- Spring Web, Validation, Data JPA, PF4J, MapStruct, and Lombok are also
  supplied by the host according to the template.
- A provider-owned contract may use `compile` scope so its JAR is included in
  `target/runtime-plugin/lib`.
- A consumer uses the contract with `provided` scope and declares PF4J/load
  ordering in the descriptor.
- Do not add another plugin's service/entity/repository implementation as a
  Maven dependency.

## Base model, entity, and DTO

| Contract | Main fields |
|---|---|
| `BaseModel` | `Long id`, audit timestamps/actors, `Integer version`, `LifecycleStatus` |
| `BaseEntity` | Same fields with JPA mapping, runtime TSID, optimistic lock |
| `BaseRequest` | `customFields` |
| `BaseSummaryResponse` | Encoded `String id`, `createdAt` |
| `BaseDetailResponse` | Summary + `updatedAt`, `createdBy`, `updatedBy`, `version`, `customFields` |
| `VersionedRequest` | `Integer getVersion()` |

Generated migrations use `BIGINT` without auto-increment. The internal ID is a
TSID-based `Long`. The public boundary uses an encoded `String` through
`IdCodec`/Sqids. Never expose raw `BIGINT` values in an API contract.

`lifecycleStatus` is base technical metadata. Do not use it as feature business
state without an explicit contract because the generator does not
automatically expose it as a CRUD/UI field.

## Generated plugin extensions

- `AppExtension`: module name/description/version and scanned base package.
- `FlywayMigrationExtension`: one plugin-owned classpath location.
- `I18nExtension`: generated and custom message basenames.

The host combines Flyway locations and message basenames from active plugins.
Resource migrations must live in the folder matching `pluginName`.

## Security

The base controller uses method authorization
`hasPermission(this, 'ACTION')` for actions such as CREATE, READ, UPDATE,
DELETE, and LOOKUP. `core-starter` does not contain the runtime authentication
or permission evaluator; the Authentication plugin must enable method security
and provide the evaluator.

UI visibility is not a substitute for backend authorization.

## Build and package

General checks:

```bash
cd ../gasi.one.api
mvn test
mvn verify
```

For a generated plugin:

```bash
cd ../gasi.one.api/plugins/<plugin>-plugin
mvn test
mvn clean verify
```

The Maven template assembles `target/runtime-plugin` during the package phase.
It contains the plugin JAR and the compile-scope libraries that must ship with
the plugin.

## Planning rules

- Use the plugin generator for the standard shell.
- Use the resource generator for standard CRUD.
- Keep custom code in packages outside manifest-owned resource paths.
- Use contract modules for cross-plugin interfaces/value objects.
- Verify `plugin.id`, Maven artifacts, PF4J dependencies, and capability IDs
  exactly; do not guess names owned by another FD.
