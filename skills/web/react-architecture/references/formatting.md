# Formatting and Oxfmt

Use this reference when creating, auditing, migrating, or verifying Oxfmt, Prettier, Biome, format scripts, or ignore configuration in a React + Vite project.

## Ownership

- New React + Vite projects use Oxfmt by default.
- Existing projects preserve the current formatter and its configuration.
- Keep one owner for formatting. Do not add Oxfmt or Prettier beside an existing formatter without an explicit request.
- Biome may own both formatting and linting.

| Evidence | Owner | Action |
| --- | --- | --- |
| `.oxfmtrc*`, `oxfmt.config.*`, Oxfmt dependency, or Oxfmt scripts | Oxfmt | Use Oxfmt for format. |
| `.prettierrc*`, `prettier.config.*`, Prettier dependency, or Prettier scripts | Prettier | Use Prettier for format. |
| `biome.json`/`biome.jsonc`, Biome dependency, or Biome scripts | Biome | Use Biome for format and lint. |
| No formatter evidence in an existing project | None | Report the gap and ask before installing a formatter. |

If multiple formatters exist, preserve them, use their scripts as the source of truth, and report conflicts. Consolidate only after an explicit request.

## New-project default

Install Oxfmt for new React + Vite projects when the scaffold does not already provide it. Oxfmt is the Oxc formatter intended to pair with the Vite template's Oxlint setup.

Use a root `.oxfmtrc.json` or a version-compatible `oxfmt.config.*`. Prefer JSON for a simple project configuration:

```json
{
  "$schema": "./node_modules/oxfmt/configuration_schema.json",
  "ignorePatterns": [
    "node_modules/**",
    "dist/**",
    "coverage/**"
  ]
}
```

Add only generated paths that exist in the project. When shadcn/ui or route generation is used, inspect the actual generated directory before adding it to `ignorePatterns`.

Use the repository-wide script names `format` and `format:check` even though Oxfmt also documents `fmt` and `fmt:check`:

```json
{
  "scripts": {
    "format": "oxfmt",
    "format:check": "oxfmt --check"
  }
}
```

Oxfmt defaults to a `printWidth` of `100`. Keep that default unless the project has a documented style requirement. Set `printWidth` to `80` only when matching an existing Prettier convention is required.

## Formatter boundaries

- Oxfmt formats JavaScript, JSX, TypeScript, TSX, JSON, CSS, GraphQL, YAML, Markdown, and other supported files.
- Use the npm package when Prettier-backed formats, dynamic configuration, editor integration, or Tailwind sorting are needed.
- Import sorting, Tailwind class sorting, and package sorting are opt-in Oxfmt features. Do not enable them without a project decision.
- Do not add `eslint-plugin-prettier` or formatting rules to Oxlint.
- Do not use Oxfmt and Prettier on the same files.

## Existing-project workflow

1. Read `package.json`, the lockfile, scripts, formatter configuration, linter configuration, ignore files, and generated-file conventions.
2. Run the existing format check before changing configuration. Record pre-existing failures.
3. Preserve the current formatter, options, ignore patterns, and script names unless the request changes them.
4. Make the smallest correction when configuration exists but its script is missing or broken.
5. Verify `format:check`, typecheck, lint, tests, and build when configured.

Completion: one formatter owns formatting, generated output is excluded through the installed tool's mechanism, and scripts run without writing files during verification.
