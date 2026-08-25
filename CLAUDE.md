# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A self-assessment tool for the maturity of Open Science infrastructures, based on the values and principles collated in the SPII project. It is a single standalone `index.html` file — there is no build step, package manager, server, or test suite. Styling comes from the [Oat](https://oat.ink) CSS framework, vendored locally as `oat.min.css`.

## Running / previewing

Just open `index.html` directly in a browser (or serve the directory with any static file server). There is nothing to install or build.

## Architecture

Everything lives in `index.html`: markup, inline `<style>`, and inline `<script>`. There is no JS framework — all interactivity is vanilla DOM manipulation.

**Content structure**: The assessment is organized as `<section>`s, one per principle (e.g. "Open", "Autonomy", "Sustainable", "Usable"). Each section contains one or more `<details class="container" name="subprinciple">` blocks — one per subprinciple. Each subprinciple has:
- a `<summary>` with an `<h3>` title and a `<meter id="subprinciple-meter">` (range 0–4) showing the currently selected maturity score
- a `<div class="row">` of four `<article class="card level-N">` elements (N = 1..4), each describing one maturity level

A commented-out block near the top of `<main>` (around line 137 of `index.html`) is the canonical template for adding a new subprinciple — copy it rather than hand-rolling the structure.

**Scoring flow** (all in the inline `<script>` blocks):
- Click/touch handlers are wired up at the bottom of the file by selecting `.level-1` through `.level-4` elements and attaching listeners that call `clicked(el, value)`.
- `clicked()` sets the sibling `<meter>` value to the clicked level and visually highlights the selected card (orange border), then calls `update_score()`.
- `update_score()` averages all subprinciple `<meter>` values that have been scored (value > 0) and writes the result into `#overall-score` / `#overall-meter`.
- `download_json()` walks all `<summary>` elements and exports `{subprinciple, value}` pairs as `results.json`.
- `download_pdf()` renders `document.body` via `jsPDF` (loaded from a CDN, `doc.html()`) into `report.pdf`. `@media print` rules hide the `.value` cards and buttons, and the `beforeprint` listener collapses all `<details>` so the PDF/print view only shows scored summaries, not the full level descriptions.

**When adding a new subprinciple or principle**: follow the existing DOM structure exactly (`id="subprinciple-meter"` on the meter, `level-1`..`level-4` classes on the cards) since the scoring/export/print logic all selects elements by these conventions rather than by unique IDs — the `id` values are intentionally duplicated across subprinciples.

## Notes

- A `<span id="watermark">DRAFT – WORK IN PROGRESS</span>` marks the tool as unfinished; remove it only when the content is considered complete.
- The linked `jspdf` and `html2canvas` scripts are pulled from `unpkg.com` / `html2canvas.hertzen.com` CDNs — there is no local vendoring for these two.
