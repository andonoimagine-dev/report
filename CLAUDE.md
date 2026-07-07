# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository overview

This repo contains a single self-contained HTML application, `index.html`, with no build system, package manager, dependencies, or test suite. There is no server component — the file is opened directly in a browser (e.g. via `file://` or any static host).

It is a shift-report generator for a specific production line ("FW-04") in an Indonesian manufacturing context (labels, dates, and generated report text are in Bahasa Indonesia). It lets a line operator/supervisor enter delivery cycles, hourly output, reject counts, and issues, then generates a formatted WhatsApp-style text report plus a "Detail Analisa" tab with computed KPIs and recommendations.

## Development

There are no build, lint, or test commands — this is plain HTML/CSS/JS in one file. To work on it:
- Edit `index.html` directly.
- "Run" it by opening the file in a browser (double-click, or `start index.html` on Windows).
- Verify changes manually in-browser; there is no automated test harness.

## Architecture (all in `index.html`)

**Structure**: one `<style>` block, one static HTML layout (two-column: left form panel / right report-preview panel), and one `<script>` block containing all app logic.

**State**: four in-memory arrays hold the repeating/dynamic sections, re-rendered from scratch on every mutation:
- `deliveries` — delivery cycles (cycle name, time, target, actual)
- `hourlies` — per-hour production actuals + notes
- `rejects` — reject counts by process/defect type
- `issues` — problem/action/status log

Simple scalar fields (date, shift, line name, part no, target/hr, operator, machine, stock/WIP, plans, escalation notes) are read directly from DOM inputs by `id` via `getState()`/`loadState()`.

**Render pattern**: each dynamic section has a `render*()` function (`renderDelivery`, `renderHourly`, `renderReject`, `renderIssues`) that wipes and rebuilds its container's `innerHTML` from the corresponding array, wiring inline `oninput`/`onclick` handlers that mutate the array by index and re-render. `renderAll()` calls all of them plus `updateLiveSummary()`. There is no virtual DOM or diffing — every edit does a full re-render of its section.

**Persistence**: `localStorage` key `fw04_draft` holds the full JSON state (`getState()`/`loadState()`), auto-saved every 60s and on manual "Simpan". `exportJSON()`/`importJSON()` provide file-based backup/restore (filename pattern `FW04_<date>_<shift>.json`). `newShift()` resets all state for a new shift after a `confirm()` prompt.

**Report generation**: `generateReport()` computes derived metrics (total achievement, avg rate/hr, gap vs. target, delivery completion, WIP totals) and builds a formatted plain-text report (emoji-annotated, monospace) into `#reportWA`, copyable via `copyReport()` (Clipboard API). `buildAnalisa()` builds the second tab (`#detailAnalisa`) with attainment %, end-of-shift projection, per-hour trend bars, and rule-based supervisor recommendations (`recs` array, threshold-driven, e.g. rate < 90% of target, reject > 5 pcs, any hour < 70% of target).

**Key business constants**: 1 palet (pallet) = 75 pcs (used in WIP Finishing conversion, `calcWIP()`); shift length assumed as 8 hours for end-of-shift projection in `buildAnalisa()`.

When modifying: keep everything inline in `index.html` (no new files/build step expected), and preserve the Indonesian-language labels/output text used throughout the UI and generated report.
