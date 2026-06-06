# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A single-file static HTML project displaying a ranked table of South Korea's top 300 construction companies by contract limit (도급한도액). No build step, no dependencies, no server required — open `index.html` directly in any browser.

**Sole file:** `index.html`

---

## File Structure

`index.html` has three logical sections:

1. **`<style>` block** — all CSS lives here; no external stylesheets.
2. **`<body>` markup** — header, toolbar, scrollable table wrapper, `<tfoot>` disclaimer.
3. **`<script>` block** — IIFE for row generation + `filterTable()` / `updateCount()` helpers.

---

## CSS Design System

| Token | Value | Usage |
|---|---|---|
| Primary color | `#1a3a5c` | Header bg, rank text, button bg, `<h1>` color |
| Page background | `#f4f6f9` | `<body>` bg |
| Even row bg | `#f7f9fc` | `tbody tr:nth-child(even)` |
| Hover row bg | `#eaf2fb` | `tbody tr:hover` |
| Row divider | `#e8ecf1` | `border-bottom` / `border-right` on `<td>` |
| Top-3 rank color | `#c0392b` | `td.rank.top3` |
| Top-10 rank color | `#e67e22` | `td.rank.top10` |
| Note text color | `#888` | `td.note` |
| Font stack | `'맑은 고딕', 'Malgun Gothic', '나눔고딕', sans-serif` | `body` |

The table header (`<thead>`) is **sticky** (`position: sticky; top: 0; z-index: 2`) so it stays visible while scrolling.

The `.table-wrap` div uses `overflow-x: auto` and the table has `min-width: 900px` for horizontal scrolling on small screens.

---

## Table Columns & CSS Classes

| # | Header (Korean) | `<td>` class | Alignment | Width | Notes |
|---|---|---|---|---|---|
| 1 | 순위 | `rank` | center | 54 px | Add `top3` for ranks 1–3 (red), `top10` for 4–10 (orange) |
| 2 | 건설회사명 | `company` | left (default) | — | `font-weight: 600` |
| 3 | 대표자 | `rep` | center | 90 px | CEO / representative name |
| 4 | 대표번호 | `tel` | center | 130 px | Phone; `white-space: nowrap` |
| 5 | 주소 | *(none)* | left | — | Full address |
| 6 | 도급한도액 | `limit` | right | 140 px | Contract limit; `white-space: nowrap` |
| 7 | 지급보증율 | `guarantee` | center | 90 px | Payment guarantee rate |
| 8 | 비고 | `note` | center | 80 px | Remarks; gray text |

Example row markup:
```html
<tr>
  <td class="rank top3">1</td>
  <td class="company">삼성물산(주) 건설부문</td>
  <td class="rep">오세철</td>
  <td class="tel">02-2145-2114</td>
  <td>서울특별시 강남구 테헤란로 521</td>
  <td class="limit">324,000억원</td>
  <td class="guarantee">100%</td>
  <td class="note"></td>
</tr>
```

---

## Hardcoded Data (Ranks 1–30)

Ranks 1–30 are hardcoded `<tr>` blocks inside `<tbody id="tableBody">`. The top 10 as of the file:

| Rank | Company | CEO |
|---|---|---|
| 1 | 삼성물산(주) 건설부문 | 오세철 |
| 2 | 현대건설(주) | 윤영준 |
| 3 | 대우건설(주) | 정원주 |
| 4 | GS건설(주) | 허윤홍 |
| 5 | 포스코이앤씨(주) | 한성희 |
| 6 | DL이앤씨(주) | 마창민 |
| 7 | 롯데건설(주) | 박현철 |
| 8 | 현대엔지니어링(주) | 홍현성 |
| 9 | SK에코플랜트(주) | 박경일 |
| 10 | HDC현대산업개발(주) | 최익훈 |

Ranks 11–30 continue with 한화건설, 호반건설, 태영건설, 계룡건설산업, 금호건설, 코오롱글로벌, 중흥건설, 두산건설, 신세계건설, KCC건설, 한신공영, 쌍용건설, 반도건설, 동부건설, 한라, 우미건설, 남광토건, 삼부토건, 효성중공업, 경남기업.

---

## JavaScript Logic

### Row generation IIFE (`index.html` ~line 474)

Runs immediately on page load. Appends empty `<tr>` rows for ranks 31–300 to `#tableBody`, then calls `updateCount()`.

```js
for (let i = 31; i <= 300; i++) { … tbody.appendChild(tr); }
updateCount();
```

**To change the generated range:** adjust the `for` loop bounds `(31; i <= 300; i++)`.

### `filterTable()`

Triggered by `oninput` on `#searchInput`. Searches the entire `textContent` of each row (all 8 columns) case-insensitively. Toggles the `.hidden-row` class (`display: none`) on non-matching rows, then calls `updateCount()`.

### `updateCount()`

Updates `#countLabel` with the pattern `표시: N / 전체: M개사` (visible / total companies).

---

## UI Controls

| Element | ID / selector | Behavior |
|---|---|---|
| Search input | `#searchInput` | `oninput="filterTable()"` — live filter |
| Print button | `.toolbar button` | `onclick="window.print()"` |
| Row count label | `#countLabel` | Updated by `updateCount()` |

---

## Print Support

A `@media print` block in `<style>`:
- Hides `.toolbar` (search + print button).
- Removes `box-shadow` from `.table-wrap`.
- Forces dark `<thead>` background via `-webkit-print-color-adjust: exact`.

---

## Adding or Editing Data

### Editing ranks 1–30
Edit the `<tr>` blocks directly in `<tbody id="tableBody">`. Apply `top3` to ranks 1–3 and `top10` to ranks 4–10 on the `<td class="rank …">` cell.

### Populating ranks 31–300

**Option A — Hardcode more rows:** Add `<tr>` blocks before `</tbody>`. Adjust the JS loop start (`let i = 31`) upward so it doesn't double-generate already-existing ranks.

**Option B — JS data array (recommended for bulk data):** Replace the loop with a data array and render rows from it, e.g.:
```js
const data = [
  { rank: 31, company: '…', rep: '…', tel: '…', addr: '…', limit: '…', guarantee: '…', note: '' },
  // …
];
data.forEach(({ rank, company, rep, tel, addr, limit, guarantee, note }) => {
  const tr = document.createElement('tr');
  tr.innerHTML = `<td class="rank">${rank}</td><td class="company">${company}</td>…`;
  tbody.appendChild(tr);
});
```

### Changing the total count (e.g., top 200 instead of 300)
1. Update the JS loop: `for (let i = 31; i <= 200; i++)`.
2. Update the `<tfoot>` disclaimer text.
3. Update the page `<title>` and `<h1>`.

---

## No Build Step

```
open index.html   # macOS
start index.html  # Windows
xdg-open index.html  # Linux
```

No npm, no bundler, no server, no external CDN dependencies.
