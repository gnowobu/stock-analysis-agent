---
name: stock-analysis
description: Use when the user asks for a deeper look at a single stock — "analyze NVDA", "look at AAPL", "deep dive on COST", "full analysis on MSFT". This skill orchestrates valuation-framework tiers, adds context (recent news, earnings, competitive position), and ends with a structured summary. Use for individual stock work; for whole-portfolio review use weekly-holdings-review.
---

# Stock Analysis

Single-stock deep dive. Combines the valuation-framework with recent 
context and qualitative judgment. Defaults to Tier 1-3 depth unless 
the user asks for "full analysis" (Tiers 1-5).

## Structure

### 1. Snapshot
- Company, ticker, sector, market cap
- Current price, 52-week range
- One-sentence business description

### 2. Recent Context (last 90 days)
- Most recent earnings: beat/miss, key takeaways
- Material news, guidance changes, M&A
- Notable analyst or insider activity (factual, no recommendations)

### 3. Valuation
Run valuation-framework at requested depth (default Tiers 1-3, 
"full" = Tiers 1-5). Follow that skill's output format.

### 4. Bull Case (3 bullets)
Strongest reasons this works as an investment over 5+ years.

### 5. Bear Case (3 bullets)
Strongest reasons this could disappoint. Required, even when bullish.

### 6. What Would Change My Mind
2-3 specific, observable things that would flip the thesis 
(e.g., "FCF margin compresses below 20% for two consecutive years").

### 7. Outside Circle of Competence?
Flag explicitly if this requires expertise the user likely lacks 
(complex financials, cyclical timing, regulatory exposure, etc.).

## Rules
- No buy/sell recommendation. No price target.
- Cite source and date for every number.
- Be honest about data gaps. "I don't have reliable Q3 FCF; using TTM as proxy."
- End with: **Confidence: High / Medium / Low** + one-line reasoning.

---

## Output — Write HTML Report

After completing the analysis, write the full report to:
`reports/YYYY-MM-DD-TICKER.html`   (e.g., `reports/2026-05-18-AAPL.html`)

Use the Write tool. Confirm in the conversation with one line:
`Report written → reports/YYYY-MM-DD-TICKER.html`

