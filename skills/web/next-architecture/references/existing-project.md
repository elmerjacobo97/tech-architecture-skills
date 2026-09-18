# Adoption in an existing Next.js App Router project

Read this reference before editing an existing Next.js application.

## Preflight

1. Confirm that `package.json` contains Next.js and that routing uses `src/app` or `app`. If the project uses Pages Router, stop and report that this skill covers App Router only.
2. Read `package.json`, lockfile, scripts, TypeScript configuration, Next.js configuration, linter, formatter, ignore files, tests, and environment declarations.
3. Inspect `src/app` and `src/` to locate layouts, route segments, providers, feature code, state, data access, forms, UI kit, auth boundaries, and existing server/client directives.
4. Run existing checks before changing anything when possible.
5. Create a delta matrix: rule, current state, risk, minimum change, and verification.

Do not begin by moving files. Gather enough evidence to know what is preserved and what is transformed.

Completion: App Router is confirmed, current ownership boundaries are documented, and baseline checks are known.

## Preservation policy

Preserve by default:

- App Router URLs, layouts, route groups, and public behavior.
- Existing Server Components and deliberate Client Component boundaries.
- Zustand, Redux, Context, or another existing state solution.
- Native fetch, Axios, an SDK, an ORM, or another existing data client.
- Existing Server Actions and Route Handlers.
- Tailwind, CSS Modules, styled-components, or another styling system.
- shadcn/ui or another existing UI kit.
- Package manager and lockfile.
- Linter, formatter, test runner, scripts, and configuration.
- Valid naming conventions and useful directory structure.

The goal is to make boundaries clear, not replace technology. Migrating state, forms, data fetching, styling, UI kit, or routing requires an explicit request and a separate scoped plan.

Completion: preserved systems and intentional migration targets are explicit before edits begin.

## Toolchain of record

Determine owners from `package.json`, scripts, and configuration:

| Evidence                                                                  | Owner    | Action                             |
| ------------------------------------------------------------------------- | -------- | ---------------------------------- |
| `biome.json` / `biome.jsonc`, dependency, or scripts for Biome            | Biome    | Use Biome for format and lint      |
| `eslint.config.*` / `.eslintrc*`, dependency, or scripts for ESLint       | ESLint   | Use ESLint for lint                |
| Oxlint configuration, dependency, or scripts                              | Oxlint   | Use Oxlint for lint                |
| `.prettierrc*` / `prettier.config.*`, dependency, or scripts for Prettier | Prettier | Use Prettier for format            |
| No evidence                                                               | None     | Report the delta before installing |

Apply these rules:

- If the project uses Biome, do not add ESLint, Oxlint, or Prettier for the same responsibility.
- If it uses ESLint or Oxlint, preserve it as linter. Prettier may remain a separate formatter.
- If Prettier is active, ensure its ignore configuration covers real generated artifacts and dependencies.
- If Prettier is not active, do not create a Prettier ignore file.
- For ESLint, Oxlint, and Biome use the ignore mechanism supported by the installed version.
- If multiple tools exist, preserve them, use current scripts as source of truth, and report conflicts. Consolidate only on explicit request.
- If no quality tool exists, ask before installing one.

Completion: quality ownership is explicit and no duplicate tool was introduced.

## App Router boundaries

Keep route files in `src/app` focused on routing and composition. Use feature modules for domain behavior:

```text
src/
├── app/
│   └── <route>/
│       ├── page.tsx
│       ├── layout.tsx
│       ├── loading.tsx
│       ├── error.tsx
│       └── route.ts
└── features/
    └── <feature>/
        ├── actions/
        ├── components/
        ├── hooks/
        ├── schemas/
        ├── services/
        ├── types/
        ├── utils/
        └── store.ts
```

Apply changes incrementally:

- Preserve existing route segments and URLs.
- Keep Server Components server-first and Client Components at the smallest interactive boundary.
- Keep services that use secrets, databases, or private SDKs server-only.
- Keep Server Actions and Route Handlers close to their feature when they are feature-specific.
- Extract to `lib`, `components`, `hooks`, or `types` only when code is genuinely shared.
- Do not create `src/pages` or move route files out of `src/app`.
- Do not add a client query provider unless the user explicitly requests TanStack Query or the existing project already depends on it.

Completion: each touched route has a clear composition role and no feature owns accidental routing or server-boundary logic.

## Dependency adoption

Do not add missing base dependencies automatically. Ask before adopting them:

