# Linting and Oxlint

Use this reference when creating, auditing, migrating, or verifying Oxlint, ESLint, Biome, lint scripts, or quality-tool configuration in a React + Vite project.

## Ownership

- New React + Vite projects use Oxlint by default.
- Existing projects preserve the current linter and its configuration.
- Keep one owner for linting. Do not install a second linter to replace a working tool.
- Load `formatting.md` when Oxfmt, Prettier, or Biome formatting is selected.

| Evidence | Owner | Action |
| --- | --- | --- |
| `.oxlintrc*`, `oxlint.config.*`, Oxlint dependency, or Oxlint scripts | Oxlint | Use the existing Oxlint configuration and scripts. |
| `eslint.config.*`, `.eslintrc*`, ESLint dependency, or ESLint scripts | ESLint | Use the existing ESLint configuration and scripts. |
| `biome.json`/`biome.jsonc`, Biome dependency, or Biome scripts | Biome | Use Biome for lint and format. |
| No linter evidence in an existing project | None | Report the gap and ask before installing a linter. |

If multiple tools exist, preserve them, use their scripts as the source of truth, and report overlapping responsibilities. Consolidate only after an explicit request.

## New-project default

The current Vite React TypeScript template includes Oxlint. Preserve its generated configuration and extend it only for a concrete project need.

Baseline configuration:

```json
{
  "$schema": "./node_modules/oxlint/configuration_schema.json",
  "plugins": ["react", "typescript", "oxc"],
  "rules": {
    "react/rules-of-hooks": "error",
    "react/only-export-components": ["warn", { "allowConstantExport": true }]
  }
}
```

This baseline covers React Hooks, TypeScript rules, Oxc rules, and the Vite fast-refresh export convention. Apply file-specific overrides when route files or test files have a documented false positive; do not disable a rule globally to hide one boundary case.

## Type-aware linting

Vite recommends type-aware Oxlint rules for production applications. Enable them only after checking the installed Oxlint, `oxlint-tsgolint`, Node.js, TypeScript, and monorepo versions:

```json
{
  "options": {
    "typeAware": true
  }
}
```

Install `oxlint-tsgolint` only when the installed documentation and project runtime support it. Keep `tsc --noEmit` as the baseline check unless the project explicitly replaces it with Oxlint type checking after comparing diagnostics and coverage.

## Plugins and rules

- Keep `react`, `typescript`, and `oxc` enabled for the default profile.
- Add `import`, `jsx-a11y`, `vitest`, or other plugins only when the project uses the corresponding boundary.
- Enable React Compiler rules only when React Compiler is selected for the project.
- Use correctness rules as errors when they indicate invalid code. Keep opinionated style or migration rules at warning level until the project adopts them deliberately.
- Keep formatting outside Oxlint. Use Oxfmt or the existing formatter.
- Keep feature boundaries, authorization, and input validation enforced by architecture and security rules. Linting is not a substitute for those checks.

## Scripts

Use the detected package manager and preserve existing script names. New Oxlint projects should expose:

```json
{
  "scripts": {
    "lint": "oxlint",
    "lint:fix": "oxlint --fix"
  }
}
```

Run `lint` in CI and verification. Keep `lint:fix` for deliberate local use. Do not run suggestions or dangerous fixes automatically.

## Ignores

- Prefer `ignorePatterns` in `.oxlintrc.json` or `oxlint.config.*` for new projects.
- Ignore `node_modules`, Vite output, coverage, and actual generated source only.
- Inspect the project before ignoring a generated TanStack Router route tree or shadcn/ui primitives.
- Keep `.eslintignore` only as an existing migration compatibility file. Do not create it for a new Oxlint project.
- For ESLint or Biome projects, use the ignore mechanism supported by the installed version.

## Existing-project workflow

1. Read `package.json`, the lockfile, scripts, linter configuration, formatter configuration, ignore files, TypeScript configuration, Vite configuration, and test setup.
2. Run existing typecheck and lint commands before changing configuration. Record pre-existing failures.
3. Preserve the current linter, plugins, rule severity, ignores, and script names unless the request changes them.
4. Make the smallest correction when configuration exists but its script is missing or broken.
5. Verify typecheck, lint, format check when configured, tests, and build.

Completion: one linter owns linting, its ignores match the installed version, scripts run directly, and exceptions are documented.
