# Adoption in an existing Nuxt project

Read this reference before editing an existing Nuxt application.

## Preflight

1. Confirm that `package.json` contains Nuxt and identify the major version. If it is Nuxt 2, stop and report that this skill covers Nuxt 3 and Nuxt 4 only.
2. Detect whether the project uses Nuxt 4's `app/` directory or Nuxt 3's root frontend directories. Read `nuxt.config.*`, `app.config.*`, `package.json`, the lockfile, scripts, TypeScript configuration, modules, linter, formatter, ignore files, tests, and environment declarations.
3. Inspect app directories, `pages`, `components`, `composables`, `layouts`, `middleware`, `plugins`, `server`, `shared`, and feature code. Locate Pinia stores, data fetching, forms, UI kit, auth boundaries, and client-only code.
4. Run existing checks before changing anything when possible.
5. Create a delta matrix: rule, current state, risk, minimum change, and verification.

Do not begin by moving directories or changing `srcDir`. Gather enough evidence to know what is preserved and what is transformed.

Completion: Nuxt version, directory convention, current ownership boundaries, and baseline checks are known.

## Preservation policy

Preserve by default:

- Nuxt major version and current `srcDir` or `app/` directory convention.
- Public routes, route names, layouts, middleware, and SSR or hybrid rendering behavior.
- Existing app and server runtime boundaries.
- Pinia, VueUse, composables, or another existing state solution.
- `useFetch`, `useAsyncData`, `$fetch`, Axios, an SDK, an ORM, or another existing data client.
- Existing Nitro API handlers, server middleware, server plugins, and auth integration.
- Tailwind, CSS Modules, scoped CSS, UnoCSS, or another styling system.
- shadcn-vue or another existing UI kit.
- Nuxt modules, package manager, lockfile, scripts, and configuration.
- Linter, formatter, test runner, and existing test setup.
- Valid naming conventions and useful directory structure.

The goal is to make runtime and feature boundaries clear, not replace technology. Migrating Nuxt major versions, `srcDir`, modules, state, forms, data fetching, styling, or UI kit requires an explicit request and a separate plan.

Completion: preserved systems and intentional migration targets are explicit before edits begin.

## Nuxt 3 and Nuxt 4 structure

Nuxt 4 new projects place frontend code in `app/`:

```text
app/
├── components/
├── composables/
├── features/
├── layouts/
├── middleware/
├── pages/
├── plugins/
└── utils/
server/
shared/
```

Nuxt 3 projects commonly place the equivalent frontend directories at the project root:

```text
components/
composables/
layouts/
middleware/
pages/
plugins/
utils/
server/
```

Apply these rules:

- Preserve the existing convention during feature work.
- Do not migrate root directories to `app/` as a cleanup task.
- Do not configure `srcDir: '.'` or another compatibility setting unless the project needs it.
- If a Nuxt 3 to Nuxt 4 migration is explicitly requested, read current upgrade documentation and treat it as a separate migration with its own verification.
- Keep `server/` at the project root for both supported versions unless the installed version documents another supported location.
- Keep `shared/` runtime-neutral when it exists. Do not create it only to move one file.

Completion: touched files follow the project's existing Nuxt version and directory convention.

## App and server boundaries

Inventory app and server runtime ownership before editing:

- `app/` or root frontend directories belong to the Nuxt application graph.
- `server/` belongs to Nitro and may use event APIs, private runtime config, and server-only clients.
- `shared/` may be imported by both graphs only when its code is runtime-neutral.
- `app/middleware/` handles navigation. `server/middleware/` handles requests.
- App plugins and Nitro plugins have separate lifecycles and must not be interchanged.
- Browser-only modules stay behind `ClientOnly`, `import.meta.client`, or a focused client lifecycle boundary.
- Private environment values stay outside `runtimeConfig.public` and never enter app components or public composables.

Move only clearly misplaced code. Do not rewrite all composables or plugins as part of a feature migration.

Completion: every touched module has one runtime owner and no private code crosses into the browser graph.

## Data and caching adoption

Do not add a new data client automatically:

- Preserve `useFetch`, `useAsyncData`, `$fetch`, Axios, SDKs, or the existing query library.
- Use stable keys that include every dynamic value affecting fetched data.
- Keep direct server reads in server services; do not add internal HTTP hops through the project's own Nitro API.
- Keep mutation invalidation and refresh behavior close to the mutation.
- Add `@tanstack/vue-query` only for an explicit client-side server-state use case.
- Review `routeRules`, caching, revalidation, and prerender behavior before changing them. Do not assume Nuxt 3 and Nuxt 4 defaults match.

Completion: data ownership, async keys, freshness policy, and invalidation behavior are explicit for every touched flow.

## Dependency adoption

Do not add missing base dependencies automatically. Ask before adopting them:

| Area | If already present | If missing and the user explicitly requests the base |
|---|---|---|
| Client state | Preserve Pinia or the current state model | Add Pinia and a small feature store only when needed |
| Server state | Preserve the current data-fetching solution | Add TanStack Vue Query only for an explicit client-side use case |
| Forms | Preserve working forms and validation | Add vee-validate, `@vee-validate/zod`, and Zod for new forms |
| Validation | Preserve current schemas | Add Zod at new input and server boundaries |
| HTTP/data | Preserve `$fetch`, `useFetch`, Axios, SDK, or ORM | Add only the requested client or integration |
| Styling | Preserve Tailwind or the current CSS system | Add Tailwind only after confirmation |
| UI | Preserve shadcn-vue or another kit | Ask before installing shadcn-vue |
| Tests | Preserve the current runner and setup | Add Vitest, Vue Test Utils, Testing Library, DOM environment, and MSW without duplication |
| Nuxt integration tests | Preserve current integration setup | Add `@nuxt/test-utils` only when explicitly requested |

