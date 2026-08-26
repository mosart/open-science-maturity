# Layout & SURF Design System alignment — implementation plan

**Status:** not started — written down for a future session to pick up.

**Goal:** The page currently looks rough because its custom `<style>` block
(`index.html:8-136`) fights the Oat framework instead of using it: ad hoc `pt`
units, hand-rolled flexbox, and invented color tokens duplicate things Oat
(`oat.min.css`) already provides. Fix the layout by leaning on Oat's existing
grid/spacing/component system, with color and type tokens matching the SURF
Design System, per `CLAUDE.md`'s stated architecture ("Oat layered with
color/font tokens hand-copied from the SURF Design System").

**Non-goals:** no new build step, no new runtime dependency, no framework —
`index.html` stays a single standalone file (see `CLAUDE.md`'s "Global
Constraints"). This is a CSS/markup pass, not a rewrite of `app-render`'s
rendering logic — `render()`/`render_*()` functions keep their current shape;
only which classes/inline styles they apply changes.

**Before starting:** `surfnet.github.io` was unreachable from this sandbox
(egress-proxy blocked) when this plan was written, so the exact SURF brand
values below are inherited from what's already hand-copied into `:root`
(`index.html:9-30`), not freshly re-verified against the live design system.
Re-check https://surfnet.github.io/DesignSystem/ for current color/type
tokens before implementing, and update the values below if they've drifted.

---

## Root cause

1. **Two overlapping color systems.** Oat already ships semantic status
   colors (`--success`, `--warning`, `--danger`, plus `--card`, `--muted`,
   `--border`, `--accent` — see `oat.min.css`'s `:root`), but the page's own
   `<style>` block invents `--compliant`/`--destructive` instead of reusing
   `--success`/`--danger`, and redefines `--warning` and `--primary`
   redundantly. Result: some elements pick up Oat's palette, others the
   custom one, and they don't visually match.
2. **`pt` units instead of Oat's spacing scale.** `body { margin: 20pt }`,
   `h2 { margin-left: 20pt }`, `section > p { margin-left: 24pt }`,
   `.status-cards { padding-right: 24pt }`, etc. all hand-pick pixel-ish
   values instead of `var(--space-1)` … `var(--space-18)`. Nothing lines up
   to a shared rhythm.
3. **No use of Oat's `.container`.** The page has no max-width — Oat defines
   `--container-max: 1280px` via `.container`, but `<body>` doesn't use it,
   so text stretches edge-to-edge on wide screens.
4. **Manual flexbox instead of Oat's grid.** `.status-cards` is a hand-rolled
   `display:flex` row; the activities checklist is a plain stack of
   `<label style="display:block">`. Oat already ships a 12-col `.row`/
   `.col-*` grid (`oat.min.css`: `.row{display:grid;grid-template-columns:
   repeat(var(--grid-cols),1fr)...}`) that both should use instead.
5. **Overridden component defaults.** Oat's `details`/`summary` already have
   sensible border/radius/hover/spacing (`oat.min.css`: `details{border:1px
   solid var(--border); ...}`, `details>*:not(summary){margin:
   var(--space-4)}`). The custom `summary { padding: 10pt }` and
   `section > p { margin-left: 24pt }` rules conflict with these defaults
   instead of extending them.
6. **`#watermark` breaks in dark mode.** `color: black` (`index.html:114`)
   is hardcoded — invisible/wrong-contrast against Oat's dark background.

---

## Task 1 — Reconcile color tokens with Oat's semantic palette

**File:** `index.html`, `:root` block (`index.html:9-30`).

- Verify current SURF hex/rgba values against
  https://surfnet.github.io/DesignSystem/ (primary blue, warning amber,
  destructive red, compliant teal, `Source Sans 3` font) and update if the
  design system has changed since these were copied.
- Stop inventing new token names. Map POSI statuses onto Oat's own semantic
  vars instead of parallel ones:
  - `compliant` → override Oat's `--success` (currently `#008032`/`#6cc070`)
    with the SURF teal value, instead of a separate `--compliant`.
  - `progress` → override Oat's existing `--warning` (Oat already defines
    it) with the SURF amber value — don't redeclare it as a "new" token.
  - `non-compliant` → override Oat's `--danger` (currently `#d32f2f`/
    `#f4807b`) with the SURF red value, instead of a separate
    `--destructive`.
  - `--primary`/`--primary-foreground` → Oat already defines both; only
    override the *values* with SURF's brand blue, don't redeclare
    `--primary-foreground` if the SURF white already matches Oat's.
- Update every consumer of the old names (`var(--compliant)`,
  `var(--destructive)`) throughout `<style>` and the JS-generated inline SVG
  (`STATUS_COLOR_VAR` in `build_badge_svg_markup`, `index.html:569-573`) to
  the reconciled `--success`/`--warning`/`--danger` names.
- Fix `#watermark`'s hardcoded `color: black` → `var(--foreground)` (Oat's
  light-dark-aware text color) so it's legible in both themes.

## Task 2 — Adopt Oat's container + spacing scale for page structure

**File:** `index.html`, `<body>` and the `body`/`section`/`h2`/`h3` rules.

- Wrap the page content in Oat's `.container` (`max-width: var(
  --container-max); margin-inline: auto; padding-inline: var(
  --container-pad)`) instead of a bare `body { margin: 20pt }` — either add
  a wrapping `<div class="container">` around `#app`'s children in `render()`,
  or apply `.container` directly to `<main id="app">`.
