# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a single-file static HTML project that displays a ranked table of South Korea's top 300 construction companies by contract limit (도급한도액).

**File:** `index.html` — the entire project lives in this one file.

## Structure of index.html

The file has three logical sections:

1. **Styled header + toolbar** — title, disclaimer notice, search input, print button, and a live row-count label.
2. **Table (`#rankTable`)** — `<thead>` with 8 columns; `<tbody id="tableBody">` containing hardcoded rows for ranks 1–30, with ranks 31–300 generated as empty rows by inline JavaScript.
3. **Inline `<script>`** — two responsibilities:
   - An IIFE that appends empty `<tr>` rows for ranks 31–300 on page load.
   - `filterTable()` / `updateCount()` helpers wired to the search input (`oninput`).

## Table Columns

| Column | Notes |
|---|---|
| 순위 | Rank (1–300). Top-3 styled red, 4–10 orange via `.top3` / `.top10` CSS classes. |
| 건설회사명 | Company name (`.company`) |
| 대표자 | Representative / CEO |
| 대표번호 | Phone number |
| 주소 | Address |
| 도급한도액 | Contract limit amount (right-aligned, `.limit`) |
| 지급보증율 | Payment guarantee rate (`.guarantee`) |
| 비고 | Remarks (`.note`) |

## Adding or Editing Data

- **Ranks 1–30** are hardcoded `<tr>` blocks inside `<tbody id="tableBody">`. Edit them directly in the HTML.
- **Ranks 31–300** are generated as empty rows. To populate them, either:
  - Add hardcoded `<tr>` rows above the `</tbody>` tag (before the script runs, the JS will skip already-existing rank numbers if you adjust the loop start), **or**
  - Replace the JS loop with a JavaScript data array and render from that.
- The JS loop range is controlled by `for (let i = 31; i <= 300; i++)` — adjust the bounds if the total count changes.

## No Build Step

Open `index.html` directly in any browser — no server, bundler, or dependencies required.
