# Web Resource Customization

Baseline sources: `resourceCustom.ts` and `docs/resource-hooks.md` at
`gasi.one.web` commit `cce0c54cdc1f7e8a0fd2e4b42123a034720bf6b8`.

## Registry

```ts
registerResourceCustom<TSummary>(resourceKey, custom)
getResourceCustom<TSummary>(resourceKey)
hasResourceCustom(resourceKey)
clearResourceCustom(resourceKey)
clearResourceCustoms()
```

`resourceKey` must exactly match the generated resource key. A registration
replaces the previous object; the registry neither merges registrations nor
defines ordering. The resource owner should normally own the registration.

## Available slots

### Page replacement

```text
ListPage, CreatePage, EditPage, DetailPage
```

Use full replacement only when generated surfaces cannot express the workflow.
For column, action, or filter changes, use a narrower slot.

### List

```text
listHeaderActions
listToolbarActions
listFilters
listFilterAction
listActiveFilters
clearListFilters
useListFilters
listColumns
listRowActions
listBulkActions
buildListFilter
```

### Form and detail

```text
formActions
detailActions
```

## Context

List callbacks receive:

```ts
type ResourceListContext = {
  resource: string;
  resourceLabel: string;
  resourcePluralLabel: string;
  params?: Record<string, string | undefined>;
};
```

A row action also receives `item: TSummary`.

## Stateful filters

`useListFilters` is a React hook called by the generated page. Its return
contract is:

```ts
type ResourceListFilterCustom = {
  filters?: ReactNode;
  filterAction?: DataTableAction;
  activeFilters?: DataTableActiveFilter[];
  filter?: GenericFilter;
  clearFilters?: () => void;
};
```

The implementation may use React state/hooks, but it must be called
unconditionally and consistently with the Rules of Hooks. Custom filters are
combined with search according to generated-page logic; verify the resulting
AND/OR structure through tests and actual requests.

Use `buildListFilter(search, context)` when server-side global search must
differ from the generator default.

## Row actions

`listRowActions(defaultActions, context)` must return the final array. Preserve
required default actions, filter prohibited actions, and add domain actions as
needed.

A permission-sensitive action uses the platform permission helper, but the
backend endpoint must enforce the same permission. If an action requires a
`version` absent from the summary, load the latest detail before confirmation.

## Registration bootstrap

Registration must execute when the plugin bundle loads or during `onStart`,
before the page renders. The baseline CLI commit has no
`ui.customizationModule` metadata. Therefore:

- do not create a custom file and assume it executes automatically;
- do not manually inject an import into a generated file that will be
  overwritten;
- use an actually reachable non-generated bootstrap, or enhance the CLI with a
  declarative import contract and regression tests.

## Testing

1. Call `clearResourceCustoms()` during setup/cleanup.
2. Register customization using the target resource key.
3. Render the generated page or directly test pure callbacks.
4. Confirm default behavior remains unless deliberately removed.
5. Test UI visibility and backend authorization separately.
6. Test filter composition, empty state, unavailable dependencies, and retry.
7. Regenerate and confirm custom files remain unchanged and present.
