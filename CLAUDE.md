# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A self-assessment tool for Open Science infrastructures against the [Principles of Open Scholarly Infrastructure (POSI) v2.0](https://openscholarlyinfrastructure.org/). It is a single standalone `index.html` file — there is no build step, package manager, server, or committed test suite. Styling comes from the [Oat](https://oat.ink) CSS framework (vendored locally as `oat.min.css`) layered with color/font tokens hand-copied from the [SURF Design System](https://surfnet.github.io/DesignSystem/).

## Running / previewing

Just open `index.html` directly in a browser (or serve the directory with any static file server). There is nothing to install or build.

## Architecture

Everything lives in `index.html`: markup, inline `<style>`, and inline `<script>`. There is no JS framework — all rendering and interactivity is vanilla DOM manipulation, driven by a single in-memory `state` object.

**Data model** (`<script id="app-logic">` in `<head>`): `FRAMEWORKS` is a list of assessment frameworks — currently one entry, POSI v2.0 (`id: "posi"`), each with a `title`/`description`/`aboutHtml` and a `sections` array (3 sections — Governance, Sustainability, Insurance — 20 principles total; the raw content lives in the `POSI` constant, referenced by `FRAMEWORKS[0].sections`). Each principle has a stable `id`, `title`, and verbatim `text` transcribed from openscholarlyinfrastructure.org. `ACTIVITIES` is `{primary, related}` — two named, boxed groups for the "Classify" picker ("Primary research life cycle activities" and "Research related activities"). `ICONS` holds the three inline SVG status icons. `initial_state()` builds a fresh `state` object: `{name, description, url, assessedOn, activities, frameworks: {[frameworkId]: {[principleId]: {status, notes}}}}`, where `status` is `"compliant"`, `"progress"`, `"non-compliant"`, or `null`, and `assessedOn` defaults to today's date (`today_iso_date()`). `compute_summary(state, frameworkId)`, `build_export_object(state)`, `merge_loaded_state(loaded)`, and `ring_segment_offsets(radius, count)` are pure functions with no DOM access — this is what makes them testable outside a browser (see Testing below). Only `"posi"` has content today, but the shape supports adding a second framework without another data-model restructure.

**Rendering** (`<script id="app-render">` in `<body>`): `render()` rebuilds `#app` entirely from the global `state` — the header (title + "About this tool" accordion), the "OS Infrastructure Description" section (name/description/URL/assessment-date fields, plus "Classify" as a sub-heading with two boxed activity columns), one section per `FRAMEWORKS` entry (title, description, "About `<framework>`" accordion, framework-level summary, then one `<details data-principle-id="...">` per principle — 3 status cards with icons + a notes `<textarea>`). The grouped sidebar nav (`render_sidebar_nav`) and footer (`render_sidebar_footer`: the assessment badge + hovercard, then a collapsed "Export / import results" accordion holding the JSON/PDF controls) are rebuilt on every `render()` too. Interaction handlers (`set_status`, `toggle_activity`, the notes/field input listeners) mutate `state` directly and call the narrower `update_summary()`/`update_badge()` rather than a full `render()`, so typing in a field doesn't lose focus. `render()` itself is only called on first load and after a JSON file is loaded (`load_json`), since those are the only times the whole DOM tree needs rebuilding from a new `state`.

**When adding a new POSI principle or activity**: add it to the `POSI`/`ACTIVITIES` data in `<script id="app-logic">` — the UI, scoring, JSON export/reload, and badge all render from that data automatically. Don't hand-edit the generated `<details>` markup; it's rebuilt by `render()` on every load.

**Export flows**:
- `download_json()` / `load_json(file)` (buttons live in the sidebar footer's "Export / import results" accordion) serialize/deserialize `state` directly via `build_export_object`/`merge_loaded_state` — no DOM scraping.
- `download_pdf()` renders `document.body` via `jsPDF` (loaded from a CDN, `doc.html()`) into `report.pdf`. Because `doc.html()` renders via html2canvas using the page's on-screen computed styles, it does not fire `beforeprint`/honor `@media print`, so `download_pdf()` does its own force-open of every `<details>` and toggles a plain `body.printing` class (mirrored by a non-media-query CSS rule) to hide the actions row, badge download buttons, and unselected status cards, restoring the original `<details>` open/closed state and removing the class once the PDF is saved. The browser's own print preview (Ctrl/Cmd+P) instead uses the `@media print` rules directly, with `beforeprint`/`afterprint` listeners that force-open all `<details>` and restore their prior state afterward.
- `build_badge_svg_markup(state)` builds a self-contained `<svg>` ("POSI Compliance" title, 3 concentric rings for Governance/Sustainability/Insurance each split into one arc per principle via `ring_segment_offsets`, the overall `%`, infrastructure name, and assessment date) — deliberately compact; the full per-section breakdown lives in `build_badge_hovercard_html(state)`, shown as a `.hovercard` on hover/focus of the badge (plain CSS, no new dependency). `download_badge_svg()`/`download_badge_png()` export the compact badge as a static image the infrastructure can host on its own site; there is no live/dynamic embed since the tool has no backend.

**Styling**: color and font tokens are hand-copied from the [SURF Design System](https://surfnet.github.io/DesignSystem/) as CSS custom properties (`--primary`, `--warning`, `--destructive`, `--compliant`, `--font-sans`) in `:root`, with dark-mode overrides under `@media (prefers-color-scheme: dark)`. This is a copy of token *values*, not a dependency on SURF's packages — the page has no new runtime dependency beyond the pre-existing jsPDF/html2canvas CDN scripts.

## Testing

There is no committed test suite — this matches the project's "no build step" philosophy. The pure functions in `<script id="app-logic">` (`compute_summary`, `build_export_object`, `merge_loaded_state`, `ring_segment_offsets`) have no DOM dependency and can be exercised from Node by extracting that script block's text (regex on `<script id="app-logic">...</script>`) and running it in a `vm.createContext`, if you need to verify logic changes without a browser. Everything else (rendering, interactivity, PDF, badge image export) is verified manually by opening `index.html` in a browser.

## Notes

- A `<span id="watermark">DRAFT – WORK IN PROGRESS</span>` marks the tool as unfinished; remove it only when the content is considered complete.
- The linked `jspdf` and `html2canvas` scripts are pulled from `unpkg.com` / `html2canvas.hertzen.com` CDNs — there is no local vendoring for these two.
