# Reading AMFI's NAV file

The Association of Mutual Funds in India publishes the latest NAV of every registered scheme each business day as one plain-text file, `https://www.amfiindia.com/spages/NAVAll.txt`. It is a published file, not a site to scrape: read it whole, once per run, and record the date the NAVs carry.

## Layout

The file isn't a plain CSV. It mixes three kinds of line:

1. **The header** — the first non-blank line, `;`-delimited column titles. Take the column order from this line every time rather than assuming it; at the time of writing it reads `Scheme Code;ISIN Div Payout/ ISIN Growth;ISIN Div Reinvestment;Scheme Name;Net Asset Value;Date`.
2. **Blank lines** — ignore them.
3. **Lines with no `;`** — section labels. Two kinds alternate:
   - a **scheme-type line**, such as `Open Ended Schemes(Equity Scheme - Large Cap Fund)`, which sets the classification (`Open Ended Schemes`) and the category (`Equity Scheme - Large Cap Fund`, the text in parentheses) for every row until the next scheme-type line;
   - a **fund-family line**, such as `HDFC Mutual Fund`, which sets the fund family for every row until the next label.

   When two label lines arrive with no data row between them, the first is the scheme type and the second is the fund family. A single label line between data rows is a new fund family under the same scheme type.
4. **Data rows** — `;`-delimited values in header order. Carry the current classification, category and fund family onto each row.

## Cleaning

- Collapse runs of whitespace in scheme names to one space.
- Treat `N.A.`, `NA`, `-`, `--` and empty values as missing — "not available", never zero.
- Parse NAV as a decimal and the date as given (`DD-Mon-YYYY`). Rows in one file can carry different dates — a scheme that didn't price that day keeps its last NAV — so keep each row's own date.
- The scheme name carries the plan and option (`- Direct Plan - Growth`, `- Regular Plan - IDCW`). Keep the full name as the row's identity; a short name derived by cutting at the first ` - ` is for display only and must never be used to match or merge rows, since it collapses distinct plans into one.

## Matching to other sources

Match an AMFI row to another source's fund by ISIN first, then scheme code. Match on name only when neither identifier is available, and then only an exact match after whitespace and case are normalized; anything looser is a candidate to show the advisor, not a match.

## What the file does not hold

NAV, date, identifiers, scheme type, category and fund family — nothing else. Returns, AUM, expense ratios, ratings and risk statistics come from a fund-data platform or the advisor's upload (SKILL.md Step 1). Never compute a return from two NAVs in this file: one file holds one date, and IDCW payouts make NAV-to-NAV change differ from total return.
