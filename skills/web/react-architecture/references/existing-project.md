# Adoption in an existing project

Read this reference before editing an existing React application.

## Preflight

1. Confirm that it is a React SPA. If `package.json` contains `next`, or the structure belongs to the Next.js router, stop and use a Next.js-specific skill.
2. Read `package.json`, the lockfile, scripts, TypeScript configuration, bundler, linter, formatter, ignore files, tests, and router.
3. Inspect `src/` and locate the entry point, routes, global state, HTTP client, forms, UI kit, and current features.
4. Run existing checks before changing anything when possible.
5. Create a delta matrix: rule, current state, risk, minimum change, and verification.

Do not begin by moving files. First gather enough evidence to know what is preserved and what is transformed.

Completion: the project classification, current structure, and baseline checks are known.

## Preservation policy

Preserve by default:

- Current router and public URLs.
- Zustand, Redux, Context, or another existing state solution.
- Axios, fetch, an SDK, or another existing HTTP client.
- Tailwind, CSS Modules, styled-components, or another styling system.
- shadcn/ui or another existing UI kit.
- Package manager and lockfile.
- Linter, formatter, test runner, and scripts.
- Valid naming conventions and structure already used by the repository.

The goal is to adopt clear boundaries, not replace technology. Migrating to TanStack Router, Zustand, Axios, Tailwind, or shadcn/ui requires an explicit request and a separate plan.

Completion: preserved systems and intentional migration targets are explicit before edits begin.

## Quality toolchain

Determine owners from evidence in `package.json`, scripts, and configuration:

| Evidence                                                              | Owner    | Action                             |
| --------------------------------------------------------------------- | -------- | ---------------------------------- |
| `biome.json` / `biome.jsonc`, dependency, or scripts for Biome        | Biome    | Use Biome for format and lint      |
| `eslint.config.*` / `.eslintrc*`, dependency, or scripts for ESLint   | ESLint   | Use ESLint for lint                |
| Oxlint configuration, dependency, or scripts                          | Oxlint   | Use Oxlint for lint                |
| `.prettierrc*` / `prettier.config.*`, Prettier dependency, or scripts | Prettier | Use Prettier for format            |
| No evidence                                                           | None     | Report the delta before installing |

Apply these rules:

- If the project already uses Biome, do not add ESLint, Oxlint, or Prettier for the same responsibility.
- If it uses ESLint or Oxlint, preserve it as the linter. Prettier may continue as a separate formatter.
- If Prettier is active, ensure `.prettierignore` covers `node_modules`, `dist`, `build`, `coverage`, and real generated artifacts.
- If Prettier is not active, do not create `.prettierignore`.
- For ESLint, Oxlint, and Biome use the ignore mechanism compatible with the installed configuration and version. Do not create obsolete ignore files automatically.
- If several tools exist, preserve them, use their scripts as source of truth, and report conflicts. Consolidate or replace only on explicit request.
- If configuration exists but coherent scripts are missing, make the minimum correction within the selected toolchain.
- If no tool exists, do not install one automatically in an existing project; ask whether the user wants to adopt one.

Completion: quality ownership is explicit and no duplicate tool was introduced.

## shadcn/ui components

If the project uses shadcn/ui, detect its base components and treat them as shared code:

- Do not automatically edit an existing component in `src/shared/components/ui/`.
- First explain the proposed change, its reason, and affected consumers.
- Ask the user and wait for explicit confirmation before modifying it.
- For a single-feature need, prefer composition, props, variants, or wrappers inside the feature.
- Keep base changes small and verify all consumers after an authorized modification.

Completion: UI base ownership is clear and no generated component changed without confirmation.

## Adoption matrix

