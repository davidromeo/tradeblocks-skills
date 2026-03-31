---
name: compare
description: Performance comparison for trading strategies. Compare backtest vs actual results, strategy vs strategy metrics, block vs block, or period vs period performance. Use when exploring differences between theoretical and live execution, understanding how two strategies relate, or analyzing performance across time periods.
compatibility: Requires TradeBlocks MCP server with trade data loaded
---

# Performance Comparison

Explore differences between strategies, execution modes, or time periods.

## Prerequisites

- TradeBlocks MCP server running
- At least one block with trade data loaded
- For backtest vs actual: Both trade log and reporting log for the same strategy
- For strategy vs strategy: Multi-strategy block or two blocks
- For block vs block: Two or more blocks

## Process

### Step 1: Identify Comparison Type

Ask the user what they want to compare:

| Type | Use Case | Primary Tool |
|------|----------|--------------|
| Backtest vs Actual | Explore theoretical vs live execution | `compare_backtest_to_actual` |
| Strategy vs Strategy (same block) | Compare strategies within a portfolio | `get_strategy_comparison` |
| Block vs Block | Compare separate portfolios side-by-side | `compare_blocks` / `block_diff` |
| Period vs Period | Analyze same strategy across time ranges | `get_period_returns` |

Ask: "What would you like to compare?"

### Step 2a: Backtest vs Actual Comparison

Use `compare_backtest_to_actual` to explore how theoretical performance compares to live execution.

**Key parameters:**
- `blockId`: Block folder name
- `scaling`: How to compare P&L fairly (see below)
- `strategy`: Optional filter to specific strategy
- `dateRange`: Optional date filter
- `matchedOnly`: Only include trades where both backtest and actual exist
- `groupBy`: Group results by `"none"`, `"strategy"`, `"date"`, `"week"`, or `"month"`
- `detailLevel`: `"summary"` (aggregate by date+strategy) or `"trades"` (individual trade comparison with field-by-field differences)
- `outliersOnly`: Only return high-slippage outliers (z-score threshold)
- `outliersThreshold`: Z-score threshold for outlier detection (default: 2)

**Scaling modes** (see [references/scaling.md](references/scaling.md)):

| Mode | What It Does | Use When |
|------|--------------|----------|
| `raw` | Shows P&L as-is | Contract sizes match between backtest and actual |
| `perContract` | Divides each P&L by contract count | Comparing per-lot performance regardless of size |
| `toReported` | Scales backtest DOWN to match actual contract count | Backtest uses more contracts than actual |

**Recommended approach:**
1. Start with `groupBy: "strategy"` and `scaling: "perContract"` for an overview
2. Drill into specific strategies with `groupBy: "month"` to spot trends
3. Use `outliersOnly: true` to find the worst slippage trades
4. Use `detailLevel: "trades"` for field-by-field comparison on outliers

**Tool returns:**
- Per-date/strategy comparison with backtest vs actual P&L
- Slippage calculation (actual minus backtest)
- Match status (whether both sides exist for each date)
- Summary totals and average slippage percentage
- Outlier detection with z-scores

Present findings from the data:
- **Total backtest P&L** vs **Total actual P&L** (at selected scaling)
- **Matched trade count** (how many dates have both)
- **Average slippage** (percentage deviation from backtest)
- **Unmatched trades** (missed fills or extra trades)
- **Outliers** (trades with unusually high slippage)

### Step 2b: Strategy vs Strategy Comparison (Same Block)

For comparing strategies within the same block, use `get_strategy_comparison`:

**Key parameters:**
- `blockId`: Block folder name
- `sortBy`: `"netPl"`, `"winRate"`, `"trades"`, `"profitFactor"`, `"name"`
- `sortOrder`: `"asc"` or `"desc"`
- `minTrades`: Minimum trades per strategy to include
- `startDate` / `endDate`: Optional date filters

**Tool returns per strategy:**
- Trade count, win rate, net P&L
- Average win, average loss
- Profit factor

Present as a ranked table:

| Strategy | Trades | Win Rate | Net P&L | Profit Factor |
|----------|--------|----------|---------|---------------|
| ... | ... | ... | ... | ... |

For deeper comparison between two specific strategies, also run:
- `get_correlation_matrix` to understand how they move together
- `get_statistics` on each (with `strategy` filter) for full metric suites

**Correlation context:**
- Very low (<0.2): Strategies move independently
- Low (0.2-0.4): Some independence
- Moderate (0.4-0.6): Shared behavior
- High (>0.6): Similar movements, less diversification

### Step 2c: Block vs Block Comparison

For comparing separate portfolio blocks:

**Option 1: Side-by-side metrics** via `compare_blocks`:
- `blockIds`: Array of block IDs (max 5)
- `metrics`: Optional filter to specific metrics (e.g., `["sharpeRatio", "maxDrawdown", "profitFactor"]`)
- `sortBy`: Sort by any metric

**Option 2: Strategy overlap analysis** via `block_diff`:
- `blockIdA`: First block (baseline)
- `blockIdB`: Second block (comparison target)
- Shows shared vs unique strategies between blocks
- Calculates performance deltas for shared strategies

Use `compare_blocks` for a quick overview, then `block_diff` when blocks share strategies and you want to understand what changed.

### Step 2d: Period vs Period Comparison

For analyzing performance across time:

Use `get_period_returns` with period type and optional date filters.

**Key parameters:**
- `period`: `"monthly"`, `"weekly"`, or `"daily"`
- `dateRange`: Optional filter to specific time range
- `normalizeTo1Lot`: Normalize for fair comparison across different position sizes
- `strategy`: Optional strategy filter

Present period breakdown:
- Performance by month/quarter/year
- Best and worst periods
- Trends or regime changes

Questions to explore:
- "How does recent performance compare to earlier results?"
- "Are there periods that stand out?"
- "Does performance vary by time of year?"

### Step 3: Present Findings

Synthesize the data into what stands out:

**Comparison Summary:**
- What was compared: [backtest vs actual / strategy A vs B / block X vs Y / period X vs Y]
- Key observation: [Most notable difference from the data]
- Magnitude: [Size of the divergence]

**What the data shows:**
- [Notable finding 1 from tool output]
- [Notable finding 2 from tool output]
- [Any patterns or anomalies]

**Context for interpretation:**
- [Possible explanations for observed differences]
- [Factors that may affect the comparison]

Present these as observations from the historical data. The user can decide what meaning to draw from the findings.

## Interpretation Reference

For detailed explanation of scaling modes, see [references/scaling.md](references/scaling.md).

## Related Skills

After comparison analysis:
- `/tradeblocks:health-check` - Deep dive into either strategy
- `/tradeblocks:portfolio` - Explore correlation and diversification
- `/tradeblocks:wfa` - Test parameter robustness

## Notes

- Backtest results typically look better than live (ideal fills, no slippage)
- Some degradation from backtest to live is expected
- High correlation between strategies means they may draw down together
- Short comparison periods have more noise than signal
- Scaling mode choice affects what story the data tells