The HTML file must be fully self-contained (embedded CSS, no external links).
Use this template — fill in all values, omit sections only when data is unavailable:

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>TICKER — Stock Analysis — YYYY-MM-DD</title>
<style>
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
  body {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', system-ui, sans-serif;
    font-size: 14px; line-height: 1.6; background: #f4f5f7; color: #1a1a2e;
    max-width: 960px; margin: 0 auto; padding: 2rem 1.5rem;
  }
  h1 { font-size: 1.5rem; font-weight: 700; margin-bottom: 0.15rem; }
  h2 { font-size: 1rem; font-weight: 700; margin: 1.5rem 0 0.5rem; color: #374151;
       padding-bottom: 0.3rem; border-bottom: 1px solid #e5e7eb; }
  .subtitle { font-size: 0.8rem; color: #6b7280; margin-bottom: 1.5rem; }

  /* Snapshot card */
  .snapshot {
    background: white; border-radius: 10px; padding: 1.5rem;
    box-shadow: 0 1px 4px rgba(0,0,0,0.08); margin-bottom: 1.5rem;
    display: grid; grid-template-columns: repeat(auto-fit, minmax(140px, 1fr)); gap: 1rem;
  }
  .metric { }
  .metric .val { display: block; font-size: 1.3rem; font-weight: 700; }
  .metric .lbl { display: block; font-size: 0.7rem; text-transform: uppercase;
                  letter-spacing: 0.06em; color: #9ca3af; margin-top: 2px; }
  .pos { color: #16a34a; }
  .neg { color: #dc2626; }

  /* Content sections */
  .section {
    background: white; border-radius: 10px; padding: 1.5rem;
    box-shadow: 0 1px 4px rgba(0,0,0,0.08); margin-bottom: 1.25rem;
  }
  .section h2 { margin-top: 0; }

  /* Bull / Bear side by side */
  .bull-bear {
    display: grid; grid-template-columns: 1fr 1fr; gap: 1rem; margin-bottom: 1.25rem;
  }
  .bull { background: #f0fdf4; border-radius: 10px; padding: 1.25rem; border-top: 3px solid #16a34a; }
  .bear { background: #fef2f2; border-radius: 10px; padding: 1.25rem; border-top: 3px solid #dc2626; }
  .bull h2, .bear h2 { border: none; margin-top: 0; padding-bottom: 0; }
  .bull h2 { color: #166534; }
  .bear h2 { color: #991b1b; }
  ul { padding-left: 1.2rem; }
  li { margin-bottom: 0.4rem; font-size: 0.875rem; }

  /* Tables */
  table { width: 100%; border-collapse: collapse; margin: 0.4rem 0; font-size: 0.85rem; }
  th { text-align: left; padding: 0.4rem 0.6rem; border-bottom: 2px solid #e5e7eb;
       font-weight: 600; color: #374151; background: #f9fafb; }
  td { padding: 0.4rem 0.6rem; border-bottom: 1px solid #f3f4f6; }
  tr:last-child td { border-bottom: none; }
  .flag { color: #dc2626; font-weight: 600; }

  /* Confidence badge */
  .confidence {
    display: inline-block; padding: 0.3em 0.8em; border-radius: 6px;
    font-size: 0.8rem; font-weight: 700; text-transform: uppercase;
    letter-spacing: 0.05em;
  }
  .conf-high { background: #dcfce7; color: #166534; }
  .conf-medium { background: #fef3c7; color: #92400e; }
  .conf-low { background: #fee2e2; color: #991b1b; }

  /* Tier label */
  .tier-label {
    font-size: 0.68rem; font-weight: 700; text-transform: uppercase;
    letter-spacing: 0.08em; color: #9ca3af; margin: 0.9rem 0 0.3rem;
  }

  /* Info note */
  .note { font-size: 0.8rem; color: #6b7280; margin-top: 0.3rem; font-style: italic; }

  /* Circle of competence warning */
  .coc-flag {
    background: #fffbeb; border-radius: 8px; padding: 1rem 1.25rem;
    border-left: 4px solid #f59e0b; font-size: 0.875rem;
  }

  /* Divider */
  hr { border: none; border-top: 1px solid #e5e7eb; margin: 1.5rem 0; }

  /* Footer */
  footer { margin-top: 2rem; font-size: 0.72rem; color: #d1d5db; text-align: center; }

  @media (max-width: 640px) { .bull-bear { grid-template-columns: 1fr; } }
  @media print {
    body { background: white; }
    .section, .bull, .bear { box-shadow: none; border: 1px solid #e5e7eb; }
  }
</style>
</head>
<body>

<h1>TICKER — Company Full Name</h1>
<p class="subtitle">Stock Analysis &nbsp;·&nbsp; YYYY-MM-DD &nbsp;·&nbsp; Data via yfinance</p>

<!-- SNAPSHOT METRICS -->
<div class="snapshot">
  <div class="metric">
    <span class="val">$000.00</span>
    <span class="lbl">Current Price</span>
  </div>
  <div class="metric">
    <span class="val">$0.0B</span>
    <span class="lbl">Market Cap</span>
  </div>
  <div class="metric">
    <span class="val">$000 – $000</span>
    <span class="lbl">52-Week Range</span>
  </div>
  <div class="metric">
    <span class="val">Sector</span>
    <span class="lbl">Sector / Industry</span>
  </div>
</div>

<p style="font-size:0.9rem; margin-bottom:1.5rem; color:#374151">
  <em>One-sentence business description.</em>
</p>

<!-- RECENT CONTEXT -->
<div class="section">
  <h2>Recent Context (Last 90 Days)</h2>
  <div class="tier-label">Most Recent Earnings</div>
  <p style="font-size:0.875rem">Beat / Miss — key takeaways here.</p>

  <div class="tier-label">Material News & Guidance</div>
  <p style="font-size:0.875rem">Significant news, guidance changes, M&A activity.</p>

  <div class="tier-label">Analyst / Insider Activity</div>
  <p style="font-size:0.875rem">Factual summary only — no recommendations.</p>
</div>

<!-- VALUATION (Tiers 1-3 default; expand to Tiers 1-5 for full analysis) -->
<div class="section">
  <h2>Valuation</h2>

  <div class="tier-label">Tier 1 — Quick Screen</div>
  <table>
    <thead><tr><th>Metric</th><th>Current</th><th>5-10yr Avg</th><th>Sector Median</th></tr></thead>
    <tbody>
      <tr><td>P/E (TTM)</td><td>00x</td><td>00x</td><td>00x</td></tr>
      <tr><td>P/E (Fwd)</td><td>00x</td><td>—</td><td>—</td></tr>
      <tr><td>EV/EBIT</td><td>00x</td><td>00x</td><td>—</td></tr>
      <tr><td>FCF Yield</td><td>0.0%</td><td>0.0%</td><td>10Y Treasury: 0.0%</td></tr>
      <tr><td>Revenue Growth (3yr CAGR)</td><td>0.0%</td><td>—</td><td>—</td></tr>
      <tr><td>Revenue Growth (5yr CAGR)</td><td>0.0%</td><td>—</td><td>—</td></tr>
    </tbody>
  </table>
  <p class="note">Verdict: <strong>Interesting / Watch / Pass</strong> — one-line reason.</p>

  <div class="tier-label">Tier 2 — Cash Flow & Quality</div>
  <table>
    <thead><tr><th>Metric</th><th>Value</th><th>Trend</th><th>Flag?</th></tr></thead>
    <tbody>
      <tr><td>FCF Trend</td><td>—</td><td>Growing / Flat / Declining</td><td>—</td></tr>
      <tr><td>FCF vs Net Income</td><td>—</td><td>—</td><td class="flag">Flag if >20% divergence</td></tr>
      <tr><td>Cash Conversion (FCF/NI)</td><td>0%</td><td>—</td><td>—</td></tr>
      <tr><td>ROIC</td><td>0.0%</td><td>—</td><td>vs ~8-10% CoC</td></tr>
      <tr><td>Gross Margin</td><td>0.0%</td><td>—</td><td>—</td></tr>
      <tr><td>Operating Margin</td><td>0.0%</td><td>—</td><td>—</td></tr>
      <tr><td>SBC % of Revenue</td><td>0.0%</td><td>—</td><td class="flag">Flag if >5%</td></tr>
    </tbody>
  </table>

  <div class="tier-label">Tier 3 — Balance Sheet & Durability</div>
  <table>
    <thead><tr><th>Metric</th><th>Value</th><th>Threshold</th><th>Flag?</th></tr></thead>
    <tbody>
      <tr><td>Net Debt / EBITDA</td><td>0.0x</td><td>&lt;3x</td><td>—</td></tr>
      <tr><td>Interest Coverage (EBIT/Int)</td><td>0.0x</td><td>&gt;5x</td><td>—</td></tr>
      <tr><td>Current Ratio</td><td>0.0</td><td>—</td><td>—</td></tr>
      <tr><td>Goodwill % of Assets</td><td>0.0%</td><td>&lt;30%</td><td>—</td></tr>
      <tr><td>Share Count (5yr trend)</td><td>—</td><td>Declining (good)</td><td>—</td></tr>
      <tr><td>Cash & Equivalents</td><td>$0.0B</td><td>—</td><td>—</td></tr>
    </tbody>
  </table>

  <!-- TIER 4 — only for full analysis -->
  <!-- 
  <div class="tier-label">Tier 4 — Intrinsic Value Estimate</div>
  <p style="font-size:0.875rem"><strong>Owner Earnings (Buffett-style):</strong> …</p>
  <p style="font-size:0.875rem"><strong>Reverse DCF:</strong> At current price of $X, market is pricing in Y% growth. Historical 10yr CAGR was Z%. Analyst consensus: W%.</p>
  <p style="font-size:0.875rem; margin-top:0.5rem"><strong>Fair Value Range: $XXX – $XXX</strong>. Most fragile assumption: …</p>
  -->

  <!-- TIER 5 — only for full analysis -->
  <!-- 
  <div class="tier-label">Tier 5 — Qualitative</div>
  <p style="font-size:0.875rem"><strong>Business model:</strong> …</p>
  <p style="font-size:0.875rem"><strong>Moat:</strong> Type — widening / stable / eroding.</p>
  <p style="font-size:0.875rem"><strong>Management:</strong> Capital allocation track record.</p>
  <p style="font-size:0.875rem"><strong>Cyclical position:</strong> Normalized / elevated / depressed.</p>
  <p style="font-size:0.875rem"><strong>Top risks (10-K):</strong> …</p>
  -->
</div>

<!-- BULL / BEAR -->
<div class="bull-bear">
  <div class="bull">
    <h2>Bull Case</h2>
    <ul>
      <li>Reason 1</li>
      <li>Reason 2</li>
      <li>Reason 3</li>
    </ul>
  </div>
  <div class="bear">
    <h2>Bear Case</h2>
    <ul>
      <li>Reason 1</li>
      <li>Reason 2</li>
      <li>Reason 3</li>
    </ul>
  </div>
</div>

<!-- WHAT WOULD CHANGE MY MIND -->
<div class="section">
  <h2>What Would Change My Mind</h2>
  <ul>
    <li>Specific observable trigger 1 (e.g., "FCF margin compresses below 20% for two consecutive quarters")</li>
    <li>Specific observable trigger 2</li>
    <li>Specific observable trigger 3 (optional)</li>
  </ul>
</div>

<!-- CIRCLE OF COMPETENCE (include only if flagged) -->
<!--
<div class="coc-flag">
  <strong>Outside Circle of Competence:</strong> This analysis requires expertise in [area].
  Caveat: …
</div>
-->

<!-- CONFIDENCE -->
<div class="section">
  <h2>Confidence</h2>
  <!-- Use class: conf-high | conf-medium | conf-low -->
  <span class="confidence conf-medium">Medium</span>
  <p style="font-size:0.875rem; margin-top:0.75rem">One-line reasoning for confidence level.</p>
</div>

<footer>Generated by Claude Code &nbsp;·&nbsp; yfinance data &nbsp;·&nbsp; Not investment advice</footer>
</body>
</html>
```
