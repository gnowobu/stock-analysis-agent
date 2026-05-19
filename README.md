# Stock Analysis — Claude Code Investment Research Agent

A Claude Code project that turns Claude into a structured investment research
assistant for a long-term, value-oriented portfolio of US large-cap stocks.

---

## What This Is

A set of prompts, skills, and an MCP integration that gives Claude a consistent
analytical framework for stock research. It is **not** a trading bot, screener,
or signal generator. It is a thinking tool for deliberate, value-focused
investors who want structured research without the fluff.

**Investor profile it's built for:**
- Style: Value investing (Buffett/Munger lineage)
- Horizon: 5+ years typical hold
- Universe: US large-cap, mostly tech and consumer
- Pace: Slow and deliberate. Bias toward inaction.

---

## Skills

Invoked as slash commands in Claude Code.

### `/valuation-framework [TICKER]`
Tiered fundamental analysis — from a 30-second quick screen to a full
intrinsic value estimate. Stop early when a stock is clearly uninteresting.

| Tier | Focus | Time |
|------|-------|------|
| 1 | Quick screen: P/E, EV/EBIT, FCF yield, growth → Interesting/Watch/Pass | 30 sec |
| 2 | Cash flow quality: FCF trends, margins, ROIC, SBC | 5 min |
| 3 | Balance sheet durability: debt, coverage, dilution | 5 min |
| 4 | Intrinsic value: owner earnings + reverse DCF (output as a range) | 15 min |
| 5 | Qualitative: moat, management, cyclical position, risks | open |

Default depth is **Tier 1**. Request more tiers explicitly ("run through Tier 3").

### `/stock-analysis [TICKER]`
Full single-stock deep dive combining valuation with context and judgment.

Structure: Snapshot → Recent Context (last 90 days) → Valuation (Tiers 1-3
default, or 1-5 for "full analysis") → Bull Case → Bear Case → What Would
Change My Mind → Circle-of-Competence Flag → Confidence Rating

Bear case is **always** required, even when analysis is bullish.

### `/weekly-holdings-review`
Structured weekly portfolio check. Requires your holdings (Symbol, Shares,
Cost Basis, Current Value — any format).

Per holding: thesis status, Tier 1 valuation snapshot, action signal.
Portfolio summary: concentration, sector allocation, attention list (max 3
items — most weeks should be 0-1).

Default answer is **hold, nothing changed.** Resist drama.

---

## Hard Rules (Never Violated)

- No specific buy/sell recommendations
- No price targets
- No position sizing advice
- No tax advice
- No confident macro predictions
- No anchoring to current price ("it dropped 20% so it's cheap" is a fallacy)

---

## Data Source

Financial data is pulled via **[yfmcp](https://github.com/narumiruna/yfmcp)**,
a yfinance MCP server. Treat it as directionally reliable, not audit-grade.
Unusual numbers should be cross-checked against primary filings.

---

## Setup

**Prerequisites:**
- [Claude Code](https://claude.ai/code) (CLI or desktop app)
- [uv](https://docs.astral.sh/uv/) — for `uvx` to run the MCP server

**Run:**
```
claude
```

The yfinance MCP server starts automatically via `.mcp.json`. No other
installation required — `uvx` handles the Python environment on first run.

**Verify MCP is connected:**
In Claude Code, run `/mcp` and confirm `yfinance` shows as connected.

---

## Input: Fidelity Export

Drop your Fidelity positions CSV export into the `input/` folder. The weekly review
reads it automatically — no reformatting needed.

**How to export from Fidelity:**
1. Accounts & Trade → Portfolio
2. Select your account
3. Click **Download** (top right) → CSV
4. Move the file to `input/`

The skill handles Fidelity's format quirks (header rows, `$`/`%` characters, pending
activity rows). yfinance provides current prices, weekly changes, and fundamentals.

## Output: Reports

Every skill writes a self-contained HTML file to `reports/`. Open it in any browser.

```
reports/2026-05-18-weekly.html
reports/2026-05-18-AAPL.html
```

The conversation confirms the path in a single line after writing. HTML reports are fully
offline — no external CSS or JS, print-friendly.

---

## Repository Layout

```
stock_analysis/
├── CLAUDE.md                         # Investor profile, principles, hard rules
├── .mcp.json                         # yfinance MCP server config
├── input/                            # Drop Fidelity CSV export here
├── reports/                          # Generated HTML reports
└── .claude/
    └── commands/
        ├── valuation-framework.md    # /valuation-framework skill
        ├── stock-analysis.md         # /stock-analysis skill
        └── weekly-holdings-review.md # /weekly-holdings-review skill (2-part)
```

---

## Example Usage

```
/valuation-framework AAPL
```
```
/stock-analysis MSFT
```
```
/stock-analysis NVDA full analysis
```
```
/weekly-holdings-review
[paste holdings]
```
```
Is Costco's moat widening or narrowing?
```

Ad-hoc questions (no slash command) default to the core principles in CLAUDE.md.
