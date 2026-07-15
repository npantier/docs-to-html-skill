# docs-to-html

A [Claude Code](https://claude.com/claude-code) skill that converts dense, table-heavy markdown docs into a single **self-contained, interactive, offline** HTML report — one file that opens with no server and no network. Emailable, hostable on S3/CloudFront, works locally.

Principles-only: the skill preserves the build mechanics and the time-costing gotchas, but leaves the visual design (layout, palette, typography) to the doc's content, its usage, and its target audience. It ships no template and no fixed styling.

## What it does

Two phases, gated by a plan-file checkpoint:

1. **Read & Plan** — reads the source doc(s), establishes audience + usage, inventories structure / big tables / status vocabulary / ticket IDs / cross-doc links / diagrams, proposes a design direction, flags anything that can't render deterministically or offline, and writes a `.plan.md` for your confirmation.
2. **Build & Verify** — on confirmation, builds the single self-contained HTML (inline CSS/JS, recreated diagrams, data-driven filterable tables, sticky TOC + scroll-spy, dark/light toggle, ticket auto-linking, print CSS), then runs a verification checklist (offline check, real-browser render, count reconciliation, print-preview).

## Requires

- Nothing mandatory — no dependencies, no build step. The ticket base URL for auto-linking is asked for at plan time, never hard-coded.
- **Optional:** a `frontend-design` skill, engaged during planning for palette / type / layout judgment when available; the skill's own **Design direction** principles apply otherwise.
- A browser is used for the Phase 2 verification render (served over local HTTP).

## Usage

```
/docs-to-html <path to a markdown doc or directory of docs>
```

The skill halts after Phase 1 with a plan for you to confirm. Example:

```
/docs-to-html docs/audit/example-audit.md
```

## Installation

The skill lives at `skills/docs-to-html/` in this repo. Claude Code discovers
personal skills one level deep — as `~/.claude/skills/<name>/SKILL.md` — so clone
the repo anywhere and symlink the inner skill directory into place:

```
git clone https://github.com/npantier/docs-to-html-skill.git ~/src/docs-to-html-skill
ln -s ~/src/docs-to-html-skill/skills/docs-to-html ~/.claude/skills/docs-to-html
```

That resolves to `~/.claude/skills/docs-to-html/SKILL.md`. Cloning the repo root
directly into `~/.claude/skills/` won't work — it buries `SKILL.md` too deep for
discovery.
