---
name: next-architecture
description: "Use when Next.js 15+ App Router projects need architecture, scaffolding, or migration with Server Components, Server Actions, Zod, Tailwind, or shadcn/ui."
license: MIT
metadata:
  author: elmerjacobo97
  version: "2.0.0"
---

# Next.js App Router Architecture

## Activation Contract

- Use for new or existing Next.js 15+ App Router projects with TypeScript.
- Use for architecture, scaffolding, and migration, not ordinary feature work.
- Stop for Pages-only projects; preserve hybrid projects unless migration is explicit.
- Load `references/versioning.md` before version-sensitive work.
- Load `references/structure.md` before scaffolding, reorganizing files, or choosing test locations.
- Load `references/linting.md` before choosing, configuring, or migrating a linter.
- Load `references/formatting.md` when Prettier is requested or detected.

## Hard Rules

1. **Version first.** Detect Next.js, package manager, app root, scripts, and instructions. Prefer installed docs; use Context7 for missing docs.
2. **Route composition.** Let `page.tsx` own route-specific metadata, `params`/`searchParams`, server reads, authentication/authorization checks, `notFound()`, redirects, and JSX. Keep reusable domain UI and client interaction in features. Never add `pages` or an empty wrapper by dogma.
3. **Server first.** Add `use client` only at the smallest interactive boundary. Keep providers deep, props serializable, secrets server-only, and private modules behind `server-only`.
4. **Feature ownership.** Features own UI, actions, services, schemas, hooks, types, utilities, and stores. Reusable infrastructure lives under `src/shared/` and never imports features; features do not import each other.
5. **Safe data.** Server Components call services or SDKs directly, not internal Route Handlers. Authenticate, authorize, validate input, and return minimal DTOs.
6. **Explicit freshness.** Choose request-time, cache, and revalidation behavior per data source. Do not assume historical `fetch` defaults.
7. **Clean React.** Keep render pure. Derive values instead of redundant state. Keep event work in handlers or Server Functions. Use Effects only for external synchronization.
8. **Minimal dependencies.** Prefer native `fetch` and existing tools. Use ESLint as the new-project linter default, honor an explicit Oxlint or Biome choice, and preserve the existing quality toolchain in existing projects. Add Zustand, TanStack Query, Axios, MSW, Playwright, or duplicate tools only for concrete use cases. Configure Prettier only when requested or already present. Use Zod at server boundaries and React Hook Form when needed.
9. **Security.** Parse untrusted input and re-check authentication, authorization, and ownership inside every Server Function and Route Handler.
10. **Predictable structure.** Use direct imports, `kebab-case` names except framework-reserved files, colocated tests, no speculative barrels, and no unapproved generated UI edits. Follow `references/structure.md`.

## Invocation and Decision Gates

Interpret text after `/next-architecture` by convention:

| Argument | Effect |
| --- | --- |
| `name`, `existing` | Name or existing-project mode |
| `ui`, `backend`, `query`, `tests`, `react-compiler` | Explicit choices; no optional tools by default |
| `eslint`, `oxlint`, `biome`, `lint`, `quality` | Select or inspect the requested linter; load `references/linting.md` |
| `prettier`, `format` | Configure Prettier when requested; load `references/formatting.md` |
| `package-manager` | `pnpm`, `npm`, `yarn`, or `bun`; preserve existing projects |

| Situation | Action |
| --- | --- |
| New project | Load `scaffold.md`; ask missing decisions |
| Existing project | Load `existing-project.md`; baseline first |
| SEO, special files, images, links, fonts, or scripts | Load `next-platform.md` |
| Files, folders, names, or test placement | Load `structure.md` |
| ESLint, Oxlint, Biome, lint, or linter scripts | Load `linting.md` |
| Prettier, Prettier config, or Prettier scripts | Load `formatting.md` |
| Forms, tests, or version-sensitive APIs | Load matching reference |

## Execution Steps

1. Classify project, app root, version, package manager, and baseline checks.
2. Read local instructions, matching references, and version-matched docs.
3. Implement the smallest base or complete feature. Preserve URLs, contracts, auth, cache behavior, scripts, and working tools.
4. Verify each step. Review boundaries, dependencies, cycles, secrets, and unrelated changes.

## Output Contract

Return classification, version gates, decisions, changed files, checks, preserved systems, and unresolved exceptions.

## References

- `references/versioning.md` - Next 15+/16/16.3+ compatibility and agent workflow.
- `references/structure.md` - Feature, shared, naming, import, and test-placement rules.
- `references/next-platform.md` - SEO, route file conventions, images, fonts, links, scripts, and verification.
- `references/linting.md` - ESLint default, existing-tool preservation, rule policy, ignores, scripts, and version gates.
- `references/formatting.md` - Optional Prettier setup, ignore rules, package scripts, and verification.
- `references/scaffold.md` - New-project setup and dependency policy.
- `references/existing-project.md` - Existing-project audit and migration.
- `references/forms.md` - Server Functions, React Hook Form, Zod, and shadcn/ui forms.
- `references/testing.md` - Test layers and runtime verification.
