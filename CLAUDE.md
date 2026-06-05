# CLAUDE.md

This file is for the Claude Code agent. Read this first before making any changes.

## Project context

**What:** A single-file HTML web app (`cpk_tool.html`) that replaces the DENSO Thailand `FM8.3.2-PE-17 (22.06R)` Excel template for recording Process Capability (Cpk) measurements.

**Who:** Sittisak C., Process Engineering at DENSO (Thailand) Co., Ltd. — the user/owner. Communicate in **Thai** by default unless the user switches languages.

**Why a single file?** The tool is shared internally as a download-and-open `.html` file. No build step, no server, no dependencies to install. CDN-loaded libraries only.

**Source template reference:** `FM8.3.2-PE-17 (22.06R)` — DENSO internal Excel form with 2 datasets, 30 measurements each, calculating Cp/Cpk per ISO 22514-style statistics. Columns C and D hold the two datasets.

## Tech stack (do not change unless explicitly asked)

- **Single HTML file** with embedded `<style>` and `<script>` tags
- **Vanilla JavaScript** — no frameworks, no JSX, no TypeScript
- **CDN libraries** (already loaded via `<link>` / `<script>`):
  - Chart.js 4.4.0 — Run Chart only
  - SheetJS (xlsx) 0.18.5 — Excel import
  - Tesseract.js 5 — image OCR import
- **Fonts** from Google Fonts: IBM Plex Sans, IBM Plex Mono, Noto Sans Thai, Noto Sans JP
- **Custom SVG** for Capability View and Distribution View (no chart library for these)

## File layout

The entire app is in `cpk_tool.html`. Roughly:

| Region | Approx. lines | Contents |
|---|---|---|
| `<head>` | 1–940 | meta, font links, CDN scripts, **all CSS** |
| `<body>` | 940–1350 | topbar, container with 4 sections + 2 chart rows, modals |
| `<script id="appScript">` | 1350–end | **all JavaScript** — i18n, state, renders, charts, import, export |

There is no module system. Everything is global within the script tag.

## State management

A single global `state` object. The workbook is **multi-tab**: `state.groups[]` holds one record per tab, and `state.common` / `state.datasets` are **live aliases** into the active group so render functions never had to change:

```js
const state = {
  groups:      [ { id, name, common:{…}, datasets:[…], spacing } ],
  activeGroup: 0,
  common:      // ALIAS → groups[activeGroup].common
  datasets:    // ALIAS → groups[activeGroup].datasets  (2–8 dataset objects)
  ui:          { /* global UI prefs: language, chart sizes, toggles, drag positions, slash params */ }
};
```

`setActiveGroup(i)` re-points the aliases, refreshes the common inputs, and re-renders. Mutate `state.datasets` **in place** (`.push`/`.splice`) or it desyncs from the group; if you must replace the array, assign `activeGroup().datasets` then `setActiveGroup`. `spacing` is **per-group** (moved out of `ui`), read via `activeGroup().spacing`.

Each `dataset` has: `id, name, mc, date, checker, tool, condition, upper, lower, data[30], color, marker, cpkOffset`. `upper`/`lower` may be `NaN` (one-sided spec).

**Persistence — local Excel file (File System Access API, Chrome/Edge only):** there is no more `localStorage`. On boot the user opens an existing `.xlsx` or creates a template (`pickOpenFile` / `createTemplateFile`). The linked `FileSystemFileHandle` is held in `fileHandle`. `saveState()` now just calls `markDirty()` → debounced (1.5s) `writeWorkbook()`; there is also an explicit Save button. A saved/unsaved chip (`#saveStatus`) reflects state.
**Workbook format (hybrid):** one human-readable sheet per group in FM8.3.2-PE-17 layout (`buildWorkbook` ↔ `parseSheetToGroup`, kept in sync via `ROW_*` consts), plus a hidden `__CPK_STATE__` sheet with the full JSON state (colors, markers, drag positions, UI prefs). On read, `__CPK_STATE__` is preferred; otherwise the readable sheets are parsed (data-only fallback).
**Merging:** restoring `ui` from `__CPK_STATE__` uses key-by-key merge (saved fills only existing keys) — do not switch to `Object.assign(state.ui, savedUi)` because that lets stale `undefined` override new defaults.

## Architecture patterns

### Render functions
Each section has its own render function. Re-rendering is cheap and idempotent:

