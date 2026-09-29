# Generator Output and File Ownership

Baseline source: target builders, templates, writer, manifest, cleaner, README,
and tests from `gasi.one.cli` commit
`fcc740631b838aed241d66619ebc9a69e853e263`.

## API plugin shell

`plugin sync --target api` creates a Maven plugin project containing:

```text
<plugin>-plugin/
├── pom.xml
├── src/assembly/runtime-plugin.xml
├── src/main/java/<package>/<Plugin>.java
├── src/main/java/<package>/extension/<Plugin>AppExtension.java
├── src/main/java/<package>/extension/<Plugin>FlywayExtension.java
├── src/main/java/<package>/extension/<Plugin>I18nExtension.java
├── src/main/resources/plugin.properties
└── src/main/resources/i18n/<code>/custom/messages_*.properties
```

Exact names follow the plugin descriptor and baseline templates.

## Web plugin shell

`plugin sync --target web` creates:

```text
<plugin>-plugin/
├── package.json
├── tsconfig.json
├── vite.config.ts
└── src/
    ├── index.ts
    └── routes.ts
```

`src/index.ts` registers plugin metadata, routes, and lookups. Vite emits a UMD
bundle into `platform-app/public/plugins` and externalizes React and shared
GASI packages.

## API resource output

API resource output is written directly into the plugin output:

```text
src/main/java/<package>/
├── presentation/controller/
├── application/dto/
├── application/mapper/
├── application/reference/
├── application/service/
├── domain/model/
├── domain/port/inbound/
├── domain/port/outbound/
├── infrastructure/entity/
├── infrastructure/adapter/
├── infrastructure/mapper/
└── infrastructure/persistence/
```

Depending on mode and metadata, the generator creates controllers, model/entity,
create/update/summary/detail DTOs, mapper, service port/implementation/hook,
repository port/adapter/repository, enums, reference provider, migrations, and
i18n.

## Web resource output

Web resource output is written to:

```text
src/features/<plural-resource>/
├── index.ts
├── components/
├── hooks/
├── i18n/
│   └── locales/
├── lookups/
├── pages/
├── routes/
├── schemas/
├── services/
└── types/
```

`crud` mode generates list/create/edit/detail pages, forms, mutation hooks and
services, schemas, types, routes, i18n, and optional lookup/nested components.
`read` mode generates only read surfaces. The generator also rewrites the
plugin's `src/routes.ts` registry from the supplied resources.

## Migrations

```text
src/main/resources/db/migration/<pluginName>/
├── V<timestamp>__create_<table>.sql
└── V<timestamp>__alter_<table>.sql
```

- Create migrations use the `create-once` strategy.
- A schema snapshot is stored in `.gasi-one/manifest.json`.
- Subsequent changes produce a new alter migration.
- Changes to uniqueness, relations, target tables, or scalar/relation
  conversion require manual review because constraint drop/recreation may be
  incomplete.
- Never edit an applied migration.

## Manifest and write strategies

Each `resource sync` merges state into `.gasi-one/manifest.json`. The manifest
records resources, schema snapshots, generated paths, migrations, i18n,
strategies, and cleanup eligibility.

| File type | Strategy | Cleanup |
|---|---|---:|
| Generated Java/React/source | `overwrite` | Yes |
| Create migration | `create-once` | Yes |
| Alter migration | `alter-on-change` | Yes |
| i18n properties | `merge-properties` | Indirect; keys are managed |
| Manifest | Regenerated/merged state | N/A |

## Custom-file rules

- Do not edit files recorded as generated/overwrite.
- Put hooks, custom action controllers, custom components, tests, and
  integration modules in paths not owned by the manifest.
- Before selecting a custom name/path, inspect the actual `plan` output and
  manifest.
- If a required bootstrap file is always overwritten, improve the generator or
  use an already reachable extension point; do not rely on a fragile post-sync
  patch.
