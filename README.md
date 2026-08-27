# Open Science Infrastructure Self-Assessment Tool

A self-assessment tool for Open Science infrastructures. Name your
infrastructure, classify the activities it supports, and score it against
one or more assessment frameworks as compliant / making progress / not
compliant with optional notes, then export the result.

Two frameworks are included today:

- **[Principles of Open Scholarly Infrastructure (POSI) v2.0](https://openscholarlyinfrastructure.org/)**
  — 20 principles across Governance, Sustainability, and Insurance.
- **SPII v0.0.1** (draft) — 19 principles across Openness, Autonomy,
  Sustainability, Interoperability, and Researcher-centric.

## Features

- **Classify** — tag the infrastructure against a research activities
  taxonomy (research life cycle activities plus related activities such as
  managing, documenting, reviewing, publishing, and evaluating), each with
  its own icon.
- **Score** — rate every principle of the selected framework(s), one tab
  per framework, with a live-updating scored/total count in the tab label
  and a colored status dot (compliant / making progress / not compliant /
  unanswered) next to every principle in the sidebar navigation, so you can
  see progress at a glance without opening each section.
- **Bring your own framework** — import a custom assessment framework
  (JSON) at runtime alongside POSI and SPII, download a template to help
  author one, and export whichever frameworks are currently loaded.
- **Export as JSON** — download your results and reload them later to
  continue or revise an assessment.
- **Export as PDF** — download a report of the full assessment.
- **Assessment badge** — download a doughnut-style SVG/PNG badge
  summarizing the score for the active framework, including the classified
  activities as small icons, suitable for hosting on the infrastructure's
  own site.

## Usage

Also live at **[surf-ori.github.io/open-science-maturity](https://surf-ori.github.io/open-science-maturity/)**
(GitHub Pages, served from this repo's `main` branch — no separate deploy
step). Or open [`index.html`](index.html) directly in a browser, or serve
the directory with any static file server; there is nothing to install or
build — it's a single standalone HTML file. The top bar's Import/Export
menus cover loading and saving a report or a custom framework by hand;
"About" and "Changelog", also in the top bar, open as dialogs.

[`examples/`](examples/) has three example POSI assessment reports based
on published self-assessments from real infrastructures (OpenAIRE,
OpenAlex, HAL+/CCSD) — linked from the About dialog as one-click loads
against the live site (see below), or load one by hand via "Import
Assessment Report".

### Loading via URL parameters

When served over http(s) (not opened directly as a `file://` page, since
browsers block that kind of cross-file fetch), `index.html` can load a
report and/or a custom framework automatically from query parameters:

- `index.html?report=report5.json` — loads and merges an exported
  assessment report.
- `index.html?framework=custom2.json` — imports a custom assessment
  framework.
- `index.html?framework=custom2.json&report=report5.json` — both at once;
  the framework loads first, so the report's scores for it (if any) apply
  on top rather than being lost.

## Citing

If you use this tool, please cite it using the metadata in
[`CITATION.cff`](CITATION.cff) — including each author's ORCID and their
affiliation's ROR identifier — which GitHub's "Cite this repository"
button uses automatically.

## License

Licensed under the [EUPL-1.2](LICENSE).

Copyright (c) 2026:
- Till Bey ([ORCID](https://orcid.org/0000-0001-7509-9875)) — [SURF](https://ror.org/009vhk114)
- Maurice Vanderfeesten ([ORCID](https://orcid.org/0000-0001-6397-4759)) — [Vrije Universiteit Amsterdam](https://ror.org/008xxew50) & [SURF](https://ror.org/009vhk114)
- Sander Bosch ([ORCID](https://orcid.org/0000-0001-6845-0911)) — [Vrije Universiteit Amsterdam](https://ror.org/008xxew50)

## Credits

Styling is provided by [Oat](https://oat.ink), layered with color and font
tokens from the [SURF Design System](https://surfnet.github.io/DesignSystem/).
