# Formatting and Prettier

Use this reference when the user explicitly requests Prettier or when an existing project has Prettier evidence: a dependency, configuration file, or script.

## Ownership

- Use the project's existing formatter when one is configured.
- Add Prettier only for an explicit request or when the project already uses it.
- If Biome, Oxfmt, or another formatter owns formatting, do not add a second formatter silently. Ask before replacing or running competing formatters.
- Preserve existing Prettier configuration. Do not create a new configuration file unless the user requests one or the project has no usable configuration and a concrete rule requires it.
- When ESLint and Prettier coexist, load `linting.md` and add `eslint-config-prettier` to avoid formatting-rule conflicts. Use its `/flat` export only with flat `eslint.config.*`; use its legacy config with `.eslintrc*`.

## Package manager

Resolve the package manager before installing dependencies or writing package scripts. Use this order:

1. Use the `packageManager` field in `package.json` when present.
2. Otherwise use the existing lockfile: `pnpm-lock.yaml` for pnpm, `package-lock.json` for npm, `yarn.lock` for Yarn, or `bun.lock`/`bun.lockb` for Bun.
3. Otherwise preserve the package manager used by existing project scripts and instructions.
4. If sources disagree or no manager is identifiable, stop and ask. Do not assume pnpm.

Use the resolved manager for dependency installation and nested script calls:

| Manager | Install Prettier | `check` command |
| --- | --- | --- |
| pnpm | `pnpm add -D prettier` | `pnpm format:check && pnpm lint && pnpm build` |
| npm | `npm install -D prettier` | `npm run format:check && npm run lint && npm run build` |
| yarn | `yarn add --dev prettier` | `yarn format:check && yarn lint && yarn build` |
| bun | `bun add -d prettier` | `bun run format:check && bun run lint && bun run build` |

When ESLint is active, install `eslint-config-prettier` with Prettier. Omit it when the project has no ESLint configuration.

## `.prettierignore`

When Prettier setup is active, create or update a root `.prettierignore`.

- Preserve existing entries and append only missing entries.
- Do not replace user comments or custom patterns.
- Add these generated, dependency, cache, coverage, and lockfile paths when relevant:

```text
node_modules/
.next/
out/
dist/
build/
coverage/
.vercel/
.turbo/
pnpm-lock.yaml
package-lock.json
yarn.lock
bun.lock
bun.lockb
```

- Do not ignore application source or the whole `public/` directory without a concrete project-specific reason.
- When shadcn/ui is used, inspect `components.json` and the project structure. Add only the actual generated primitives directory, commonly `components/ui/`, `src/components/ui/`, or `src/shared/components/ui/`.
- For new projects following this skill's structure, use `src/shared/components/ui/` for generated shadcn/ui primitives.
- Do not add every candidate shadcn path when only one exists.

## Package scripts

When Prettier setup is active, ensure `package.json` contains:

```json
{
  "scripts": {
    "format": "prettier --write .",
    "format:check": "prettier --check .",
    "check": "<resolved-manager> format:check && <resolved-manager> lint && <resolved-manager> build"
  }
}
```

Replace `<resolved-manager>` with the command from the package-manager table. Add missing scripts without changing unrelated scripts. If an existing `format`, `format:check`, or `check` script has a different purpose, preserve it and ask before replacing it. Create `check` only when `lint` and `build` scripts exist; otherwise report the missing scripts instead of creating a broken command.

## Verification

Run the resolved manager's `format:check` script. Confirm `.prettierignore` excludes generated output, lockfiles, and the actual shadcn/ui primitives directory. Then run the configured typecheck, tests, lint, and build commands. Report pre-existing failures separately from failures introduced by Prettier setup.
