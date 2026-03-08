# Symfony UX -- Agent Skills

![Trigger warning animals](https://repository-images.githubusercontent.com/1171056549/6dee3166-2b48-449b-ad26-667ee821ff35)

AI agent skills for the **Symfony UX** frontend stack.

Built on the [Agent Skills](https://agentskills.io/specification) open standard.
Compatible with Claude Code, Gemini CLI, OpenAI Codex, Cursor, Windsurf, and any platform that supports `SKILL.md`.

---

## Skills

Each skill is a self-contained directory following the [agentskills.io](https://agentskills.io/specification) specification: a `SKILL.md` with YAML frontmatter and markdown instructions, plus optional `references/` for API docs, patterns and gotchas.

| Skill | Description | References |
|---|---|:---:|
| **symfony-ux** | Orchestrator -- decision tree for choosing between Stimulus, Turbo, TwigComponent and LiveComponent | -- |
| **stimulus** | Stimulus controllers, targets, values, actions, classes, outlets, lifecycle | api, patterns, gotchas |
| **turbo** | Turbo Drive, Frames, Streams, Mercure integration, `<twig:Turbo:Stream:*>` | api, patterns, gotchas |
| **twig-component** | TwigComponent props, blocks, computed properties, anonymous components, HTML syntax | api, patterns |
| **live-component** | LiveComponent props, actions, data-model binding, forms, emit, defer/lazy | api, patterns, gotchas |

## Project context files

Optional entry-point files for your project root. They provide a quick decision tree and key rules so the agent knows when to reach for which skill.

| File | Platform |
|---|---|
| `CLAUDE.md` | Claude Code |
| `AGENTS.md` | OpenAI Codex |
| `GEMINI.md` | Gemini CLI |
| `llms.txt` | Web / any LLM ([llmstxt.org](https://llmstxt.org) standard) |

## Repository structure

```
.
├── .claude-plugin/
│   └── plugin.json
├── README.md
├── LICENSE
├── CLAUDE.md
├── AGENTS.md
├── GEMINI.md
├── llms.txt
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
    │       └── patterns.md
    └── live-component/
        ├── SKILL.md
        └── references/
            ├── api.md
            ├── patterns.md
            └── gotchas.md
```

## Installation

### Claude Code Plugin (recommended)

This repository is installable as a [Claude Code plugin](https://code.claude.com/docs/en/plugins). Skills are automatically discovered and namespaced under `symfony-ux:`.

```bash
# Test locally
claude --plugin-dir /path/to/symfony-ux-skills

# Or install from a marketplace (if available)
claude plugin install symfony-ux
```

### Manual installation

Copy each skill directory into the skills location for your platform.

**Claude Code:**

```bash
# User-level (available in all projects)
cp -r skills/stimulus    ~/.claude/skills/
cp -r skills/turbo       ~/.claude/skills/
cp -r skills/twig-component  ~/.claude/skills/
cp -r skills/live-component  ~/.claude/skills/
cp -r skills/symfony-ux  ~/.claude/skills/

# Or project-level (shared via git)
mkdir -p .claude/skills
cp -r skills/* .claude/skills/
```

**Gemini CLI:**

```bash
mkdir -p ~/.gemini/skills
cp -r skills/* ~/.gemini/skills/
```

**OpenAI Codex:**

```bash
# User-level
mkdir -p ~/.codex/skills
cp -r skills/* ~/.codex/skills/

# Or project-level
mkdir -p .codex/skills
cp -r skills/* .codex/skills/
```

### 2. Add project context (optional)

Copy the entry-point file for your platform to your project root:

```bash
cp CLAUDE.md /path/to/project/   # Claude Code
cp AGENTS.md /path/to/project/   # OpenAI Codex
cp GEMINI.md /path/to/project/   # Gemini CLI
```

These files reference skills by name. The agent discovers them automatically from the skills directory.

## Coverage

Targets Symfony UX 2.22--2.28+, Symfony 7.2 / 7.4 / 8.0, PHP 8.4+.

Documented features include:

- `<twig:Turbo:Stream:*>` Twig component syntax (since UX 2.22)
- `TurboStreamResponse` helper
- LiveProp URL binding and validation modifiers (`min_length`, `max_length`, `min_value`, `max_value`)
- CVE-2025-47946 -- `ComponentAttributes` injection advisory
- `defer` / `lazy` loading for LiveComponents
- UX Toolkit (copy-paste UI components)

## Author

Simon Andre -- [smnandre.dev](https://smnandre.dev) -- smn.andre@gmail.com

## License

MIT
