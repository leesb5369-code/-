# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a single-file static HTML page that displays a ranked table of South Korea's top 300 construction companies by contract limit (도급한도액). It is a self-contained reference/lookup table with client-side search and print support — no backend, no build tooling, no dependencies.

**File:** `index.html` — the entire project (markup, CSS, and JS) lives in this one file. There is no `package.json`, test suite, or linter configured.

## Structure of index.html

The file has three logical sections, in order:

1. **`<style>` block (head)** — all CSS, scoped by element/class selectors (no framework). Defines the color palette, layout (flex toolbar, sticky table header, responsive overflow wrapper), row styling, and a `@media print` block that hides the toolbar and flattens colors for printing.
2. **Body markup**:
   - `<header>` — title (`<h1>`) and a disclaimer `<p class="subtitle">` noting the data is for reference only.
   - `.toolbar` — search `<input id="searchInput">` (wired via `oninput="filterTable()"`), a print `<button onclick="window.print()">`, and `<span id="countLabel">` showing visible/total counts.
   - `.table-wrap > table#rankTable` — `<thead>` with the 8 columns below; `<tbody id="tableBody">` containing **hardcoded `<tr>` rows for ranks 1–30**; `<tfoot>` with a note explaining that ranks 31–300 are generated at runtime.
3. **Inline `<script>`** at the bottom of `<body>`, with two responsibilities:
   - An IIFE that runs on load, appending empty `<tr>` rows for ranks 31–300 to `#tableBody` via `for (let i = 31; i <= 300; i++)`, then calls `updateCount()`.
   - `filterTable()` — reads `#searchInput`, lowercases it, and toggles a `.hidden-row` class (display:none) on any `<tbody>` row whose `textContent` doesn't match; then calls `updateCount()`.
   - `updateCount()` — counts total vs. non-`.hidden-row` rows in `#tableBody` and writes `표시: {visible} / 전체: {total}개사` into `#countLabel`.

## Table Columns

| Column | CSS class | Notes |
|---|---|---|
| 순위 | `.rank` (+ `.top3` / `.top10`) | Rank 1–300. Ranks 1–3 styled red via `.top3`, ranks 4–10 orange via `.top10`, 11+ unstyled (default navy). |
| 건설회사명 | `.company` | Company name, bold |
| 대표자 | `.rep` | Representative / CEO, centered |
| 대표번호 | `.tel` | Phone number, centered, format `xx(x)-xxxx-xxxx` |
| 주소 | *(none)* | Address, left-aligned |
| 도급한도액 | `.limit` | Contract limit amount, right-aligned, format `"123,000억원"` |
| 지급보증율 | `.guarantee` | Payment guarantee rate, centered, e.g. `"100%"` |
| 비고 | `.note` | Remarks, centered, gray text — currently empty for all rows |

## Adding or Editing Data

- **Ranks 1–30** are hardcoded `<tr>` blocks inside `<tbody id="tableBody">` (lines ~163–462). Edit the `<td>` text content directly, keeping the existing class names so styling (alignment, top3/top10 colors, etc.) is preserved.
- **Ranks 31–300** are generated as empty placeholder rows by the IIFE in the inline `<script>`. To populate a given rank with real data, either:
  - Replace/add a hardcoded `<tr>` for that rank above `</tbody>` **and** raise the JS loop's start value past it (e.g. change `for (let i = 31; ...)` to `for (let i = 41; ...)` once ranks 31–40 are hardcoded) — the loop does **not** automatically detect or skip existing rows, so failing to adjust the bounds will produce duplicate rank rows; **or**
  - Replace the entire loop with a JS data array (`[{rank, company, rep, tel, addr, limit, guarantee, note}, …]`) and render `<tr>` markup from it — preferable if populating many rows at once.
- Keep data formatting consistent with existing rows: phone numbers as `지역번호-국번-번호`, contract limits as `"00,000억원"`, guarantee rates as `"000%"`.
- New/edited rows are automatically picked up by `filterTable()` and `updateCount()` since both query `#tableBody tr` live — no script changes needed for search/count to work.

## Styling Conventions

- Primary brand color (`#1a3a5c`, navy) is used for the header underline, table header background, and default rank text. Top-3 ranks use `#c0392b` (red); ranks 4–10 use `#e67e22` (orange).
- Korean sans-serif font stack: `'맑은 고딕', 'Malgun Gothic', '나눔고딕', sans-serif`.
- `.hidden-row { display: none; }` is the sole mechanism for search filtering — rows are never removed from the DOM, only hidden.
- The `@media print` block hides `.toolbar` and forces the dark header background to print via `-webkit-print-color-adjust: exact`.

## Development Workflow

- **No build step, no install, no server.** Open `index.html` directly in any browser to view/test changes.
- **No automated tests or linters** are configured for this repo. Verify changes manually in a browser:
  - Confirm the row count label updates correctly after edits (`표시: N / 전체: 300개사` when unfiltered).
  - Type into the search box and confirm matching rows (by company, rep, phone, or address text) remain visible while others get `.hidden-row`.
  - Use the 인쇄 (print) button / browser print preview to confirm the print stylesheet still hides the toolbar and renders the table cleanly.
- Since everything is inline in one file, prefer small, targeted edits (`Edit`/string replacement) over rewriting the whole file.
