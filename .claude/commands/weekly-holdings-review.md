---
name: weekly-holdings-review
description: Use when the user asks to run a weekly portfolio review, holdings check, or market check. Triggers include "run weekly review", "weekly check", "review my portfolio", "go through my holdings". Outputs a two-part HTML report: Part 1 reviews current holdings from a Fidelity CSV export (reads from input/ folder); Part 2 is a forward-looking market pulse using web search — weekly recap, notable earnings, hot stocks, and undervalued opportunities in US large-cap tech/consumer.
---

# Weekly Holdings Review

Two-part report, written to a single HTML file:

- **Part 1 — Holdings Review:** Current positions from Fidelity export
- **Part 2 — Market Pulse:** Forward-looking intelligence — what moved, what's hot, what's worth a look

---

## PART 1: HOLDINGS REVIEW

### Setup — Read Fidelity Export

Use the Glob tool to find the CSV file in the `input/` folder.
Use the Read tool to read it.

**Fidelity export format:**
- The file begins with account header rows — skip until you find the row where the first column is `Symbol`
- Relevant columns: `Symbol`, `Description`, `Quantity`, `Last Price`, `Current Value`, `Today's Gain/Loss Percent`, `Total Gain/Loss Dollar`, `Total Gain/Loss Percent`, `Percent Of Account`, `Cost Basis Per Share`, `Cost Basis Total`
- Strip `$` and `,` from price/value fields before parsing
- Strip `%` from percentage fields before parsing
- Skip rows with no ticker (cash rows, "Pending Activity", totals, blank rows)
- If multiple accounts are present, combine positions with the same ticker

After parsing, use the yfinance MCP to fetch the **weekly price change** for each ticker — Fidelity's export only shows today's change.

If no file is found in `input/`, ask the user to export from Fidelity:
> Accounts & Trade → Portfolio → select account → Download (CSV) → place in the `input/` folder

---

### Per-Holding Format

For each equity holding produce this exact structure. Keep it tight — no walls of text.

```
TICKER — Company Name
─────────────────────
Position: $X  (Y% of portfolio)
Weekly: ±X%   |   Total return: ±X%
```

**VALUATION SNAPSHOT** *(Tier 1 from valuation-framework)*
| Metric | Current | 5yr Avg | Note |
|--------|---------|---------|------|
| P/E (TTM) | — | — | — |
| P/E (Fwd) | — | — | — |
| FCF Yield | — | — | vs 10Y Treasury: X% |

Verdict: **Cheap / Fair / Rich** vs history — one line.

**ACTION SIGNAL**
Add zone / Trim zone / **Hold** (default)

**RED FLAGS THIS WEEK**
One bullet, or "none."

---

### Portfolio Summary

After all holdings:
- Total value and today's change (from export)
- Concentration: top 3 positions as % of portfolio
- Sector allocation (broad)
- Cash %
- Attention list — max 3 items, most weeks 0–1

**Rules for Part 1:**
- Default answer is "hold, nothing changed." Resist drama.
- No news + move <5% → "no signal."
- Cite source and date for every number.
- Do not invent an original thesis — it is not tracked here.

---

## PART 2: MARKET PULSE & OPPORTUNITIES

Use WebSearch extensively. This section is forward-looking intelligence for a value investor in US large-cap tech and consumer. Goal: surface what actually matters, not noise.

### 2a. Market Recap (This Week)
Search: major index moves this week, macro catalysts (Fed, inflation data, earnings season).
Output: 2–3 tight bullets. Factual only. No predictions.

### 2b. Notable Earnings (Last 2 Weeks)
Search: significant earnings beats and misses in tech and consumer sectors.
For each: `TICKER — beat/miss — one-line takeaway — any implication for holdings or watch list`
Focus on companies in or adjacent to the user's current holdings.
Skip if nothing material.

### 2c. Stocks Worth Watching
Search: US large-cap tech and consumer stocks getting attention for **fundamental** reasons.
Filter ruthlessly:
- Must have a real valuation anchor — not just price action or momentum
- Exclude meme stocks, speculative names, anything outside the circle of competence
- Distinguish clearly: momentum-driven vs. fundamentals-driven

For each (max 4):
**TICKER — Why it's worth noting**
- Fundamental reason (earnings quality, competitive development, valuation reset)
- Valuation context — pull Tier 1 metrics from yfinance if useful
- Circle of competence flag if needed

