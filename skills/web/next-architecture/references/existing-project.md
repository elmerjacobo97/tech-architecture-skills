# Existing Next.js App Router Project

Read `versioning.md` and `next-platform.md` before editing. Audit first. Make the smallest change that clarifies ownership without replacing working systems.

## Preflight

1. Confirm `package.json` contains Next.js and locate `app` or `src/app` from the project root. If only `pages` or `src/pages` exists, stop and report the scope mismatch.
2. If both `app` and `src/app` exist, stop and resolve which tree is authoritative before editing.
3. Read `package.json`, lockfile, scripts, TypeScript config, Next config, environment declarations, ignore files, agent instructions, linter, formatter, tests, and deployment configuration.
4. Inspect route segments, layouts, providers, `use client` and `use server` boundaries, data access, auth, forms, state, UI kit, metadata, manifest, robots/sitemap, share images, icons, links, fonts, and existing error/loading files.
5. Run existing typecheck, lint, format check, tests, type generation, and build commands when possible. Record failures before changes.
6. Create a delta matrix with `rule`, `current state`, `risk`, `minimum change`, and `verification`.

Completion: App Router, app root, installed Next version, ownership boundaries, baseline checks, and intentional migration targets are explicit.

## Preservation policy

Preserve by default:

- URLs, route groups, layouts, metadata, loading states, error boundaries, and public behavior.
- Indexability policy, canonical URLs, sitemap and robots behavior, Open Graph/Twitter assets, icons, manifests, and public media paths.
- Existing Server Components, Client Components, providers, Server Functions, Route Handlers, and request interception.
- The current package manager, lockfile, scripts, aliases, linter, formatter, test runner, and deployment settings.
- Existing state, data-fetching, HTTP, ORM, SDK, styling, UI kit, auth, and validation systems.
- Useful route-local colocation and naming conventions.

Do not add a second library for the same responsibility. Do not migrate state, forms, data fetching, styling, UI kit, router, middleware/proxy, or cache model without an explicit request and scoped plan.

## Toolchain of record

Determine ownership from installed dependencies, scripts, and configuration:

| Evidence | Owner | Action |
| --- | --- | --- |
| Biome config, dependency, or scripts | Biome | Use Biome for format and lint. |
| ESLint config, dependency, or scripts | ESLint | Use ESLint for lint. Run its direct CLI on Next.js 16+. |
| Oxlint config, dependency, or scripts | Oxlint | Use Oxlint for lint. |
| Prettier config, dependency, or scripts | Prettier | Use Prettier for format. |
| No evidence | None | Report the gap and ask before installing a tool. |

Keep one owner per responsibility. Preserve multiple existing tools and report conflicts; consolidate only when explicitly requested. Do not add obsolete `next lint` scripts to Next.js 16 projects.

## Boundaries and ownership

- Keep route files in `app` or `src/app` focused on routing and composition.
- Let page files own route-specific metadata, `params`/`searchParams`, server reads, authentication/authorization checks, `notFound()`/redirects, and JSX. Do not move route-specific UI merely to make a page empty.
- Keep feature-specific actions, services, schemas, components, hooks, types, utilities, and tests together.
- Treat feature services that access private data as a server-only Data Access Layer. Add `server-only` when an accidental client import must fail.
- Make DAL functions authenticate, authorize the requested resource, select only needed fields, and return safe DTOs.
- Keep shared infrastructure independent from features. Features do not import one another directly.
- Keep Server Components as the default. Move only the smallest interactive leaf behind `use client` and keep its props serializable.
- Place context and providers as deep as their consumers allow.
- Keep `middleware.ts` for existing Next.js 15 projects. Use `proxy.ts` for new Next.js 16 code when the installed docs support it. Treat either as an optimistic request filter, never as the only security boundary.
- Keep Server Functions and Route Handlers independently authenticated and authorized. A page or layout guard does not protect them.

## Dependency adoption

Ask before adding missing base dependencies in an existing project:

