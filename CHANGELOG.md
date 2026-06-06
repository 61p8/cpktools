# Changelog

All notable changes to this project. Format loosely based on [Keep a Changelog](https://keepachangelog.com).

## [Unreleased]

### Added — charts
- **Distribution View styling parity:** the per-dataset **color + marker chips** now appear above the Distribution View too (shared with Capability — editing either row updates both), and each bell curve now draws its dataset **marker at the peak**.
- **Capability View auto-compact:** once you drag datasets closer together (custom spacing), the chart automatically crops the empty band along the cross axis on screen and in copies. The value axis keeps its length; the value ruler, USL/LSL lines, and legend ride up next to the clustered datasets. **Reset Spacing** restores the full chart.

### Changed — export buttons
- Copy buttons are relabelled by destination: **Copy Excel** (copies a PNG image) and **Copy Powerpoint** (copies vector SVG for Capability/Distribution; PNG for the canvas-based Run Chart). Behaviour is unchanged — they copy to the clipboard for pasting.
- Removed the separate **Save PNG / Save SVG** buttons from all three charts.

## [v2.0.0] — 2026-06

### Changed — Excel-file backend (replaces browser cache)
- Persistence moved from `localStorage` to a **local Excel file** via the File System Access API (**Chrome / Edge only**, served over http(s)/localhost; opening via `file://` is blocked with a notice).
- On open, the app prompts to **link a `.xlsx`** — open an existing workbook or create a new template.
- **Multi-tab workbook:** each sheet/tab is one Cpk group. Switch via a horizontally-scrollable tab bar (drag/wheel to pan); add / rename / remove groups in-app.
- **Hybrid format:** each group sheet is human-readable (FM8.3.2-PE-17 layout) plus a hidden `__CPK_STATE__` sheet storing full state (colors, markers, chart positions, UI prefs) for exact restore.
- **Auto-save** (debounced) writes back to the linked file, plus an explicit **Save** button and a saved/unsaved status chip.

### Added — charts
- **Capability View ±kσ option:** choose **3σ or 4σ** (Capability View only). Draws a ±kσ process-spread band alongside Min/X̄/Max, and the chart's Cpk label uses the selected multiplier (CPU = (USL−X̄)/kσ, etc.). The Section 04 stats table stays on standard 3σ.
- **Tidier chart toolbars:** primary controls stay inline; secondary controls (font, decimals, line width, slash, aspect, reset) collapse into a per-chart **⚙ settings popover** that does not cover the plot, so adjustments preview live.

### Fixed
- **One-sided spec capability:** with only USL (or only LSL), the tool no longer blanks all indices. `Cp` is shown only when both limits exist; **CPU** and **CPL** are shown independently, and `Cpk` = the available one-sided index.
- **Run chart spec lines:** USL/LSL no longer sit on the chart border. The Y-axis now has headroom, USL/LSL are drawn as single shared lines in distinct colors (USL red, LSL blue) with labels, and the out-of-spec zones are lightly shaded.

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
