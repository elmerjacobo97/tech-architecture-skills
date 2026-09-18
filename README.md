[English](README.md) · [Español](README-es.md)

<p align="center">
  <h1 align="center">Tech Architecture Skills</h1>
  <p align="center">Stop re-deciding the same architecture on every project. Fix the stack once, per technology.</p>
</p>

<p align="center">
  <img alt="License" src="https://img.shields.io/github/license/elmerjacobo97/tech-architecture-skills">
  <img alt="Latest Release" src="https://img.shields.io/github/v/release/elmerjacobo97/tech-architecture-skills">
  <img alt="GitHub Stars" src="https://img.shields.io/github/stars/elmerjacobo97/tech-architecture-skills?style=social">
  <img alt="Skills" src="https://img.shields.io/badge/skills-4-blue">
</p>

## Quick start

```bash
npx skills@latest add elmerjacobo97/tech-architecture-skills
```

## What this is

One skill per technology stack. Each one fixes the project's technical base — folder structure, state management, routing, HTTP layer, models, storage, testing — and gives the agent a concrete pattern to follow when scaffolding a new project or evolving an existing one.

The problem it solves: an LLM asked to "build a Next.js app" makes dozens of architecture decisions on its own (folder layout, client vs server state, data fetching, naming) and hides them inside the code. These skills turn those decisions into an explicit, versioned rulebook the agent reads before writing anything.

## Skills

| Skill | Stack | Use when |
| --- | --- | --- |
| [`next-architecture`](./skills/web/next-architecture/SKILL.md) | Next.js App Router, Server Components, Server Actions, Route Handlers, React Hook Form, Zod, Tailwind CSS, shadcn/ui | Creating, scaffolding, structuring, or migrating a Next.js project |
| [`react-architecture`](./skills/web/react-architecture/SKILL.md) | React SPA + Vite, TanStack Router, TanStack Query, Zustand, React Hook Form, Zod, Axios, Tailwind CSS, shadcn/ui | Building a React SPA — not for Next.js |
| [`nuxt-architecture`](./skills/web/nuxt-architecture/SKILL.md) | Nuxt 4, App directory, Nitro server routes, Pinia, composables, `useFetch`, vee-validate, Zod, Tailwind CSS, shadcn-vue | Creating or migrating a Nuxt application |
| [`flutter-architecture`](./skills/mobile/flutter-architecture/SKILL.md) | Riverpod (codegen), `hooks_riverpod`, go_router, dio, freezed 3.0, fpdart (`Either`), optional Clean Architecture tier | Starting or structuring a Flutter app |

Each skill is **feature-based by default**: organize by vertical feature, not by global technical layer. `flutter-architecture` additionally documents Clean Architecture as an opt-in tier for large projects.

## How a skill behaves

- **Loads on demand.** The agent picks the skill when the task matches its description (new project, restructuring, "how should I organize this?").
- **Also works as a slash command.** `/next-architecture nombre=my-app` — free-text arguments are interpreted by convention (`nombre`, `tier`, backend alignment, etc.).
- **Index + references.** `SKILL.md` holds the scope, the fixed stack table, the workflow, and the arguments. The depth lives in `references/` (`scaffold.md`, `existing-project.md`, and stack-specific files) so the main file stays cheap to load.

## Pairs with spec-driven development

These skills define **how a project is built**; specs define **what gets built**. Used together, the spec's decisions section inherits the stack and the agent implements against both. The spec workflow lives in [spec-flow-skills](https://github.com/elmerjacobo97/spec-flow-skills).

## Installation

### Option 1 — skills.sh (recommended, Claude Code and more)

```bash
npx skills@latest add elmerjacobo97/tech-architecture-skills
```

Install a single skill:

```bash
npx skills@latest add elmerjacobo97/tech-architecture-skills -s next-architecture
```

Uninstall:

```bash
npx skills@latest remove elmerjacobo97/tech-architecture-skills
```

### Option 2 — Other agents (Cursor, Codex, Antigravity, opencode)

```bash
git clone https://github.com/elmerjacobo97/tech-architecture-skills ~/.tech-architecture-skills
cd ~/your-project
~/.tech-architecture-skills/scripts/install-to-agent.sh <agent>
```

`<agent>` can be `claude`, `cursor`, `codex`, `antigravity`, or `opencode`.

| Agent | What gets written |
| --- | --- |
| `claude` | Symlinks each skill into `.claude/skills/` |
| `cursor` | Generates `.cursor/rules/<name>.mdc` files |
| `codex` | Adds a `## Skills` block to `AGENTS.md` and copies skills into `.codex/skills/` |
| `antigravity` | Copies skills into `.antigravity/skills/` |
| `opencode` | Generates `.opencode/commands/<name>.md` files |

### Option 3 — Manual

```bash
mkdir -p ~/.claude/skills
cp -r skills/web/next-architecture ~/.claude/skills/
cp -r skills/mobile/flutter-architecture ~/.claude/skills/
```

## Adding a new stack

1. Create `skills/<area>/<tech>-architecture/` with `SKILL.md` and `references/`.
2. Fix the stack table first — state, routing, HTTP, models, storage, testing. Those are decisions the skill must not ask again.
3. Write the `description` in English, starting with "Use when…".
4. Validate the YAML frontmatter (a `: ` inside an unquoted description silently removes the skill from the skills.sh index):

```bash
ruby -ryaml -e 'Dir.glob("skills/**/SKILL.md").each { |f| t=File.read(f); YAML.safe_load(t[/\A---\n(.*?)\n---/m,1]); puts "OK #{f}" }'
```

5. Commit with [Conventional Commits](https://www.conventionalcommits.org/) — releases are automated with release-please.

Candidate stacks: `laravel-architecture` (backend), and whatever you work with next.

## License

MIT
