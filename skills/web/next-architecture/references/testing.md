# Testing and Runtime Verification

Use this reference when enabling tests, adding a feature with behavior, or finishing a migration.

## Choose the test layer

| Behavior | Test layer |
| --- | --- |
| Pure utility, schema, store, synchronous service, or synchronous component | Vitest; use React Testing Library for component behavior. |
| Route Handler contract | Unit or integration test with real `Request`/`Response` objects and boundary validation. |
| Server Function validation and authorization | Unit/integration test the action or its server-only data access path. Include unauthorized, forbidden, invalid, and success cases. |
| Async Server Component | E2E or an installed Next-compatible integration path. Vitest and Jest currently do not fully support async Server Components. |
| Browser navigation, cookies, redirects, streaming, or critical user journey | Playwright or the project's existing browser runner. Keep the suite small and behavior-focused. |
| Mocked network boundary | MSW only when the test actually needs a mocked HTTP boundary. Prefer the real feature service for service tests. |

Do not install every test tool by default. Preserve the existing runner and setup in an existing project.

## Official tool boundaries

- Use Vitest plus React Testing Library for synchronous components, hooks, utilities, schemas, stores, and services. Follow the installed Next.js guide for `@vitejs/plugin-react`, `jsdom`, and TypeScript path resolution.
- Use Jest plus React Testing Library only when the project already uses Jest or requires its snapshot/configuration ecosystem. `next/jest` configures the Next.js compiler, styles, images, fonts, environment files, and `.next` exclusions.
- Use Playwright for browser behavior, async Server Components, redirects, cookies, streaming, loading states, and critical journeys. Run E2E against `next build` plus `next start` or a configured production-like web server when possible.
- Use the project's existing runner. Do not migrate Vitest to Jest or add Playwright only to test a pure function.

## Testing Next.js boundaries

- Keep route pages and layouts server-testable when they only compose synchronous JSX. Test route-specific metadata, params, `notFound()`, and redirects through focused server/integration tests or E2E when framework control flow is involved.
- Test Client Components that use `next/navigation` with the test runner's module mock (`vi.mock` or `jest.mock`). Mock only hooks the component uses, return stable values, expose spies for `push`/`replace`/`back`, and reset mocks between tests.
- Prefer passing destinations, search values, and callbacks as props to deeply interactive components. Keep `useRouter`, `usePathname`, and `useSearchParams` in the smallest adapter so most behavior tests do not need framework mocks.
- In real App Router tests, verify `useSearchParams` consumers inside their required `Suspense` boundary. A unit mock proves component behavior; it does not prove navigation, URL updates, focus, or prefetch behavior.
- Treat `redirect()` and `notFound()` as framework control flow. Assert destination or missing-resource behavior at an integration/E2E boundary; do not assert private error digests or implementation-specific thrown objects.
- Test modules that import `cookies()`/`headers()` from `next/headers` or import `server-only` in a Node/integration environment. Add a narrow test-only alias only when the runner cannot resolve a server marker; keep a boundary check that prevents private modules from entering the client graph.

Example Vitest navigation mock:

```ts
import { vi } from "vitest";

const router = vi.hoisted(() => ({
  push: vi.fn(),
  replace: vi.fn(),
  back: vi.fn(),
}));

vi.mock("next/navigation", () => ({
  useRouter: () => router,
  usePathname: () => "/dashboard",
  useSearchParams: () => new URLSearchParams("tab=overview"),
}));
```

Keep this mock local or in a focused test helper. Do not globally replace navigation behavior for tests that should exercise real routes.

## Experimental Next testmode

- Before adopting it, run `next experimental-test --help` through the project's package manager and confirm the installed Next.js version supports it. Do not assume it exists in Next.js 15.
- In supported Next.js 16 projects, `next experimental-test` currently supports the Playwright runner. The `next/experimental/testmode/playwright` entrypoint provides Playwright's `test`/`expect` plus a `next` fixture with `onFetch()` for intercepting server-side fetches. The `/msw` entrypoint combines the fixture with MSW.
- Keep test files under the runner's supported pattern, currently `{app,pages}/**/*.spec.{t,j}s`, only when using this testmode. Existing Playwright layouts may keep their own `tests/` convention.
- Use testmode fetch interception for controlled server-fetch responses that must be observed through a real browser route. Use ordinary MSW, fixtures, or a test database for unit/integration tests.
- Treat this API as experimental and partially documented. Do not make it the default test runner or add it only to replace stable Playwright coverage.

## Instant navigation regression tests

- Use `@next/playwright` `instant()` only when the project explicitly enables `cacheComponents` and configures the route segment's `instant` expectation.
- Use it to assert the UI available during hard or soft navigation before deferred content resolves. Use ordinary Playwright when `cacheComponents` is disabled.
- Do not enable `cacheComponents` or `instant` only to add a test; they are separate architecture decisions.

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
6. Run E2E against the production-like server when behavior depends on streaming, prefetching, redirects, cookies, or route transitions. Report dev-only verification separately.

If runtime verification is unavailable, report build-only verification and the missing signal. Do not claim that a route works from typecheck alone.

## Test style

- Test observable behavior through accessible roles, labels, text, and user interactions.
- Use `user-event` or the project's current interaction helper instead of implementation-level event simulation when available.
- Keep tests near the module under test; shared setup belongs in `src/test/`.
- For new modules, follow `structure.md`: colocate `foo.test.ts(x)` beside `foo`, use `__tests__/` only for small clusters, keep feature tests with their feature, and keep browser journeys in `e2e/` or the existing project convention.
- Use one representative runtime test for repeated patterns, then add coverage for meaningful differences.
