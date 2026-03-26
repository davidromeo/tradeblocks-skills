---
name: market-data
description: Import and set up market data for TradeBlocks analysis. Guides through importing daily OHLCV, VIX term structure, and intraday option bars from API, CSV, or DuckDB sources. Use when market data is missing, regime analysis shows no matches, replay returns empty paths, or enrich_trades shows warnings.
compatibility: Requires TradeBlocks MCP server. API imports require MASSIVE_API_KEY env var.
---

# Market Data Setup

Guide the user through importing market data so other skills (DC analysis, health checks, replay) have the data they need.

## When This Skill Triggers

- `enrich_trades` returns warnings about missing market data
- `analyze_regime_performance` returns "No trades matched to market data"
- `replay_trade` returns 0 minute bars / $0 P&L
- `batch_exit_analysis` returns mostly `noTrigger` with $0 P&L
- User says "import market data", "set up market data", "I need VIX data", etc.

## Step 1: Diagnose What's Missing

Ask the user what they're trying to do, then check what data exists:

```sql
-- Check daily data coverage
SELECT ticker, COUNT(*) as rows,
  MIN(date) as earliest, MAX(date) as latest
FROM market.daily GROUP BY ticker ORDER BY ticker
```

```sql
-- Check intraday data coverage
SELECT
  CASE WHEN ticker LIKE 'SPX%' OR ticker LIKE 'SPXW%' THEN 'SPX options'
       WHEN ticker LIKE 'QQQ%' THEN 'QQQ options'
       WHEN ticker = 'SPX' THEN 'SPX underlying'
       ELSE ticker END as category,
  COUNT(DISTINCT ticker) as tickers,
  COUNT(*) as total_bars
FROM market.intraday GROUP BY category
```

```sql
-- Check context derived (regime/term structure)
SELECT COUNT(*) as rows, MIN(date) as earliest, MAX(date) as latest
FROM market._context_derived
```

Present what's available and what's missing for their goal.

## Step 2: Determine the Right Import

### For Regime Analysis (VIX regimes, term structure, enriched trades)

**Minimum needed:** Daily OHLCV for the underlying + VIX context (VIX/VIX9D/VIX3M)

| What to import | Tool | Example |
|---------------|------|---------|
| Underlying daily (SPX) | `import_from_api` | ticker: "SPX", target_table: "daily" |
| Underlying daily (QQQ) | `import_from_api` | ticker: "QQQ", target_table: "daily" |
| VIX context (all 3 tickers) | `import_from_api` | ticker: "VIX", target_table: "context" |
| Enrichment (RSI, Vol_Regime, etc.) | `enrich_market_data` | Auto-runs after import, or manual |

**Order matters:** Import daily first, then context, then enrichment runs automatically.

**Date range:** Match the trade data range. Check with:
```sql
SELECT MIN(date_opened) as earliest, MAX(date_opened) as latest
FROM trades.trade_data WHERE block_id = '<block>'
```

### For Trade Replay (minute-level P&L paths, greeks, exit trigger simulation)

**Needed:** Intraday 1-minute bars for each option leg in the trades

The replay engine auto-fetches on cache miss if `MASSIVE_API_KEY` is set. For bulk pre-loading:

1. Get the option tickers from recent trades:
```sql
SELECT DISTINCT legs, date_opened FROM trades.trade_data
WHERE block_id = '<block>' ORDER BY date_opened DESC LIMIT 10
```

2. Parse the OCC tickers from the legs string (the replay tool does this automatically)

3. Import each option ticker:
```
import_from_api: ticker="SPXW260320P06410000", target_table="intraday", timespan="1m"
```

**Note:** Option data availability varies by provider. Massive.com typically has data from ~2022 onward for SPX/SPY options. Older trades won't have replay data.

### For TradingView CSV Import

TradingView exports have a Unix timestamp column called `time`. The mapping handles date+time extraction automatically.

**Daily bars:**
```json
{
  "file_path": "~/Downloads/SPX_daily.csv",
  "ticker": "SPX",
  "target_table": "daily",
  "column_mapping": {
    "time": "date",
    "open": "open",
    "high": "high",
    "low": "low",
    "close": "close"
  }
}
```

**Intraday bars:**
```json
{
  "file_path": "~/Downloads/SPX_15m.csv",
  "ticker": "SPX",
  "target_table": "intraday",
  "column_mapping": {
    "time": "date",
    "open": "open",
    "high": "high",
    "low": "low",
    "close": "close"
  }
}
```

Use `dry_run: true` first to validate the mapping before writing.

### For External DuckDB Import

If the user has market data in another DuckDB file:

```json
{
  "db_path": "~/data/market.duckdb",
  "query": "SELECT date, open, high, low, close FROM ext_import_source.spx_daily",
  "ticker": "SPX",
  "target_table": "daily",
  "column_mapping": {
    "date": "date",
    "open": "open",
    "high": "high",
    "low": "low",
    "close": "close"
  }
}
```

## Step 3: Execute the Import

Run the imports based on what's needed. Common recipes:

### Recipe: Full SPX Setup (regime + enrichment)
1. `import_from_api`: ticker="SPX", from="2016-01-01", to="today", target_table="daily"
2. `import_from_api`: ticker="VIX", from="2016-01-01", to="today", target_table="context"
3. Enrichment runs automatically after each import

### Recipe: Full QQQ Setup
1. `import_from_api`: ticker="QQQ", from="2016-01-01", to="today", target_table="daily"
2. VIX context (same as above — shared across all underlyings)
3. Enrichment runs automatically

### Recipe: Replay Data for a Block
1. Run `batch_exit_analysis` or `replay_trade` — auto-fetches on cache miss if API key is set
2. Or manually import specific option tickers via `import_from_api` with target_table="intraday"

## Step 4: Verify

After importing, verify the data is available:

```sql
-- Check daily coverage
SELECT ticker, COUNT(*) as rows, MIN(date) as earliest, MAX(date) as latest
FROM market.daily GROUP BY ticker ORDER BY ticker
```

```sql
-- Check regime data populated
SELECT Vol_Regime, COUNT(*) as days
FROM market._context_derived
GROUP BY Vol_Regime ORDER BY Vol_Regime
```

```sql
-- Check term structure populated
SELECT Term_Structure_State, COUNT(*) as days
FROM market._context_derived
WHERE Term_Structure_State IS NOT NULL
GROUP BY Term_Structure_State
```

If `Term_Structure_State` is all NULL, VIX3M data is missing — run the context import.

## Common Issues

| Symptom | Cause | Fix |
|---------|-------|-----|
| `enrich_trades` shows null VIX3M fields | VIX3M not in market.daily | Import with target_table="context" |
| `Term_Structure_State` all NULL | VIX3M data missing | Import with target_table="context" |
| `analyze_regime_performance` returns 0 matched | No daily data for that ticker | Import daily OHLCV for the underlying |
| `replay_trade` returns 0 bars | No intraday option data cached | Set MASSIVE_API_KEY for auto-fetch, or import manually |
| Enrichment fields missing (RSI, ATR) | `enrich_market_data` not run | Run it manually for the ticker |
| API import returns 0 rows | Date range outside provider coverage | Try a more recent date range |
| CSV import fails on column mapping | Wrong column names | Use `dry_run: true` first to preview |

## What NOT to Do

- Don't import daily data without also importing VIX context — regime analysis needs both
- Don't assume all date ranges have data — Massive.com option data starts ~2022
- Don't skip enrichment — without it, RSI, Vol_Regime, and other derived fields won't exist
- Don't import the same data twice without checking — upserts are safe but waste API calls
