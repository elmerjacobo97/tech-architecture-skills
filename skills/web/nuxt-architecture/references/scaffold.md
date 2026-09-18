# New Nuxt project scaffold

Read this reference only when creating a new Nuxt application. Nuxt 4 is the default.

## Target structure

```text
project-root/
├── app/
│   ├── app.vue
│   ├── app.config.ts
│   ├── assets/
│   ├── components/
│   │   └── ui/                 # only after shadcn-vue confirmation
│   ├── composables/
│   ├── features/
│   │   └── <feature>/
│   │       ├── components/
│   │       ├── composables/
│   │       ├── schemas/
│   │       ├── services/
│   │       ├── stores/
│   │       ├── types/
│   │       └── utils/
│   ├── layouts/
│   ├── middleware/
│   ├── pages/
│   ├── plugins/
│   └── utils/
├── server/
│   ├── api/
│   ├── features/
│   ├── middleware/
│   ├── plugins/
│   └── utils/
├── shared/
├── public/
├── .env.example
├── <quality-config>
├── nuxt.config.ts
├── vitest.config.ts            # when tests are enabled
└── package.json
```

Keep `shared/` runtime-neutral. Do not create empty feature folders, a second router, or duplicate Nuxt reserved directories.

## Sequence

### 1. Confirm decisions

Confirm the project name, server integration, test scope, and whether to install shadcn-vue. Use pnpm by default unless the user specifies another package manager. Tailwind is the default styling system. Use Nuxt 4 for new projects unless the user selects Nuxt 3.

Completion: name, Nuxt version, package manager, UI choice, server integration, test scope, and quality-tool policy are known.

### 2. Create the application

Use the current `nuxi` initialization command and flags for TypeScript, Nuxt 4, and the selected package manager. Consult current Nuxt documentation before running it; do not hardcode a stale command or assume the Nuxt 3 directory layout.

If the destination already exists but is empty, work inside it instead of creating an accidental nested directory. If it contains files, classify it as an existing project and switch to the existing-project reference.

Completion: the application starts with the intended Nuxt version, TypeScript, App directory convention, and package manager.

### 3. Install the base

Install runtime dependencies for a new project:

```text
@pinia/nuxt
pinia
vee-validate
@vee-validate/zod
zod
```

Add Tailwind using the current official Nuxt integration. Do not add Axios or `@tanstack/vue-query` by default. Add TanStack Vue Query only after an explicit request for client-side server-state management.

Install test dependencies when tests are enabled:

```text
vitest
@vue/test-utils
@testing-library/vue
happy-dom
msw
```

Keep `@nuxt/test-utils` opt-in for integration tests that need a running Nuxt application. Keep Playwright opt-in for end-to-end tests.

Completion: every installed dependency has a requested role and no duplicate state, data, UI, or test solution was added.

### 4. Resolve the quality toolchain

Inspect what `nuxi` and the repository provide. Preserve an existing choice. If no formatter or linter is present, ask which combination to use:

- Biome for format and lint.
- ESLint for lint plus Prettier for format.
- Oxlint for lint plus Prettier for format.

Configure only the selected combination and preserve Nuxt-compatible rules. Add scripts only for active tools. Use the ignore mechanism supported by the installed versions.

Completion: one tool owns each quality responsibility, scripts are coherent, and no obsolete ignore file was introduced.

### 5. Configure the Nuxt base

Configure or verify:

- TypeScript strict mode and the current Nuxt 4 `app/` directory convention.
- `app/app.vue`, `app/pages`, `app/layouts`, and app-level `assets` when needed.
- A minimal `nuxt.config.ts` with only required modules and configuration.
- `@pinia/nuxt` in `modules` when Pinia is installed.
- Server-first rendering and no unnecessary client-only wrapper.
- Private runtime configuration from environment variables, with public values under `runtimeConfig.public` only when they are safe to expose.
- Environment validation in a server-only module using Zod when the project has required environment variables.
- Test setup under `test/` or the project's chosen location when tests are enabled.
- `.env.example` with variable names and never real secrets.

Do not create example stores, services, schemas, composables, or empty features before a real feature needs them.

Completion: the root route renders, Nuxt modules are intentional, runtime config is safe, and configuration contains no unused scaffold residue.

### 6. Resolve UI

If the user confirms shadcn-vue:

1. Initialize it with the current official setup.
2. Keep generated primitives under `app/components/ui/` or the configured components directory.
3. Generate components only when a feature needs them.
4. Prefer composition, variants, props, or feature wrappers for customization.
5. Ask before editing an existing generated component.

If the user does not confirm:

1. Keep Tailwind CSS.
2. Place shared custom UI under `app/components/`.
3. Place domain-specific UI under its feature.
4. Do not initialize shadcn-vue.

Completion: UI choice is explicit and generated base components are protected from unapproved edits.

### 7. Add the first feature

Keep `app/pages` route files focused on URL, page metadata, page-level orchestration, and composition. Put frontend domain code in `app/features/<feature>/`. Put private server behavior in `server/features/<feature>/` and keep `server/api` handlers thin.

Use `useFetch` or `useAsyncData` for page and component data. Use `$fetch` directly in server services. Use Pinia only for shared client state. Use vee-validate and Zod for interactive forms, then validate again in Nitro handlers.

Completion: the first feature has visible app/server boundaries, stable async keys, validated inputs, and no unnecessary client-only code.

### 8. Resolve app and server boundaries

Review every plugin, composable, runtime config value, middleware file, and server handler:

- App middleware handles navigation concerns.
- Server middleware handles request concerns.
- App plugins remain focused on app integrations.
- Nitro plugins remain focused on server integrations.
- Private modules and secrets remain in the server graph.
- `shared/` contains only runtime-neutral code.

Completion: no browser bundle receives private server code or secrets, and no boundary is handled by the wrong Nuxt directory.

### 9. Verify

Run the project's actual scripts, at minimum when available:

```text
<package-manager> install
<package-manager> exec vue-tsc --noEmit
<package-manager> run format:check
<package-manager> exec vitest run
<package-manager> run lint
<package-manager> run build
```

Use the current Nuxt typecheck command when the installed version provides it. Use `@nuxt/test-utils` only for explicitly requested Nuxt integration tests. Omit checks that have no configured equivalent and report the omission. The scaffold is complete only when available typecheck, formatting, tests, lint, and build checks pass, or an external failure is documented.

Completion: available checks pass and the final diff contains only requested scaffold and feature work.
