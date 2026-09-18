# Project Structure and Naming

Use this reference for new projects and for any change that moves files. The structure is a default, not a reason to reorganize working code without an explicit migration request.

## Target structure

```text
src/
├── app/                         # routes, layouts, metadata, and route special files
├── features/
│   └── <feature>/               # vertical domain slice, created when needed
│       ├── actions/             # Server Functions owned by the feature
│       ├── components/          # domain UI
│       ├── hooks/               # feature hooks
│       ├── schemas/             # feature input and response contracts
│       ├── services/            # feature data access and domain operations
│       ├── types/               # feature-only types
│       ├── utils/               # feature-only helpers
│       └── store.ts             # only when several feature components need client state
├── shared/
│   ├── components/              # reusable UI; ui/ for generated primitives when used
│   ├── hooks/                   # reusable hooks
│   ├── lib/                     # clients, environment, query keys, and infrastructure
│   ├── schemas/                 # contracts shared by multiple boundaries
│   ├── types/                   # types shared by multiple features
│   └── utils/                   # runtime-safe reusable helpers
└── test/                        # shared test setup and helpers only

e2e/                             # browser journeys, only when enabled
```

Create directories when real code needs them. Do not create empty feature modules, example stores, placeholder schemas, or a second shared namespace. Keep `public/`, config files, and generated output at project root.

## Ownership and imports

Use this dependency direction:

```text
app      -> features -> shared
app      -> shared
features -> external dependencies
shared   -> external dependencies
```

- Route files may import features and shared modules.
- A feature may import its own modules and `src/shared/`.
- `src/shared/` may import other shared modules and external dependencies only.
- Features never import another feature directly. Promote a real abstraction to `shared/` after a second consumer exists.
- Keep route-private code in the route segment or a Next private folder such as `_components/`; do not move it to `features/` only to satisfy the diagram.
- Keep server-only data access inside a feature service or shared server-only module. Add `server-only` when an accidental client import must fail.
- Use aliases such as `@/features/...` and `@/shared/...` consistently. Import modules directly; do not add barrels only to shorten paths.

## Naming

- Use `kebab-case` for application files and directories: `login-form.tsx`, `use-current-user.ts`, `auth.service.ts`.
- Preserve Next.js reserved names and suffixes: `page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`, `not-found.tsx`, `route.ts`, `middleware.ts`/`proxy.ts`, and metadata files.
- Use PascalCase for React component symbols, camelCase for functions and variables, and `UPPER_SNAKE_CASE` only for true constants.
- Name a module for one responsibility. Do not hide unrelated exports behind `index.ts`.
- Create `actions/`, `services/`, `schemas/`, `hooks/`, or `store.ts` only when the feature has that behavior.

## Test placement

- Keep a test beside the module when it proves that module: `login-form.tsx` plus `login-form.test.tsx`.
- Use `__tests__/` only for a small, coherent cluster that would make neighboring files harder to scan.
- Keep shared Vitest/Jest setup, MSW handlers, and render helpers in `src/test/`.
- Keep Playwright journeys in `e2e/` or the existing project convention. The experimental Next testmode exception uses the runner's required `{app,pages}/**/*.spec.{t,j}s` pattern.
- Mirror feature ownership in tests: feature behavior stays with the feature; route and browser behavior stays with the route/E2E layer.
- Do not test generated UI primitives, third-party internals, or a thin wrapper when its owner is already covered.

## Existing projects

- Preserve a valid existing structure, naming convention, test runner, and alias unless migration is explicitly requested.
- If an existing project has `src/components`, `src/hooks`, `src/lib`, or `src/types`, do not mass-move them or create duplicate parallel directories as cleanup.
- New code should follow the requested project convention only after its ownership boundary is clear. A deliberate legacy exception belongs in the final report.
- Move one complete feature at a time. Verify imports, client/server boundaries, tests, and runtime behavior before repeating.
