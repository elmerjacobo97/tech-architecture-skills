# New Next.js project scaffold

Read this reference only when creating a new Next.js App Router application.

## Target structure

```text
project-root/
├── src/
│   ├── app/
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   ├── globals.css
│   │   └── ...
│   ├── features/
│   │   └── <feature>/
│   │       ├── actions/
│   │       ├── components/
│   │       ├── hooks/
│   │       ├── schemas/
│   │       ├── services/
│   │       ├── types/
│   │       ├── utils/
│   │       └── store.ts
│   ├── components/
│   │   └── ui/                 # only after shadcn/ui confirmation
│   ├── hooks/
│   ├── lib/
│   │   ├── env.ts
│   │   ├── query-keys.ts       # only when query keys are needed
│   │   └── ...
│   ├── types/
│   └── test/
│       ├── handlers.ts
│       ├── server.ts
│       └── setup.ts
├── e2e/                        # only when requested or the app is critical
├── .env.example
├── <quality-config>
├── tsconfig.json
├── next.config.*
├── vitest.config.*             # when tests are enabled
└── package.json
```

Do not create empty feature folders without a real feature. Do not create `src/pages`, a second router, or barrels.

## Sequence

### 1. Confirm decisions

Confirm the project name, backend or server integration, test scope, and whether to install shadcn/ui. Use pnpm by default unless the user specifies another package manager. Tailwind is the default styling system for a new project.

Completion: name, package manager, UI choice, server integration, test scope, and quality-tool policy are known.

### 2. Create the application

Use the current `create-next-app` command and flags for TypeScript, App Router, `src`, Tailwind, and the selected package manager. Consult current Next.js documentation before running it; do not hardcode a stale command.

If the destination already exists but is empty, work inside it instead of creating an accidental nested directory. If it contains files, classify it as an existing project and switch to the existing-project reference.

Completion: the application starts with App Router, TypeScript, `src`, the intended package manager, and the chosen initial styling setup.

### 3. Install the base

Install runtime dependencies for a new project:

```text
zustand
react-hook-form
@hookform/resolvers
zod
```

React Hook Form and `@hookform/resolvers` are the form base: shadcn/ui forms with Zod run through `zodResolver` (see `forms.md`). Add TanStack Query only after an explicit request for client-side server-state management. Add Axios only when the user requests it; prefer `fetch` or the project's SDK.

Install test dependencies when tests are enabled:

```text
vitest
@testing-library/react
@testing-library/jest-dom
jsdom
msw
```

Do not add Playwright unless the user requests end-to-end tests or identifies the application as critical.

Completion: every installed dependency has a requested role and no duplicate client, state, UI, or test solution was added.

### 4. Resolve the quality toolchain

Inspect what `create-next-app` and the repository already provide. Preserve an existing choice. If no formatter or linter is present, ask which combination to use:

- Biome for format and lint.
- ESLint for lint plus Prettier for format.
- Oxlint for lint plus Prettier for format.

Configure only the selected combination and preserve the Next.js-compatible rules it needs. Add scripts only for active tools. Use the ignore mechanism supported by the installed versions.

Completion: one tool owns each quality responsibility, scripts are coherent, and no obsolete ignore file was introduced.

### 5. Configure the App Router base

Configure or verify:

- TypeScript strict mode and the project's `@/*` alias when it does not conflict.
- `src/app/layout.tsx`, `page.tsx`, and `globals.css`.
- A server-first root layout. Add client providers only when a consumer requires them.
- Environment validation in `src/lib/env.ts` using Zod, with private variables kept server-side.
- Test setup under `src/test/` when tests are enabled.
- A minimal `next.config.*` that contains only required project configuration.

Do not create domain services, example stores, example schemas, or empty features before a real feature needs them.

Completion: the root route renders, server/client boundaries are intentional, and configuration contains no unused scaffold residue.

### 6. Resolve UI

If the user confirms shadcn/ui:

1. Initialize it with the current official setup.
2. Keep generated primitives under `src/components/ui/`.
3. Generate components only when a feature needs them.
4. Prefer composition, variants, props, or feature wrappers for customization.
5. Ask before editing an existing generated component.
6. Do not write tests for generated primitives; test the project-owned components built on top.

If the user does not confirm:

1. Keep Tailwind CSS.
2. Place shared custom UI under `src/components/`.
3. Place domain-specific UI under its feature.
4. Do not initialize shadcn/ui.

Completion: UI choice is explicit and generated base components are protected from unapproved edits.

### 7. Add the first feature

Keep `src/app` route files focused on URL and composition. Put domain code in `src/features/<feature>/`. Add `actions/` only for Server Actions, `services/` for data access, `schemas/` for validation, and `store.ts` only when shared client state is real.

Use Server Components for server-side reads. Use Client Components only for browser interaction. Use Server Actions for UI mutations and Route Handlers for external HTTP contracts, webhooks, or client-only consumers.

Completion: the first feature has visible boundaries, validated inputs, and no unnecessary client boundary.

### 8. Verify

Run the project's actual scripts, at minimum when available:

```text
<package-manager> install
<package-manager> exec tsc --noEmit
<package-manager> run format:check
<package-manager> exec vitest run
<package-manager> run lint
<package-manager> run build
```

Use `next typegen` or the current Next.js type-generation command when the installed version provides it. Omit checks that have no configured equivalent and report the omission. The scaffold is complete only when available TypeScript, formatting, tests, lint, and build checks pass, or an external failure is documented.

Completion: available checks pass and the final diff contains only requested scaffold and feature work.
