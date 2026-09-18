---
name: next-architecture
description: Use when creating, scaffolding, structuring, or incrementally migrating Next.js App Router projects with feature-based architecture, Server Components, Server Actions, Route Handlers, React Hook Form, Zod, Tailwind CSS, or shadcn/ui.
---

# Structured Next.js App Router

Skill for defining the technical base, architecture, and incremental evolution of Next.js applications using the App Router. Organize by vertical feature, not by global technical layer.

## Scope

- Applies to Next.js applications using the App Router and TypeScript.
- Covers two modes: new project and existing project.
- New projects receive a coherent base with React Hook Form, Zod, Tailwind CSS, and tests; client state remains opt-in.
- Existing projects are audited first and current decisions are preserved.
- React SPA/Vite projects and Next.js Pages Router projects are outside this skill. Stop and use the appropriate guide when detected.

## Invocation and arguments

The text after `/next-architecture` is available as `ARGUMENTS`. Interpret these keys by convention:

| Key               | Values                               | Effect                                                                                      |
| ----------------- | ------------------------------------ | ------------------------------------------------------------------------------------------- |
| `name`            | text                                 | Name used when creating a new project                                                       |
| `existing`        | no value                             | Forces existing-project mode                                                                |
| `ui`              | `shadcn` / `tailwind`                | Avoids asking about UI in a new project                                                     |
| `backend`         | `server` / `rest` / `none` / `other` | Sets the requested server integration; `server` is the default for a new App Router project |
| `query`           | `tanstack` / `none`                  | Adds TanStack Query only when explicitly requested                                          |
| `tests`           | `yes` / `no`                         | Forces installation of the default test base                                                |
| `package-manager` | `pnpm` / `npm` / `yarn` / `bun`      | Selects the package manager for a new project; ignored for existing projects                |

If a new project lacks a required decision, ask before installing. The UI decision is required when `ui` is absent: ask whether to install shadcn/ui. The default is Tailwind CSS without shadcn/ui. Use pnpm by default unless the user explicitly selects another package manager.

## Decision policy

### New project

Use Next.js with TypeScript, the App Router, a `src` directory, and the selected package manager. Consult current Next.js documentation before using scaffold flags or version-sensitive APIs.

Use this base:

- React local state for local UI; add a client store only when state is shared across multiple components and cannot remain in a feature boundary.
- React Hook Form plus `@hookform/resolvers` for forms.
- Zod for input, environment, and external-data validation.
- Tailwind CSS for styling.
- Vitest, React Testing Library, jsdom, and MSW for unit, component, and API tests unless the user opts out.

Do not add TanStack Query by default. Add it only when the user requests client-side server-state management or explicitly passes `query:tanstack`. Do not add Axios by default; use `fetch` or the project's SDK, and add a separate HTTP client only when the user requests or the project already uses one.

Ask before installing shadcn/ui. If confirmed, initialize it using the current official setup and generate components only when a feature needs them.

Playwright, additional icons, date libraries, toast libraries, i18n, Sentry, CI, ORM packages, and provider SDKs remain opt-in unless the request requires them.

### Existing project

Read `references/existing-project.md` before editing. Audit the repository and create a delta matrix against these rules.

- Confirm that the project uses the App Router. If it uses Pages Router, stop and report that this skill does not cover it.
- Detect the package manager from `packageManager` in `package.json` and lockfiles; preserve it.
- Preserve the existing state solution, data-fetching approach, HTTP client or SDK, UI kit, linter, formatter, test runner, scripts, and useful structure.
- Do not add missing Zustand, Zod, React Hook Form, Tailwind, TanStack Query, or other base dependencies automatically. Ask before adding them.
- If shadcn/ui exists, preserve it. If it is absent, ask before installing it.
- Adopt feature boundaries without replacing working systems.
- Migrate to Zustand, React Hook Form, Zod, Tailwind, shadcn/ui, TanStack Query, or another tool only after an explicit request and a scoped plan.
- Avoid installing a second library that solves the same problem.
- Apply changes by feature or small layer, with verification after each step.

## App Router architecture

Use this structure as the default for a new project:

