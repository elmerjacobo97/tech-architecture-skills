---
name: nuxt-architecture
description: Use when creating, scaffolding, structuring, or incrementally migrating Nuxt applications with Nuxt 4, the App directory, Vue, Nitro server routes, Pinia, composables, useFetch, vee-validate, Zod, Tailwind CSS, or shadcn-vue.
---

# Structured Nuxt Application

Skill for defining the technical base, architecture, and incremental evolution of Nuxt applications. Organize by vertical feature while keeping Nuxt app code, Nitro server code, and runtime-neutral shared code in clear boundaries.

## Scope

- Applies to Nuxt 4 applications and existing Nuxt 3 applications.
- Uses Nuxt 4 for new projects unless the user requests another version.
- Covers two modes: new project and existing project.
- New projects receive Pinia, vee-validate, Zod, Tailwind CSS, and a test base.
- Existing projects are audited first and current conventions, modules, and structure are preserved.
- Nuxt 2, Vue-only SPAs, and non-Nuxt Vue projects are outside this skill. Stop and use the appropriate guide when detected.

## Invocation and arguments

The text after `/nuxt-architecture` is available as `ARGUMENTS`. Interpret these keys by convention:

| Key | Values | Effect |
|---|---|---|
| `name` | text | Name used when creating a new project |
| `existing` | no value | Forces existing-project mode |
| `version` | `nuxt4` / `nuxt3` | Selects the version for a new project; Nuxt 4 is the default |
| `ui` | `shadcn-vue` / `tailwind` | Avoids asking about UI in a new project |
| `backend` | `nitro` / `rest` / `none` / `other` | Sets the requested server integration; `nitro` is the default for a new Nuxt project |
| `query` | `vue-query` / `none` | Adds TanStack Vue Query only when explicitly requested |
| `tests` | `yes` / `no` | Forces installation of the default test base |
| `package-manager` | `pnpm` / `npm` / `yarn` / `bun` | Selects the package manager for a new project; ignored for existing projects |

If a new project lacks a required decision, ask before installing. The UI decision is required when `ui` is absent: ask whether to install shadcn-vue. The default is Tailwind CSS without shadcn-vue. Use pnpm by default unless the user explicitly selects another package manager.

## Decision policy

### New project

Use Nuxt 4 with TypeScript, the current App directory structure, and the selected package manager. Consult current Nuxt documentation before using `nuxi` commands, directory configuration, module setup, or version-sensitive APIs.

Install this base:

- `@pinia/nuxt` and Pinia for shared client state when needed.
- `vee-validate`, `@vee-validate/zod`, and Zod for interactive forms and validation.
- Tailwind CSS using the current Nuxt integration.
- Vitest, Vue Test Utils, Vue Testing Library, a DOM environment, and MSW for unit, component, and mocked API tests unless the user opts out.

Use `$fetch`, `useFetch`, and `useAsyncData` for data access by default. Do not add Axios or `@tanstack/vue-query` by default. Add TanStack Vue Query only when the user requests client-side server-state management or explicitly passes `query:vue-query`.

Ask before installing shadcn-vue. If confirmed, initialize it using the current official setup and generate components only when a feature needs them.

Playwright, `@nuxt/test-utils`, additional UI libraries, date libraries, toast libraries, i18n, Sentry, CI, database clients, and provider SDKs remain opt-in unless the request requires them.

### Existing project

Read `references/existing-project.md` before editing. Audit the repository and create a delta matrix against these rules.

- Confirm the project uses Nuxt 3 or Nuxt 4. If it uses Nuxt 2, stop and report that this skill does not cover it.
- Detect the package manager from `packageManager` in `package.json` and lockfiles; preserve it.
- Detect whether the project uses Nuxt 4's `app/` directory or Nuxt 3's root frontend directories. Preserve the existing layout unless migration is explicitly requested.
- Preserve existing state, data-fetching, HTTP, UI, module, auth, linter, formatter, test runner, scripts, and useful structure.
- Do not add missing Pinia, vee-validate, Zod, Tailwind, TanStack Vue Query, or other base dependencies automatically. Ask before adding them.
- If shadcn-vue exists, preserve it. If it is absent, ask before installing it.
- Adopt feature boundaries without replacing working systems.
- Migrate to Pinia, vee-validate, Zod, Tailwind, shadcn-vue, TanStack Vue Query, or another tool only after an explicit request and a scoped plan.
- Avoid installing a second module or library that solves the same problem.
- Apply changes by feature or small layer, with verification after each step.

## Nuxt architecture

Use this structure as the default for a new Nuxt 4 project:

