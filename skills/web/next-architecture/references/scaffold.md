# New Next.js App Router Project

Read `versioning.md` and `next-platform.md` first. Create a new project only after the project name, target Next.js version, package manager, server integration, UI choice, test scope, indexability policy, and quality-tool policy are known.

## Target structure

```text
project-root/
├── src/
│   ├── app/                       # routes, layouts, metadata, special files when needed
│   ├── features/<feature>/       # domain-owned code, created when needed
│   │   ├── actions/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── schemas/
│   │   ├── services/              # server data access when feature-owned
│   │   ├── types/
│   │   ├── utils/
│   │   └── store.ts               # only for shared client state
│   ├── components/                # genuinely shared UI
│   ├── hooks/                     # genuinely shared hooks
│   ├── lib/                       # shared clients, env, utilities
│   ├── types/                     # genuinely shared types
│   └── test/                      # setup only when tests are enabled
├── public/                        # only when static assets exist
├── .env.example
├── next.config.*
├── tsconfig.json
└── package.json
```

Do not create empty feature folders, example stores, example schemas, domain services, or test handlers before a real feature needs them. Do not create `src/pages` or barrel files.

Route-local colocation is valid when code belongs to one segment. Use `features/` for reusable domain ownership across routes. Do not move code only to satisfy a folder diagram.

## Sequence

### 1. Create the app

Use the `create-next-app` command and flags documented for the selected Next.js version. Select TypeScript, App Router, `src`, Tailwind, the chosen linter, and the existing alias convention. Use Turbopack when supported and not explicitly declined.

Use `AGENTS.md` and `CLAUDE.md` generation when the selected Next.js version supports it. Never overwrite project instructions without preserving user content.

If the destination contains files, classify it as an existing project and switch to `existing-project.md`. If it is empty, work inside it instead of creating an accidental nested directory.

### 2. Install only requested base

Use this policy:

| Need | Default |
| --- | --- |
| Boundary validation | Add Zod for new projects that validate input or environment. |
| Validated client forms | Add React Hook Form and `@hookform/resolvers` when the first such form exists. |
| Shared client state | Add Zustand only when multiple components need shared client state that cannot remain local. |
| Client server state | Add TanStack Query only for an explicit client-side server-state use case. |
| HTTP | Use native `fetch` or the existing SDK. Add Axios only for an explicit requirement. |
| Unit/component tests | Add Vitest, React Testing Library, `@testing-library/dom`, `@testing-library/jest-dom`, jsdom, and the TypeScript/Vite adapters required by the installed versions. |
| API mocks | Add MSW only when tests need mocked network boundaries. |
| Browser tests | Add Playwright for async Server Components, critical user journeys, or an explicit E2E request. |

Do not add icons, date libraries, toast libraries, i18n, Sentry, CI, ORM packages, provider SDKs, or a second state, HTTP, UI, test, linter, or formatter tool without a concrete use case.

### 3. Configure the base

- Keep TypeScript strict and configure `@/*` only when it does not conflict with the selected alias.
- Keep the root layout a Server Component. Add client providers only for real consumers and place them as deep as possible.
- Validate private environment variables in a server-only module such as `src/lib/env.ts`. Keep public variables explicitly prefixed according to Next.js rules.
- Keep `next.config.*` minimal. Add `cacheComponents`, experimental flags, runtime settings, or provider configuration only for a stated requirement and supported version.
- Choose public versus private indexability before adding `metadata`, `robots.ts`, `sitemap.ts`, Open Graph/Twitter images, manifests, or icons. Create only files required by that policy and product behavior.
- Add `src/test/` setup only when tests are enabled.

### 4. Add the first feature

Keep `src/app` route files focused on URL-specific composition. A page may own metadata, `params`/`searchParams`, server reads, authentication/authorization checks, `notFound()`/redirects, and route-specific JSX. Put reusable domain UI, client interaction, schemas, Server Functions, and feature tests in the owning feature. Do not make a page empty only to satisfy a folder diagram. Use Server Components for server reads, Client Components for browser interaction, Server Functions for UI mutations, and Route Handlers for public HTTP contracts, webhooks, integrations, or client-only consumers.

Place auth and authorization in the data access path, not only in the page. Return minimal DTOs and structured expected errors. Revalidate or redirect after successful mutations when affected UI requires it.

### 5. Resolve UI

If shadcn/ui is confirmed, initialize it using the current official setup and generate primitives only when a feature needs them. Generate `field` before a validated shadcn form and add `input-group` only for grouped controls. Ask before editing an existing generated component.

Without shadcn/ui, keep Tailwind and place shared custom UI in `src/components/` and domain UI in its feature.

### 6. Verify

Run the project's actual scripts and omit unavailable checks:

```text
<package-manager> exec tsc --noEmit
<package-manager> run format:check
<package-manager> run test -- --run
<package-manager> run lint
<package-manager> exec next typegen
<package-manager> run build
```

Use the installed version's current type-generation command. Run `next dev` and verify one real route when runtime tooling is available. For Next.js 16.3+ with Turbopack, use `next-dev-loop` when available. Report external failures separately from failures introduced by the scaffold.

For public routes, inspect rendered metadata, canonical URLs, robots policy, sitemap entries, share images, icons, internal links, and responsive images. For private routes, confirm they are not included in crawlable metadata or sitemap output.

Completion: every dependency has a role, root route renders, available checks pass, and no unused scaffold residue remains.
