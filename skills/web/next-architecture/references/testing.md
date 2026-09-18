# Testing and Runtime Verification

Use this reference when enabling tests, adding a feature with behavior, or finishing a migration.

## Choose the test layer

| Behavior | Test layer |
| --- | --- |
| Pure utility, schema, store, synchronous service, or synchronous component | Vitest; use React Testing Library for component behavior. |
| Route Handler contract | Unit or integration test with real `Request`/`Response` objects and boundary validation. |
| Server Function validation and authorization | Unit/integration test the action or its server-only data access path. Include unauthorized, forbidden, invalid, and success cases. |
| Async Server Component | E2E or an installed Next-compatible integration path. Vitest currently does not fully support async Server Components. |
| Browser navigation, cookies, redirects, streaming, or critical user journey | Playwright or the project's existing browser runner. Keep the suite small and behavior-focused. |
| Mocked network boundary | MSW only when the test actually needs a mocked HTTP boundary. Prefer the real feature service for service tests. |

Do not install every test tool by default. Preserve the existing runner and setup in an existing project.

## What to verify

- Forms: valid submit, server validation error, pending state, accessible error output, reset, disabled state, and no duplicate submit.
- Dialogs: open/close, action-specific labels, defaults, mutation error, pending state, and success close.
- Server Functions and Route Handlers: input parsing, authentication, resource authorization, safe return values, failure status, and cache invalidation.
- Server/client boundaries: no private import reaches a Client Component; only serializable minimal props cross the boundary.
- Data loading: loading UI, error UI, not-found behavior, redirect behavior, parallel requests, and expected cache freshness.
- Routes: changed URL works in a production-like build and important client navigation still renders the intended shell and streamed content.
- SEO: title, description, canonical, robots, sitemap inclusion, Open Graph/Twitter metadata, JSON-LD, manifest, and icon URLs match route policy.
- Platform files: `loading.tsx` renders an accessible fallback, `error.tsx` recovers safely, and `not-found.tsx` handles missing resources without leaking details.
- Media and navigation: `next/image` has meaningful alt text, stable dimensions, responsive sizes, allowed sources, and correct LCP behavior; internal links use `next/link` and external links use `<a>`.

## What not to verify

- Generated shadcn/ui primitives or third-party internals.
- Static copy, class-name spelling, implementation details, or large DOM snapshots.
- Thin wrappers that only forward to a tested service or query library.
- The same business rule in both application and backend test suites when the backend owns that rule.
- A mocked browser journey when a direct unit or integration test proves the behavior more cheaply.

## Runtime loop

After meaningful route or boundary changes:

1. Run the project's typecheck or dev compiler.
2. Visit the changed route with `next dev` when possible.
3. Check server logs, browser console, visible behavior, metadata, loading/error boundaries, not-found behavior, and client navigation.
4. For Next.js 16.3+ with Turbopack, use the official `next-dev-loop` workflow when installed. It combines `/_next/mcp` compilation/runtime data with browser inspection.
5. Run configured tests and `next build` before finishing. Run `next typegen` or the installed equivalent when available.

If runtime verification is unavailable, report build-only verification and the missing signal. Do not claim that a route works from typecheck alone.

## Test style

- Test observable behavior through accessible roles, labels, text, and user interactions.
- Use `user-event` or the project's current interaction helper instead of implementation-level event simulation when available.
- Keep tests near the module under test; shared setup belongs in `src/test/`.
- Use one representative runtime test for repeated patterns, then add coverage for meaningful differences.
