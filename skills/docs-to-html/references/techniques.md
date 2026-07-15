# Techniques reference

Mechanics for turning a dense markdown doc into a single self-contained, interactive HTML report — the self-containment rules, the reusable component kit, the data-table pattern, the cross-cutting behaviors, the gotchas that cost time, and the checklist to run before shipping.

## Self-containment

Inline **all** CSS in one `<style>` and **all** JS in one `<script>`. No `<link>`, no `<script src>`, no `@import`, no `url(https://…)`.

Use system-font stacks only — no webfonts. Distinctive is still possible: pair a system *serif* display stack (`ui-serif, "Iowan Old Style", Palatino, Georgia`) with a system sans body and a mono stack for IDs/code. Serif headings + sans body reads editorial, not default.

Recreate diagrams rather than pulling in a runtime (Mermaid). A dependency-graph flowchart, for example, becomes a few flex rows of styled nodes and arrows — it keeps the file offline and keeps it smaller.

Verify offline-ness with one grep before shipping: `grep -nE '(<link|<script[^>]*src|@import|url\(http)' file.html`. It should return empty — but the pattern matches text anywhere in the file, so a doc whose own content is about `@import`, `<link>`, or `<script src>` will show hits inside `<code>`/escaped body spans that are not real resource loads. When that happens the browser network pass (see the checklist) is authoritative — confirm zero external requests fire — or scope the grep to the `<head>`/`<style>` block.

## Design direction

Choose the visual direction deliberately before building — don't default into it. If the `frontend-design` skill is available, use it for palette, type, and layout judgment; with or without it, the direction must fit this file's constraints: all CSS/JS inline, system fonts only, fully offline (no webfonts, no CDN). Produce a compact token system and record it in the plan so it can be confirmed before the build:

- **Palette** — 4–6 named hex values, defined for light *and* dark, driven by CSS custom properties.
- **Type** — display / body / mono roles as system-font stacks. A characterful pairing survives without webfonts: a system serif display against a sans body, a mono face for IDs and code.
- **Layout + signature** — a one-line layout concept plus exactly one signature element drawn from the doc's own subject. Spend boldness in one place and keep the rest quiet.

Ground every choice in the doc's subject and vocabulary. Avoid the three templated AI-default looks unless one is a deliberate fit: (1) cream background + high-contrast serif + terracotta accent, (2) near-black + a single acid-green/vermilion accent, (3) broadsheet hairline rules in dense columns.

## Component techniques

These are techniques, not finished styles — all driven by CSS custom properties, so a light/dark toggle is ~20 lines.

- **Status pills** — map source glyphs to semantic, labeled, colored chips (`.pill.built|partial|gap|deferred|donotport` is an example vocabulary). **Never render raw emoji as data** — chips are accessible, filterable, and print correctly. Keep a matching legend.
- **Priority chips** — a second, visually distinct chip family (filled / outline / strikethrough) so priority never reads as status.
- **Stat cards** (`.stat`) for headline numbers; **callout cards** (`.callout.risk|action`) for risk/action lists.
- **Sticky TOC with scroll-spy**, built *from the headings* by querying `[data-toc]` — not hand-maintained. Indent `h3` entries. `data-num` renders a section number. Set `scroll-margin-top` on anchor targets so a jump lands below the sticky chrome.
- **Theme toggle** with `localStorage` persistence, driven by `data-theme` on `<html>`.
- **Responsive + print stylesheets.** Print CSS force-shows filtered/hidden rows (`tr[hidden]{display:table-row!important}`), expands collapsed sections, and drops the chrome (topbar / toc / toolbar).

## Data-driven tables

For a doc dominated by one large repeating table, render static `<tr>` rows carrying `data-*` attributes; vanilla JS filters by toggling `hidden`:

```html
<tr data-domain="FLT" data-status="partial" data-pri="CORE"> … </tr>
```

Search is a substring match on `tr.textContent`. Filters are `Set` membership on `data-status` / `data-pri`, plus a domain `<select>`. The live count reads "showing N of TOTAL".

Compute per-section tallies and the global distribution **from the rendered rows at load time** — don't hand-type counts. They drift, and computing them from the DOM doubles as a fidelity self-check.

**Key learning — static rows beat a JS data array.** Long evidence text full of apostrophes, quotes, backticks, and `<`/`>` is error-prone to escape into JS string literals. Static HTML rows need only HTML-entity escaping (`&` `<` `>`), stay readable in source, and still deliver the full filter/search/count UX. Reach for a JS array only when rows are short and quote-clean.

## Cross-cutting behaviors

**Ticket / ID auto-linking.** One JS pass walks text nodes (skipping `A`/`CODE`/`SCRIPT`/`STYLE`) and wraps IDs matching a per-org regex in links to the ticket base URL. Parameterize the base URL and the ID regex — it's whatever tracker the doc uses (Jira, GitHub Issues, Linear, …), never a hard-coded host.

**Cross-doc links become in-page nav.** A markdown `[x](other.md)` link becomes an in-page view switch (`onclick="__show('crosswalk')"`), not a dead file link.

**Fidelity is non-negotiable.** Every row, note, ticket, and section appears — a faithful rendering, not a summary. Convert `->` to `→`; escape literal entity text (`&#39;` becomes `&amp;#39;`) so it displays verbatim.

## Gotchas

The seven that cost time:

1. `file://` is blocked in Playwright/Chromium automation — serve the dir over HTTP (`python3 -m http.server`) and navigate to localhost.
2. Sandboxed shells can't bind a listen port ("Operation not permitted" on `socket.bind`) — run the local HTTP server with the sandbox disabled for that one command.
3. Inline-CSS edits get cached — after editing, hard-reload with a cache-busting query (`?v=2`) or you screenshot stale styles and chase a phantom bug.
4. Default table headers to non-sticky — a sticky toolbar + `position:sticky` `thead` overlap, and a short table's header floats mid-body as it scrolls past the sticky line. Leave `thead` non-sticky unless a table is genuinely long. Set `scroll-margin-top` on anchor targets so TOC jumps clear the sticky chrome.
5. Emoji encoding — source status glyphs may be multi-byte; convert to semantic classes early rather than carrying emoji through the pipeline.
6. `favicon.ico` 404 in the console is browser-automatic and harmless — not a real dependency.
7. Arrows / entities in notes — convert ASCII `->` to `→`; escape literal `&#39;`-style entity text as `&amp;#39;`.

## Verification checklist

Do all of these:

- Row/section counts match the source (`grep -o 'data-domain="[A-Z0-9]*"' | sort | uniq -c`).
- Reconcile the distribution to source totals — watch for off-by-N from control elements that also carry `data-*` (e.g. filter buttons carrying `data-status` inflate the raw count).
- Render in a real browser; check the console (only the favicon 404 is acceptable).
- Exercise every interaction — view switch, theme toggle, each filter, priority filter, domain select, search, reset — assert the live count after each.
- Screenshot both views in both themes.
- Confirm zero external resource loads. The offline grep from Self-containment is the quick first pass, but it false-positives on docs whose content mentions `@import` / `<link>` / `<script src>` — for those, treat the browser network panel (zero external requests) as authoritative.
- Print-preview is clean (chrome hidden, hidden rows shown).
