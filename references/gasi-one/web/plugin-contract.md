# Web Plugin Contract

Baseline sources: `gasi.one.web` commit
`cce0c54cdc1f7e8a0fd2e4b42123a034720bf6b8` and the plugin template at
`gasi.one.cli` commit `fcc740631b838aed241d66619ebc9a69e853e263`.

## Workspace boundaries

```text
core-api       public types and extension points
core-starter   plugin/resource runtime, HTTP client, state, permissions, hooks
core-ui        UI components, DataTable, i18n, toast, utilities
platform-app   React/Vite host and plugin loader
plugins/*      business-feature bundles
```

A feature plugin imports only public exports from shared packages. React, React
Router, and `@gasi/core-*` packages must be externalized from the UMD bundle so
the plugin uses the host singletons.

## PluginDefinition

Main contract:

```text
id: plugin.<name>
name
version
requiresPlatform
startupPolicy: mandatory | optional
description?
dependsOn?
extensions?
onStart?
onStop?
```

Runtime states:

```text
registered → starting → started → stopping → stopped
                         ↘ error
```

The host validates `requiresPlatform` compatibility, dependency ordering and
ranges, consistency between manifest and bundle metadata, and startup policy.
A failed mandatory plugin prevents application rendering; an optional plugin
is logged and skipped.

## Supported extensions

`PluginExtension` may contain:

- `routes` for the route extension point;
- `resources` for resource metadata;
- `menus` for menu entries;
- `lookups` for lookup presets;
- `guard` for an authentication guard;
- `pluginId`, assigned by the registry during startup.

The generated plugin shell currently registers route and lookup extensions.

## Routes and permissions

`RouteDefinition` contains:

```text
path, component, public?, layout?, title?, order?, resource?, action?
```

Standard web actions:

```text
read, create, update, delete, download, upload
```

`resolvePermission` produces `${resource}:${action}` and defaults the action to
`read`. If `resource` is absent, the route permission is undefined. Public
routes must use `public: true`; a protected feature route must not be made
public merely to avoid an Authentication dependency.

Ensure that generated route resources/actions semantically match the backend
permission evaluator. Do not guess capitalization or aliases; inspect the plan
output and the authentication contract used by the deployment.

## Host manifest

An installable plugin entry:

```json
{
  "id": "plugin.catalog",
  "version": "1.0.0",
  "requiresPlatform": ">=1.0.0 & <2.0.0",
  "startupPolicy": "mandatory",
  "url": "/plugins/catalog-plugin-1.0.0.umd.js",
  "dependsOn": []
}
```

Manifest metadata must match the metadata registered by the bundle. A plain URL
is still accepted only for a legacy optional bundle and is not recommended for
new plugins.

## Build and run

```bash
cd ../gasi.one.web
npm install
npm run build -w plugins/<plugin>-plugin
npm run build
npm run lint
npm test
```

The generated plugin shell defines only a local `build` script. Do not claim
that `npm test -w <plugin>` or `npm run lint -w <plugin>` is available unless
the script has actually been added.

Run the host:

```bash
npm run dev
```

The default development proxy sends `/platform-app` to backend port 8080.

## Bundle output

The plugin build emits a UMD bundle at:

```text
platform-app/public/plugins/<plugin>-plugin-<version>.umd.js
```

The root build builds plugins before the platform, then Vite carries the
manifest and bundles into the host distribution.

## Customization rules

- Register custom/resource extensions before the related page renders.
- Do not create a second plugin registry, React instance, or QueryClient.
- Do not edit generated `src/index.ts`/`src/routes.ts` without generator
  support or a safe non-generated bootstrap.
- The backend remains authoritative for permissions and data disclosure.