| Area | Preserve | Add only for |
| --- | --- | --- |
| Client state | Current state solution | Shared client state that cannot remain local; use one small feature store. |
| Server state | Current data-fetching solution | Explicit client-side server-state requirements. |
| Forms | Current forms and validation | New validated forms; use React Hook Form, resolver, and Zod when complexity warrants them. |
| Validation | Current schemas | New input, environment, or external-data boundaries. |
| HTTP/data | `fetch`, SDK, Axios, ORM, or existing client | Explicit integration need. |
| UI | Current UI kit and styling | Explicit shadcn/ui or styling decision. |
| Tests | Current runner and setup | New coverage; add MSW only for mocked network boundaries and Playwright for async RSC or real journeys. |

## Migration order

### Phase 1: configuration and docs

- Resolve the installed Next.js version and read `versioning.md` plus local version-matched docs.
- Read `next-platform.md` and record whether routes are public, private, or mixed before changing metadata or crawl files.
- Confirm package manager, lockfile, scripts, aliases, strict TypeScript, Next config, and quality-tool ownership.
- Preserve current configuration unless a missing setting blocks the requested work.

Completion: baseline still runs and version-specific conventions are selected from installed evidence.

### Phase 2: server and client boundaries

- Inventory `use client`, `use server`, server-only modules, environment access, providers, Server Functions, Route Handlers, and middleware/proxy.
- Move only clearly server-only logic away from client module graphs.
- Validate untrusted inputs at the server boundary and authorize inside every mutation and handler.
- Keep rendering free of database writes, cookie mutations, cache invalidation, and other side effects.

Completion: no private module or secret crosses into the client graph, and each mutation has server-side validation and authorization.

### Phase 3: infrastructure

- Consolidate environment parsing without changing public variable names.
- Keep data access in feature services or the existing client boundary.
- Preserve cache and revalidation semantics. Do not enable or adopt Cache Components as a side effect of reorganization.
- Add query keys only when the project already uses a query library. Add MSW setup only when API coverage uses it.

Completion: no duplicated clients, providers, stores, validators, query systems, or test infrastructure exist.

### Phase 4: one tracer feature

Migrate one complete feature before repeating:

```text
features/<feature>/
├── actions/
├── components/
├── hooks/
├── schemas/
├── services/
├── types/
└── utils/
```

- Preserve route URLs, auth behavior, cache behavior, loading/error states, and public contracts.
- Move domain UI, data access, schemas, Server Functions, and tests together.
- Keep server reads on the server and client behavior at the smallest boundary.
- Extract shared code only after a second real consumer exists.

Completion: one feature passes behavior and available checks with visible boundaries and no unjustified cross-feature dependency.

### Phase 5: repeat with controlled scope

Repeat feature by feature. After each feature, run typecheck, relevant tests, lint, format check, type generation, and build when configured. Use runtime verification for changed routes when possible.

## Security rules

- Validate `FormData`, JSON, route params, search params, cookies, headers, and external responses before use.
- Re-check authentication and authorization inside every Server Function and Route Handler, including resource ownership and role checks.
- Keep private environment variables, server SDKs, database clients, and session secrets server-only.
- Return minimal action payloads and DTOs. Do not serialize raw database records or sensitive error details.
- Verify webhook signatures against the raw request body before parsing or acting.
- Use expected errors as structured return values and unexpected failures through error boundaries. Do not hide errors with broad catches.
- Consider rate limiting for expensive or externally reachable mutations.
- Do not change URLs, API contracts, persistence, auth, or cache behavior as a side effect of folder reorganization.

## Final verification

Run configured typecheck, lint, format check, tests, Next.js type generation, and build. Use the installed package manager and scripts. Run a runtime route check when possible; use `next-dev-loop` only when its version and Turbopack requirements are met.

Review the diff and confirm:

- No Pages Router layer or duplicate app tree was introduced.
- No unnecessary `use client` boundary or private import reached the client graph.
- No feature cycles, new speculative barrels, duplicated tools, or unused dependencies were introduced.
- Existing package manager, scripts, URLs, auth, persistence, cache, loading, error, and public behavior remain intact unless explicitly changed.
- Public metadata, canonical URLs, robots/sitemap policy, share images, icons, manifests, and image/link conventions remain intact unless explicitly changed.

Completion: available checks pass, exceptions are documented, and diff contains only requested adoption work.