| Area         | If already present                           | If missing and user explicitly requests the base                                                                                                   |
| ------------ | -------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| Client state | Preserve current store or state model        | Add Zustand and create a small feature store only when needed                                                                                      |
| Server state | Preserve current data-fetching solution      | Add TanStack Query only for an explicit client-side use case                                                                                       |
| Forms        | Preserve working forms and validation        | Use React Hook Form plus Zod for new forms; with shadcn/ui add `react-hook-form` + `@hookform/resolvers` and `shadcn add field` in the scoped plan |
| Validation   | Preserve current schemas                     | Add Zod at new input and server boundaries                                                                                                         |
| HTTP/data    | Preserve fetch, SDK, Axios, or ORM           | Add only the requested client or integration                                                                                                       |
| Styling      | Preserve Tailwind or the existing CSS system | Add Tailwind only after confirmation                                                                                                               |
| UI           | Preserve shadcn/ui or another kit            | Ask before installing shadcn/ui                                                                                                                    |
| Tests        | Preserve runner and setup                    | Add Vitest, Testing Library, jsdom, and MSW without duplicating tools                                                                              |

Completion: every new dependency has explicit user intent and a single owner in the architecture.

## shadcn/ui

If the project uses shadcn/ui:

- Detect generated components under `components/ui/` and treat them as shared base code.
- Do not edit an existing generated component automatically.
- Explain the proposed change, reason, and affected consumers.
- Obtain explicit confirmation before changing it.
- Prefer composition, variants, props, or feature wrappers for local needs.
- For new or changed validated forms, follow `forms.md`: install `react-hook-form` and `@hookform/resolvers` in the scoped plan, generate the `field` primitive, and keep the Zod schema in its own feature module.
- Verify all consumers after an authorized base-component change.
- Do not write tests for generated components under `components/ui/`. They are third-party base code: tests target project-owned code (features, wrappers, hooks, services, and shared components built on top of the primitives).

If the project does not use shadcn/ui, ask before initializing it. Tailwind alone is not confirmation to install shadcn/ui.

Completion: UI base ownership is clear and no generated component changed without confirmation.

## Migration order

### Phase 1: configuration boundaries

- Confirm package manager, lockfile, and scripts.
- Stop and ask if multiple conflicting lockfiles claim ownership.
- Preserve or add an alias only when it does not conflict.
- Audit strict TypeScript, Next.js configuration, linter, formatter, and ignores.
- Fix only missing configuration in the selected toolchain.

Completion: baseline checks still pass and every new import follows one alias convention.

### Phase 2: server and client boundaries

- Inventory `"use client"`, server-only modules, environment access, providers, Server Actions, and Route Handlers.
- Move only clearly server-only logic away from client modules.
- Keep providers as low as their consumers allow.
- Validate Server Action and Route Handler inputs with Zod and re-check auth inside each mutation.

Completion: no private module or secret crosses into the client graph, and every mutation has a server-side authorization check.

### Phase 3: infrastructure

- Consolidate shared environment parsing in one module.
- Keep data access in feature services or the existing shared client boundary.
- Add query keys only when TanStack Query is present.
- Add MSW setup only when API test coverage is being adopted.

Completion: infrastructure has clear entry points and no duplicated clients, providers, or validation modules.

### Phase 4: migrate one feature

Migrate one complete feature as a tracer bullet:

```text
features/<feature>/
├── actions/
├── components/
├── hooks/
├── schemas/
├── services/
├── types/
├── utils/
└── store.ts       # only when applicable
```

- Keep the route page and layout in `src/app`.
- Move domain UI, services, schemas, and tests together.
- Separate interactive form components from Server Components.
- Keep server reads on the server and client behavior at the smallest boundary.
- Extract only genuinely shared code.

Completion: the feature preserves behavior, URLs, auth, cache semantics, and tests while its boundaries are visible.

### Phase 5: controlled repetition

Repeat per feature, not through a global folder rewrite. After each feature run typecheck, lint, relevant tests, and build when feasible. Defer router, UI-kit, state, and data-client migrations until explicitly requested.

Completion: all agreed features are migrated and no unjustified cross-feature dependencies remain.

## Security rules

- Re-check authentication and authorization inside every Server Action and Route Handler.
- Validate JSON, FormData, search parameters, route parameters, and external responses before use.
- Keep private environment variables and server SDKs out of Client Components.
- Verify webhook signatures against the raw request body before parsing or acting on it.
- Return safe error messages from public HTTP boundaries; keep sensitive diagnostics server-side.
- Do not change URLs, API contracts, persistence, or cache behavior as a side effect of reorganizing folders.
- Do not touch unrelated worktree changes.

Completion: touched server boundaries validate input, enforce auth, protect secrets, and preserve external contracts.

## Final verification

Run existing scripts for typecheck, lint, format check when a formatter exists, tests, Next.js type generation when available, and build. Add MSW tests for new services or API flows where applicable.

Review the diff and confirm:

- No Pages Router layer was introduced.
- No unnecessary `"use client"` boundary was added.
- No private module or secret is imported into a Client Component.
- No imports between features were introduced.
- No new barrels were added.
- No clients, providers, stores, validators, UI kits, or quality tools were duplicated.
- Existing package manager, scripts, URLs, auth, and cache behavior remain intact unless explicitly changed.

Completion: available checks pass, exceptions are documented, and the diff contains only the requested adoption work.
