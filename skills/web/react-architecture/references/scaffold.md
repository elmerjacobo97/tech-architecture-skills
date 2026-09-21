# New project scaffold

Read this reference only when creating a new React SPA application.

## Structure

```text
project-root/
├── src/
│   ├── main.tsx
│   ├── routes/
│   │   ├── __root.tsx
│   │   └── ...
│   ├── features/
│   │   └── <feature>/
│   │       ├── components/
│   │       ├── hooks/
│   │       ├── schemas/
│   │       ├── services/
│   │       ├── types/
│   │       ├── utils/
│   │       └── store.ts
│   ├── shared/
│   │   ├── components/
│   │   │   └── ui/           # only after shadcn/ui confirmation
│   │   ├── hooks/
│   │   ├── lib/
│   │   │   ├── api-client.ts
│   │   │   ├── env.ts
│   │   │   ├── query-client.ts
│   │   │   └── query-keys.ts
│   │   ├── schemas/
│   │   ├── types/
│   │   └── utils/
│   ├── test/
│   │   ├── handlers.ts
│   │   ├── server.ts
│   │   ├── setup.ts
│   │   └── render.tsx
│   └── styles/
│       └── globals.css
├── e2e/                      # only when requested AND journeys hit a real backend
├── .env.example
├── <quality-config>          # Biome, ESLint, Oxlint, Oxfmt, or Prettier config
├── .prettierignore           # only when Prettier is selected
├── tsconfig.json
├── vite.config.ts
├── vitest.config.ts
└── package.json
```

Do not create empty feature folders without a real feature. Do not create `src/pages` or barrels. Use `kebab-case` for application files and folders except TanStack Router reserved names. Place module tests beside their modules as `foo.test.ts(x)`; keep shared setup in `src/test/` and real-backend browser journeys in `e2e/`.

## Sequence

### 1. Confirm decisions

Confirm the name, backend, and whether to install shadcn/ui. Use pnpm by default in new projects unless the user specifies another package manager. When `ui` is absent, ask before installing.

For a normal application, leave Playwright out. Ask or install it only when the user says the application is critical or needs end-to-end tests against a real backend. Never add `page.route` as a stand-in for MSW. See `references/testing.md`.

Completion: project name, backend, UI choice, package manager, test scope, and quality-tool policy are known. New projects use Oxlint + Oxfmt unless explicitly changed.

### 2. Create the application

Use Vite with React + TypeScript and the selected package manager. Consult current documentation before using flags or commands that may have changed.

If the folder already exists but is empty, work inside it without creating an accidental nested directory.

Completion: the application exists with React, TypeScript, Vite, and the intended package manager.

### 3. Install the base

Install runtime dependencies:

```text
@tanstack/react-router
@tanstack/react-query
@tanstack/router-plugin
zustand
react-hook-form
@hookform/resolvers
zod
```

Add `axios` if the project consumes a REST API. Omit it when `backend: none`.

Install test tools:

```text
vitest
@testing-library/react
@testing-library/jest-dom
@testing-library/user-event
jsdom
msw
```

Add Tailwind using the current Vite integration. Do not hardcode versions; let the package manager resolve versions compatible with the project.

Completion: every installed dependency has a requested role and no duplicate solution was added.

### 4. Resolve the quality toolchain

Read `linting.md` and `formatting.md`. Review what the scaffold includes. If it already includes Biome, ESLint, Oxlint, Oxfmt, or Prettier, preserve that choice. If it includes none, use Oxlint + Oxfmt by default:

- Oxlint for lint plus Oxfmt for format.
- Biome for format and lint when explicitly selected.
- ESLint for lint; keep Oxfmt for format unless Prettier is explicitly selected too.

Configure only the selected combination:

- Biome: `biome.json` or `biome.jsonc`, with `lint`, `format`, and `format:check` scripts.
- ESLint: `eslint.config.*` or a configuration compatible with the installed version, plus its `lint` script.
- Oxlint: configuration compatible with the installed version, plus `lint` and optional `lint:fix` scripts.
- Oxfmt: configuration compatible with the installed version, plus `format` and `format:check` scripts.
- Prettier: compatible configuration, `.prettierignore`, and `format` and `format:check` scripts.

If Prettier is active, `.prettierignore` must cover `node_modules`, `dist`, `build`, `coverage`, and real generated artifacts. If Oxfmt is active, use its configuration ignore patterns. If neither tool is active, do not create formatter ignore files. Do not install duplicate tools or create ignores for tools that are not used.

Add Git hooks only when they are part of the requested standard; do not add a second lint or format chain.

Completion: one tool owns each quality responsibility, scripts are coherent, and linting does not also format files.

### 5. Configure the base

Configure:

- `@/*` in TypeScript pointing to `src/*`.
- The `@` alias in Vite.
- The TanStack Router plugin and route-tree generation.
- Strict TypeScript and unused-code checks.
- A `QueryClient` with conservative defaults: `refetchOnWindowFocus: false` and a stale time around two minutes, unless the product needs otherwise.
- `api-client.ts`, `query-client.ts`, `query-keys.ts`, and `env.ts` under `src/shared/lib/`.
- `vitest.config.ts` and MSW setup under `src/test/`. See `references/testing.md`.
- `.env.example` with variable names and never real secrets.

Keep route files thin and free of domain logic.

Completion: aliases, providers, query infrastructure, environment validation, and test setup have clear owners.

### 6. Resolve UI

If the user confirms shadcn/ui:

1. Initialize shadcn/ui with the current version.
2. Configure aliases so generated components live in `src/shared/components/ui/`.
3. Generate components only when a screen needs them.
4. Use Lucide for icons when it is the kit convention.
5. Treat generated components as shared base code: ask and wait for confirmation before editing an existing component.
6. For feature-specific customization, prefer composition, props, variants, or wrappers inside the feature.

If the user does not confirm:

1. Keep Tailwind CSS.
2. Create custom components in `src/shared/components/` or inside the corresponding feature.
3. Do not install or initialize shadcn/ui.

Completion: UI choice is explicit and generated base components are protected from unapproved edits.

### 7. Create minimum infrastructure

Create providers, the root route, query client, HTTP client, environment validation, and test setup. Do not create example stores, services, or schemas that do not belong to a requested feature.

Completion: the base runs without placeholder domain modules.

### 8. Verify

Run the project's actual scripts, at minimum:

```text
<package-manager> install
<package-manager> exec tsc --noEmit
<package-manager> run format:check
<package-manager> exec vitest run
<package-manager> run lint
<package-manager> run build
```

If `format:check` does not apply because no formatter is configured, omit it and report the decision. If a script does not exist, use its configured equivalent or report its absence. The scaffold is complete only when available TypeScript, format, tests, lint, and build checks pass, or an external failure is documented.

Completion: available checks pass and the final diff contains only requested scaffold and feature work.
