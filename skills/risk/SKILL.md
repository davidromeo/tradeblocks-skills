---
name: risk
description: Risk analysis for trading strategies including Kelly criterion calculations, tail risk metrics, Monte Carlo projections, stress testing, and drawdown attribution. Use when exploring position sizing, capital allocation, or understanding worst-case characteristics.
compatibility: Requires TradeBlocks MCP server with trade data loaded
---

# Risk Analysis

Explore risk characteristics and position sizing metrics for trading strategies.

## Prerequisites

- TradeBlocks MCP server running
- Block with trade data (10+ trades minimum for meaningful metrics)

## Process

### Step 1: Understand User Goals

Risk analysis serves different purposes. Ask what the user wants to understand:

| Goal | Primary Tool | Also Consider |
|------|-------------|---------------|
| "What does Kelly suggest?" | `get_position_sizing` | Monte Carlo for drawdown context |
| "What are worst-case scenarios?" | `run_monte_carlo` | `stress_test` for named scenarios |
| "How did I do during COVID/bear markets?" | `stress_test` | Drawdown attribution |
| "What caused my biggest drawdown?" | `drawdown_attribution` | Tail risk for future risk |
| "How correlated are my strategies?" | `get_tail_risk` | Position sizing per strategy |
| "What if I change allocations?" | `what_if_scaling` | Marginal contribution |

Ask: "What aspect of risk would you like to explore?"

Then use `list_blocks` to identify the target block.

### Step 2: Position Sizing (Kelly Criterion)

Call `get_position_sizing` with the user's capital base.

**Key parameters:**
- `capitalBase`: Starting capital (required)
- `kellyFraction`: "full", "half" (default), or "quarter"
- `maxAllocationPct`: Cap per strategy (default: 25%)
- `minTrades`: Minimum trades for valid calculation (default: 10)
- `sortBy`: Sort by "kelly", "winRate", "payoffRatio", "allocation", "name"
- `useMarginReturns`: Use percentage returns based on margin (better for compounding strategies)

**Tool returns:**
- Win rate and payoff ratio (inputs to Kelly formula)
- Raw Kelly percentage (what the formula suggests)
- Adjusted allocations at full/half/quarter Kelly
- Per-strategy breakdown if multiple strategies exist
- Warnings (e.g., "Portfolio Kelly exceeds 25%", "negative Kelly")

**Important context for Kelly:**
- Full Kelly is mathematically optimal but assumes perfect knowledge of edge
- Half Kelly is commonly used to account for estimation uncertainty
- Negative Kelly indicates historical losses exceeded wins (Kelly formula doesn't apply)

### Step 3: Stress Testing

Call `stress_test` to see how the portfolio performed during named historical stress scenarios.

**Key parameters:**
- `blockId`: Block folder name
- `scenarios`: Optional list of specific scenario names (omit for all built-in scenarios)
- `customScenarios`: User-defined scenarios with custom date ranges
- `includeEmpty`: Include scenarios with no trades (default: false)

**Tool returns per scenario:**
- Scenario name and date range
- Trade count, win rate, net P&L during that period
- Profit factor and max drawdown

Present as a stress test table:

| Scenario | Dates | Trades | Win Rate | Net P&L | Max DD |
|----------|-------|--------|----------|---------|--------|
| ... | ... | ... | ... | ... | ... |

Key insight: If a strategy had zero trades during a stress period, it had no exposure — that's important context (either it didn't exist yet, or its filters kept it out).

### Step 4: Monte Carlo Projections

Call `run_monte_carlo` for probabilistic projections:

**Key parameters:**
- `includeWorstCase`: Enable worst-case injection (default: true)
- `worstCasePercentage`: Percentage of worst-case trades (default: 5%)
- `worstCaseMode`: "pool" (adds to resample pool) or "guarantee" (ensures worst appears)
- `resampleMethod`: "trades" (default), "daily", or "percentage" (for compounding)
- `simulationLength`: Number of trades/days to project forward
- `normalizeTo1Lot`: Normalize for fair comparison across position sizes

Focus on:
- 5th percentile outcome (valueAtRisk.p5)
- Probability of profit
- Mean and median max drawdown
- Distribution of terminal equity

### Step 5: Drawdown Attribution

Call `drawdown_attribution` to identify what caused the worst drawdown.

**Key parameters:**
- `blockId`: Block folder name
- `strategy`: Optional filter
- `topN`: Number of top contributors (default: 5)

