---
name: react-architecture
description: Use when creating, scaffolding, structuring, or incrementally migrating React SPA projects with feature-based architecture, TanStack Router, TanStack Query, Zustand, React Hook Form, Zod, Axios, Tailwind CSS, or shadcn/ui. Use only for React SPA/Vite projects; do not apply to Next.js.
---

# Structured React SPA

Skill for defining the technical base, architecture, and incremental evolution of React SPA applications. Organize by vertical feature, not by global technical layer.

## Scope

- Applies to React SPA applications, normally created with Vite and TypeScript.
- Covers two modes: new project and existing project.
- New projects receive a coherent base.
- Existing projects are audited first and current decisions are preserved.
- Next.js is outside this skill. If the repository uses `next`, a Next.js `app/` or `pages/` structure, stop and use a Next.js-specific guide.

## Invocation and arguments

The text after `/react-architecture` is available as `ARGUMENTS`. Interpret these keys by convention:

| Key                      | Values                          | Effect                                                                               |
| ------------------------ | ------------------------------- | ------------------------------------------------------------------------------------ |
| `name` / `nombre`        | text                            | Name used when creating a new project                                                |
| `existing` / `existente` | no value                        | Forces existing-project mode                                                         |
| `ui`                     | `shadcn` / `tailwind`           | Avoids asking about UI in a new project                                              |
| `backend`                | `rest` / `none` / `other`       | Defines whether an HTTP client is prepared; `rest` is the new-project default        |
| `router`                 | `tanstack` / `other`            | `tanstack` is the new-project default; existing projects preserve the current router |
| `playwright`             | `yes` / `no` / `si`             | Forces end-to-end test installation                                                  |
| `package-manager`        | `pnpm` / `npm` / `yarn` / `bun` | Selects the package manager for a new project; ignored for existing projects         |

If a new project lacks required information, ask before installing. The UI decision is required: ask whether to install shadcn/ui when it is not specified. Default: Tailwind CSS without shadcn/ui.

## Decision policy

### New project

Use React + TypeScript + Vite, install the standard base, and create the structure described in `references/scaffold.md`. Use pnpm by default. Change it only when the user explicitly selects another package manager through `package-manager` or the request.

Included base:

- TanStack Router for file-based routing.
- TanStack Query for server state and fetching.
- Axios as the HTTP client for REST APIs when the project has a backend or API integration.
- Zustand for shared client state when it is genuinely needed.
- React Hook Form + Zod + `zodResolver` for forms.
- Tailwind CSS for styling.
- Vitest + React Testing Library + MSW for unit, component, and mocked API tests.
- TypeScript in strict mode.

Install shadcn/ui only after asking and receiving an affirmative answer. Omit Axios when `backend: none`. Playwright, additional icons, `date-fns`, `sonner`, i18n, Sentry, and CI remain opt-in.

### Existing project

Read `references/existing-project.md` before editing. Audit first and create a delta matrix against these rules.

- Detect the package manager from `packageManager` in `package.json` and the lockfile, and preserve it.
- Preserve the current router, state solution, HTTP client, UI kit, linter, formatter, and useful structure.
- Add the preferred architecture without replacing existing systems.
- Migrate to TanStack Router, Zustand, Axios, Tailwind, or shadcn/ui only when the user explicitly requests it.
- Avoid installing a second library that solves the same problem.
- Apply changes by feature or small layer, with verification after each step.

## Quality toolchain

Treat formatter and linter as project decisions, not a fixed stack. Audit `package.json`, scripts, and configuration files before installing or changing tools.

| Evidence                                                              | Owner    | Action                                                                          |
| --------------------------------------------------------------------- | -------- | ------------------------------------------------------------------------------- |
| `biome.json` / `biome.jsonc`, dependency, or scripts for Biome        | Biome    | Use Biome for format and lint                                                   |
| `eslint.config.*` / `.eslintrc*`, dependency, or scripts for ESLint   | ESLint   | Use ESLint for lint                                                             |
| Oxlint configuration, dependency, or scripts                          | Oxlint   | Use Oxlint for lint                                                             |
| `.prettierrc*` / `prettier.config.*`, Prettier dependency, or scripts | Prettier | Use Prettier for format                                                         |
| No evidence                                                           | None     | Ask in a new project; report the delta before installing in an existing project |

Selection rules:

- Biome can own both format and lint.
- ESLint or Oxlint can own lint; Prettier can own format.
- Do not install all four tools by default or introduce a second tool for the same responsibility.
- If several already exist, preserve them, use current scripts as source of truth, and report conflicts. Consolidate or replace only on explicit request.
- In a new project, respect the tool included by the scaffold. If none is included, ask which combination to use.

## Ignore files

- If Prettier is active, ensure `.prettierignore` covers real project artifacts: `node_modules`, `dist`, `build`, `coverage`, and generated files.
- If Prettier is not active, do not create `.prettierignore`.
- For ESLint, Oxlint, and Biome use the ignore mechanism compatible with the installed configuration and version. Do not create obsolete ignore files automatically.
- Ensure coherent `lint`, `format`, and `format:check` scripts when the corresponding tool is active.

## shadcn/ui components

When the project uses shadcn/ui:

- Treat `src/shared/components/ui/` as generated, shared base components.
- Use these components without editing generated files on your own initiative.
- If a need requires modifying a shadcn/ui component, first explain what will change, why, and the impact on other consumers.
- Ask the user and wait for explicit confirmation before editing existing shadcn/ui components.
- If customization is feature-specific, prefer composition, props, variants, or a feature wrapper over changing the base component.
- Generate new components only when the UI needs them; do not modify base components for a local need.

