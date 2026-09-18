# Next.js Version and Agent Workflow

Read this reference before using a framework API, CLI flag, file convention, or cache behavior.

## Resolve the version

1. Read `package.json` and the lockfile. Use the installed `next` version as the source of truth.
2. Resolve the project root from `next.config.*`, then locate `app` or `src/app`. Do not infer the root from the agent's current directory.
3. Read root `AGENTS.md`, `CLAUDE.md`, and other project instructions before editing. Preserve managed sections.
4. For Next.js 16.2+, read the version-matched documentation under `node_modules/next/dist/docs/` when available. For Next.js 16.3+, expect Next.js to manage an `AGENTS.md` block in projects that enable agent rules.
5. If local documentation is unavailable or a third-party library is involved, resolve Context7's library first and then query its docs. Do not mix APIs from different Next.js generations.

When sources disagree, prefer this order: installed package documentation, project configuration and scripts, official documentation matching the installed version, Context7 current documentation, then memory. Stop and report an unresolved conflict instead of guessing.

## Compatibility matrix

| Installed version | Use | Gate |
| --- | --- | --- |
| Next.js 15+ | App Router, async `params` and `searchParams`, Server Functions, Route Handlers, Server Components | Await route params and request APIs where the installed types require it. |
| Next.js 15.x | `middleware.ts` remains the existing request interception convention | Preserve existing middleware. Do not rename it unless the target version and request explicitly support migration. |
| Next.js 16+ | `proxy.ts` replaces and deprecates the `middleware.ts` convention; direct ESLint or Biome scripts | Use `proxy.ts` for new code when the installed docs support it. `next build` no longer runs lint automatically. |
| Next.js 16+ | Cache Components and `use cache` when enabled by project config and installed version | Treat `cacheComponents: true` as an explicit migration decision. Do not add it during ordinary scaffolding or folder cleanup. |
| Next.js 16.3+ | Bundled agent docs, managed agent rules, Next.js MCP, and the official `next-dev-loop` workflow | Use bundled docs and runtime verification when available. `next-dev-loop` requires Next.js 16.3+ and Turbopack. |

## Version-sensitive rules

- Type dynamic route `params` and `searchParams` from the installed Next.js types. In Next.js 15+, the current App Router shape is promise-based; do not copy synchronous examples from older docs.
- Type Route Handler `context.params` as a promise when required. Use generated `RouteContext` only after the installed project exposes it through `next typegen` or the current equivalent.
- Treat `use server` as a Server Function directive. It does not mark a component as a Server Component; App Router components are server components by default unless marked `use client`.
- Keep `fetch` caching, `use cache`, `cacheLife`, `cacheTag`, `revalidatePath`, and `revalidateTag` choices tied to the installed version and project cache configuration. Record freshness and invalidation intent near the data access code.
- Treat Proxy as an early optimistic filter, not the only authentication or authorization layer. Enforce access again in the data layer, Server Function, and Route Handler.
- Use React Compiler only when the user or existing project configuration selects it. Do not add manual memoization as a substitute for understanding state, purity, or measured performance.

## Agent verification

- Prefer a running `next dev` loop for changed routes when the project provides a browser or Next.js MCP workflow.
- For Next.js 16.3+ with Turbopack, use `next-dev-loop` when installed. Cross-check compilation issues, server errors, browser console, DOM behavior, and relevant React boundaries.
- If runtime tooling is unavailable, run the project's scripts and `next build`; report that runtime behavior was not checked.
- Never delete or move `.next` while a running development server depends on it. Use a separate `distDir` for isolated production output.
