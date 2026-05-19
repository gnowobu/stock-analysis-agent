---
name: valuation-framework
description: Use when the user asks to analyze, evaluate, or value a single stock — including queries like "look at NVDA", "is AAPL cheap", "value MSFT", "should I look at COST". Runs a tiered fundamental analysis from quick screen to intrinsic value estimate. Focus on US large-cap value-investing lens. Do not use for whole-portfolio reviews (use weekly-holdings-review instead).
---

# Valuation Framework

Tiered analysis for a single stock. Run only the tiers requested. Default 
to Tier 1 if unspecified. Stop early when a stock is clearly uninteresting. 
State which tier you're in. Cite source and date for every number.

## Tier 1 — Quick Screen (30 sec)
Goal: Is this worth deeper work?

Pull and present:
- Current price, market cap
- P/E trailing & forward — vs 5-10yr company history, vs sector median
- EV/EBIT or EV/EBITDA — same comparisons
- FCF yield (FCF / Market Cap) — compare to 10Y treasury yield as floor
- Revenue growth (3yr & 5yr CAGR)

End with: **Verdict: Interesting / Watch / Pass** + one-line reason.
If Pass, stop unless asked to continue.

## Tier 2 — Cash Flow & Quality (5 min)
Goal: Is the business actually good?

- FCF trend (last 5-10 yrs): growing / flat / declining / lumpy?
- FCF vs Net Income — should track. Flag divergence >20%.
- Cash conversion = FCF / Net Income (healthy: >80% multi-year avg)
- ROIC or ROCE — vs ~8-10% cost of capital
- Gross margin & operating margin trend
- CapEx as % of operating cash flow (capital intensity)
- Stock-based comp as % of revenue (especially for tech)

Call out red flags:
- FCF persistently below net income
- Margins compressing
- ROIC declining
- SBC >5% of revenue without clear justification

## Tier 3 — Balance Sheet & Durability (5 min)
Goal: Can this survive a bad scenario?

- Net debt / EBITDA (concern if >3x for non-utilities)
- Interest coverage = EBIT / Interest (concern if <5x)
- Current ratio, quick ratio
- Goodwill as % of total assets (concern if >30%)
- Share count over 5-10 yrs — buybacks (good) or dilution (flag)
- Cash & equivalents

## Tier 4 — Intrinsic Value Estimate (15 min)
Goal: What is this business worth, roughly?

Present TWO methods, show the range:

**A) Owner Earnings (Buffett-style)**
Owner Earnings ≈ Net Income + D&A − Maintenance CapEx − ΔWorking Capital
Apply 10-20x multiple depending on quality/growth.

**B) Reverse DCF**
Given current price, what growth rate is the market pricing in?
Compare to historical growth, analyst consensus, my view.

Output a **range, not a point**. Example:
"Fair value range: $180–$240. Current $210. Market pricing in ~12% growth; 
10-yr historical was 9%."

State key assumptions explicitly. Flag the most fragile one.

## Tier 5 — Qualitative
Goal: Do I understand WHY this business earns what it earns?

- Business model in 2 sentences
- Moat: which kind? (cost, network, switching, brand, scale, regulatory)
- Moat trajectory: widening / stable / eroding?
- Management quality: capital allocation track record
- Cyclical position: earnings normalized, elevated, or depressed?
- Top 2-3 risks from latest 10-K Risk Factors

## Output Format (any tier)
- Numbers in a table when comparing across periods or peers
- Cite source and date for every number
- End with: **Confidence: High / Medium / Low** + one-line reasoning
- End with the **bear case** in 2-3 sentences, even if analysis is bullish