## Architecture

Base structure:

```text
src/
├── main.tsx
├── routes/                 # route definitions and screen composition
├── features/
│   └── <feature>/
│       ├── components/     # domain-owned components
│       ├── hooks/          # queries, mutations, and feature hooks
│       ├── schemas/        # form validation and contracts
│       ├── services/       # external access for the feature
│       ├── types/
│       ├── utils/
│       └── store.ts        # only when the feature needs shared state
├── shared/
│   ├── components/
│   │   └── ui/             # only when shadcn/ui is used
│   ├── hooks/
│   ├── lib/
│   ├── schemas/
│   ├── types/
│   └── utils/
├── test/                   # MSW + render helpers (setup, handlers, server, render)
└── styles/
    └── globals.css
```

Dependency rules:

- `routes` may import from `features` and `shared`.
- A feature may import its own modules and `shared`.
- `shared` imports only from `shared` or external dependencies.
- A feature does not import another feature directly.
- If two features need an abstraction, move it to `shared` after confirming that it is genuinely shared.
- Screens live in `routes`, not in `src/pages` or as page components inside a feature.
- Route files stay thin: they define routing, layout, and composition; domain logic lives in features.
- Use TanStack Router's reserved conventions (`__root.tsx`, `index.tsx`, `$param.tsx`, pathless groups, and so on) only where the router requires them. Use `kebab-case` for other files.

## Testing

Read `references/testing.md` for wiring. Default stack: Vitest + RTL + jsdom + MSW. Playwright is opt-in.

A test is worth writing if it would fail when user-visible or API-visible behavior breaks. Do not test generated `components/ui/`, third-party internals, thin `useQuery` wrappers, static copy, or DOM snapshots.

Form checklist: valid submit, invalid state, pending, no duplicate submit, reset.
Dialog checklist: open/close, action labels, mutation error, pending, success closes.
Create and edit independently even when they share a form module.

Playwright only for journeys that lose meaning if the API is mocked (real cookies, CORS, deployed build + backend). Never use `page.route` as a stand-in for MSW. Do not run a large mocked browser suite on every MR.

## Non-negotiable rules

1. **Direct imports.** Import each module from its source file. Do not create barrel files that only re-export, especially `index.ts`.
2. **Consistent aliases.** `@/*` points to `src/*`. Configure the alias in TypeScript and the bundler, and use `@/...` for non-local internal imports.
3. **Predictable names.** Use `kebab-case` for application files and folders: `login-form.tsx`, `use-current-user.ts`, `auth.service.ts`.
4. **Well-placed state.** TanStack Query manages server state. Zustand manages shared client state. State used by one component remains in local React state.
5. **Small stores.** Create at most one Zustand store per feature and only when multiple components need shared state. Never create a global mega-store.
6. **Encapsulated HTTP.** Configure Axios in `shared/lib/api-client.ts`. Each REST feature exposes operations through `services/<feature>.service.ts`. Components and hooks do not call Axios directly.
7. **Focused hooks.** `queries.ts` and `mutations.ts` only coordinate TanStack Query with services. They do not contain HTTP logic or extensive transformations.
8. **Centralized query keys.** Define factories in `shared/lib/query-keys.ts` and reuse them in queries, mutations, and invalidations.
9. **Boundary validation.** Validate environment variables in `shared/lib/env.ts` with Zod. Validate user input and, when appropriate, external responses with Zod schemas.
10. **Sensitive data.** Keep access tokens in memory. Prefer refresh tokens in `httpOnly` cookies; use another storage mechanism only as an explicit, documented exception. Never persist access tokens in `localStorage`.
11. **Nearby tests.** Co-locate tests with the module under test. Shared setup lives in `src/test/`. Use `__tests__/` when several small files form one unit. Do not test generated shadcn `ui/` components.
12. **Playwright is a sliver.** Add it only when requested or the app is critical, and only for real-edge journeys. Mocked `page.route` tests belong in Vitest + MSW.
13. **Conditional UI.** Ask about shadcn/ui in new projects. In existing projects detect `components.json` and the current implementation; preserve it. Existing Tailwind remains Tailwind.
14. **Protected base components.** Ask for explicit confirmation before editing existing shadcn/ui components. Prefer composition for local needs.
15. **Adaptable toolchain.** Select formatter and linter from project evidence. Keep one owner per responsibility and matching ignore configuration.
16. **Respectful configuration.** In existing projects make the minimum required configuration changes and preserve compatible scripts, aliases, and conventions.
17. **Mandatory verification.** Before finishing, run available typecheck, lint, format check, tests, and build; fix failures introduced by the change.

## Work order

1. Classify the project as new React SPA, existing React SPA, or Next.js. Completion: classification is explicit.
2. Read the matching reference: `references/scaffold.md` for new or `references/existing-project.md` for existing. Completion: decisions and delta are identified.
3. Confirm only missing decisions. Completion: package manager, UI, quality toolchain, and scope are known.
4. Implement the base or incremental migration. Completion: every change respects boundaries and import rules.
5. Verify and review the diff. Completion: available checks pass and no references to removed tooling remain.

## Library documentation

Before assuming current syntax, configuration, or behavior for Vite, TanStack, shadcn/ui, Tailwind, Vitest, or another library, consult current documentation through `ctx7`. Do not pin versions or APIs based on memory alone.