- Replace every hardcoded `pt` margin/padding in the custom `<style>` block
  (`h2`, `h3`, `section > p`, `div.row`, `section`, `summary`,
  `.status-cards`, `.taxonomy-group`, `#summary-section`) with the matching
  `var(--space-*)` token, picking the closest existing step rather than
  introducing new magic numbers.
- Let Oat's own `details`/`summary` spacing (`details > *:not(summary) {
  margin: var(--space-4) }`) do its job instead of re-declaring
  `section > p { margin-left: 24pt }` for the principle text — that rule is
  currently fighting Oat's own margin on the `<p>` inside each `<details>`.
- Re-check `h2`/`h3` sizing: replace `font-size: x-large`/`medium` with
  Oat's `--text-*` clamp scale (`--text-2`/`--text-3` are natural section/
  subsection sizes) so headings scale responsively like the rest of the
  design system instead of using static browser keyword sizes.

## Task 3 — Use Oat's grid for the activities checklist and status cards

**File:** `index.html`, `render_classify_section()`/`render_activity_checkbox()`
(`index.html:315-358`) and `render_principle()`'s status-cards block
(`index.html:389-411`).

- Activities checklist: currently one long vertical stack of
  `<label style="display:block">`. Wrap the flat items (and each grouped
  sub-list) in Oat's `.row` + `.col-*` grid so they lay out in 2–3 responsive
  columns instead of a single column that runs the length of the page —
  e.g. `<div class="row">` with each checkbox `<label>` as `class="col-4"`
  (adjust span to taste against the existing `--grid-cols: 12` default).
- Status cards: replace the manual `.status-cards { display:flex; gap:8pt }`
  with Oat's own grid (`class="row"` on `.status-cards`, `col-4` on each
  `.status-card` so the three options split evenly) so spacing/gap come from
  `--grid-gap` instead of a custom `padding-right`.
- Confirm `.status-card`'s `.card` base styling (background/border/radius/
  shadow from Oat) actually renders now that it's inside a proper grid cell
  instead of a flex item — screenshot-check in a browser.

## Task 4 — Style the name input and summary as first-class components

**File:** `index.html`, `render_header()` (`index.html:282-300`) and
`render_summary_section()`/`#summary-section` styling (`index.html:92-102`,
`302-313`).

- The infrastructure-name `<input>` currently gets inline
  `style.width = "200pt"` with no visual treatment beyond the browser
  default. Check whether Oat styles bare `input[type=text]` already (it
  likely does, given `--input`/`--radius-medium` tokens exist); if so, drop
  the inline style entirely and let it inherit; if not, add a small rule
  using `var(--input)`/`var(--radius-medium)`/`var(--space-2)` consistent
  with button styling.
- Turn the "Overall: N% compliant" summary into a prominent stat tile: wrap
  it in `.card`, size the percentage with a large `--text-1`/`--text-2`
  weight, and color it via the reconciled status tokens from Task 1 (e.g.
  green-ish when mostly compliant) rather than plain inline text next to a
  `<span>`.

## Task 5 — Verify print/PDF export still works

**Files:** `index.html`'s `@media print` block and `body.printing` rules
(`index.html:120-136`), `download_pdf()` (`index.html:525-544`).

- The print/PDF paths hide `#actions`, `#badge-section button`, and
  `.status-card:not(.selected)` by selector — confirm those selectors still
  match after Tasks 1-4 change surrounding markup/classes (e.g. if
  `.status-cards` gains a `.row` class alongside its own, selectors keyed on
  `.status-card` are unaffected, but double-check nothing renamed).
- Re-run the manual verification steps from
  `docs/superpowers/plans/2026-08-25-posi-self-assessment.md` (Task 4) for
  both the in-app "Download report" button and the browser's native
  Ctrl/Cmd+P print preview, in both light and dark OS themes.

## Task 6 — Manual verification checklist

Since there's no committed test suite (see `CLAUDE.md`'s Testing section),
verify by opening `index.html` directly in a browser:

- [ ] Page content is centered with a readable max-width on a wide monitor.
- [ ] Heading sizes scale smoothly when resizing the window (clamp-based).
- [ ] Activities checklist lays out in multiple columns on desktop, collapses
      to one column on narrow/mobile widths.
- [ ] Status cards (Compliant / Making progress / Not compliant) sit in an
      even 3-column row with consistent gap, each showing `.card` styling
      (background, border, radius, shadow) and the correct selected-state
      color border.
- [ ] Toggle OS/browser dark mode — every color (status cards, watermark,
      links, focus ring, badge preview) remains legible and uses the
      reconciled tokens, not a hardcoded light-mode color.
- [ ] The "Overall" summary tile is visually prominent and its color tracks
      the reconciled status tokens.
- [ ] "Download report" (PDF) and the browser's native print preview both
      still hide the actions row / unselected status cards and force-open
      all `<details>`, matching current behavior.
- [ ] The doughnut badge preview/SVG/PNG export still renders with the
      reconciled `--success`/`--warning`/`--danger` colors (not the old
      `--compliant`/`--destructive` names, which will silently fall back to
      `#cccccc` in `status_color()` if left unmigrated anywhere).

## Suggested execution

This is sized for `superpowers:writing-plans` → `superpowers:executing-plans`
or `superpowers:subagent-driven-development` once picked up — Tasks 1-2
(tokens + spacing) are prerequisites for Tasks 3-4 (grid + components), and
Task 5 (print/PDF) should run last since it depends on the final markup.
