---
name: mutual-fund-analysis
description: Analyze and compare one or more mutual fund schemes for an advisor across short, medium and long horizons (1m, 3m, 6m, 1y, 2y, 3y, 5y): pulls each fund's NAV, AUM, fund-family AUM, expense ratio, minimum investment, risk rating, credit/agency rating and trailing returns beside its category average, plus the risk statistics the source computes (standard deviation, Sharpe, Sortino, alpha, beta, R-squared, Treynor, upside/downside capture), from connected fund-data platforms (Morningstar, FactSet, S&P Global, MSCI, Franklin Templeton, Vanguard, BlackRock, Invesco), the public AMFI NAV file for Indian schemes, or an upload. Builds a per-horizon factor scorecard showing where each fund sits against its category and how volatile that category is, and cites the source on every figure. Does not recommend buying, selling or switching a fund, does not label funds "good" or "bad", and does not forecast returns. Triggers on "mutual fund analysis", "/mutual-fund-analysis", "analyze [fund]", "compare these funds", "how has [fund] done against its category", "fund scorecard", "screen [fund family]'s schemes", or any request to study mutual fund schemes' performance and the factors behind it.
---

# Mutual Fund Analysis

Give an advisor a clear, sourced read on one or more mutual fund schemes: how each has performed against its own category over every horizon from one month to five years, how much risk it took to get there, how large and how backed by its fund family it is, and which of those factors actually separates the funds in front of them. The advisor weighs it and decides; this skill never ranks a fund as a buy.

