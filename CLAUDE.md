# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This repo packages **architecture and scaffolding skills** — one per technology stack — for AI coding agents. Each skill fixes the technical base of a project (folder structure, state management, routing, data layer, testing) so the agent stops improvising those decisions on every new feature.

Skills are written in the language that fits the stack's audience; **the `description` field is always English** because that is what the skills.sh index shows.

## Repository layout

```
skills/
├── web/
│   ├── next-architecture/     # Next.js App Router
│   ├── react-architecture/    # React SPA (Vite)
│   └── nuxt-architecture/     # Nuxt 4
└── mobile/
    └── flutter-architecture/  # Flutter
```

- Buckets group skills by area (`web/`, `mobile/`, `backend/` for future stacks such as Laravel).
- Each skill is a directory with a `SKILL.md` plus a `references/` folder holding the detailed material (`scaffold.md`, `existing-project.md`, and stack-specific files). `SKILL.md` stays lean and points at the references by relative path.

## Skill authoring conventions

The YAML frontmatter of `SKILL.md` declares:

```yaml
---
name: next-architecture
description: Use when creating, scaffolding, structuring, or incrementally migrating ...
---
```

Rules:

- `name` must match the folder name exactly.
- `description` is in English, starts with "Use when…", and names the stack plus its concrete tools — that is the trigger the agent matches on.
- Do **not** set `disable-model-invocation` here: these are reference skills, and the agent must be able to load them on its own when the task matches. They remain invocable as slash commands (`/next-architecture <args>`) with free-text arguments interpreted by convention.
- **Validate the frontmatter before pushing.** A plain-scalar `description` containing `: ` (colon + space) is invalid YAML and the skill is silently skipped by the skills.sh indexer:

```bash
ruby -ryaml -e 'Dir.glob("skills/**/SKILL.md").each { |f| t=File.read(f); YAML.safe_load(t[/\A---\n(.*?)\n---/m,1]); puts "OK #{f}" }'
```

- Quote the description when it contains punctuation that YAML could misread.

## Adding a new stack

1. Create `skills/<area>/<tech>-architecture/SKILL.md` + `references/`.
2. Fix the stack table first (state, routing, HTTP, models, storage, testing) — these are decisions, not questions the skill asks later.
3. Keep `SKILL.md` as the index: scope, fixed stack, arguments, workflow, and links to the references.
4. English `description`, validated YAML, commit with a Conventional Commit (`feat: add laravel-architecture skill`).

## Distribution

The repo is consumed in two ways:

1. **skills.sh** (`npx skills@latest add elmerjacobo97/tech-architecture-skills`) — auto-discovers public GitHub repos with `skills/**/SKILL.md`. Just push to GitHub.
2. **Multi-agent installer** (`scripts/install-to-agent.sh <agent>`) — translates skills for Cursor, Codex, Antigravity, and opencode. Run from the _target_ repo, not this one.

`scripts/link-skills.sh` symlinks every skill into `~/.claude/skills` for local development.

## Releases

[release-please](https://github.com/googleapis/release-please) derives versions from [Conventional Commits](https://www.conventionalcommits.org/): `feat:` bumps minor, `fix:` bumps patch, `docs:`/`chore:`/`refactor:` do not bump.

## No build or test commands

There is no package manager, build step, or test suite. All skills are plain Markdown files.
