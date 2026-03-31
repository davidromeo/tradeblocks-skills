---
name: wfa
description: Walk-forward analysis for trading strategies. Tests whether optimized parameters hold up on out-of-sample data. Use when checking parameter robustness, detecting potential overfitting, or validating a backtest.
compatibility: Requires TradeBlocks MCP server with trade data loaded
---

# Walk-Forward Analysis

Test whether strategy parameters hold up when applied to data the optimizer never saw.

## What is Walk-Forward Analysis?

Walk-forward analysis (WFA) tests parameter robustness by:

1. **Dividing history into segments**
2. **In-Sample (IS):** The data used to find "optimal" parameters
3. **Out-of-Sample (OOS):** Data the optimizer never saw, used to test those parameters
4. **Rolling forward:** Repeat across the entire history

```
|------ IS Period 1 ------|-- OOS 1 --|
              |------ IS Period 2 ------|-- OOS 2 --|
                            |------ IS Period 3 ------|-- OOS 3 --|
```

If OOS performance is close to IS performance, the parameters may be capturing real patterns. If OOS significantly underperforms, the parameters may be fitting to noise.

## Prerequisites

- TradeBlocks MCP server running
- Block with sufficient trade history (50+ trades for meaningful analysis)

## Process

### Step 1: Select Strategy

Use `list_blocks` to show available blocks.

Ask:
- "Which backtest contains the strategy you want to analyze?"
- "Do you want to test a specific strategy or the full portfolio?"

Note the block ID and optional strategy filter for subsequent steps.

### Step 2: Understand User Goals

Walk-forward analysis answers different questions:

| Goal | What to Look For |
|------|------------------|
| Test if parameters are robust | Overall WF efficiency, OOS vs IS degradation |
| Check for potential overfitting | High IS but low OOS performance |
| Evaluate consistency | How many OOS periods were profitable |
| Understand parameter sensitivity | Parameter stability across windows |
| Test strategy weight combinations | Use `parameterRanges` with strategy weights |
| Test position sizing approaches | Use `parameterRanges` with Kelly/fraction params |

Ask: "What are you trying to understand about this strategy?"

### Step 3: Run Analysis

Call `run_walk_forward` with the selected block.

**Core parameters:**
- `blockId`: Block folder name
- `strategy`: Filter to specific strategy
- `isWindowCount`: Number of in-sample windows (default: 5)
- `oosWindowCount`: Number of out-of-sample windows (default: 1)
- `optimizationTarget`: Metric to optimize (default: "sharpeRatio")
  - Options: "netPl", "profitFactor", "sharpeRatio", "sortinoRatio", "calmarRatio", "cagr", "avgDailyPl", "winRate"
- `minInSampleTrades`: Minimum trades in IS period (default: 10)
- `minOutOfSampleTrades`: Minimum trades in OOS period (default: 3)
- `normalizeTo1Lot`: Normalize trades to 1-lot (useful for pct_of_portfolio sizing)

**Explicit window sizing (overrides window counts):**
- `inSampleDays`: Explicit IS period in days
- `outOfSampleDays`: Explicit OOS period in days
- `stepSizeDays`: Days to slide forward each period (defaults to OOS days)

**Parameter ranges for grid search:**
- `parameterRanges`: Define sweep ranges as `{paramName: [min, max, step]}`

| Parameter | What It Tests | Example |
|-----------|--------------|---------|
| `kellyMultiplier` | Kelly fraction scaling | `[0.25, 1.0, 0.25]` → tests 0.25x/0.5x/0.75x/1.0x |
| `fixedFractionPct` | Fixed fraction sizing | `[1, 4, 1]` → tests 1-4% |
| `fixedContracts` | Fixed contract count | `[1, 5, 1]` → tests 1-5 contracts |
| `maxDrawdownPct` | Max drawdown constraint | `[15, 25, 5]` → rejects combos exceeding 15-25% |
| `maxDailyLossPct` | Max single-day loss constraint | `[2, 5, 1]` |
| `consecutiveLossLimit` | Max consecutive losers | `[3, 7, 1]` |
| `strategy:StrategyName` | Per-strategy weight | `[0, 1, 0.5]` → tests include/exclude |

Multiple parameters create a grid search across all combinations.

**Risk constraints:**
- `enableCorrelationConstraint`: Reject highly correlated strategy combinations (default: false)
- `maxCorrelationThreshold`: Max allowed correlation (default: 0.7)
- `enableTailRiskConstraint`: Reject high tail dependence combinations (default: false)
- `maxTailDependenceThreshold`: Max allowed tail dependence (default: 0.5)
- `minProfitFactor`: Reject combinations below this profit factor
- `minSharpeRatio`: Reject combinations below this Sharpe
- `requirePositiveNetPl`: Reject combinations with losses

