# GASI:One Framework Reference

This directory contains a verified baseline of framework contracts taken from
the implementation repositories. It allows specifications, plans, and tasks to
retain a reliable reference when `gasi.one.api`, `gasi.one.web`, or
`gasi.one.cli` is not available in the workspace.

These documents do not replace source code or release artifacts. When the
repositories are available, source code at a newer commit remains the technical
authority, and this baseline must be refreshed before it is used for
implementation decisions.

## Start here

1. Read [baseline.md](./baseline.md) for the versions and commits inspected.
2. For CRUD planning, read [CLI commands](./cli/commands.md),
   [resource schema](./cli/resource-schema.md), and
   [generated files](./cli/generated-files.md).
3. For backend customization, read the [API plugin contract](./api/plugin-contract.md)
   and [CRUD hooks](./api/crud-hooks.md).
4. For frontend customization, read the [web plugin contract](./web/plugin-contract.md)
   and [resource customization](./web/resource-customization.md).
5. Review the [cross-repository contracts](./cross-repository-contracts.md)
   before defining APIs, permissions, IDs, queries, errors, or file ownership.

## Index

| Area | Document | Primary purpose |
|---|---|---|
| Baseline | [baseline.md](./baseline.md) | Commits, versions, capture date, and freshness rules |
| CLI | [commands.md](./cli/commands.md) | Commands that actually exist and their safe workflow |
| CLI | [resource-schema.md](./cli/resource-schema.md) | Resource JSON contract and generator capabilities |
| CLI | [generated-files.md](./cli/generated-files.md) | Outputs, manifest, migrations, and write strategies |
| CLI | [extension-points.md](./cli/extension-points.md) | Generator boundaries and hook/custom requirements |
| API | [plugin-contract.md](./api/plugin-contract.md) | Plugin structure, dependencies, IDs, migrations, and builds |
| API | [crud-hooks.md](./api/crud-hooks.md) | CRUD extension-point signatures and ordering |
| API | [response-errors.md](./api/response-errors.md) | Response envelope, pagination, queries, and errors |
| API | [capability-audit.md](./api/capability-audit.md) | Cross-plugin contracts and audit boundaries |
| Web | [plugin-contract.md](./web/plugin-contract.md) | Plugin runtime, routes, permissions, builds, and manifest |
| Web | [resource-customization.md](./web/resource-customization.md) | Resource customization registry and slots |
| Web | [query-contract.md](./web/query-contract.md) | Services, hooks, filters, sorting, and cache keys |
| Web | [generated-files.md](./web/generated-files.md) | Generated web files and ownership |
| Integration | [cross-repository-contracts.md](./cross-repository-contracts.md) | JSON → API → web → database mapping |

## Usage rules

- Record the baseline commit in `plan.md` whenever a decision depends on a
  framework or generator capability.
- Mark a claim as **verified** only when it appears in these documents or has
  been checked again against the source repository.
- Mark unsupported requirements as a **gap**, not as an assumed generator
  capability.
- Do not copy framework implementation into a feature plugin.
- Do not edit output owned by `.gasi-one/manifest.json`.
- After updating a framework repository, reverify the affected sections and
  update the baseline commit and date.

## Confidence classification

These documents use the following terms:

- **Verified**: inspected directly in source, documentation, or tests at the
  baseline commit.
- **Intended**: stated by documentation but not proven by execution during the
  baseline capture.
- **Gap**: not supported by the inspected public contract.
- **Owned by another FD/plugin**: outside the responsibility of the framework
  or the feature currently being planned.
