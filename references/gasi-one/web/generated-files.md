# Web Output and File Ownership

Baseline sources: `gasi.one.cli` templates/targets at commit
`fcc740631b838aed241d66619ebc9a69e853e263` and the `gasi.one.web` runtime at
commit `cce0c54cdc1f7e8a0fd2e4b42123a034720bf6b8`.

## Plugin shell

```text
plugins/<plugin>-plugin/
├── package.json
├── tsconfig.json
├── vite.config.ts
└── src/
    ├── index.ts
    └── routes.ts
```

The generated `package.json` only contains the `build: vite build` script.
`src/index.ts` registers the plugin and its route/lookup extensions.
`src/routes.ts` is the resource registry and is also generator-owned.

## CRUD resource

```text
src/features/<plural-resource>/
├── index.ts
├── components/<Resource>Form.tsx
├── hooks/<resource>.hooks.ts
├── i18n/index.ts
├── i18n/locales/<locale>.ts
├── lookups/<resource>.lookup.ts
├── pages/<Resource>ListPage.tsx
├── pages/<Resource>CreatePage.tsx
├── pages/<Resource>EditPage.tsx
├── pages/<Resource>DetailPage.tsx
├── routes/<resource>.routes.tsx
├── schemas/<resource>.schema.ts
├── services/<resource>.service.ts
└── types/<resource>.types.ts
```

Optional files depend on `lookup`, parent/nested configuration, `read` mode,
and layout.

## Generated type contract

```text
Summary: id + summary fields
Detail: id + version + detail fields
CreateRequest: create fields
UpdateRequest: version + update fields
```

In the baseline, generated `Summary`/`Detail` types do not automatically include
all API base metadata such as `createdAt`, `updatedAt`, `createdBy`, and
`updatedBy`. If the UI needs this metadata, extend the type in a custom file or
improve the generator; do not edit the generated type directly.

Create/update Zod schemas are generated from the JSON rules. The update schema
always contains an integer `version`. Services and hooks wrap the base framework.

## Ownership

The source files above and the plugin's `src/routes.ts` are recorded in
`.gasi-one/manifest.json` with the overwrite strategy. i18n is merged by key.
Consequently:

- do not directly edit generated pages, forms, types, schemas, services, hooks,
  routes, indexes, or locale files;
- use JSON for standard changes;
- use `registerResourceCustom` and files in non-generated paths for custom UI;
- use a local custom service for domain endpoints;
- always review `resource plan --target web` and the manifest before sync.

## Verification

```bash
cd ../gasi.one.web
npm run build -w plugins/<plugin>-plugin
npm run build
npm run lint
npm test
```

After building, confirm that the bundle is listed in the host manifest, the
plugin reaches `started`, its routes are reachable, permissions match, and
requests use the correct proxy/base URL.