**Strategy selection:**
- `selectedStrategies`: Filter to specific strategies (default: all)
- `tickerFilter`: Filter by underlying ticker

**For shorter histories (< 100 trades):**
- Reduce `isWindowCount` to 3
- Lower minimum trade counts

**For longer histories (> 500 trades):**
- Consider `isWindowCount` of 7+
- Use explicit `inSampleDays` and `outOfSampleDays` for more control

### Step 4: Interpret Results

The tool returns a verdict with three components, each rated as "good", "moderate", or "concerning":

**Walk-Forward Efficiency (degradationFactor):**
- Measures how well IS performance transfers to OOS
- Calculated as: OOS Performance / IS Performance

| Efficiency | Rating | What It Suggests |
|------------|--------|------------------|
| >= 80% | Good | OOS retained most of IS performance |
| 60-79% | Moderate | Some degradation, but meaningful signal may remain |
| < 60% | Concerning | Significant gap between IS and OOS |

*Thresholds based on Pardo's work, adjusted upward because TradeBlocks uses ratio metrics (Sharpe, profit factor) which should degrade less than raw returns.*

**Parameter Stability:**
- Coefficient of variation < 30% = "good" (stable parameters)
- 30-50% variation = "moderate"
- > 50% variation = "concerning" (parameters sensitive to data window)

**Consistency Score:**
- Percentage of OOS periods that were profitable
- >= 70% = "good"
- 50-70% = "moderate" (around random chance)
- < 50% = "concerning"

### Step 5: Present Findings

Synthesize the analysis into what it reveals about the strategy:

**Walk-Forward Results:**
- Efficiency: [value]% ([rating]) - OOS retained [value]% of IS performance
- Stability: [rating] - Parameters [were consistent / showed variation] across windows
- Consistency: [value]% of OOS periods profitable ([rating])
- Overall verdict: [good/moderate/concerning]

**What this suggests:**
- [If efficiency is high]: OOS performance tracked IS reasonably well
- [If efficiency is low]: Significant gap between optimized and real-world performance
- [If stability is low]: Optimal parameters varied significantly between windows
- [If consistency is low]: Many OOS periods were unprofitable

**Individual period breakdown** (if relevant):
- Show IS vs OOS performance for each window
- Highlight any windows with unusual behavior

**Grid search results** (if parameterRanges used):
- Best parameter combination found
- How stable was the "best" across windows
- Adjacent parameter performance (smooth gradient = robust, cliff = overfit)

Present these as insights about what the historical data shows, not as trading advice.

## Interpretation Reference

For detailed walk-forward concepts, see [references/wfa-guide.md](references/wfa-guide.md).

## Related Skills

After walk-forward analysis:
- `/tradeblocks:health-check` - Full metrics review
- `/tradeblocks:risk` - Position sizing analysis
- `/tradeblocks:optimize` - Parameter exploration

## Common Scenarios

### "Is my backtest overfit?"

1. Run basic WFA with default settings
2. If efficiency < 60%, the strategy may be fitting to noise
3. Check parameter stability — high variation means parameters aren't robust
4. Low consistency (< 50% OOS profitable) is a red flag

### "Which strategy combination is most robust?"

1. Use `parameterRanges` with `strategy:StrategyName` weights
2. Enable `enableCorrelationConstraint` and `enableTailRiskConstraint`
3. The optimizer will reject high-correlation, high-tail-risk combinations
4. Check if the "winning" combination is consistent across OOS periods

### "What Kelly fraction should I use?"

1. Use `parameterRanges` with `kellyMultiplier: [0.25, 1.0, 0.25]`
2. WFA tests each fraction across time windows
3. If full Kelly dominates IS but half Kelly wins OOS, that's a signal to size conservatively

### "How many trades do I need?"

- 50+ trades: Reasonable for basic WFA
- 100+ trades: Can use more windows for finer resolution
- 200+ trades: Can use explicit day-based windows
- <50 trades: Reduce window count to 3, interpret with extreme caution

## Common Issues

**"Insufficient trades for walk-forward analysis"**
- Need at least 20 trades total
- Reduce window count or expand date range

**Low consistency but decent efficiency:**
- Small sample size causing noise
- Consider running with more windows if data permits

**Individual periods show high variance:**
- May indicate regime changes during the historical period
- The tool's `periods` array shows per-window breakdown

**WFE looks inflated due to position sizing growth:**
- Use `normalizeTo1Lot: true` to remove sizing bias
- Especially important for pct_of_portfolio strategies where later trades are larger

## Notes

- WFA tests parameter robustness, not profitability
- Strong WFA results don't guarantee future performance
- Weak WFA results suggest the optimization may be fitting to noise
