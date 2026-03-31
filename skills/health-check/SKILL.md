---
name: health-check
description: Strategy health check for trading backtests. Analyzes performance metrics, runs stress tests, and surfaces risk indicators. Use when evaluating a strategy's historical performance and stress characteristics.
compatibility: Requires TradeBlocks MCP server with trade data loaded
---

# Strategy Health Check

Surface key performance metrics and stress test results to help understand a strategy's characteristics.

## Prerequisites

- TradeBlocks MCP server must be running
- At least one block with trade data loaded

## Process

### Step 1: Select Strategy

List available blocks and help the user choose what to analyze.

```
Use list_blocks to show available options.
```

Ask clarifying questions:
- "Which backtest would you like to analyze?"
- "Do you want to analyze the full portfolio or a specific strategy within it?"

If analyzing a specific strategy, note it for filtering in subsequent steps.

**Shortcut for multi-strategy portfolios:** If the user wants a comprehensive portfolio-level health check, consider using `portfolio_health_check` (see Step 6) instead of running Steps 2-5 individually. It combines correlation, tail risk, Monte Carlo, walk-forward, and profile-aware dimensions in one call.

### Step 2: Gather Basic Metrics

Run `get_statistics` for the selected block (with strategy filter if specified).

Present key metrics with context:

| Metric | What It Measures |
|--------|------------------|
| Sharpe Ratio | Risk-adjusted return (higher = better return per unit risk) |
| Sortino Ratio | Downside risk-adjusted return (focuses only on losses) |
| Max Drawdown | Largest peak-to-trough decline (lower = less historical pain) |
| Win Rate | Percentage of trades that were profitable |
| Profit Factor | Gross wins / gross losses (>1 means profitable overall) |
| Net P&L | Total profit after commissions |

**Key insight:** A strategy can have low win rate but high profit factor if average wins exceed average losses significantly. Neither metric alone tells the full story.

### Step 3: Stress Testing

Run `stress_test` to see how the strategy performed during named historical market stress scenarios (COVID crash, 2022 bear, VIX spikes, etc.).

**Key parameters:**
- `blockId`: Block folder name
- `scenarios`: Optional list of specific scenario names (omit to run all built-in scenarios)
- `customScenarios`: User-defined scenarios with custom date ranges
- `includeEmpty`: Include scenarios with no trades (default: false)

**Tool returns per scenario:**
- Scenario name and date range
- Trade count, win rate, net P&L during that period
- Profit factor and max drawdown during the stress period

Present a stress test summary table:

| Scenario | Dates | Trades | Win Rate | Net P&L | Max DD |
|----------|-------|--------|----------|---------|--------|
| COVID Crash | | | | | |
| 2022 Bear | | | | | |
| VIX Spike Events | | | | | |

Flag any scenario where the strategy had significant losses or drawdown.

### Step 4: Monte Carlo Projections

Run `run_monte_carlo` to project performance under uncertainty.

Key parameters to understand:
- `resampleMethod`: "trades" resamples individual trade P&L (default)
- `includeWorstCase`: Injects synthetic worst-case scenarios (default: true)
- `worstCasePercentage`: How much of simulation is worst-case (default: 5%)

Focus on these outputs:
- **5th percentile outcome**: What the data suggests in a bad scenario (1 in 20 chance of worse)
- **Probability of profit**: How often simulations ended profitable
- **Mean max drawdown**: Typical drawdown across simulations

Present these as "what the historical data suggests could happen" - not predictions.

### Step 5: Drawdown Attribution

Run `drawdown_attribution` to identify which strategies contributed most to losses during the portfolio's maximum drawdown period.

**Key parameters:**
- `blockId`: Block folder name
- `strategy`: Optional filter to specific strategy
- `topN`: Number of top contributors to return (default: 5)

**Tool returns:**
- Drawdown period (peak date to trough date)
- Peak and trough equity values
- Per-strategy P&L attribution during the drawdown
- Each strategy's percentage contribution to the total loss

Present which strategies drove the worst drawdown. This is critical for understanding concentrated risk.

### Step 6: Portfolio Health Check (Multi-Strategy Blocks)

For multi-strategy blocks, run `portfolio_health_check` to get a comprehensive one-call assessment.

**Tool returns a layered report:**
- **Verdict**: Overall status (HEALTHY / MODERATE_CONCERNS / SIGNIFICANT_CONCERNS)
- **Grades**: A-F across 9 dimensions (diversification, tail risk, robustness, consistency, regime coverage, day coverage, concentration risk, correlation risk, scaling alignment)
- **Flags**: Specific warnings and passes with details
- **Key numbers**: Sharpe, Sortino, max drawdown, avg correlation, avg tail dependence, MC probability of profit, walk-forward efficiency

Surface the verdict, any warning flags, and the dimension grades. Focus attention on any grade below B.

### Step 7: Summary

Synthesize findings into a clear picture of what the data shows:

**Metrics Summary:**
- Sharpe Ratio: [value] - [context: >1.0 considered acceptable by many, >2.0 considered excellent]
- Max Drawdown: [value] - [context: <20% relatively low, >40% significant]
- Profit Factor: [value] - [context: >1.5 considered good, >2.0 excellent]

**Stress Test Insights:**
- [Scenario results — which stress periods hurt, which held up]
- [Any scenarios with zero trades (no exposure during that period)]

**Monte Carlo Projections:**
- 5th percentile scenario: [value]
- Probability of profit: [value]

**Drawdown Attribution:**
- Worst drawdown period: [peak date] to [trough date]
- Primary contributor: [strategy] ([percentage] of total loss)

**What stands out:**
- [Highlight any notably strong or weak metrics]
- [Note any warnings from the tools]
- [Stress scenarios that caused outsized damage]

Let the user draw their own conclusions about whether this fits their risk tolerance.

## Interpretation Reference

For detailed explanations of each metric, see [references/metrics.md](references/metrics.md).

## Related Skills

After health check, the user may want to:
- `/tradeblocks:wfa` - Test if optimized parameters hold up on unseen data
- `/tradeblocks:risk` - Deep dive into position sizing and tail risk analysis
- `/tradeblocks:portfolio` - Portfolio-level correlation and diversification

## Notes

- Always use trade-based calculations when filtering by strategy (daily logs represent full portfolio)
- Historical performance doesn't guarantee future results
- Stress tests only cover known historical scenarios — unknown risks aren't captured
