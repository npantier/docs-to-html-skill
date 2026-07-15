---
name: docs-to-html
description: >-
  Use when the user asks to turn dense markdown doc(s) into a single
  self-contained, interactive, offline HTML report or page — one file that opens
  with no server and no network. Trigger on "docs to html", "markdown to html
  report", "make an interactive HTML report", "self-contained HTML page",
  "single-file report", or a status / inventory / audit doc that needs status
  pills, a sticky TOC with scroll-spy, filterable tables, a dark/light toggle,
  ticket auto-linking, or print CSS. Principles-only — it preserves the build
  mechanics and gotchas but leaves the visual design to the doc, its usage, and
  its audience. Inventory-first with a confirmation checkpoint; halts at content
  that can't render deterministically or offline rather than guessing.
argument-hint: "<path to a markdown doc or directory of docs>"
---

# Docs → self-contained HTML report

Turn dense, table-heavy markdown into a single self-contained, interactive,
**offline** HTML file — emailable, hostable on S3/CloudFront, opens locally, works
with no server or network. Two phases: **Read & Plan** (produces a `.plan.md` and
halts) then on confirmation **Build & Verify**.

This skill is a **guided report builder — not a generic md→html converter and not a
fixed template.** It preserves the *engineering mechanics and the time-costing
gotchas*; it ships **no** visual design. Layout, palette, typography, and component
styling are your call, driven by the doc's content, its intended usage, and its
target audience. The build mechanics, reusable component techniques, gotcha
catalog, and verification checklist live in
[`references/techniques.md`](references/techniques.md) — read it before building.

**Provenance:** distilled from one real conversion (n=1). It teaches technique and
flags risk honestly; it carries no proven/provisional confidence matrix.

## Decision framework (settle these in Phase 1)

| Decision | Default | Go the other way when |
|----------|---------|-----------------------|
| One file vs many | One self-contained file | Docs are truly independent audiences, never cross-referenced |
| Multiple source docs in one file | Top-level **view switcher** (tabs) | A doc is short enough to inline as another section |
| Rendering a big table | **Data-driven** once a repeating table exceeds ~50 rows | Small tables → plain static HTML tables |
| Dependencies | Fully self-contained (inline CSS/JS, no CDN/webfont) | An interactive lib is unavoidable — then justify it |
| Diagrams (Mermaid etc.) | Recreate as inline styled HTML/SVG | Diagram is complex enough that hand-rebuild isn't worth it |

## Phase 1 — Read & Plan (static; ends at a checkpoint)

**Emit no HTML and touch no source files in Phase 1.**

1. **Resolve the doc set from the live tree.** If a path argument was given,
   Glob/Read it. If none was given, **ask which doc(s) to convert** — offer the
   markdown files found in the current dir as candidates, or take a typed
   path/glob; don't silently convert the whole cwd. Surface contradictions —
   `.mdx`, embedded HTML, missing files — rather than proceeding on the stated
   format.
2. **Establish context.** Target audience and intended usage (emailed / hosted /
   opened locally offline). This drives the design direction — ask if not stated.
3. **Inventory.** Structure and headings; big repeating tables (data-driven
   candidates); status vocabulary; ticket/ID patterns; cross-doc links; diagrams;
   embedded HTML/scripts.
4. **Propose a design direction** — described, not coded. *Structure:* split
   mode, which table(s) are data-driven, the status→pill vocabulary, the ticket
   base URL (ask the user — never hard-code an org's issue-tracker host; skip
   ticket-linking if the doc has no IDs). *Aesthetics:* set a deliberate visual
   direction — engage the `frontend-design` skill if available, else apply the
   **Design direction** principles in
   [`references/techniques.md`](references/techniques.md) — and produce a token
   system (palette, system-font type roles, layout + one signature element)
   within the self-containment constraints. Both parts go in the plan.
5. 🚦 **Flag content that can't render deterministically or offline — HALT, don't
   guess.** Embedded `<script>`, remote/external assets, dynamic content,
   unresolved relative image paths, non-deterministic table shapes, ambiguous
   status words. Surface each; do not fabricate a rendering.
6. **Write the plan file and HALT.** Filename
   `docs-to-html_{slug}_{short-hash}.plan.md` where `{slug}` is a kebab-case label
   for the target (basename) and `{short-hash}` is `git rev-parse --short HEAD` (or
   `no-git`). Save under `.cursor/plans/` if it exists, else `docs/plans/`, else the
   repo/project root. Present the plan inline and ask the user to confirm before
   Phase 2.

The plan file carries: detected setup (source docs + formats), inventory, a
construct→HTML-feature mapping table (heading→TOC entry, status word→pill, pipe
table→filterable table, `PROJ-123`→auto-link), the **design token system**
(palette, type roles, layout + signature), `## Planned build (Phase 2)` as a
`- [ ]` checklist, and `## 🚦 Escalations (will halt — not auto-resolved)` as plain
`-` bullets.

## Phase 2 — Build & Verify (on confirmation)

Approval of the plan is approval to build the listed items in one pass (no per-edit
gate); the 🚦 escalation branches still halt. Read
[`references/techniques.md`](references/techniques.md) for every mechanic below.

7. **Build the single self-contained file** per the approved design direction —
   inline CSS in one `<style>`, inline JS in one `<script>`, system fonts, diagrams
   recreated as styled HTML/SVG. Write to `<docname>.html` next to the source (or
   the out-path you were given). **Never overwrite an existing file without
   confirming.**
8. **Emit big table(s) as `data-*`-tagged `<tr>` rows.** Vanilla JS drives substring
   search, `Set`-membership filters, a domain `<select>`, and a live "showing N of
   TOTAL"; per-section tallies and the global distribution are computed from the
   rendered rows at load — never hand-typed.
9. **Cross-cutting.** Ticket auto-linker (base URL + regex from the plan), cross-doc
   links rewritten to in-page view switches, arrow/entity normalization.
10. **Fidelity pass.** Every row, note, ticket, and section from the source appears
    — a faithful rendering, not a summary. Tables→tables, lists→lists, code spans
    kept.
11. **Verify** (full checklist in `references/techniques.md`): the offline grep is
    clean (self-referential content can false-positive — trust the browser network
    panel); render in a real browser served over HTTP (that one command runs with
    the sandbox disabled — a sandboxed shell can't bind a listen port); exercise
    every interaction in both themes; reconcile counts to source totals;
    print-preview is clean.
12. **Report.** What was built, the verify results, and everything that needed a
    human — halted content, the chosen ticket base URL, any overwrite decision.

## Reference

- Build mechanics, component techniques, gotcha catalog, verification checklist →
  [`references/techniques.md`](references/techniques.md)
