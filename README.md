# Open Science infrastructure maturity assessment

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
- **Score** — rate every principle of the selected framework(s), with a
  live-updating compliance summary per framework.
- **Export as JSON** — download your results and reload them later to
  continue or revise an assessment.
- **Export as PDF** — download a report of the full assessment.
- **Assessment badge** — download a doughnut-style SVG/PNG badge
  summarizing the score for the active framework, including the classified
  activities as small icons, suitable for hosting on the infrastructure's
  own site.

## Usage

Open [`index.html`](index.html) directly in a browser, or serve the
directory with any static file server. There is nothing to install or
build — it's a single standalone HTML file.

## Citing

If you use this tool, please cite it using the metadata in
[`CITATION.cff`](CITATION.cff) (GitHub's "Cite this repository" button uses
this automatically).

## License

Licensed under the [EUPL-1.2](LICENSE).
Copyright (c) 2026 Till Bey, Maurice Vanderfeesten, and Sander Bosch.

## Credits

Styling is provided by [Oat](https://oat.ink), layered with color and font
tokens from the [SURF Design System](https://surfnet.github.io/DesignSystem/).