- `renderDsTable()` — per-dataset info table (Section 02)
- `renderDataGrid()` — 30-row measurement input grid (Section 03)
- `renderStats()` — statistics table (Section 04)
- `renderDsStyleRow()` — color/marker chips above Capability chart
- `drawCapChart()` — Capability View SVG (rebuilds entire SVG)
- `drawBellChart()` — Distribution View SVG (rebuilds entire SVG)
- `drawRunChart()` — Run Chart canvas via Chart.js

### Master update
`onStateChange(opts)` re-renders everything (with skip flags to avoid focus loss). Called by most user actions.

### Drag handling
Drag handles use SVG element delegation (one listener per chart SVG). Drag targets are identified by `data-*` attributes:

- `[data-ds-handle="i"]` — drag a dataset's Min-Max bar to change horizontal position (Capability)
- `[data-label-handle="usl|lsl"]` — drag USL/LSL label along its spec line (Capability)
- `[data-bell-label-handle="usl|lsl"]` — same idea for Distribution View
- `[data-cpk-handle="i"]` — drag a per-dataset Cpk label anywhere (independent per dataset)
- `[data-resize-handle="cap|bell|run"]` — corner resize handle for each chart wrapper

### USL/LSL spec lines
The current behavior: **USL and LSL are shared across all datasets**, sourced from `state.datasets[0].upper` / `.lower`. Inputs for `upper`/`lower` on datasets `1..N-1` are disabled and visually locked. `renderDsTable()` enforces this sync at the start of each render.

There is one global spec line per limit (USL, LSL), drawn full-width across the plot. The label group (3 slashes + label + value) is independently draggable along the line.

### Slash terminator
The "/// label" at the end of each USL/LSL line is parameterized by:
- `state.ui.specSlashLen` (default 10)
- `state.ui.specSlashOffset` (default 4)
- `state.ui.specSlashAngle` (default 30°)
Shared between Capability and Distribution.

### i18n
Three language tables (`en`, `th`, `ja`) in `const I18N = {...}`. Helper `t(key)` returns the current-language string. `applyLang()` walks DOM nodes with `data-i18n="key"` and sets their `innerHTML`.

When adding text-bearing UI:
1. Add the string to **all three** language tables in `I18N`
2. Use `data-i18n="yourKey"` on the element
3. The default content of the tag can be the EN string (fallback if i18n init fails)

### Export pipeline
- `svgToPngBlob(svgEl, scale=2)` — serialize SVG → image → canvas → PNG blob
- `svgToSvgBlob(svgEl)` — serialize SVG with embedded font styles → SVG blob (for PowerPoint 365 import)
- Clipboard: tries `ClipboardItem` for image/png and image/svg+xml, falls back to `writeText` for SVG markup, then to file download

### Locked default-spec rule
In `renderDsTable`, before rendering rows, USL/LSL of `ds[0]` is force-synced into `ds[1..N-1]`. The `upper`/`lower` inputs for those datasets are rendered with `disabled` and `class="synced-locked"`. Do not break this — the user explicitly chose option (ข): one input, all locked.

## Conventions

### Communication style
- **Language:** Thai by default. Mirror the user's casual register.
- **Avoid heavy formatting** in chat unless the user is comparing options or you have a multi-item list. Prose is preferred.
- **Tone:** direct, no fluff, no overuse of emojis. One emoji per message is fine if it serves clarity (e.g. 🚀 for shipping, ✓ for confirmation).

### Workflow (very important)
The user works in **comment-then-batch** mode:

1. They send comments **one at a time** — usually short ("1 USL/LSL ปรับขนาดไม่ได้", "2 ย้าย Cpk ไปด้านล่าง")
2. Your job: **acknowledge** each comment, **accumulate** the list, **do not implement yet**
3. When they say **"ลุยเลย"** (or "ลุย" or similar Thai go-ahead), implement everything in one batch
4. The user has corrected past versions of you to slow down and wait — respect this. Don't implement after each comment.

If a comment is ambiguous, ask **one short clarifying question** before adding to the list.

### Decision defaults
When the user says "ทำเหมือนเดิม" or doesn't specify minor details, **make a sensible default decision and state it clearly**. Don't ask for clarification on minor things. They will redirect if needed.

### Code style
- 2-space indentation
- Single quotes for strings in JS
- Template literals for interpolation
- No semicolons inside HTML attributes
- SVG attributes via `svgEl(tag, {attrs}, text?)` helper — always use this for SVG, not `document.createElementNS` directly
- Color values: use the CSS variables defined in `:root` (e.g. `var(--accent)`) for theme colors. Direct hex (`#D2232A`) only for hard-coded chart colors that must match the design system