```text
app/
├── app.vue                       # root application component
├── app.config.ts                 # non-secret reactive app configuration
├── assets/                       # build-processed assets
├── components/                   # shared auto-imported Vue components
│   └── ui/                       # generated shadcn-vue components, if used
├── composables/                  # shared auto-imported composables
├── features/
│   └── <feature>/
│       ├── components/           # feature-owned Vue components
│       ├── composables/          # feature-owned composables
│       ├── schemas/              # input and response validation
│       ├── services/             # feature data access and domain operations
│       ├── stores/               # feature Pinia stores, only when needed
│       ├── types/
│       └── utils/
├── layouts/
├── middleware/                   # navigation middleware
├── pages/                        # file-based routes
├── plugins/                      # app plugins
└── utils/                        # shared app utilities
├── server/
│   ├── api/                      # Nitro API handlers
│   ├── features/                 # server-side feature services
│   ├── middleware/               # request middleware
│   ├── plugins/                  # Nitro plugins
│   └── utils/
├── shared/                       # runtime-neutral app/server code
├── public/                       # static files served as-is
├── nuxt.config.ts
├── vitest.config.ts              # when tests are enabled
└── package.json
```

Nuxt 3 projects may use the equivalent frontend directories at the project root:

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

Do not move a Nuxt 3 project into `app/` merely to match Nuxt 4. Preserve its structure unless the user explicitly requests a migration.

Keep route files in `app/pages` or `pages` focused on route metadata, page composition, and page-level data orchestration. Put domain logic in `features`; shared app logic in `app/composables`, `app/utils`, and `app/components`; Nitro logic in `server`; runtime-neutral code in `shared`.

Do not create empty feature directories. Do not create a second router or duplicate Nuxt's reserved directories.

## App and server boundaries

- Treat Nuxt app code and Nitro server code as separate runtime graphs.
- Keep secrets, database clients, private SDKs, and server credentials in `server/` or server-only modules.
- Keep `runtimeConfig` private values on the server. Only values under `runtimeConfig.public` are available to the browser.
- Use `app.config.ts` for non-secret reactive application configuration, never for credentials.
- Keep `shared/` runtime-neutral. It must not import Vue app APIs, browser-only globals, Nitro event APIs, Node-only modules, or private server clients.
- Use `app/middleware/` for navigation decisions and `server/middleware/` for request-time behavior. Do not use one as a substitute for the other.
- Use plugins only for genuine app or Nitro integration. Keep plugin scope narrow and avoid turning plugins into service containers.
- Use `ClientOnly` or `import.meta.client` only for browser-only code that cannot render during SSR. Prefer SSR-compatible components first.
- Keep providers and global state setup as low-impact as possible; do not turn all pages into client-only rendering.

## Data fetching and caching

- Use `useFetch` for common page and component data fetching, with stable keys and typed responses.
- Use `useAsyncData` when fetching requires custom logic, multiple sources, or a non-standard key.
- Use `$fetch` directly inside server services and Nitro handlers when calling an upstream service or server-side resource.
- Do not call the application's own Nitro API from server code when the service can be called directly. Avoid unnecessary internal HTTP hops.
- Use a unique `key` for dynamic `useAsyncData` or `useFetch` calls. Include route parameters and other inputs that affect the result.
- Make server, lazy, dedupe, refresh, and cache behavior explicit when freshness or rendering timing matters.
- Use route rules, revalidation, or caching directives only after checking the installed Nuxt and Nitro versions and understanding freshness and invalidation behavior.
- Keep server-state data out of Pinia unless the user explicitly chooses a client cache design. Use `useFetch` or `useAsyncData` for normal Nuxt data flows.
- Add TanStack Vue Query only for an explicitly requested client-side server-state use case; do not wrap every page in a query provider.
- Re-fetch or invalidate affected data after mutations when the UI requires it. Keep invalidation close to the mutation.

## Nitro server routes

- Put HTTP handlers under `server/api/` and `server/routes/` using current Nitro filename conventions.
- Keep handlers thin: parse the request, authenticate and authorize, call a server feature service, and return a safe response.
- Validate JSON bodies, query parameters, route parameters, headers, cookies, and external responses as `unknown` with Zod at the server boundary.
- Re-check authentication and authorization inside every mutating handler. Never rely only on navigation middleware or page guards.
- Use the raw request body when verifying webhook signatures before parsing or acting on it.
- Return appropriate status codes and safe error messages. Keep sensitive diagnostics in server logs only.
- Keep server services under `server/features/<feature>/` or the existing project boundary. Do not put private server logic in `app/` or `shared/`.

## Pinia, composables, and forms

- Use Pinia for client state shared across multiple components. Keep local UI state in Vue refs and reactive state.
- Create at most one small Pinia store per feature when needed. Avoid a global mega-store.
- Keep server data out of Pinia unless there is an explicit reason to create a client cache.
- Put reusable app behavior in composables and keep each composable focused on one concern.
- Keep feature composables inside their feature when they are not genuinely shared.
- Use vee-validate with Zod schemas for new interactive forms.
- Validate again in Nitro handlers or server services; client validation is not an authorization or integrity boundary.
- Return structured expected validation errors. Reserve thrown errors for exceptional failures.
- Keep form components focused on interaction and field presentation. Keep mutation and server authorization on the server.

## Pages, layouts, middleware, and metadata

- Use file-based pages and layouts for URL and shared page composition.
- Use `definePageMeta` for route metadata, middleware declarations, and page-level options.
- Use `useSeoMeta` or the current Nuxt head utilities for metadata, with values derived from trusted content.
- Use `error.vue`, `createError`, `showError`, `clearError`, `navigateTo`, and `abortNavigation` according to the current Nuxt conventions.
- Keep authentication and authorization checks in server boundaries even when navigation middleware provides a faster user experience.
- Use `loading` and pending states from the data-fetching composable rather than hiding asynchronous work in global client state.