| Area         | If already present                   | If missing and the user asks to adopt the base               |
| ------------ | ------------------------------------ | ------------------------------------------------------------ |
| Routing      | Preserve the router and routes       | Configure TanStack Router incrementally                      |
| Server state | Preserve the current solution        | Add TanStack Query for new flows                             |
| Client state | Preserve the current store           | Create a feature store only when needed                      |
| HTTP         | Preserve the current client          | Create a shared client and services                          |
| Forms        | Preserve working forms               | Use RHF + Zod for new forms                                  |
| UI           | Preserve Tailwind or the current kit | Ask before adding shadcn/ui                                  |
| Tests        | Preserve the current runner          | Add Vitest + Testing Library + MSW without duplicating tools |
| Aliases      | Preserve valid aliases               | Add `@/*` when it does not conflict                          |

Completion: each adoption has a stated reason, limited scope, and verification path.

## Recommended order

### Phase 1: configuration boundaries

- Confirm the package manager and scripts.
- Respect `packageManager` in `package.json` and the lockfile. If conflicting lockfiles exist, stop and ask.
- Configure the `@/*` alias only if no equivalent or conflicting alias exists.
- Adjust TypeScript strictness gradually; fix errors in groups rather than silencing them.
- Audit the linter, formatter, and ignore files; complete only missing configuration in the selected toolchain.

Completion: baseline checks still pass and every new import follows one alias convention.

### Phase 2: shared infrastructure

- Consolidate the HTTP client in `shared` without duplicating instances.
- Centralize query keys when TanStack Query already exists.
- Validate environment variables from one module.
- Prepare MSW setup under `src/test/` when API coverage is being adopted.
- If Playwright exists, audit whether tests hit a real backend. Mocked `page.route` suites belong in Vitest + MSW; keep only a post-deploy sliver.

Completion: infrastructure has one clear entry point and no feature calls the transport outside its service.

### Phase 3: migrate one feature

Migrate one complete feature as a tracer bullet:

```text
features/<feature>/
├── components/
├── hooks/
├── schemas/
├── services/
├── types/
├── utils/
└── store.ts        # only when applicable
```

- Move domain components with their tests.
- Separate forms from screens.
- Keep screens and route composition in `src/routes/`.
- Change imports to direct aliases.
- Extract only genuinely reusable code to `shared`.
- Remove barrels when touching an area, updating imports to direct files.

Completion: the feature preserves behavior, routes, and tests while its boundaries are visible.

### Phase 4: controlled repetition

Repeat by feature, not through a global rewrite. After each feature run typecheck, lint, and affected tests. Defer router, UI kit, or state migrations until the user requests them.

Completion: all agreed features are migrated and no unjustified cross-feature dependencies remain.

## Security rules

- Do not delete files until all consumers are found and updated.
- Do not move screens into `features`; they must remain in `routes`.
- Do not introduce `src/pages` as a parallel layer.
- Do not create `index.ts` files to re-export modules.
- Do not install shadcn/ui only because the project uses Tailwind.
- Do not modify existing shadcn/ui components without the user's explicit confirmation.
- Prefer composition or wrappers over changing a base component for a local need.
- Do not install Prettier, ESLint, Oxlint, or Biome only because a configuration file is missing; first identify the project's decision.
- Do not create `.prettierignore` when Prettier is inactive.
- Do not change URLs, API contracts, or persistence as a side effect of organizing folders.
- Do not touch unrelated worktree changes.

Completion: structural changes preserve consumers, contracts, and unrelated work.

## Final verification

Run existing scripts for typecheck, lint, `format:check` when a formatter exists, tests, and build. Add MSW tests for new services or API flows. Review the diff and confirm:

- No avoidable long relative imports remain.
- No imports between features exist.
- No new barrels exist.
- HTTP clients, query clients, stores, or UI kits were not duplicated.
- Formatters or linters were not duplicated, and ignore files match the active toolchain.
- The router and URLs remain unchanged unless migration was explicitly requested.
- The project preserves the correct package manager and lockfile.

Completion: available checks pass, exceptions are documented, and the final diff contains only requested adoption work.