### 2d. Potential Undervalued Opportunities
Search: large-cap tech or consumer stocks that have dropped significantly (>10%) recently 
despite fundamentally intact businesses.
For each candidate: pull Tier 1 metrics from yfinance, note the drop, assess whether 
fundamentals justify the decline.
Max 3 items. **Skip entirely if nothing genuine** — do not manufacture opportunities.

**Rules for Part 2:**
- Circle of competence first — filter to US large-cap tech/consumer unless something 
  outside is genuinely exceptional and you flag it clearly
- "Hot" means fundamentally interesting, not just high price momentum
- No buy/sell recommendations — frame as "worth investigating" or "on watch"
- Web data may be stale or biased — say so when relevant
- Quantify uncertainty: "I found limited data on X" beats a confident-sounding guess

---

## OUTPUT — Write HTML Report

After completing both parts, write the full report to:
`reports/YYYY-MM-DD-weekly.html`

Use the Write tool. Confirm in the conversation with one line:
`Report written → reports/YYYY-MM-DD-weekly.html`

The HTML file must be fully self-contained (embedded CSS, no external links).
Use this template — fill in all sections, do not skip Part 2:

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Weekly Holdings Review — YYYY-MM-DD</title>
<style>
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
  body {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', system-ui, sans-serif;
    font-size: 14px; line-height: 1.6; background: #f4f5f7; color: #1a1a2e;
    max-width: 980px; margin: 0 auto; padding: 2rem 1.5rem;
  }
  h1 { font-size: 1.4rem; font-weight: 700; margin-bottom: 0.15rem; }
  h2 { font-size: 1rem; font-weight: 700; margin: 1.25rem 0 0.5rem; color: #374151; }
  .subtitle { font-size: 0.8rem; color: #6b7280; margin-bottom: 1.75rem; }

  /* Part divider */
  .part-header {
    font-size: 0.7rem; font-weight: 700; text-transform: uppercase; letter-spacing: 0.1em;
    color: #9ca3af; border-bottom: 2px solid #e5e7eb; padding-bottom: 0.4rem;
    margin: 2.5rem 0 1.25rem;
  }
  .part-header:first-of-type { margin-top: 0; }

  /* Portfolio summary card */
  .summary-card {
    background: white; border-radius: 10px; padding: 1.5rem;
    box-shadow: 0 1px 4px rgba(0,0,0,0.08); margin-bottom: 1.75rem;
    display: grid; grid-template-columns: repeat(auto-fit, minmax(130px, 1fr)); gap: 1rem;
  }
  .metric .val { display: block; font-size: 1.4rem; font-weight: 700; }
  .metric .lbl { display: block; font-size: 0.7rem; text-transform: uppercase;
                  letter-spacing: 0.06em; color: #9ca3af; margin-top: 2px; }
  .pos { color: #16a34a; } .neg { color: #dc2626; } .neu { color: #1a1a2e; }

  /* Holding cards */
  .card {
    background: white; border-radius: 10px; padding: 1.25rem 1.5rem;
    box-shadow: 0 1px 4px rgba(0,0,0,0.08); margin-bottom: 1rem;
    border-left: 5px solid #e5e7eb;
  }
  .card.intact       { border-left-color: #16a34a; }
  .card.watch        { border-left-color: #d97706; }
  .card.flag         { border-left-color: #dc2626; }

  .card-header { display: flex; justify-content: space-between; align-items: flex-start;
                  margin-bottom: 0.75rem; gap: 0.5rem; flex-wrap: wrap; }
  .card-title { font-size: 1rem; font-weight: 700; }
  .card-sub { font-size: 0.8rem; color: #6b7280; margin-top: 2px; }

  /* Badges */
  .badge {
    display: inline-block; padding: 0.2em 0.65em; border-radius: 5px;
    font-size: 0.68rem; font-weight: 700; text-transform: uppercase; letter-spacing: 0.05em;
  }
  .b-hold   { background: #f3f4f6; color: #374151; }
  .b-add    { background: #dcfce7; color: #166534; }
  .b-trim   { background: #fef3c7; color: #92400e; }
  .b-watch  { background: #fef3c7; color: #92400e; }
  .b-hot    { background: #fce7f3; color: #9d174d; }
  .b-value  { background: #dbeafe; color: #1e40af; }
  .b-coc    { background: #f3f4f6; color: #6b7280; }

  /* Section label */
  .slabel {
    font-size: 0.67rem; font-weight: 700; text-transform: uppercase;
    letter-spacing: 0.08em; color: #9ca3af; margin: 0.85rem 0 0.3rem;
  }

  /* Tables */
  table { width: 100%; border-collapse: collapse; margin: 0.4rem 0; font-size: 0.83rem; }
  th { text-align: left; padding: 0.4rem 0.6rem; border-bottom: 2px solid #e5e7eb;
       font-weight: 600; color: #374151; background: #f9fafb; }
  td { padding: 0.4rem 0.6rem; border-bottom: 1px solid #f3f4f6; }
  tr:last-child td { border-bottom: none; }

  /* Action row */
  .action-row { display: flex; gap: 0.5rem; align-items: center; margin: 0.4rem 0;
                 font-size: 0.85rem; color: #6b7280; }

  /* Red flag */
  .red-flag { font-size: 0.83rem; color: #dc2626; margin-top: 0.25rem; }

  /* Attention list */
  .attention {
    background: #fffbeb; border-radius: 10px; padding: 1.25rem 1.5rem;
    border-left: 5px solid #f59e0b; margin-bottom: 1.5rem;
  }
  .attention h2 { color: #92400e; margin-top: 0; }
  ol { padding-left: 1.2rem; }
  ol li { margin-bottom: 0.35rem; font-size: 0.875rem; }

  /* Market pulse cards */
  .pulse-card {
    background: white; border-radius: 10px; padding: 1.25rem 1.5rem;
    box-shadow: 0 1px 4px rgba(0,0,0,0.08); margin-bottom: 1rem;
  }
  .pulse-card h2 { margin-top: 0; }
  ul { padding-left: 1.2rem; }
  ul li { margin-bottom: 0.35rem; font-size: 0.875rem; }

  /* Watch item */
  .watch-item { border-top: 1px solid #f3f4f6; padding-top: 0.75rem; margin-top: 0.75rem; }
  .watch-item:first-child { border-top: none; padding-top: 0; margin-top: 0; }
  .watch-ticker { font-weight: 700; font-size: 0.9rem; }
  .watch-why { font-size: 0.83rem; color: #374151; margin: 0.2rem 0; }
  .watch-val { font-size: 0.78rem; color: #6b7280; }

  /* Recap bullets */
  .recap-bullet { font-size: 0.875rem; padding: 0.35rem 0; border-bottom: 1px solid #f3f4f6; }
  .recap-bullet:last-child { border-bottom: none; }

  hr { border: none; border-top: 1px solid #e5e7eb; margin: 1.5rem 0; }
  footer { margin-top: 2rem; font-size: 0.72rem; color: #d1d5db; text-align: center; }

  @media print {
    body { background: white; }
    .card, .pulse-card, .attention { box-shadow: none; border: 1px solid #e5e7eb; }
  }
</style>
</head>
<body>

<h1>Weekly Holdings Review</h1>
<p class="subtitle">YYYY-MM-DD &nbsp;·&nbsp; Fidelity export + yfinance + web search</p>

<!-- ══════════════════════════════════════════ -->
<div class="part-header">Part 1 — Holdings Review</div>

<!-- PORTFOLIO SUMMARY -->
<div class="summary-card">
  <div class="metric">
    <span class="val neu">$000,000</span>
    <span class="lbl">Total Value</span>
  </div>
  <div class="metric">
    <span class="val pos">+0.0%</span>
    <span class="lbl">Today's Change</span>
  </div>
  <div class="metric">
    <span class="val pos">+0.0%</span>
    <span class="lbl">Weekly Change</span>
  </div>
  <div class="metric">
    <span class="val neu">0.0%</span>
    <span class="lbl">Cash</span>
  </div>
  <div class="metric">
    <span class="val neu">N</span>
    <span class="lbl">Positions</span>
  </div>
</div>

<!-- ATTENTION LIST (omit section entirely if 0 items) -->
<div class="attention">
  <h2>This Week's Attention List</h2>
  <ol>
    <li><strong>TICKER</strong> — reason requiring attention</li>
  </ol>
</div>

<!-- HOLDING CARD (repeat for each equity position) -->
<!-- card class: intact | watch | flag -->
<div class="card intact">
  <div class="card-header">
    <div>
      <div class="card-title">TICKER &nbsp;·&nbsp; Company Name</div>
      <div class="card-sub">
        $00,000 &nbsp;(00% of portfolio) &nbsp;·&nbsp;
        <span class="pos">+0.0% wk</span> &nbsp;·&nbsp;
        <span class="pos">+00.0% total</span>
      </div>
    </div>
    <!-- badge: b-hold | b-add | b-trim -->
    <span class="badge b-hold">Hold</span>
  </div>

  <div class="slabel">Valuation Snapshot</div>
  <table>
    <thead><tr><th>Metric</th><th>Current</th><th>5yr Avg</th><th>Note</th></tr></thead>
    <tbody>
      <tr><td>P/E (TTM)</td><td>00x</td><td>00x</td><td>—</td></tr>
      <tr><td>P/E (Fwd)</td><td>00x</td><td>—</td><td>—</td></tr>
      <tr><td>FCF Yield</td><td>0.0%</td><td>0.0%</td><td>10Y Treasury: 0.0%</td></tr>
    </tbody>
  </table>
  <p style="font-size:0.78rem; color:#6b7280; margin-top:0.25rem">
    Verdict: <strong>Fair / Cheap / Rich</strong> vs history
  </p>

  <div class="slabel">Action Signal</div>
  <div class="action-row">
    <span class="badge b-hold">Hold</span>
    No signal this week.
  </div>

  <div class="slabel">Red Flags</div>
  <p class="red-flag">None.</p>
</div>

<!-- PORTFOLIO BREAKDOWN -->
<hr>
<h2>Portfolio Breakdown</h2>
<table>
  <thead><tr><th>Holding</th><th>Value</th><th>% Portfolio</th><th>Total Return</th><th>Sector</th></tr></thead>
  <tbody>
    <tr><td>TICKER</td><td>$00,000</td><td>00%</td><td class="pos">+00%</td><td>Sector</td></tr>
  </tbody>
</table>
<p style="font-size:0.83rem; margin-top:0.75rem; color:#374151">
  <strong>Concentration:</strong> Top 3 — TICKER (00%), TICKER (00%), TICKER (00%) — combined 00%
</p>

<!-- ══════════════════════════════════════════ -->
<div class="part-header">Part 2 — Market Pulse &amp; Opportunities</div>

<!-- 2a. MARKET RECAP -->
<div class="pulse-card">
  <h2>Market Recap — This Week</h2>
  <div class="recap-bullet">Major indexes: S&amp;P 500 ±X%, Nasdaq ±X%, Dow ±X%</div>
  <div class="recap-bullet">Key catalyst: description</div>
  <div class="recap-bullet">Macro note: description</div>
</div>

<!-- 2b. NOTABLE EARNINGS -->
<div class="pulse-card">
  <h2>Notable Earnings (Last 2 Weeks)</h2>
  <div class="watch-item">
    <div class="watch-ticker">TICKER &nbsp;<span class="badge b-value">Beat</span></div>
    <div class="watch-why">Key takeaway from earnings.</div>
    <div class="watch-val">Implication for holdings or watch list.</div>
  </div>
  <!-- repeat watch-item for each -->
</div>

<!-- 2c. STOCKS WORTH WATCHING -->
<div class="pulse-card">
  <h2>Stocks Worth Watching</h2>
  <div class="watch-item">
    <div class="watch-ticker">
      TICKER &nbsp;
      <!-- badge: b-hot (momentum), b-value (undervalued), b-watch (developing), b-coc (outside CoC) -->
      <span class="badge b-hot">Hot</span>
    </div>
    <div class="watch-why">Fundamental reason — why this is worth attention.</div>
    <div class="watch-val">Valuation context: P/E 00x vs 5yr avg 00x · FCF yield 0.0%</div>
  </div>
  <!-- repeat watch-item for each, max 4 -->
</div>

<!-- 2d. POTENTIAL UNDERVALUED (omit section entirely if nothing genuine) -->
<div class="pulse-card">
  <h2>Potential Undervalued Opportunities</h2>
  <div class="watch-item">
    <div class="watch-ticker">TICKER &nbsp;<span class="badge b-value">Dropped X%</span></div>
    <div class="watch-why">Why it dropped. Are fundamentals intact?</div>
    <div class="watch-val">P/E 00x · FCF yield 0.0% · Assessment: warranted / potentially overdone</div>
  </div>
  <!-- repeat, max 3 -->
</div>

<footer>Generated by Claude Code &nbsp;·&nbsp; Fidelity export · yfinance · web search &nbsp;·&nbsp; Not investment advice</footer>
</body>
</html>
```
