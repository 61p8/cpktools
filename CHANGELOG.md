# Changelog

All notable changes to this project. Format loosely based on [Keep a Changelog](https://keepachangelog.com).

## [v1.0.1] — 2026-07

Repo review fixes. No feature changes.

### Fixed
- `.gitignore` was committed under the wrong filename (`download`) and never took effect — renamed back so backups, Excel files, and measurement JSON stay out of the repo
- Cleared USL/LSL became `0` after a page reload (NaN → null through the localStorage JSON round-trip slipped past `isNaN()`), silently producing wrong T/Cp/Cpk — spec limits are now checked with a strict numeric guard (`isNum`) everywhere, and empty inputs are stored as `null`
- Run Chart legend/tooltip hid any dataset whose *name* ended in "USL"/"LSL" (regex filter) — now filtered by the internal spec-line flag instead

### Security
- Added SRI `integrity` + `crossorigin` attributes to all three CDN scripts (SheetJS, Chart.js, Tesseract.js)
- Pinned Tesseract.js to exact version 5.1.1 (was floating `@5`, which would break SRI on upstream release)

## [v1.0.0] — 2026-06

Initial release. Single-file HTML app, ~4000 lines, ~150 KB.

### Core
- Single-file HTML with embedded CSS + JS — no build step
- Vanilla JavaScript, CDN-loaded libraries only
- localStorage persistence (key: `cpk-tool-state-v2`)
- Print-friendly A4 layout

### Data
- Support for 2–8 datasets (add/remove on the fly)
- Per-dataset fields: name, M/C, date, checker, tool, condition, USL, LSL
- USL/LSL **shared** across all datasets (sourced from ds[0], inputs on ds[1+] locked)
- 30 measurement rows per dataset
- Paste-from-clipboard support (paste a column of numbers anywhere in the grid)
- Auto-computed statistics: n, X̄, σ (sample), Cp, Cpk (upper), Cpk (lower), Cpk (min), Max, Min

### Charts

#### Capability View
- Vertical or horizontal orientation
- Drag any dataset's Min-Max bar to reposition spacing
- Min / X̄ / Max markers with toggleable visibility
- Custom marker shapes per dataset (circle, square, triangle, diamond, star, cross)
- Per-dataset color picker
- USL/LSL spec lines drawn as single full-width line with draggable 3-slash label
- **Cpk label per dataset** — independently draggable, default position below Min
- Bottom legend (marker + dataset name)
- Spec line stroke width adjustable (1–6 px)
- Slash terminator: length / gap / angle independently adjustable

#### Distribution View (Bell curve)
- Normal PDF curve overlay for each dataset (filled + outline)
- Vertical or horizontal orientation
- Toggleable: Mean line, ±3σ, ±4σ, USL/LSL spec lines
- Inline legend at top-left of plot
- Spec line stroke width adjustable
- Shares slash terminator settings with Capability View

#### Run Chart
- Chart.js line chart, n=1…30 per dataset
- USL/LSL reference lines per dataset
- Custom marker shapes match Capability/Distribution

### Chart controls (per chart)
- Drag-to-resize handle (bottom-right corner)
- Aspect ratio presets: Free / 16:9 / 4:3 / 1:1
- Live size readout overlay
- Font size +/-
- Decimal places +/-

### Export
- **Copy PNG** to clipboard (Capability, Distribution, Run)
- **Copy SVG** to clipboard (Capability, Distribution only — Run is canvas-based)
- **Save PNG** as file
- **Save SVG** as file (Capability, Distribution)
- **Save JSON** — full state dump including stats and UI prefs
- Print A4

### Import
- **Excel** — compatible with FM8.3.2-PE-17 template (reads USL/LSL from row 8/9, data from rows 13–42 in columns C..J)
- **Image (OCR)** — Tesseract.js extracts numeric column; user reviews and applies to chosen dataset

### Internationalization
- Full UI translation: English / Thai / 日本語
- Language switcher in topbar
- Per-language font stack (IBM Plex Sans for EN, Noto Sans Thai for TH, Noto Sans JP for JA)

### UI
- Industrial / blueprint aesthetic
- DENSO red accent (`#D2232A`)
- IBM Plex Sans + Mono typography
- Topbar with brand (title + version) on left, action buttons on right
- 4-section workflow: Common Info → Per-Dataset → Measurement Data → Statistics
- 2 chart rows: [Capability | Distribution] then [Run Chart full-width]