```text
src/
├── app/                          # route segments, layouts, and composition
│   ├── layout.tsx
│   ├── page.tsx
│   ├── globals.css
│   └── ...
├── features/
│   └── <feature>/
│       ├── actions/              # Server Actions when the feature needs them
│       ├── components/           # feature-owned UI
│       ├── hooks/                # client hooks and optional query hooks
│       ├── schemas/              # input and response validation
│       ├── services/             # feature data access and domain operations
│       ├── types/
│       ├── utils/
│       └── store.ts              # only when shared client state is needed
├── components/
│   ├── ui/                       # generated shadcn/ui components, if used
│   └── ...                       # shared UI composed from feature needs
├── hooks/                        # hooks genuinely shared across features
├── lib/                          # shared clients, env, query keys, and utilities
├── types/                        # types genuinely shared across features
└── test/                         # shared test setup and MSW handlers
```

Keep route files in `src/app` thin. They define URL structure, metadata, layouts, loading and error boundaries, authorization gates, and page composition. Domain logic belongs in `features`; shared infrastructure belongs in `lib`; reusable UI belongs in `components`.

Use App Router conventions deliberately: `layout.tsx`, `page.tsx`, `loading.tsx`, `error.tsx`, `not-found.tsx`, `template.tsx`, `default.tsx`, route groups, dynamic segments, parallel routes, intercepting routes, and `route.ts` only where the route requires them. Do not create `src/pages` as a parallel layer.

## Server and client boundaries

- Treat every component as a Server Component by default.
- Add `"use client"` only at the smallest interactive boundary that needs browser APIs, event handlers, or client state.
- Keep server-only code, secrets, database clients, and private SDKs behind server-only modules.
- Pass serializable data from Server Components to Client Components. Pass Server Actions only through supported Server Function boundaries.
- Keep providers as low in the tree as their consumers allow; do not turn the root layout into a Client Component without a concrete need.
- Do not import server-only services into Client Components.
- Keep environment variables without `NEXT_PUBLIC_` on the server. Validate environment variables once with Zod.

## Data, mutations, and caching

- Server Components call feature services or SDKs directly for server-side reads. Do not call the application's own Route Handler from a Server Component.
- Use native `fetch` or the existing SDK for server data access. Encapsulate feature-specific access in `features/<feature>/services/`.
- Use Server Actions for mutations initiated by the application's UI when they are the right server boundary.
- Use Route Handlers for public HTTP contracts, webhooks, integrations, or access needed by external or client-only consumers.
- Validate every untrusted input as `unknown` with Zod at the server boundary, including `FormData`, JSON bodies, search parameters, route parameters, and external responses where appropriate.
- Re-check authentication and authorization inside every Server Action and Route Handler. Never rely only on a page-level guard.
- Make caching and revalidation explicit. Inspect the installed Next.js version and current documentation instead of assuming historical `fetch` defaults.
- Choose a cache policy per data source. Use revalidation or cache directives only when freshness and invalidation behavior are understood.
- Revalidate or redirect after successful mutations when the affected UI requires it. Keep invalidation close to the mutation.
- Add TanStack Query only for an explicitly requested client-side server-state use case; do not wrap all App Router data fetching in a client query provider.

## Forms and client state

- Use React Hook Form for new interactive forms and Zod schemas through `zodResolver`.
- A shadcn/ui form with Zod validation requires `react-hook-form` and `@hookform/resolvers` (plus `zod`). Install them when the first validated form is added; in an existing project, include them in the scoped plan and ask before installing. Follow `references/forms.md` for the canonical schema module, dependencies, and component wiring.
- Keep validation schemas in their own module under the owning feature (`features/<feature>/schemas/<resource>.ts`) and infer form values with `z.infer`; never declare the schema inline in the component.
- Validate again in the Server Action or Route Handler; client validation is not an authorization or integrity boundary.
- Return structured action results for expected form errors. Reserve thrown errors for exceptional failures.
- Use a client store only for state shared across multiple components that cannot remain local to a feature. Keep component-local state in React.
- Create at most one small store per feature when needed. Avoid a global mega-store.
- Keep server state out of client stores. Use Server Components, native data access, or explicitly requested TanStack Query for server state.

## Resource dialogs and forms

For a resource with multiple actions, give each action its own self-contained dialog module:

