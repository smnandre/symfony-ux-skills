# Symfony UX -- Agent Skills

![Trigger warning animals](https://repository-images.githubusercontent.com/1171056549/6dee3166-2b48-449b-ad26-667ee821ff35)

AI agent skills for the [Symfony UX](https://ux.symfony.com) frontend stack -- Stimulus, Turbo, TwigComponent, LiveComponent, UX Icons and UX Map.

By [Simon Andre](https://github.com/smnandre)

## Skills

| Skill | What it does | When the agent activates it | Refs |
|---|---|---|:---:|
| **[symfony-ux](skills/symfony-ux/)** | Orchestrator / decision tree | The developer asks "which UX tool should I use?" or a question that spans multiple packages | -- |
| **[stimulus](skills/stimulus/)** | Stimulus controllers, targets, values, actions, outlets | Client-side JS behavior -- toggles, dropdowns, modals, wrapping a JS library | api, patterns, gotchas |
| **[turbo](skills/turbo/)** | Turbo Drive, Frames, Streams, Mercure | Partial page updates, SPA-like nav, real-time server pushes -- no JS to write | api, patterns, gotchas |
| **[twig-component](skills/twig-component/)** | TwigComponent props, blocks, computed properties, anonymous components | Reusable UI building blocks -- buttons, cards, alerts, design system | api, patterns, gotchas |
| **[live-component](skills/live-component/)** | LiveComponent props, actions, data-model, forms, emit, defer/lazy | Reactive server-rendered UI -- live search, validation, dependent selects | api, patterns, gotchas |
| **[ux-icons](skills/ux-icons/)** | SVG icons via Iconify, local files, aliases, CLI | Rendering icons in Twig -- Lucide, Heroicons, Tabler, Material Design, etc. | api, patterns, gotchas |
| **[ux-map](skills/ux-map/)** | Interactive maps with Leaflet / Google Maps | Maps with markers, polygons, polylines, circles, events, LiveComponent integration | api, patterns, gotchas |

**Upstream packages:**
[symfony/stimulus-bundle](https://packagist.org/packages/symfony/stimulus-bundle) --
[symfony/ux-turbo](https://packagist.org/packages/symfony/ux-turbo) --
[symfony/ux-twig-component](https://packagist.org/packages/symfony/ux-twig-component) --
[symfony/ux-live-component](https://packagist.org/packages/symfony/ux-live-component) --
[symfony/ux-icons](https://packagist.org/packages/symfony/ux-icons) --
[symfony/ux-map](https://packagist.org/packages/symfony/ux-map)

## Installation

### Claude Code Plugin

This repository is a [Claude Code plugin marketplace](https://docs.claude.com/en/docs/claude-code/plugins). Add the marketplace, then install the plugin. Skills are namespaced under `symfony-ux-skills:`.

```bash
claude plugin marketplace add smnandre/symfony-ux-skills
claude plugin install symfony-ux-skills@symfony-ux-skills
```

`smnandre/symfony-ux-skills` is the GitHub repository. `symfony-ux-skills@symfony-ux-skills` reads as `plugin@marketplace`.

To load the plugin locally without installing it:

```bash
claude --plugin-dir /path/to/symfony-ux-skills
```

### Vercel's Skills CLI

```bash
npx skills add smnandre/symfony-ux-skills
```

### Manual installation

Copy each skill directory into your platform's skills location:

```bash
# Claude Code (project-level, shared via git)
mkdir -p .claude/skills && cp -r skills/* .claude/skills/

# Claude Code (user-level, available everywhere)
cp -r skills/* ~/.claude/skills/

# Gemini CLI
mkdir -p ~/.gemini/skills && cp -r skills/* ~/.gemini/skills/

# OpenAI Codex
mkdir -p .codex/skills && cp -r skills/* .codex/skills/
```

Then optionally copy the context file for your platform to your project root:

```bash
cp CLAUDE.md /path/to/project/   # Claude Code
cp AGENTS.md /path/to/project/   # OpenAI Codex
cp GEMINI.md /path/to/project/   # Gemini CLI
```

## How it works

Agent skills are structured knowledge files that teach AI coding agents *how* to use a library. Instead of relying on training data (which may be outdated or incomplete), the agent reads the skill at runtime and gets accurate, version-specific guidance: API references, common patterns, and known gotchas.

Each skill follows a **progressive disclosure** pattern:

1. **Description** (YAML frontmatter) -- always loaded; tells the agent *when* to activate the skill (~100 words)
2. **SKILL.md body** -- loaded on activation; quick-reference with the most important rules and examples
3. **references/** -- loaded on demand; deep API docs, advanced patterns, and common pitfalls

This means the agent only pulls in what it needs, keeping context windows lean.

Built on the [Agent Skills](https://agentskills.io/specification) open standard. Compatible with **Claude Code**, **Gemini CLI**, **OpenAI Codex**, **Cursor**, **Windsurf**, and any platform that supports `SKILL.md`.

## Project context files

Optional files for your project root. They give the agent a quick decision tree and key rules so it knows which skill to reach for.

| File | Platform |
|---|---|
| `CLAUDE.md` | Claude Code |
| `AGENTS.md` | OpenAI Codex |
| `GEMINI.md` | Gemini CLI |
| `llms.txt` | Web / any LLM ([llmstxt.org](https://llmstxt.org)) |

## Repository structure

```
.
├── CLAUDE.md                   # Context file for Claude Code
├── AGENTS.md                   # Context file for OpenAI Codex
├── GEMINI.md                   # Context file for Gemini CLI
├── llms.txt                    # Context file for web / LLMs
├── .claude-plugin/
│   └── plugin.json             # Claude Code plugin manifest
├── gemini-extension.json       # Gemini CLI extension manifest
└── skills/
    ├── symfony-ux/
    │   └── SKILL.md
    ├── stimulus/
    │   ├── SKILL.md
    │   └── references/
    │       ├── api.md
    │       ├── patterns.md
    │       └── gotchas.md
    ├── turbo/
    │   ├── SKILL.md
    │   └── references/
    │       ├── api.md
    │       ├── patterns.md
    │       └── gotchas.md
    ├── twig-component/
    │   ├── SKILL.md
    │   └── references/
    │       ├── api.md
    │       ├── patterns.md
    │       └── gotchas.md
    ├── live-component/
    │   ├── SKILL.md
    │   └── references/
    │       ├── api.md
    │       ├── patterns.md
    │       └── gotchas.md
    ├── ux-icons/
    │   ├── SKILL.md
    │   └── references/
    │       ├── api.md
    │       ├── patterns.md
    │       └── gotchas.md
    └── ux-map/
        ├── SKILL.md
        └── references/
            ├── api.md
            ├── patterns.md
            └── gotchas.md
```

## Coverage

Targets **Symfony UX 2.22 -- 2.28+**, Symfony 7.2 / 7.4 / 8.0, PHP 8.4+.

Documented features include `<twig:Turbo:Stream:*>` component syntax (UX 2.22), `TurboStreamResponse` helper, LiveProp URL binding with validation modifiers, `defer` / `lazy` loading for LiveComponents, UX Toolkit (copy-paste UI components), Iconify on-demand icons with `ux:icons:lock` CLI, and UX Map with Leaflet/Google Maps renderers including polygons, polylines, circles and `ComponentWithMapTrait`.

## Author

Simon Andre -- [smnandre.dev](https://smnandre.dev) -- [GitHub](https://github.com/smnandre) -- [Twitter](https://x.com/simonandre)

## License

MIT
