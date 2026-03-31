---
name: optimize
description: Parameter exploration for trading backtests. Analyzes trade data to find patterns across parameters like time of day, DTE, delta ranges, and market conditions. Use when exploring which parameters performed differently or understanding strategy behavior across conditions.
compatibility: Requires TradeBlocks MCP server with trade data loaded
---

# Parameter Exploration

Explore trade data to understand how performance varies across different parameters.

## What This Skill Does

Uses predictive field analysis, field statistics, filter curves, and market-based filter suggestions to answer questions like:
- "Which fields correlate with P&L?"
- "What does the VIX distribution look like for my trades?"
- "Is there an S/L ratio threshold that improves results?"
- "What market-based filters would have helped?"

**Important:** This skill helps surface patterns in historical data. Past patterns may not persist. See [references/optimization.md](references/optimization.md) for overfitting context.

## Prerequisites

- TradeBlocks MCP server running
- Block with trade data loaded
- Market data imported for market-based filter suggestions (daily OHLCV + VIX context)
- Sufficient trade count for meaningful analysis (50+ trades for better signal)

## Process

### Step 1: Identify Exploration Goal

Ask what the user wants to explore:

| Goal | Primary Tool |
|------|-------------|
| Find which fields predict P&L | `find_predictive_fields` |
| Understand a specific field's distribution | `get_field_statistics` |
| Test filter thresholds on a field | `filter_curve` |
| Get market-based filter suggestions | `suggest_filters` |
| Validate existing entry filters | `validate_entry_filters` |

Ask: "What aspect of your strategy would you like to explore?"

Use `list_blocks` to identify the target block.

### Step 2: Discover Predictive Fields

Run `find_predictive_fields` to rank all numeric fields by correlation with P&L.