```text
features/<feature>/components/
├── <resource>-create-dialog.tsx
├── <resource>-edit-dialog.tsx
└── <resource>-delete-dialog.tsx
```

- Add only actions the resource supports; use `<resource>-revert-dialog.tsx` for a revert action when needed.
- Each dialog owns its whole flow in one file: fields, `useForm` with `zodResolver`, submit wiring, mutation call, labels, and close behavior.
- Never create a shared `<resource>-form.tsx`. Duplicating fields between create and edit is accepted by design: independence beats reuse here — a divergent edit flow is cheaper than a shared form carrying mode props, conditional defaults, and action branches.
- Extract a shared field block only when all three hold: the fields are identical, the block has no action-specific behavior (no defaults, labels, or mutation), and three or more dialogs use it. It is then a presentational `FieldGroup`, never a component that owns `useForm`.
- Keep schemas as contracts: one module per resource in `features/<feature>/schemas/`, shared between create and edit only while validation is identical. When edit adds or changes fields, give it its own module (`<resource>-edit.ts`). Never declare a schema inline in the component.
- Do not combine modes behind `isEdit`, `mode`, or action-specific conditional branches in one dialog.

When shadcn/ui is present, use its current form composition:

- Read `references/forms.md` for the canonical wiring: schema module, `useForm` with `zodResolver`, `Controller`, submit, and reset.
- Add the missing form primitives with `shadcn add field`; add `input-group` only when the interface needs grouped inputs.
- Wrap related fields in `FieldGroup`.
- Wrap each control in `Field` with `data-invalid` and, when disabled, `data-disabled`.
- Connect controlled primitives through `Controller`; `register` remains valid for native inputs when it keeps the interface simpler.
- Set `aria-invalid` on invalid controls.
- Render validation through `FieldError errors={[fieldState.error]}`.
- Keep schemas in `schemas/`, one module per resource, and infer form values from the schema.

## Feature ownership

- A route composes one or more features; it does not own resource behavior.
- A feature owns its components, forms, hooks, schemas, services, types, and utilities.
- Shared infrastructure may be imported by features; shared infrastructure never imports a feature.
- Features do not import one another directly. Move shared contracts or data access to a neutral module with one clear owner.
- Name files by resource and action so the file name reveals its responsibility.

## Form and dialog verification

- Test each form's valid submit, invalid state, reset behavior, and disabled state.
- Test each dialog's open/close behavior, action-specific labels, mutation errors, pending state, and success close behavior.
- Test each dialog independently — its own form, defaults, labels, mutation, pending state, and close behavior. There is no shared form module to cover.

## Quality and tooling

Treat formatter and linter as project decisions, not a fixed stack. Audit `package.json`, scripts, and configuration before changing tools.

| Evidence                                                                  | Owner    | Action                                                        |
| ------------------------------------------------------------------------- | -------- | ------------------------------------------------------------- |
| `biome.json` / `biome.jsonc`, dependency, or scripts for Biome            | Biome    | Use Biome for format and lint                                 |
| `eslint.config.*` / `.eslintrc*`, dependency, or scripts for ESLint       | ESLint   | Use ESLint for lint                                           |
| Oxlint configuration, dependency, or scripts                              | Oxlint   | Use Oxlint for lint                                           |
| `.prettierrc*` / `prettier.config.*`, dependency, or scripts for Prettier | Prettier | Use Prettier for format                                       |
| No evidence                                                               | None     | Ask in a new project; report the delta in an existing project |

Rules:

- Keep one owner per responsibility. Biome may own both formatting and linting; ESLint or Oxlint may own linting while Prettier owns formatting.
- Do not install all four tools by default or introduce a second tool for the same responsibility.
- Preserve existing scripts and use them as the source of truth.
- Configure ignores through the mechanism supported by the installed tool and version.
- Add coherent `lint`, `format`, and `format:check` scripts only when the corresponding tool is active.
- Use strict TypeScript and the project's existing alias convention. Configure `@/*` to `src/*` only when it does not conflict with an existing alias.

## shadcn/ui

When the project uses shadcn/ui:

