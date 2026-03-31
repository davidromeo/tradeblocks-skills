---
name: portfolio
description: Portfolio analysis for trading strategies. Explores correlation, diversification, and combined performance characteristics. Use when understanding how strategies relate, exploring diversification effects, or analyzing portfolio composition.
compatibility: Requires TradeBlocks MCP server with trade data loaded
---

# Portfolio Analysis

Explore how strategies relate and what combining them means for portfolio characteristics.

## What This Skill Does

Surfaces data to help understand portfolio dynamics:
- **Health Check**: One-call comprehensive portfolio assessment
- **Correlation**: How do strategies move relative to each other?
- **Structure**: Where does the portfolio have overlap or blind spots?
- **Marginal Contribution**: Which strategies help or hurt risk-adjusted returns?
- **Similarity**: Are any strategies redundant?
- **What-If Scaling**: How would changing allocations affect the portfolio?

## Prerequisites

- TradeBlocks MCP server running
- Multiple strategy blocks or a multi-strategy block loaded
- Strategy profiles recommended for regime coverage and structure analysis

## Process

### Step 1: Portfolio Health Check

Start with `portfolio_health_check` for a comprehensive one-call assessment.

**Key parameters:**
- `blockId`: Block folder name
- `correlationThreshold`: Flag pairs above this (default: 0.5)
- `tailDependenceThreshold`: Flag tail pairs above this (default: 0.5)
- `profitProbabilityThreshold`: MC profit probability warning (default: 0.95)
- `wfeThreshold`: Walk-forward efficiency warning (default: -0.15)

**Tool returns a layered report:**
- **Verdict**: HEALTHY / MODERATE_CONCERNS / SIGNIFICANT_CONCERNS
- **Grades**: A-F across 9 dimensions:

| Dimension | What It Measures |
|-----------|-----------------|
| Diversification | Correlation between strategy pairs |
| Tail Risk | Joint extreme co-movement risk |
| Robustness | Walk-forward efficiency (OOS vs IS) |
| Consistency | Monte Carlo probability of profit |
| Regime Coverage | Strategy performance across VIX regimes |
| Day Coverage | Trading day coverage across the week |
| Concentration Risk | Allocation by structure, underlying, DTE |
| Correlation Risk | Profile-aware overlap (same underlying + DTE + days) |
| Scaling Alignment | Backtest vs live per-contract P&L deviation |

- **Flags**: Specific warnings with details (high correlation pairs, tail dependence pairs, etc.)
- **Key numbers**: Sharpe, Sortino, max drawdown, avg correlation, avg tail dependence, MC stats, WFE

Present the verdict, flag any grades below B, and surface the specific warning details.

### Step 2: Correlation Analysis

Use `get_correlation_matrix` for detailed pairwise correlation data.

**Key parameters:**
- `blockId`: Block folder name
- `method`: "kendall" (robust, rank-based, default), "spearman" (rank), "pearson" (linear)
- `alignment`: "shared" (only days both traded) or "zero-pad" (fill missing with 0)
- `timePeriod`: "daily", "weekly", or "monthly" aggregation
- `normalization`: "raw" (absolute P&L), "margin" (P&L/margin), "notional" (P&L/notional)
- `minSamples`: Minimum shared periods for valid calculation (default: 10)
- `highlightThreshold`: Flag pairs above this (default: 0.7)

**Interpreting correlation values:**

| Correlation | What It Indicates |
|-------------|-------------------|
| < 0.2 | Very low - strategies move independently |
| 0.2 - 0.4 | Low - mostly independent movement |
| 0.4 - 0.6 | Moderate - some shared behavior |
| 0.6 - 0.8 | High - significant shared movement |
| > 0.8 | Very high - strategies behave similarly |

See [references/correlation.md](references/correlation.md) for why Kendall's tau is often more informative than Pearson for trading returns.

### Step 3: Portfolio Structure Map

Use `portfolio_structure_map` to see a Vol_Regime x Trend_Direction matrix across all profiled strategies.

**Key parameters:**
- `blockId`: Optional — omit to aggregate across all blocks
- `minTrades`: Thin-data warning threshold (default: 10)

**Tool returns:**
- 18-cell matrix (6 Vol_Regimes x 3 Trend_Directions) with per-strategy stats
- **Overlap detection**: 2+ strategies active in the same cell
- **Blind spots**: Cells with zero trades across all strategies
- **Thin-data warnings**: Cells with fewer trades than threshold

This is the key tool for understanding portfolio construction — where the portfolio has coverage and where it doesn't.

### Step 4: Marginal Contribution

Use `marginal_contribution` to see how each strategy affects portfolio risk-adjusted returns.

**Key parameters:**
- `blockId`: Block folder name
- `targetStrategy`: Calculate for specific strategy only (optional)
- `topN`: Number of top contributors to return (default: 5)