### Testing
There is no automated test framework. To verify changes don't break things:

1. **JS syntax check:**
   ```bash
   python3 -c "import re; html=open('cpk_tool.html').read(); m=re.search(r'<script id=\"appScript\">(.*?)</script>', html, re.DOTALL); open('/tmp/t.js','w').write(m.group(1))"
   node --check /tmp/t.js
   ```

2. **HTML tag balance:**
   ```python
   import re
   html = open('cpk_tool.html').read()
   for tag in ['html','head','body','div','section','script','style','svg']:
       o = len(re.findall(r'<' + tag + r'[\s>]', html))
       c = len(re.findall(r'</' + tag + r'>', html))
       print(f'{"✓" if o==c else "✗"} {tag}: {o}/{c}')
   ```

3. **jsdom smoke test:**
   ```js
   // Verify boot doesn't error + key elements render
   const { JSDOM } = require('jsdom');
   const html = require('fs').readFileSync('cpk_tool.html', 'utf8');
   const stub = `<script>
     window.XLSX = { read: () => ({SheetNames:[],Sheets:{}}) };
     window.Chart = class { constructor(){ this.destroy=()=>{}; this.resize=()=>{}; }};
     window.Tesseract = { recognize: () => Promise.resolve({data:{text:''}}) };
   </script>`;
   const dom = new JSDOM(html.replace('</head>', stub+'</head>'), {
     runScripts: 'dangerously', resources: 'usable', pretendToBeVisual: true,
   });
   setTimeout(() => {
     // assert key selectors exist
     console.log(dom.window.document.querySelectorAll('#capChart line').length, 'lines');
   }, 800);
   ```

4. **Open in browser** — always do this final check yourself if possible. The JSDOM test catches structural issues but not visual ones.

### Backup before risky edits
If making structural changes, copy the file first:
```bash
cp cpk_tool.html cpk_tool.backup.html
```
Delete the backup after verification.

## Implementation gotchas (real bugs that already happened)

1. **`Object.assign(state.ui, obj.ui)`** — DON'T do this. If the saved state in localStorage has a field set to `undefined` (e.g., from an older schema), it will overwrite the default. Use the key-by-key merge already in `loadState()`.

2. **SVG drag listeners** — the SVG's children are recreated on every `drawCapChart()`. Do not attach pointer listeners to individual child elements; they will be wiped. Use **event delegation** on the SVG itself (already done in `setupCapChartDrag()`).

3. **`fill: 'none'` on hit rects** — makes the rect non-clickable. Use `fill: 'transparent'` or `fill: 'rgba(0,0,0,0)'` for invisible-but-clickable hit areas.

4. **`pointerLeave` on individual handles** breaks drag if pointer briefly leaves the element. Use `setPointerCapture` on the SVG and don't bind `pointerleave`.

5. **Workbook state schema** — state now persists inside the linked `.xlsx` (hidden `__CPK_STATE__` sheet), not `localStorage`. When changing the saved schema, keep `readWorkbookIntoState` tolerant (normalize via `normalizeGroup` / `Object.assign(newDataset(i), d)`) so older saved files still open. Keep `buildWorkbook` and `parseSheetToGroup` row layout in sync via the `ROW_*` constants.

6. **Excel column inclusion** — `handleExcel()` only includes a column if it has at least one actual data value. Don't fall back to "include if upper/lower exist" because old templates have cached `0` values from formulas.

## Things that should NOT change (without explicit user approval)

- The single-file architecture (no build, no bundler)
- The Thai-first communication
- The "comment-then-batch with ลุยเลย" workflow
- Shared USL/LSL across datasets (locked at ds[0])
- IBM Plex font family (the user likes this aesthetic)
- DENSO red accent color `#D2232A`
- The topbar layout: brand (title + version stacked) on left, actions on right

## Open ideas (not promised)

The user has mentioned but not committed to:

- Code obfuscation / wrapping as Electron `.exe` for distribution
- Hosting on internal DENSO intranet
- A "PowerPoint export" button that generates a `.pptx` directly (currently they copy PNG/SVG and paste)

If the user picks any of these up, ask 1-2 clarifying questions about scope before starting.

## When in doubt

- Read this file again
- Look at how similar features are implemented elsewhere in `cpk_tool.html` and follow the pattern
- Test before shipping
- Ship one file: `cpk_tool.html`. Don't introduce new files unless the user asks