- Treat generated files under `components/ui/` as shared base components.
- Use composition, variants, props, or feature wrappers for local customization.
- Explain the change, reason, and affected consumers before editing an existing generated component.
- Obtain explicit confirmation before modifying an existing generated component.
- Generate new components only when a real feature needs them.
- Do not write tests for generated components under `components/ui/`. They are third-party base code: tests target project-owned code (features, wrappers, hooks, services, and shared components built on top of the primitives).
- Choose the primitive that matches the interface intent: repeated resources and list items as `Card`, statuses as `Badge`, actions as `Button`, single-line fields as `Input`, view switching as `Tabs`, and so on. Do not hand-roll `div` structures with utility classes when a project primitive covers the need.
- When a needed primitive is missing, add it with `shadcn add <name>` before inventing local markup. Compose flows from the correct primitives instead of restyling one generic element repeatedly.
- Before building a validated form, ensure `react-hook-form` and `@hookform/resolvers` are installed (include them in the scoped plan and ask first in an existing project) and generate the `field` primitive; add `input-group` only when the interface needs grouped inputs. Keep the Zod schema in its own feature module and follow `references/forms.md`.
- Never ship demo or debug surfaces (color swatches, component galleries, foundation previews) inside product routes. Placeholder pages are temporary: once the real feature exists, remove the scaffold and present the real UX.

When the project does not use shadcn/ui:

- Keep Tailwind CSS if it is already present.
- Ask before initializing shadcn/ui.
- Build feature-specific UI in the feature and genuinely shared UI in `components/`.

## Non-negotiable rules

1. **App Router only.** Keep route behavior in `src/app`; do not introduce `src/pages`.
2. **Thin route files.** Pages and layouts compose features; they do not become domain-service files.
3. **Feature boundaries.** A feature may import its own modules and shared modules. Shared modules never import a feature. Features do not import one another directly.
4. **Server by default.** Keep Server Components and server-only services on the server; place client behavior at the smallest necessary boundary.
5. **Validated boundaries.** Parse untrusted input as `unknown` with Zod at every mutation, API, auth, and external-data boundary.
6. **Authorization in mutations.** Re-check session and permissions inside each Server Action and Route Handler.
7. **State placement.** Use React local state for local UI, an opt-in feature store for genuinely shared client state, and server data tools for server state.
8. **No unnecessary clients.** Do not add Axios or TanStack Query by default; use `fetch` or existing SDKs unless an explicit requirement justifies another client.
9. **Direct imports.** Avoid barrels that only re-export modules, especially new `index.ts` files.
10. **Predictable names.** Use `kebab-case` for application files and folders where the framework does not reserve a filename. Declare components and functions with inline named exports (`export function ThemeToggle() {}`) instead of a trailing `export { ... }` list; keep trailing export lists only for generated files (shadcn/ui) and re-export modules.
11. **Secrets stay server-side.** Never expose private environment variables, tokens, or server SDKs to the client bundle.
12. **Protected UI base.** Ask before modifying existing shadcn/ui components.
13. **Respect existing systems.** Preserve working package managers, scripts, routes, UI kits, clients, state solutions, and toolchains.
14. **Version-aware APIs.** Consult current Next.js and library documentation before relying on version-sensitive behavior.
15. **Mandatory verification.** Run available typecheck, lint, format check, tests, and build commands before finishing; fix introduced failures.
16. **Right primitives.** With shadcn/ui, build interfaces from the primitives that match the intent (Card for items, Badge for statuses, Button for actions, Input for fields) instead of hand-rolled markup, and never ship demo or debug surfaces in product routes.

## Work order

1. Classify the project as new Next.js App Router, existing Next.js App Router, Pages Router, or non-Next. Completion: classification is explicit.
2. Read the matching reference: `references/scaffold.md` for new projects or `references/existing-project.md` for existing projects. Completion: required decisions and current delta are known.
3. Confirm only missing decisions. Completion: package manager, UI choice, test scope, quality toolchain, and server integration scope are known.
4. Implement the base or incremental migration. Completion: route files, feature boundaries, server/client boundaries, and dependency choices follow this skill.
5. Verify and review the diff. Completion: available checks pass, no forbidden router layer was introduced, and no unrelated files changed.

## Library documentation

Before assuming syntax, configuration, or behavior for Next.js, React, Tailwind, Zustand, React Hook Form, Zod, shadcn/ui, Vitest, or another library, consult current documentation through `ctx7`. Do not pin versions or APIs from memory alone.
