# Next.js Platform Conventions

Read `versioning.md` first. Apply only conventions supported by the installed Next.js version and required by the route. Do not create special files as empty checklist items.

## Metadata and SEO

- Use the Metadata API from `next`. Use a static `metadata` export when values are known at build time and `generateMetadata` for route data. Do not export both from one file.
- Set `metadataBase` once in the root layout from the canonical public origin. Use a title template, default description, `<html lang>`, and site-wide Open Graph/Twitter defaults there; override route-specific values in the closest page or layout.
- Await `params` and `searchParams` in dynamic metadata functions when required by the installed version. Preserve parent metadata when extending nested Open Graph or Twitter images.
- Set `alternates.canonical` for public canonical URLs. Do not build canonical URLs from untrusted request input or expose internal preview origins.
- Use `robots.ts` for site-wide crawl policy and the Metadata API's `robots` fields for route-specific private, authenticated, preview, or staging pages. Do not rely on `robots.txt` alone to produce `noindex`; crawlers must be able to fetch a page to see its meta robots directive. Use `sitemap.ts` only for public canonical URLs that should be crawled. Generate dynamic entries from the same source of truth as public content; never include user-specific or private URLs.
- Use `opengraph-image` and `twitter-image` file conventions or Metadata fields for share previews. Keep generated images deterministic, public, and version-gated. Do not make client-only state a prerequisite for SEO metadata.
- Render JSON-LD in a Server Component when structured data improves a public page. Validate its values and escape `<` before placing serialized JSON in a script tag. Do not expose secrets or claim schema types the page does not support.
- Add `manifest.ts`, `icon.tsx`, `icon.svg`, `apple-icon`, or equivalent files only when the product needs installability, browser icons, or platform metadata. Keep icon paths real and verify them in production.
- Do not use `next/head`, `react-helmet`, or ad hoc document-head mutation in App Router routes.

Example static metadata:

```tsx
import type { Metadata } from "next";

export const metadata: Metadata = {
  title: "About",
  description: "Public description for this route.",
  alternates: { canonical: "/about" },
};
```

Example dynamic metadata:

```tsx
import type { Metadata } from "next";

export async function generateMetadata({
  params,
}: {
  params: Promise<{ slug: string }>;
}): Promise<Metadata> {
  const { slug } = await params;
  const item = await getPublicItem(slug);

  return {
    title: item.title,
    description: item.description,
    alternates: { canonical: `/items/${item.slug}` },
  };
}
```

## Route File Conventions

| File | Rule |
| --- | --- |
| `layout.tsx` | Persistent shell, providers, and inherited metadata. Keep it server-side unless a real consumer requires a client boundary. |
| `loading.tsx` | Instant segment fallback. Use a lightweight, accessible skeleton that matches the route shell. It is a Server Component by default. |
| `error.tsx` | Segment error boundary. It must be a Client Component, show a safe message, log through the project mechanism, and expose the recovery callback from the installed version (`retry` in current Next.js 16 docs). |
| `global-error.tsx` | Last-resort root error UI. It must be version-supported, client-side, and define its own `html` and `body`. |
| `not-found.tsx` | UI for `notFound()` in its segment. Keep it accessible and do not leak lookup details. |
| `global-not-found.tsx` | Use only when supported and needed for unmatched routes outside the normal layout; define its own document shell. |
| `template.tsx` | Use only when a segment must remount on navigation. Do not replace `layout.tsx` by default. |
| `route.ts` | Public HTTP boundary for webhooks, integrations, or non-UI consumers. Authenticate and validate independently from page or layout guards. |

- Model expected validation and business failures as values in Server Functions. Reserve `error.tsx` for uncaught render or server failures.
- Place `notFound()` before a boundary that may stream when a correct 404 status matters. Streaming can commit a `200` response; Next.js still emits `noindex` for streamed not-found UI.
- Put special files at the narrowest segment that owns their behavior. Do not create global fallbacks when a route-local boundary is sufficient.
- Keep loading UI server-renderable and error fallback controls usable after hydration. Include labels, focus-visible controls, and keyboard access.

## Images and Fonts

- Use `next/image` for content and layout images. Provide meaningful `alt`; use `alt=""` for decorative images.
- Provide both `width` and `height`, use a static import, or use `fill` inside a positioned parent. Add `sizes` whenever CSS makes an image responsive or `fill` is used.
- Use the installed version's current LCP option. Next.js 16 uses `preload`; older versions may document `priority`. Set it for one likely above-the-fold LCP image, not every image.
- Keep remote image configuration strict with `images.remotePatterns`. Do not use deprecated broad `domains`, unrestricted wildcard hosts, or user-controlled image URLs without validation.
- Avoid `unoptimized` except for a known reason such as a small SVG, animation, or an image service that already optimizes output. Treat `dangerouslyAllowSVG` as a security decision requiring a restrictive policy.
- Use `placeholder="blur"` only with a real `blurDataURL` or a static import that provides one. Do not trade layout stability for decorative loading effects.
- Use `next/font` for project fonts. Keep font loading in a layout or shared boundary; do not add CSS font imports that create avoidable layout shift.

## Links and Navigation

- Use `next/link` for internal application routes. Use native `<a>` for external URLs, downloads, email, telephone, or other browser protocols.
- Give links descriptive accessible names. Do not use click handlers or `window.location` for ordinary internal navigation.
- Keep default prefetch behavior unless a measured network or privacy constraint requires `prefetch={false}`. Do not prefetch user-specific or expensive routes without a reason.
- Use `redirect()` in Server Components, Server Functions, or Route Handlers when server control flow is required. Use `useRouter` only for client event-driven navigation.
- Validate dynamic destinations and preserve same-origin constraints. Never pass untrusted URLs into redirects or external links without an explicit policy.
- Keep route changes compatible with loading, error, not-found, focus, and back-button behavior.

## Scripts and Verification

- Use `next/script` for third-party scripts and select a loading strategy from the installed docs. Load only scripts required by the route or product.
- Use a plain `<script type="application/ld+json">` only for server-rendered JSON-LD. Do not use raw scripts for third-party code when `next/script` applies.
- Verify rendered title, description, canonical, robots, Open Graph/Twitter tags, JSON-LD, manifest, and icon URLs for public routes.
- Verify public sitemap entries exclude private routes and that `robots.txt` matches the indexability policy.
- Visit routes with JavaScript disabled when progressive rendering or SEO matters. Check image alt text, dimensions, responsive `sizes`, internal link navigation, focus behavior, loading UI, and error recovery.