**Key parameters:**
- `blockId`: Block folder name
- `strategy`: Optional filter to specific strategy
- `strategyName`: Strategy profile name (auto-filters to that strategy's trades, adds profile context)
- `targetField`: Field to correlate against (default: `"pl"`)
- `minSamples`: Minimum trades with valid values (default: 30)

**Tool returns:**
- All numeric fields ranked by absolute correlation with P&L
- Correlation direction (positive/negative)
- Sample size per field
- Fields skipped due to insufficient data

**Interpreting results — exclude output-derived fields** (netPl, plPct, rom, isWinner, maxProfit, maxLoss, profitCapturePercent, rMultiple, etc.) and focus on actionable entry/market fields:

| Field | What It Means | Actionable? |
|-------|--------------|-------------|
| `openingShortLongRatio` | Entry quality / structure health | Yes - potential min threshold filter |
| `openingVix` | VIX at entry | Yes - potential VIX range filter |
| `durationHours` | Holding period | Yes - exit timing |
| `movement` | Underlying movement during trade | Context for regime sensitivity |
| `premium` | Entry pricing | Yes - potential min/max filter |
| `gap` | Opening gap | Yes - potential gap filter |
| `hourOfDay` | Entry time | Yes - time-of-day filter |
| `numContracts` | Position size | Context for sizing effects |

Flag any actionable field with |correlation| > 0.1 as worth investigating further.

### Step 3: Examine Field Distribution

For fields identified in Step 2, use `get_field_statistics` to understand the data shape.

**Key parameters:**
- `blockId`: Block folder name
- `field`: Field name to analyze
- `strategy`: Optional filter
- `histogramBuckets`: Number of histogram buckets (default: 10, max: 50)

**Tool returns:**
- Range (min/max), mean, median, standard deviation
- Percentiles (p5, p10, p25, p50, p75, p90, p95)
- Histogram with bucket counts

This reveals:
- Where trades concentrate
- Whether there are outliers
- The natural breakpoints for filter thresholds

### Step 4: Test Filter Thresholds

Use `filter_curve` to sweep thresholds and see performance at each cutoff point.

**Key parameters:**
- `blockId`: Block folder name
- `field`: Field to sweep thresholds on
- `strategy`: Optional filter
- `mode`: `"lt"` (field < threshold), `"gt"` (field > threshold), `"both"` (show both directions)
- `thresholds`: Custom threshold values to test (auto-generated from percentiles if omitted)
- `percentileSteps`: Which percentiles to use for auto thresholds (default: [5, 10, 25, 50, 75, 90, 95])

**Tool returns per threshold:**
- Trade count remaining after filter
- Win rate, profit factor, net P&L of remaining trades
- Comparison to unfiltered baseline

Present as a table showing performance at each threshold:

| Threshold | Mode | Trades | Win Rate | PF | Net P&L | vs Baseline |
|-----------|------|--------|----------|-----|---------|-------------|
| ... | ... | ... | ... | ... | ... | ... |

**What to look for:**
- **Smooth gradient**: Performance improves gradually as threshold tightens — genuine signal
- **Sharp cliff**: Good at one value, terrible at adjacent — likely noise/overfit
- **Wide plateau**: A range of values all work well — robust filter candidate

### Step 5: Market-Based Filter Suggestions

Run `suggest_filters` to get data-driven filter suggestions based on market conditions.

**Key parameters:**
- `blockId`: Block folder name
- `strategy`: Optional filter to specific strategy
- `strategyName`: Strategy profile name (cross-references against existing profile filters)
- `minImprovementPct`: Only suggest filters with >= X% win rate improvement (default: 3)

**Tool returns:**
- Standalone filter suggestions with before/after metrics
- Composite filters (multi-field combinations where cross-field correlations are strong)
- Each suggestion includes the market field, threshold, direction, and improvement metrics

**Market fields analyzed:**
- VIX levels (open, prior close), VIX IVR/IVP
- Gap percentage, prior range vs ATR
- Vol regime, term structure state
- RSI, realized vol (5D/20D)
- Day of week, OpEx flag

Present the top suggestions ranked by improvement. For each, note:
- How many trades would be excluded
- The improvement in win rate and profit factor
- Whether the suggestion aligns with the strategy's thesis

### Step 6: Validate Existing Filters (If Profile Exists)

If the strategy has a profile with entry filters, run `validate_entry_filters`:

**Key parameters:**
- `blockId`: Block folder name
- `strategyName`: Strategy name matching a stored profile

**Tool returns:**
- Per-filter comparison: entered vs filtered-out trades (full stat suite)
- Ablation study: removes one filter at a time and tests pairs
- `profile_update_hints` when filters appear counterproductive

Surface any filters that are hurting rather than helping — where filtered-out trades actually outperform entered trades.

### Step 7: Present Findings

Synthesize what the data shows:

**Predictive Fields Summary:**
- Top actionable fields: [field 1] (r=[value]), [field 2] (r=[value])
- Fields with no signal: [list any with |r| < 0.05]

**Filter Curve Results (if tested):**
- Field tested: [field name]
- Optimal range: [threshold] with [trades remaining]
- Robustness: [smooth gradient / sharp cliff / wide plateau]

**Market Filter Suggestions (if run):**
- Top suggestion: [filter] — [improvement]% win rate improvement, excludes [N] trades
- [Any suggestions that align with strategy thesis]

**Sample Size Context:**
- [Note which analyses have sufficient data]
- [Flag any with <30 trades in key buckets]

**For further validation:**
- Run walk-forward analysis to test if pattern persists on unseen data (`/tradeblocks:wfa`)
- Collect more trades to increase sample sizes
- Check if pattern aligns with strategy thesis

Present these as patterns in the historical data. The user can decide what weight to give these observations.

## Interpretation Reference

For detailed guidance on interpreting optimization results and avoiding overfitting, see [references/optimization.md](references/optimization.md).

## Related Skills

After parameter exploration:
- `/tradeblocks:wfa` - Test if patterns hold on out-of-sample data
- `/tradeblocks:health-check` - Full metrics review
- `/tradeblocks:dc-analysis` - DC-specific deep dive with curve fit detection

## Common Scenarios

### "Which fields predict P&L for my strategy?"

1. Run `find_predictive_fields` with the block (and optional strategy filter)
2. Filter out output-derived fields (netPl, plPct, isWinner, etc.)
3. Present actionable fields ranked by |correlation|
4. For top fields, run `get_field_statistics` to understand distributions

### "Is there a VIX filter that would help?"

1. Run `get_field_statistics` on `openingVix` to see the distribution
2. Run `filter_curve` on `openingVix` to test thresholds
3. Also run `suggest_filters` to see if VIX-based filters are suggested
4. Compare: does the filter_curve sweet spot align with suggest_filters recommendation?

### "What market conditions should I avoid?"

1. Run `suggest_filters` to get data-driven suggestions
2. For each suggestion, run `filter_curve` on the underlying field to check robustness
3. Look for smooth gradients, not sharp cliffs
4. Cross-reference with strategy thesis — does the filter make sense mechanically?

### "Are my current filters actually helping?"

1. Run `validate_entry_filters` with the strategy profile name
2. Review per-filter entered vs filtered-out comparison
3. Check the ablation study for filter interactions
4. Surface any `profile_update_hints`

## Data Quality Notes

- **Historical patterns may not persist** - markets and conditions change
- **Small sample sizes are noisy** - 30+ trades per bucket for meaningful comparison
- **Multiple testing inflates apparent significance** - be skeptical of "best" findings
- **Validate with walk-forward** - use `/tradeblocks:wfa` for out-of-sample testing
- **Consider why** - patterns with logical explanations are more likely to persist

## Notes

- Exploration surfaces patterns; it doesn't prove causation
- The "best" parameter from historical data often regresses toward average
- Robustness across parameters often matters more than optimization to one value
- Consider the trading thesis - does the pattern make sense?
