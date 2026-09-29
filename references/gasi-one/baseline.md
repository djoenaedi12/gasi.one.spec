# GASI:One Framework Baseline

## Inspected snapshot

This baseline was captured on **2026-09-29** from three sibling repositories on
the `main` branch. None of the commits below had a release tag detectable by
`git describe`; therefore, the full commit hash identifies the snapshot.

| Repository | Origin | Commit | Commit time |
|---|---|---|---|
| `gasi.one.api` | `https://github.com/djoenaedi12/gasi.one.api.git` | `88417135daeeb2b10c2df3fb8c23d345b734849c` | 2026-09-29 12:33:04 +07:00 |
| `gasi.one.web` | `https://github.com/djoenaedi12/gasi.one.web.git` | `cce0c54cdc1f7e8a0fd2e4b42123a034720bf6b8` | 2026-09-24 14:16:41 +07:00 |
| `gasi.one.cli` | `https://github.com/djoenaedi12/gasi.one.cli.git` | `fcc740631b838aed241d66619ebc9a69e853e263` | 2026-09-24 14:16:05 +07:00 |

The repositories had clean working trees when inspected. This statement only
describes their state at capture time and does not guarantee their subsequent
state.

## Platform and toolchain versions

### API

| Component | Verified version |
|---|---|
| Maven platform | `gasi.one:modular-app:1.0.0` |
| Java target | 25 |
| Spring Boot parent | 4.1.1 |
| PF4J | 3.15.0 |
| PF4J Spring | 0.10.0 |
| MapStruct | 1.6.3 |
| JaCoCo | 0.8.15 |
| Runtime ID | Hypersistence TSID 2.1.4 and Sqids 0.1.0 |

The target database runtime is MariaDB with Flyway. This baseline does not pin
the MariaDB server version; deployment documentation must record it.

### Web

| Component | Verified version/range |
|---|---|
| Platform package | 1.0.0 |
| React | ^19.2.0 |
| TypeScript | ~5.9.3 |
| Vite | ^7.3.1 |
| React Router | ^7.13.1 |
| TanStack Query | ^5.90.21 |
| TanStack Table | ^8.21.3 |
| React Hook Form | ^7.71.2 |
| Zod | ^4.3.6 |
| Tailwind CSS | ^4.2.1 |

The documented web prerequisites are Node.js 20.19+ or 22.12+.

### CLI

| Component | Verified version/range |
|---|---|
| `gasi-one` package | 0.1.0 |
| Node.js engine | >=18 |
| Test runner | `node --test` through `npm test` |

### Local toolchain at capture time

| Tool | Version |
|---|---|
| OpenJDK | 25.0.2 LTS |
| Maven | 3.6.3 |
| Node.js | 22.23.2 |
| npm | 10.9.8 |
| Git | 2.34.1 |

## Sources inspected

- API: root and module POMs, `core-api`/`core-starter` READMEs, public
  contracts, base controller/service, hook registry, exception handler, and
  plugin bootstrap/configuration.
- Web: root/workspace package files, plugin registry/loader, route contract,
  resource customization registry, base services/hooks, and plugin/resource
  hook documentation.
- CLI: README, command dispatcher, resource/plugin/contract normalizers and
  validators, target builders, templates, manifest/writer/cleaner, and
  regression tests.

## Verification results at capture time

| Repository/module | Command | Result |
|---|---|---|
| `gasi.one.cli` | `npm test` | 22 passed, 0 failed |
| `gasi.one.web` | `npm test` | 7 passed, 0 failed |
| `gasi.one.api` | `mvn -pl core-api,core-starter test` | 15 passed, 0 failed; reactor succeeded |

These tests verify foundational contracts and runtime behavior in the snapshot;
they do not verify a particular feature plugin or full integration with the
database or Authentication.

## Freshness rules

Reverify this baseline when any of the following occurs:

- a source repository commit changes;
- a major or minor version of the platform, Spring Boot, React, Vite, or CLI
  changes;
- the resource JSON gains an undocumented key;
- hook, capability, or resource-customization signatures change;
- generator templates or write strategies change;
- the error envelope, query/filter contract, permission format, or ID codec
  changes.

When repositories are unavailable, this baseline may be used for planning with
the label **historical baseline**. A `sync`, custom implementation, or high-risk
schema change still requires verification against the repository or release
that will actually be used.

## Known differences

FD plans created before this snapshot may still mention Spring Boot 4.0.3. The
API source at the baseline commit uses **4.1.1**. Every plan must be aligned or
explicitly state that it targets a different baseline.
