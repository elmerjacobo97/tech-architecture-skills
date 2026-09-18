[English](README.md) · [Español](README-es.md)

<p align="center">
  <h1 align="center">Tech Architecture Skills</h1>
  <p align="center">Deja de re-decidir la misma arquitectura en cada proyecto. Fija el stack una vez, por tecnología.</p>
</p>

<p align="center">
  <img alt="License" src="https://img.shields.io/github/license/elmerjacobo97/tech-architecture-skills">
  <img alt="Latest Release" src="https://img.shields.io/github/v/release/elmerjacobo97/tech-architecture-skills">
  <img alt="GitHub Stars" src="https://img.shields.io/github/stars/elmerjacobo97/tech-architecture-skills?style=social">
  <img alt="Skills" src="https://img.shields.io/badge/skills-4-blue">
</p>

## Inicio rápido

```bash
npx skills@latest add elmerjacobo97/tech-architecture-skills
```

## Qué es esto

Un skill por stack tecnológico. Cada uno fija la base técnica del proyecto — estructura de carpetas, manejo de estado, ruteo, capa HTTP, modelos, storage, testing — y le da al agente un patrón concreto que seguir al crear un proyecto nuevo o evolucionar uno existente.

El problema que resuelve: un LLM al que le pides "hazme una app Next.js" toma decenas de decisiones de arquitectura por su cuenta (organización de carpetas, estado cliente vs servidor, fetching, nombres) y las esconde dentro del código. Estos skills convierten esas decisiones en un manual explícito y versionado que el agente lee antes de escribir nada.

## Skills

| Skill | Stack | Cuándo usarlo |
| --- | --- | --- |
| [`next-architecture`](./skills/web/next-architecture/SKILL.md) | Next.js App Router, Server Components, Server Actions, Route Handlers, React Hook Form, Zod, Tailwind CSS, shadcn/ui | Crear, estructurar o migrar un proyecto Next.js |
| [`react-architecture`](./skills/web/react-architecture/SKILL.md) | React SPA + Vite, TanStack Router, TanStack Query, Zustand, React Hook Form, Zod, Axios, Tailwind CSS, shadcn/ui | Construir una SPA React — no aplica a Next.js |
| [`nuxt-architecture`](./skills/web/nuxt-architecture/SKILL.md) | Nuxt 4, App directory, Nitro server routes, Pinia, composables, `useFetch`, vee-validate, Zod, Tailwind CSS, shadcn-vue | Crear o migrar una aplicación Nuxt |
| [`flutter-architecture`](./skills/mobile/flutter-architecture/SKILL.md) | Riverpod (codegen), `hooks_riverpod`, go_router, dio, freezed 3.0, fpdart (`Either`), tier opcional de Clean Architecture | Arrancar o estructurar una app Flutter |

Cada skill es **feature-based por default**: organiza por feature vertical, no por capa técnica global. `flutter-architecture` además documenta Clean Architecture como tier opcional para proyectos grandes.

## Cómo se comporta un skill

- **Se carga solo cuando aplica.** El agente lo elige cuando la tarea coincide con su descripción (proyecto nuevo, reestructurar, "¿cómo organizo esto?").
- **También funciona como slash command.** `/next-architecture nombre=mi-app` — los argumentos de texto libre se interpretan por convención (`nombre`, `tier`, alineación con backend, etc.).
- **Índice + referencias.** `SKILL.md` contiene el alcance, la tabla de stack fija, el flujo y los argumentos. La profundidad vive en `references/` (`scaffold.md`, `existing-project.md` y archivos específicos del stack), para que el archivo principal sea barato de cargar.

## Se combina con spec-driven development

Estos skills definen **cómo se construye el proyecto**; los specs definen **qué se construye**. Usados juntos, la sección de decisiones del spec hereda el stack y el agente implementa contra ambos. El workflow de specs vive en [spec-flow-skills](https://github.com/elmerjacobo97/spec-flow-skills).

## Instalación

### Opción 1 — skills.sh (recomendada, Claude Code y más)

```bash
npx skills@latest add elmerjacobo97/tech-architecture-skills
```

Instalar un solo skill:

```bash
npx skills@latest add elmerjacobo97/tech-architecture-skills -s next-architecture
```

Desinstalar:

```bash
npx skills@latest remove elmerjacobo97/tech-architecture-skills
```

### Opción 2 — Otros agentes (Cursor, Codex, Antigravity, opencode)

```bash
git clone https://github.com/elmerjacobo97/tech-architecture-skills ~/.tech-architecture-skills
cd ~/tu-proyecto
~/.tech-architecture-skills/scripts/install-to-agent.sh <agent>
```

`<agent>` puede ser `claude`, `cursor`, `codex`, `antigravity` u `opencode`.

| Agente | Qué escribe |
| --- | --- |
| `claude` | Symlinks de cada skill en `.claude/skills/` |
| `cursor` | Genera `.cursor/rules/<name>.mdc` |
| `codex` | Agrega un bloque `## Skills` a `AGENTS.md` y copia los skills a `.codex/skills/` |
| `antigravity` | Copia los skills a `.antigravity/skills/` |
| `opencode` | Genera `.opencode/commands/<name>.md` |

### Opción 3 — Manual

```bash
mkdir -p ~/.claude/skills
cp -r skills/web/next-architecture ~/.claude/skills/
cp -r skills/mobile/flutter-architecture ~/.claude/skills/
```

## Agregar un stack nuevo

1. Crear `skills/<área>/<tech>-architecture/` con `SKILL.md` y `references/`.
2. Fijar primero la tabla de stack — estado, ruteo, HTTP, modelos, storage, testing. Esas son decisiones que el skill no debe volver a preguntar.
3. Escribir la `description` en inglés, empezando con "Use when…".
4. Validar el frontmatter YAML (un `: ` dentro de una descripción sin comillas elimina el skill del índice de skills.sh en silencio):

```bash
ruby -ryaml -e 'Dir.glob("skills/**/SKILL.md").each { |f| t=File.read(f); YAML.safe_load(t[/\A---\n(.*?)\n---/m,1]); puts "OK #{f}" }'
```

5. Commit con [Conventional Commits](https://www.conventionalcommits.org/) — los releases son automáticos con release-please.

Stacks candidatos: `laravel-architecture` (backend), y lo que venga después.

## Licencia

MIT