## Quality and tooling

Treat formatter and linter as project decisions, not a fixed stack. Audit `package.json`, scripts, and configuration before changing tools.

| Evidence | Owner | Action |
|---|---|---|
| `biome.json` / `biome.jsonc`, dependency, or scripts for Biome | Biome | Use Biome for format and lint |
| `eslint.config.*` / `.eslintrc*`, dependency, or scripts for ESLint | ESLint | Use ESLint for lint |
| Oxlint configuration, dependency, or scripts | Oxlint | Use Oxlint for lint |
| `.prettierrc*` / `prettier.config.*`, dependency, or scripts for Prettier | Prettier | Use Prettier for format |
| No evidence | None | Ask in a new project; report the delta in an existing project |

Rules:

- Keep one owner per responsibility. Biome may own both formatting and linting; ESLint or Oxlint may own linting while Prettier owns formatting.
- Do not install all four tools by default or introduce a second tool for the same responsibility.
- Preserve existing scripts and use them as the source of truth.
- Configure ignores through the mechanism supported by the installed tool and version.
- Add coherent `lint`, `format`, and `format:check` scripts only when the corresponding tool is active.
- Use strict TypeScript and the project's existing alias convention. Add aliases only when they do not conflict with Nuxt auto-imports or existing paths.

## shadcn-vue

When the project uses shadcn-vue:

- Treat generated files under `app/components/ui/` or the project's configured UI directory as shared base components.
- Use composition, variants, props, or feature wrappers for local customization.
- Explain the change, reason, and affected consumers before editing an existing generated component.
- Obtain explicit confirmation before modifying an existing generated component.
- Generate new components only when a real feature needs them.

When the project does not use shadcn-vue:

- Keep Tailwind CSS if it is already present.
- Ask before initializing shadcn-vue.
- Build feature-specific UI in the feature and genuinely shared UI in the configured components directory.

## Non-negotiable rules

1. **Nuxt version first.** Identify Nuxt 3 or Nuxt 4 before relying on directory or module conventions.
2. **Runtime boundaries.** Keep app, server, and runtime-neutral shared code in their respective graphs.
3. **Thin route files.** Pages, layouts, and Nitro handlers compose features; they do not become domain-service files.
4. **Feature boundaries.** A feature may import its own modules and shared modules. Shared modules never import app-only or server-only code. Features do not import one another directly.
5. **SSR by default.** Keep rendering SSR-compatible and isolate browser-only behavior to the smallest component or lifecycle boundary.
6. **Validated boundaries.** Parse untrusted input as `unknown` with Zod at every form, API, auth, and external-data boundary.
7. **Authorization in handlers.** Re-check session and permissions inside every mutating Nitro handler or server service.
8. **State placement.** Use local Vue state for local UI, Pinia for shared client state, and Nuxt data composables for server data.
9. **No unnecessary clients.** Do not add Axios or TanStack Vue Query by default; use `$fetch`, `useFetch`, `useAsyncData`, or existing SDKs.
10. **Stable async keys.** Include every dynamic input that changes fetched data in the `useFetch` or `useAsyncData` key.
11. **Secrets stay server-side.** Never expose private runtime config, credentials, tokens, database clients, or server SDKs to the app bundle.
12. **Direct imports.** Avoid barrels that only re-export modules, especially new `index.ts` files.
13. **Predictable names.** Use `kebab-case` for application files and folders where Nuxt does not reserve a filename.
14. **Protected UI base.** Ask before modifying existing shadcn-vue components.
15. **Respect existing systems.** Preserve working package managers, modules, scripts, routes, UI kits, clients, state solutions, and toolchains.
16. **Version-aware APIs.** Consult current Nuxt, Nitro, Vue, and library documentation before relying on version-sensitive behavior.
17. **Mandatory verification.** Run available typecheck, lint, format check, tests, and build commands before finishing; fix introduced failures.

## Work order

1. Classify the project as new Nuxt 4, existing Nuxt 4, existing Nuxt 3, Nuxt 2, or non-Nuxt. Completion: classification is explicit.
2. Read the matching reference: `references/scaffold.md` for new projects or `references/existing-project.md` for existing projects. Completion: required decisions and current delta are known.
3. Confirm only missing decisions. Completion: package manager, UI choice, test scope, quality toolchain, server integration, and optional query client are known.
4. Implement the base or incremental migration. Completion: runtime boundaries, feature boundaries, module choices, and Nuxt conventions are respected.
5. Verify and review the diff. Completion: available checks pass, no version-incompatible structure was introduced, and no unrelated files changed.

## Library documentation

Before assuming syntax, configuration, or behavior for Nuxt, Nitro, Vue, Pinia, Tailwind, vee-validate, Zod, shadcn-vue, Vitest, or another library, consult current documentation through `ctx7`. Do not pin versions or APIs from memory alone.