**Tool returns:**
- Drawdown period (peak date to trough date)
- Peak and trough equity values
- Per-strategy P&L attribution during the drawdown
- Percentage contribution to total loss

This answers "which strategies caused the most pain" during the worst period.

### Step 6: Tail Risk (Multi-Strategy)

Call `get_tail_risk` for extreme co-movement analysis (requires 2+ strategies).

**Key parameters:**
- `tailThreshold`: Percentile for "tail" events (default: 0.1 = worst 10%)
- `varianceThreshold`: For effective factors calculation (default: 0.8)
- `normalization`: "raw", "margin", or "notional"

**Tool returns:**
- Joint tail risk matrix (do strategies fail together?)
- Effective factors (how many independent risk sources exist)
- Risk level: LOW (<0.3), MODERATE (0.3-0.5), HIGH (>0.5)
- Copula correlation (statistical dependency structure)

### Step 7: What-If Scaling

Call `what_if_scaling` to explore how allocation changes affect risk metrics.

**Key parameters:**
- `blockId`: Block folder name
- `strategyWeights`: Weight per strategy (e.g., `{"StrategyA": 0.5, "StrategyB": 1.5}`)
- `strategies`: Multi-strategy mode with per-strategy block source and scale factor
- `showUncapped`: Also show results without maxContractsPerTrade ceiling

**Tool returns:**
- Before/after portfolio metrics comparison
- Per-strategy breakdown at new weights
- Profile-aware: respects contract ceilings from profiles

Common questions this answers:
- "What if I removed this underperforming strategy?"
- "What if I doubled my best performer?"
- "What if I halved everything during high VIX?"

### Step 8: Present Findings

Synthesize findings into what the data reveals:

**Position Sizing Metrics:**
- Win rate: [value]% | Payoff ratio: [value]
- Kelly formula suggests: [value]% (based on historical data)
- At half Kelly: [dollar amount] of [capital base]
- [Any warnings from the tool]

**Stress Test Results:**
- [Worst scenario]: [P&L and drawdown during that period]
- [Best scenario]: [how the strategy held up]
- Scenarios with no exposure: [list]

**Monte Carlo Projections:**
- 5th percentile return: [value]
- Probability of profit: [value]%
- Mean max drawdown: [value]%

**Drawdown Attribution:**
- Worst drawdown: [peak] to [trough] ([magnitude])
- Primary contributor: [strategy] at [percentage] of total loss

**Tail Risk (if applicable):**
- Average joint tail risk: [value] ([LOW/MODERATE/HIGH])
- Effective factors: [value] of [strategy count]
- [Note any high-risk pairs]

**What stands out:**
- [Highlight notable findings]
- [Surface any warnings from the tools]
- [Note relationships between metrics]

Present these as insights from the historical data, letting the user decide what fits their situation.

## Interpretation References

- [references/kelly-guide.md](references/kelly-guide.md) - Kelly criterion explained
- [references/tail-risk.md](references/tail-risk.md) - Understanding fat tails

## Common Scenarios

### "I have $100,000 - what does Kelly suggest?"

1. Run `get_position_sizing` with `capitalBase: 100000`
2. Review the Kelly percentages and warnings
3. Surface both raw Kelly and half-Kelly figures
4. Note that Kelly assumes independent trades and known edge

### "Do my strategies fail together?"

1. Run `get_tail_risk`
2. Look at joint tail risk matrix for high values
3. Check effective factors (closer to 1 = more correlated risk)
4. High correlation means drawdowns may compound

### "What's the worst realistic outcome?"

1. Run `stress_test` for historical named scenarios
2. Run `run_monte_carlo` with worst-case injection for probabilistic projection
3. Run `drawdown_attribution` to understand what drove the historical worst
4. Combine: stress tests show what happened, Monte Carlo shows what might happen

### "Should I change my allocations?"

1. Run `what_if_scaling` with proposed weights
2. Compare before/after Sharpe, drawdown, net P&L
3. Check if the change improves risk-adjusted returns or just shifts risk

## Related Skills

After risk analysis:
- `/tradeblocks:health-check` - Full metrics overview
- `/tradeblocks:portfolio` - Correlation and diversification analysis
- `/tradeblocks:wfa` - Test parameter robustness

## Notes

- Kelly assumes independent trades; real trades may be correlated
- Historical volatility may underestimate future extremes
- Monte Carlo resamples history - it can't predict unknown risks
- Stress tests only cover known historical scenarios
- Negative Kelly means the formula doesn't apply (no positive edge in historical data)