**Tool returns:**
- Baseline portfolio Sharpe and Sortino
- Per-strategy marginal Sharpe and Sortino impact
- Most beneficial and least beneficial strategies

**Interpretation:**
- **Negative marginal Sharpe**: Removing this strategy would LOWER the portfolio Sharpe — it's contributing positively
- **Positive marginal Sharpe**: Removing this strategy would RAISE the portfolio Sharpe — it may be hurting risk-adjusted returns
- Note: marginal contribution measures risk-adjusted impact, not absolute P&L. A profitable strategy can hurt Sharpe if it adds volatility.

### Step 5: Regime Allocation Advisor

Use `regime_allocation_advisor` to cross-reference strategy profiles' expected regimes with actual performance.

**Key parameters:**
- `blockId`: Optional — omit to aggregate across all profiled strategies
- `minTrades`: Minimum trades per regime cell for reliable stats (default: 5)

**Tool returns per strategy per regime:**
- Whether the regime was expected (from profile) or unexpected
- Win rate, P&L, trade count in that regime
- Classifications:
  - `thesis_aligned`: Strategy performs as expected in its target regimes
  - `thesis_violation`: Strategy underperforms in regimes it should handle
  - `hidden_edge`: Strategy performs well in regimes not marked as expected

Surface any thesis violations (strategies failing where they shouldn't) and hidden edges (opportunities the profile doesn't capture).

### Step 6: Strategy Similarity

Use `strategy_similarity` to detect potentially redundant strategies.

**Key parameters:**
- `blockId`: Block folder name
- `correlationThreshold`: Min correlation to flag (default: 0.7)
- `tailDependenceThreshold`: Min tail dependence to flag (default: 0.5)
- `minSharedDays`: Minimum shared trading days (default: 30)
- `topN`: Number of most similar pairs (default: 5)

**Tool returns:**
- Most similar strategy pairs ranked by combined similarity score
- Correlation, tail dependence, and trading day overlap for each pair
- Flags for strategies that may be adding risk without diversification benefit

If two strategies are highly correlated AND have high tail dependence, they'll likely draw down together — the "diversification" between them is illusory.

### Step 7: What-If Scaling

Use `what_if_scaling` to explore allocation changes.

**Key parameters:**
- `blockId`: Block folder name
- `strategyWeights`: Weight per strategy, e.g., `{"5/7 17D": 0.5, "Pickle RIC": 1.5}`. Unspecified default to 1.0. Weight 0 = exclude.
- `strategies`: Multi-strategy mode — array with per-strategy block source and scale factor
- `showUncapped`: Also show results without maxContractsPerTrade ceiling

**Tool returns:**
- Before/after comparison of portfolio metrics
- Per-strategy breakdown at new weights
- Profile-aware: respects maxContractsPerTrade ceilings from profiles
- Flags when ignoreMarginReq is set

Common scenarios:
- "What if I removed this strategy?" → set its weight to 0
- "What if I doubled this allocation?" → set weight to 2.0
- "What if I halved everything except my best performer?" → adjust weights accordingly

### Step 8: Present Findings

Synthesize the data:

**Portfolio Health:**
- Verdict: [HEALTHY / MODERATE_CONCERNS / SIGNIFICANT_CONCERNS]
- Dimensions needing attention: [any grades below B]
- Key flags: [specific warnings]

**Correlation Findings:**
- Average correlation across pairs: [value]
- Highest correlation pair: [pair] at [value]
- Lowest correlation pair: [pair] at [value]

**Structure:**
- Regime blind spots: [cells with no coverage]
- Overlap areas: [cells with 2+ strategies]
- Concentration: [by underlying, DTE, structure type]

**Marginal Contribution:**
- Most beneficial: [strategy] (marginal Sharpe: [value])
- Least beneficial: [strategy] (marginal Sharpe: [value])

**What stands out:**
- [Notable patterns]
- [Any redundant strategy pairs]
- [Thesis violations or hidden edges]

Present these as insights from the historical data. The user can decide what fits their risk tolerance and portfolio goals.

## Interpretation References

- [references/correlation.md](references/correlation.md) - Understanding correlation methods
- [references/diversification.md](references/diversification.md) - Diversification concepts and tail risks

## Related Skills

After portfolio analysis:
- `/tradeblocks:compare` - Deep comparison of specific strategy pairs or blocks
- `/tradeblocks:risk` - Position sizing and Kelly analysis
- `/tradeblocks:health-check` - Full metrics on any individual strategy

## Notes

- Correlation is measured on aggregated returns, not trade-by-trade
- Past correlation patterns may not persist in future market conditions
- Tail correlation (crisis behavior) is often higher than normal correlation
- Low correlation doesn't guarantee protection - both can lose for different reasons
- Sample size matters - 10 shared data points is minimum, more is better
- Strategy profiles are required for regime coverage, structure map, and scaling alignment analysis
