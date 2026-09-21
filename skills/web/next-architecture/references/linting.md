# Linting and Quality

Use this reference when creating, auditing, migrating, or verifying ESLint, Oxlint, Biome, lint scripts, or quality-tool configuration in a Next.js project.

## Ownership

- New Next.js projects use ESLint by default.
- Existing projects preserve the current linter and its configuration.
- Keep one owner per responsibility. Do not install a second linter to replace a working tool.
- Load `formatting.md` when Prettier is selected or already present.

| Evidence | Owner | Action |
| --- | --- | --- |
| `eslint.config.*`, `.eslintrc*`, ESLint dependency, or ESLint scripts | ESLint | Use the existing ESLint configuration and scripts. |
| `.oxlintrc*`, `oxlint.config.*`, Oxlint dependency, or Oxlint scripts | Oxlint | Use the existing Oxlint configuration and scripts. |
| `biome.json`/`biome.jsonc`, Biome dependency, or Biome scripts | Biome | Use Biome for lint and format. |
| `.oxfmtrc*`, `oxfmt.config.*`, Oxfmt dependency, or Oxfmt scripts | Oxfmt | Use Oxfmt for format; do not add Prettier silently. |
| No linter evidence in an existing project | None | Report the gap and ask before installing a linter. |

If multiple tools exist, preserve them, use their scripts as the source of truth, and report overlapping responsibilities. Consolidate only after an explicit request.

## New-project ESLint

Use the current `create-next-app` options for the installed Next.js version. Select ESLint unless the user explicitly chooses Oxlint or Biome.

For Next.js versions that support the current flat configuration, use the official Next.js presets:

- `eslint-config-next/core-web-vitals` for Next.js, React, React Hooks, and Core Web Vitals rules.
- `eslint-config-next/typescript` for TypeScript rules based on `typescript-eslint/recommended`.
- `eslint-config-prettier/flat` only when Prettier is also the formatter.

Use a version-matched configuration. Current Next.js 16 examples use this shape:

```js
import { defineConfig, globalIgnores } from 'eslint/config'
import nextVitals from 'eslint-config-next/core-web-vitals'
import nextTs from 'eslint-config-next/typescript'

export default defineConfig([
  ...nextVitals,
  ...nextTs,
  globalIgnores([
    '.next/**',
    'out/**',
    'build/**',
    'next-env.d.ts',
  ]),
])
```

Use the installed Next.js documentation when the preset shape or available exports differ. Do not copy a Next.js 16 configuration into a Next.js 15 project without checking the installed version.

## Rule policy

- Prefer official recommended presets over a copied list of individual rules.
- Keep `core-web-vitals` enabled for new applications unless a concrete compatibility issue is documented.
- Keep TypeScript linting at the recommended level by default. Use type-aware or strict presets only after checking project size, parser configuration, and runtime cost.
- Keep `tsc --noEmit` as the TypeScript baseline. Linting does not replace type checking.
- Keep formatting outside ESLint. Do not add formatting rules or `eslint-plugin-prettier` when Prettier or another formatter owns formatting.
- Do not assume `eslint-config-next` provides accessibility coverage. Add `jsx-a11y` only when the project explicitly adopts that rule set.
- Keep feature boundaries, server-only boundaries, authorization, and data validation enforced by the architecture and security rules in this skill. A linter is not the only boundary.

For an existing complex ESLint configuration with conflicting React, React Hooks, import, parser, or resolver plugins, use `@next/eslint-plugin-next` directly instead of spreading a conflicting shareable config. Preserve existing rule ownership and document overrides.

## Version gates

- Next.js 15: preserve an existing `next lint` script unless migration is explicitly requested and the installed documentation supports the replacement.
- Next.js 16+: `next lint` is removed. Use the direct ESLint CLI through the project script.
- Next.js 16+: `next build` does not run lint automatically. Keep lint as an explicit verification command.
- Flat config uses `eslint.config.*` and `globalIgnores`. Do not create `.eslintignore` for a new flat-config project.

## Scripts

Use the detected package manager and preserve existing script names. New ESLint projects should expose:

```json
{
  "scripts": {
    "lint": "eslint .",
    "lint:fix": "eslint . --fix"
  }
}
```

Run `lint` in CI and verification. Keep `lint:fix` for deliberate local use. Do not run auto-fix as an implicit build step.

When a project uses Oxlint or Biome instead, use their current direct CLI and configuration. Do not make the `lint` script call one tool through another tool.

## Ignores

- Ignore generated output, dependencies, coverage, and real generated source only.
- For ESLint flat config, use `globalIgnores` in `eslint.config.*`.
- For Oxlint, prefer `ignorePatterns` in `.oxlintrc.json` or `oxlint.config.*` for new configuration.
- For Biome, use `files.includes`, tool-specific includes, and VCS integration supported by the installed version.
- Inspect the actual project before ignoring shadcn/ui primitives, generated route trees, type output, or provider-generated files.
- Never ignore application source or all of `public/` without a project-specific reason.

## Existing-project workflow

1. Read `package.json`, the lockfile, scripts, linter configuration, formatter configuration, ignore files, TypeScript configuration, and Next configuration.
2. Run the existing typecheck and lint commands before changing configuration. Record pre-existing failures.
3. Preserve the current linter, plugins, rule severity, ignores, and script names unless the request changes them.
4. Make the smallest correction when configuration exists but its script is missing or broken.
5. Verify typecheck, lint, format check when configured, tests, Next.js type generation, and build.

Completion: one linter owns linting, its ignores match the installed version, scripts run directly, and exceptions are documented.
