# gasi.one.cli Commands

Baseline source: `gasi.one.cli` commit
`fcc740631b838aed241d66619ebc9a69e853e263`.

## Available domains and actions

| Domain | `validate` | `plan` | `sync` | `clean` | Additional actions |
|---|---:|---:|---:|---:|---|
| `resource` | Yes | Yes | Yes | Yes | — |
| `plugin` | Yes | Yes | Yes | Yes | `package`, `install` |
| `contract` | Yes | Yes | Yes | Yes | API target only |

There is no `generate` command and no combined `api,web` target. Run API and
web targets as separate invocations.

## Command forms

```text
gasi-one resource validate -f <resources.json>
gasi-one resource plan     -f <resources.json> -o <output-dir> --target api|web
gasi-one resource sync     -f <resources.json> -o <output-dir> --target api|web
gasi-one resource clean    -f <resources.json> -o <output-dir> --target api|web

gasi-one plugin validate   -f <plugin.json>
gasi-one plugin plan       -f <plugin.json> -o <output-dir> --target api|web
gasi-one plugin sync       -f <plugin.json> -o <output-dir> --target api|web
gasi-one plugin clean      -f <plugin.json> -o <output-dir> --target api|web

gasi-one contract validate -f <contract.json>
gasi-one contract plan     -f <contract.json> -o <output-dir> --target api
gasi-one contract sync     -f <contract.json> -o <output-dir> --target api
gasi-one contract clean    -f <contract.json> -o <output-dir> --target api
```

Local repository invocation:

```bash
node ./bin/gasi-one.js <domain> <action> ...
```

## Required generation workflow

1. Run `validate` against the source JSON.
2. Run `plan` for the API target.
3. Review every reported create/update/delete operation and migration.
4. Run `plan` for the web target.
5. Review every web file and route.
6. Run `sync` only for an approved plan.
7. Inspect `.gasi-one/manifest.json` and the repository diff.
8. Run the target repository's build and tests.

Example:

```bash
cd ../gasi.one.cli

node ./bin/gasi-one.js plugin validate -f definitions/<domain>/plugin.json
node ./bin/gasi-one.js resource validate -f definitions/<domain>/resources.json

node ./bin/gasi-one.js plugin plan -f definitions/<domain>/plugin.json -o ../gasi.one.api/plugins --target api
node ./bin/gasi-one.js plugin plan -f definitions/<domain>/plugin.json -o ../gasi.one.web/plugins --target web

node ./bin/gasi-one.js resource plan -f definitions/<domain>/resources.json -o ../gasi.one.api/plugins/<plugin>-plugin --target api
node ./bin/gasi-one.js resource plan -f definitions/<domain>/resources.json -o ../gasi.one.web/plugins/<plugin>-plugin --target web
```

After review, replace `plan` with `sync`. Do not change other arguments
without running the plan again.

## Contract scaffold

`contract sync` creates only a Maven shell. Capability interfaces, records,
enums, and providers still require deliberate design. Contract modules are
normally created under `../gasi.one.api/contracts` and added to
`contracts/pom.xml` when they must participate in the reactor build.

```bash
node ./bin/gasi-one.js contract validate -f definitions/<domain>/<name>-contract.json
node ./bin/gasi-one.js contract plan -f definitions/<domain>/<name>-contract.json -o ../gasi.one.api/contracts --target api
node ./bin/gasi-one.js contract sync -f definitions/<domain>/<name>-contract.json -o ../gasi.one.api/contracts --target api
```

## Plugin package and installation

The CLI can package API and web artifacts into a `.one` file, validate its
manifest and checksums, and install it into explicit targets:

```bash
node ./bin/gasi-one.js plugin package \
  -f definitions/<domain>/plugin.json \
  --api-dir ../gasi.one.api/plugins/<plugin>-plugin/target/runtime-plugin \
  --web-file ../gasi.one.web/platform-app/public/plugins/<bundle>.umd.js \
  -o ../gasi.one.api/plugins/<plugin>-plugin/target/<plugin>.one

node ./bin/gasi-one.js plugin install \
  -f <plugin>.one \
  --api-target ../gasi.one.api/runtime-plugins \
  --web-target ../gasi.one.web/platform-app/public/plugins
```

## Repository verification commands

```bash
cd ../gasi.one.cli
npm test
npm run validate:example
npm run plan:example:api
npm run plan:example:web
```

Smoke scripts run `sync` against example output and therefore perform writes.
Use them only with a disposable or explicitly approved target.

## `clean` warning

`clean` destructively removes files recorded in the manifest:

- generated source owned by the resource;
- migrations owned by the resource;
- resource-owned i18n keys and i18n files that become empty;
- empty generated directories.

Do not include `clean` in the normal implementation workflow. Always target
an exact output directory and review the manifest scope first.
