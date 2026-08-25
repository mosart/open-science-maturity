# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A self-assessment tool for Open Science infrastructures against the [Principles of Open Scholarly Infrastructure (POSI) v2.0](https://openscholarlyinfrastructure.org/). It is a single standalone `index.html` file — there is no build step, package manager, server, or committed test suite. Styling comes from the [Oat](https://oat.ink) CSS framework (vendored locally as `oat.min.css`) layered with color/font tokens hand-copied from the [SURF Design System](https://surfnet.github.io/DesignSystem/).

## Running / previewing

Just open `index.html` directly in a browser (or serve the directory with any static file server). There is nothing to install or build.

## Architecture

Everything lives in `index.html`: markup, inline `<style>`, and inline `<script>`. There is no JS framework — all rendering and interactivity is vanilla DOM manipulation, driven by a single in-memory `state` object.

**Data model** (`<script id="app-logic">` in `<head>`): `ACTIVITIES` is the "supports the following activities" taxonomy (flat labels plus one grouped entry for the Research Lifecycle sub-stages). `POSI` is the full Principles of Open Scholarly Infrastructure v2.0 content (3 sections — Governance, Sustainability, Insurance — 20 principles total), each principle with a stable `id`, `title`, and verbatim `text` transcribed from openscholarlyinfrastructure.org. `initial_state()` builds a fresh `state` object: `{name, activities, posi: {[id]: {status, notes}}}`, where `status` is `"compliant"`, `"progress"`, `"non-compliant"`, or `null`. `compute_summary(state)`, `build_export_object(state)`, `merge_loaded_state(loaded)`, and `ring_segment_offsets(radius, count)` are pure functions with no DOM access — this is what makes them testable outside a browser (see Testing below).

**Rendering** (`<script id="app-render">` in `<body>`): `render()` rebuilds `#app` entirely from the global `state` — name input, activity checkboxes, one `<details data-principle-id="...">` per POSI principle (3 status cards + a notes `<textarea>`), the live summary, the doughnut badge preview, and the actions row. Interaction handlers (`set_status`, `toggle_activity`, the notes/name input listeners) mutate `state` directly and call the narrower `update_summary()`/`update_badge()` rather than a full `render()`, so typing in a textarea doesn't lose focus. `render()` itself is only called on first load and after a JSON file is loaded (`load_json`), since those are the only times the whole DOM tree needs rebuilding from a new `state`.

**When adding a new POSI principle or activity**: add it to the `POSI`/`ACTIVITIES` data in `<script id="app-logic">` — the UI, scoring, JSON export/reload, and badge all render from that data automatically. Don't hand-edit the generated `<details>` markup; it's rebuilt by `render()` on every load.

**Export flows**:
- `download_json()` / `load_json(file)` serialize/deserialize `state` directly via `build_export_object`/`merge_loaded_state` — no DOM scraping.
- `download_pdf()` renders `document.body` via `jsPDF` (loaded from a CDN, `doc.html()`) into `report.pdf`. `@media print` rules hide the actions row and status cards, and the `beforeprint` listener collapses all `<details>` so the PDF/print view only shows scored summaries.
- `build_badge_svg_markup(state)` builds a self-contained `<svg>` — 3 concentric rings (Governance/Sustainability/Insurance), each ring split into one arc per principle via `ring_segment_offsets`, colored by that principle's status. `download_badge_svg()`/`download_badge_png()` export it as a static image the infrastructure can host on its own site; there is no live/dynamic embed since the tool has no backend.

**Styling**: color and font tokens are hand-copied from the [SURF Design System](https://surfnet.github.io/DesignSystem/) as CSS custom properties (`--primary`, `--warning`, `--destructive`, `--compliant`, `--font-sans`) in `:root`, with dark-mode overrides under `@media (prefers-color-scheme: dark)`. This is a copy of token *values*, not a dependency on SURF's packages — the page has no new runtime dependency beyond the pre-existing jsPDF/html2canvas CDN scripts.

## Testing

There is no committed test suite — this matches the project's "no build step" philosophy. The pure functions in `<script id="app-logic">` (`compute_summary`, `build_export_object`, `merge_loaded_state`, `ring_segment_offsets`) have no DOM dependency and can be exercised from Node by extracting that script block's text (regex on `<script id="app-logic">...</script>`) and running it in a `vm.createContext`, if you need to verify logic changes without a browser. Everything else (rendering, interactivity, PDF, badge image export) is verified manually by opening `index.html` in a browser.

## Notes

- A `<span id="watermark">DRAFT – WORK IN PROGRESS</span>` marks the tool as unfinished; remove it only when the content is considered complete.
- The linked `jspdf` and `html2canvas` scripts are pulled from `unpkg.com` / `html2canvas.hertzen.com` CDNs — there is no local vendoring for these two.