The method follows the open study *Analysis of Mutual Funds* (A. Sood, A. Arora, D. Nellurvalappil, SFU CMPT 733, https://github.com/abhinavsood/mutual-fund-analysis), which built per-horizon factor metrics for Indian schemes from AMFI and MoneyControl data. Its factor definitions are in @${CLAUDE_PLUGIN_ROOT}/skills/mutual-fund-analysis/references/factor-scorecard.md, and its lessons — chiefly that a "good fund" classifier trained on labels derived from its own features looks accurate only because of class imbalance — are why this skill stops at a descriptive scorecard.

## Inputs

Required: **one or more funds**, by name, ticker, ISIN, AMFI scheme code, or a fund family plus category ("HDFC's large-cap schemes"). Optional: the horizons to cover (default all seven), a benchmark the advisor wants used in place of the category average, a market (default: inferred from the identifiers — an AMFI code or an Indian AMC name means India, a ticker or CUSIP means US), and a share class or plan (Direct vs. Regular, Growth vs. IDCW, Class A vs. I).

**Fund identity is settled before any figure is pulled.** A name often matches several schemes — share classes, Direct and Regular plans, Growth and dividend options, a fund and its feeder. Never proceed on a name match alone: if a lookup returns more than one candidate, show them (name, identifier, plan/class, fund family) and ask which. Two plans of one scheme are two funds here — their NAVs, expense ratios and returns differ — so never merge them into one row.

**This is fund analysis, not a client review.** It reads no CRM and no household. If the advisor asks how a fund fits a particular client, analyze the fund here and point at **/portfolio-rebalance-review** for the household's drift and model fit.

Once identities are settled, don't ask "should I start?" — say which sources you're pulling from and go.

## Step 1: Find the sources

Find each fund-data platform's tools in your own tool list, or with a `ToolSearch` by the platform's own name. Real connector tool names use opaque prefixes, so never guess one. A search that finds nothing for a platform means you don't have its tools in this session, not that the fund doesn't exist; `ListConnectors` coming back empty means **unknown**, never "nothing is connected".

Use, in this order of preference, for each field:

1. **A connected fund-data platform** — Morningstar, FactSet, S&P Global, MSCI for any fund; Franklin Templeton, Vanguard, BlackRock, Invesco for their own funds. Where a platform's provider folder exists under `${CLAUDE_PLUGIN_ROOT}/providers/`, read every file in it before your first call and follow it.
2. **For Indian schemes, AMFI's published NAV file** — the latest NAV for every registered scheme, updated each business day. Read it with web fetch where that is available in this session; @${CLAUDE_PLUGIN_ROOT}/skills/mutual-fund-analysis/references/amfi-nav-format.md says how to parse it. It carries NAV, date, scheme code, ISIN, scheme type/category and fund family only — no returns, AUM, ratings or risk statistics.
3. **An upload or paste from the advisor** — a factsheet, a platform export, a CSV. Label each figure "per the [source] factsheet/export, [date]".

Every figure in the analysis comes from one of these three. Never fill a field from memory of a fund, a prior run, or a news item.

> **Connector Placeholder Convention:** if no fund-data platform is reachable and the funds aren't Indian schemes AMFI covers, say: *"This is where I'd pull [fund]'s performance and risk data from a fund-data platform such as Morningstar or FactSet once that connector is available."* Then offer the fallback — upload a factsheet or export, or paste the figures — and write the analysis now with "— pending [source]" cells. A file of pending cells is still the deliverable; a question with no file behind it is a stall.

## Step 2: Pull the fields

For each fund, pull what the source holds, with its as-of date:

- **Identity:** full scheme name, identifier(s), plan/share class, fund family, category as the source classifies it, benchmark the fund states.
- **Size and cost:** latest NAV, scheme AUM, fund-family AUM, expense ratio, minimum initial investment, exit/redemption load.
- **Ratings:** the source's own risk rating (e.g. SEBI riskometer level, Morningstar risk), and any credit or agency rating it reports (e.g. CRISIL rank, Morningstar star rating) — each under its own name, never one substituted for another.
- **Returns, per horizon (1m, 3m, 6m, 1y, 2y, 3y, 5y):** the fund's trailing return and its **category average** for the same horizon and as-of date, and the category's best and worst if the source reports them. Note whether returns over a year are annualized or cumulative — the source decides, and you say which.
- **Risk statistics the source computes:** standard deviation, Sharpe, Sortino, alpha, beta, R-squared, Treynor, upside/downside capture — with the period and benchmark each was computed over.
- **Distributions:** latest dividend/IDCW and any bonus, only if the source reports them.

**Pass through, never recalculate.** A return, ratio or risk statistic is the one the source computed. Never derive Sharpe from a return and a standard deviation, annualize a cumulative return, or fill a missing horizon by compounding shorter ones. A statistic the source doesn't return is "not available", never a different one in its place.

**Compare like with like.** A fund's return and its category average must share horizon, as-of date, and basis (annualized or cumulative; net of fees or gross). Where they don't, show both with their dates and don't compute anything from the pair. Two sources reporting the same fund at the same date and disagreeing beyond rounding is a conflict: show both, pick neither, and ask the advisor which is right before the analysis is final.

## Step 3: Build the factor scorecard

Compute the five factor metrics in @${CLAUDE_PLUGIN_ROOT}/skills/mutual-fund-analysis/references/factor-scorecard.md for each fund and horizon the data allows — risk score, credit-rating score, family-confidence ratio, beat-category flag and category-volatility ratio. These are the only figures this skill computes, each is simple arithmetic on figures the source gave, and the file shows the inputs beside every one so the advisor can check it.

Rules for the scorecard:

- **A missing input makes the metric "not available", never zero.** The original study imputed missing values as 0, which made an unrated fund score as the riskiest and a fund with no return look like a laggard. Don't.
- **No composite score and no label.** Don't average the five into one number, round it to good/bad, or rank funds by it. The study did exactly that and found the labels too rare (54–139 "good" in 1,296 schemes) to train anything meaningful; a composite also hides which factor moved. Show the five side by side.
- **Horizons stay separate.** A fund that beats its category over five years and trails it over one is reported as both; never net them.
- **Category volatility describes the category, not the fund.** Say so wherever it appears.

## Step 4: What separates these funds

With two or more funds, write a short **Observations** section: for each horizon, which factors differ most across the funds analyzed, and where a fund's position against its category changes across horizons. Describe the data in front of you — "Fund B beat its category over 3y and 5y but trailed it over 6m and 1y" — never a cause ("because its manager…") the sources don't state, and never a generalization beyond these funds.

With ten or more funds of one category the advisor may ask which factors matter most. Answer descriptively: how each factor varies with the fund's excess return over its category in this set, with the count of funds behind it, and say plainly that a sample this size shows association, not cause, and that past returns don't predict future ones. Don't train a model on the scorecard to produce "importances".

Flags for the advisor's attention, each citing the row it comes from:

- **Stale data:** any figure whose as-of date is more than ~10 business days older than the newest in the analysis.
- **Thin history:** a fund too young to have a 3y or 5y return — say "fund launched [date]; no 5y record", not "not available".
- **Small or concentrated:** scheme AUM that is a very small share of its family's AUM, or very small in absolute terms — liquidity and closure risk worth discussing, stated as a figure, not a verdict.
- **Rating gaps:** a fund with no risk or credit rating from any source.

If a flag can't be checked because an input is missing, name the flag and say what's missing.

## Step 5: Output

Write the analysis to a markdown file following @${CLAUDE_PLUGIN_ROOT}/skills/mutual-fund-analysis/templates/fund-analysis-template.md. If the file already exists in the working folder, ask before replacing it. In order:

- **Executive Summary** — two or three plain-English sentences on what the data shows across the funds and horizons. The chat reply opens with these same sentences.
- **Sources** — each source read, with its status (`OK`, `NOT CONNECTED`, `NOTHING FOUND`, `PARTIAL`) and as-of date; for AMFI, the file's date. A platform's disclaimer or terms appear only as a link it returned, never as text you write or paraphrase.
- **Fund profiles** table, then **Returns vs. category** (one table, a row per fund and a column pair per horizon), then **Risk statistics**, then the **Factor scorecard**, then **Observations and flags**.
- **For the advisor** — every question the analysis was written around: which plan/class to use if one was assumed, whether to use the stated benchmark instead of the category average, and anything a "— pending [source]" cell is waiting for.

Cell markers: **"not available"** where a source answered and doesn't hold the field (a line under the table gives its reason); **"— pending [source]"** where nothing has answered yet. A zero only where the source reports zero.

If any part will go to a client, recommend running it through **/compliance** first: performance figures in client material carry SEC Marketing Rule (or, for Indian schemes, SEBI/AMFI) presentation requirements this advisor-facing file doesn't meet.

**Summary-first close:** the chat opens with the Executive Summary, then offers two to four directions this run can deliver (a deeper look at one fund, another horizon, adding a fund to the comparison), the format question (.docx, .pdf or .xlsx, only if they ask), and the For the advisor questions.

## Out of Scope

- **No recommendations.** No buy, sell, hold, switch or "top pick", no good/bad label, no ranking by a composite score. A fund idea a platform surfaces is the platform's, attributed to it (@${CLAUDE_PLUGIN_ROOT}/partials/rules/platform-analytics.md).
- **No forecasts.** No projected NAV or return, and no "expected" return from past ones.
- **No recalculated statistics.** Risk and return figures are the source's; Step 3's five metrics are the only arithmetic.
- **No transactions.** Nothing is bought, sold or switched from here.
- **No scraping.** Fund-data sites without a connector or a published file (as AMFI publishes) are not read; the advisor can upload an export instead.

## Important Notes

- Never fabricate a NAV, return, rating, AUM or risk statistic. Missing is "not available" or "— pending [source]" (Steps 2 and 5).
- Settle each fund's identity, plan and share class before pulling anything (Inputs).
- Compare a fund with its category only at the same horizon, date and basis (Step 2).
- Five factor metrics, side by side, per horizon — never a composite or a label (Step 3).
- Past performance does not guarantee future results; the analysis says so once, in the file.