Completion: every new dependency has explicit user intent and a single owner in the architecture.

## shadcn-vue

If the project uses shadcn-vue:

- Detect generated components under the configured components directory and treat them as shared base code.
- Do not automatically edit an existing generated component.
- Explain the proposed change, reason, and affected consumers.
- Obtain explicit confirmation before changing it.
- Prefer composition, variants, props, or feature wrappers for local needs.
- Verify all consumers after an authorized base-component change.

If the project does not use shadcn-vue, ask before initializing it. Tailwind alone is not confirmation to install shadcn-vue.

Completion: UI base ownership is clear and no generated component changed without confirmation.

## Quality toolchain

Determine owners from evidence in `package.json`, scripts, and configuration:

| Evidence | Owner | Action |
|---|---|---|
| `biome.json` / `biome.jsonc`, dependency, or scripts for Biome | Biome | Use Biome for format and lint |
| `eslint.config.*` / `.eslintrc*`, dependency, or scripts for ESLint | ESLint | Use ESLint for lint |
| Oxlint configuration, dependency, or scripts | Oxlint | Use Oxlint for lint |
| `.prettierrc*` / `prettier.config.*`, dependency, or scripts for Prettier | Prettier | Use Prettier for format |
| No evidence | None | Report the delta before installing |

Apply these rules:

- If the project uses Biome, do not add ESLint, Oxlint, or Prettier for the same responsibility.
- If it uses ESLint or Oxlint, preserve it as the linter. Prettier may remain a separate formatter.
- If Prettier is active, ensure its ignore configuration covers real generated artifacts and dependencies.
- If Prettier is not active, do not create a Prettier ignore file.
- For ESLint, Oxlint, and Biome use the ignore mechanism compatible with the installed version.
- If multiple tools exist, preserve them, use current scripts as source of truth, and report conflicts. Consolidate or replace only on explicit request.
- If no tool exists, do not install one automatically in an existing project; ask whether the user wants to adopt one.

Completion: quality ownership is explicit and no duplicate tool was introduced.

## Migration order

### Phase 1: configuration boundaries

- Confirm Nuxt version, `srcDir` or `app/` convention, package manager, lockfile, and scripts.
- Stop and ask if multiple conflicting lockfiles claim ownership.
- Audit `nuxt.config.*`, `app.config.*`, modules, aliases, strict TypeScript, linter, formatter, and ignores.
- Preserve configuration that is valid for the installed version.
- Fix only missing configuration in the selected toolchain.

Completion: baseline checks still pass and every new import follows one compatible alias convention.

### Phase 2: runtime boundaries

- Inventory app composables, server services, runtime config, plugins, middleware, API handlers, and browser-only dependencies.
- Move only clearly server-only logic away from app modules.
- Keep shared code runtime-neutral.
- Validate Nitro handler inputs with Zod and re-check auth inside each mutation.

Completion: no private module or secret crosses into the app graph, and every mutation has a server-side authorization check.

### Phase 3: infrastructure

- Consolidate environment parsing in one server-only module.
- Keep data access in feature services or the existing shared client boundary.
- Add Pinia stores only for actual shared client state.
- Add query keys only when TanStack Vue Query is present.
- Add MSW setup only when API test coverage is being adopted.

Completion: infrastructure has clear entry points and no duplicated clients, stores, providers, modules, or validation modules.

### Phase 4: migrate one feature

Migrate one complete feature as a tracer bullet:

```text
app/features/<feature>/
├── components/
├── composables/
├── schemas/
├── services/
├── stores/
├── types/
└── utils/
```

- Keep the page and layout in the project's Nuxt pages and layouts directories.
- Keep Nitro handlers under `server/api` or `server/routes`.
- Move domain UI, composables, schemas, services, stores, and tests together.
- Separate interactive forms from server mutations.
- Preserve SSR, route, auth, and cache behavior.
- Extract only genuinely shared code.

Completion: the feature preserves behavior, URLs, runtime boundaries, auth, cache semantics, and tests while its ownership is visible.

### Phase 5: controlled repetition

Repeat per feature, not through a global directory rewrite. After each feature run typecheck, lint, relevant tests, and build when feasible. Defer Nuxt major-version, `srcDir`, module, UI-kit, state, and data-client migrations until explicitly requested.

Completion: all agreed features are migrated and no unjustified cross-feature dependencies remain.

## Security rules

- Re-check authentication and authorization inside every mutating Nitro handler and server service.
- Validate JSON, FormData, query parameters, route parameters, headers, cookies, and external responses before use.
- Keep private runtime config, environment variables, server SDKs, and database clients out of app components and public composables.
- Verify webhook signatures against the raw request body before parsing or acting on it.
- Return safe error messages from public HTTP boundaries; keep sensitive diagnostics server-side.
- Do not change routes, API contracts, persistence, rendering mode, or cache behavior as a side effect of reorganizing folders.
- Do not touch unrelated worktree changes.

Completion: touched server boundaries validate input, enforce auth, protect secrets, and preserve external contracts.

## Final verification

Run existing scripts for typecheck, lint, format check when a formatter exists, tests, Nuxt integration tests when explicitly configured, and build. Add MSW tests for new services or API flows where applicable.

Review the diff and confirm:

- No Nuxt 3 project was silently migrated to the Nuxt 4 `app/` structure.
- No private module or secret is imported into app code.
- No unnecessary browser-only boundary was added.
- No feature imports another feature directly.
- No new barrels were added.
- Data clients, Pinia stores, modules, providers, validators, UI kits, or quality tools were not duplicated.
- Existing package manager, scripts, routes, auth, SSR, and cache behavior remain intact unless explicitly changed.

Completion: available checks pass, exceptions are documented, and the final diff contains only requested adoption work.
