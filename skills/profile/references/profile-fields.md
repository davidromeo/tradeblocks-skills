# Profile Field Reference

## Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `blockId` | string | Block ID from `list_blocks`. Not the test name — these often differ. |
| `strategyName` | string | Human-readable name (e.g., "Monday 2/4 DC - QQQ"). Should match across blocks if the same strategy. |
| `structureType` | string | Option structure type. See Structure Types below. |
| `greeksBias` | string | Primary greeks exposure. Verify with `decompose_greeks`, don't assume. |

## Other Top-Level Fields

| Field | Type | Description |
|-------|------|-------------|
| `underlying` | string | Underlying symbol (SPX, QQQ, etc.). Practically always needed. |
| `thesis` | string | Free-text description of the strategy thesis |
| `capLosses` | boolean | Losses are capped by structure (verify from P/L distribution) |
| `capProfits` | boolean | Profits are capped by structure |
| `reEntry` | boolean | Strategy supports re-entry on same day (verify from trade data) |
| `closeOnCompletion` | boolean | Close entire position when any leg hits target |
| `ignoreMarginReq` | boolean | Strategy ignores standard margin requirements (OO setting) |
| `requireTwoPricesPT` | boolean | Profit target requires two prices (OO setting) |

## Structure Types

| Type | Legs | Example |
|------|------|---------|
| `double_calendar` | 4 legs, 2 DTE groups, puts + calls | Short 2 DTE / Long 4 DTE at 20Δ |
| `iron_condor` | 4 legs, same expiry, OTM puts + calls | Short 30Δ put + call, long wings |
| `calendar_spread` | 2 legs, same strike, different expiry | Short front / Long back |
| `vertical_spread` | 2 legs, same expiry, different strikes | Bull put spread, bear call spread |
| `butterfly` | 3 strikes, same expiry | Long wings, short body |
| `reverse_iron_condor` | 4 legs, opposite of iron condor | Long OTM, short ITM |
| `short_put_spread` | 2 puts, same expiry | Short higher / Long lower |
| `short_call_spread` | 2 calls, same expiry | Short lower / Long higher |
| `straddle` | 2 legs, same strike, same expiry | Short ATM put + call |
| `strangle` | 2 legs, different OTM strikes, same expiry | Short OTM put + call |

## Greeks Bias Values

| Value | Meaning | Common Structures |
|-------|---------|-------------------|
| `theta_positive` | Profits from time decay | Iron condors, calendars, straddles |
| `vega_negative` | Profits from vol crush | Short straddles, iron condors |
| `vega_positive` | Profits from vol expansion | Long straddles, reverse iron condors |
| `delta_neutral` | No directional bias | Balanced structures |
| `delta_positive` | Bullish directional bias | Bull spreads, long calls |
| `delta_negative` | Bearish directional bias | Bear spreads, long puts |
| `gamma_scalp` | Profits from gamma rebalancing | Long straddles with hedging |

**Important:** A double calendar can be theta_positive AND vega_negative or vega_positive depending on the DTE spread. Always verify with `decompose_greeks`.

## Leg Fields

| Field | Type | Description |
|-------|------|-------------|
| `type` | string | `long_put`, `short_call`, `long_call`, `short_put` |
| `strike` | string | Strike selection description: "20-delta", "ATM", "5-point-offset" |
| `strikeMethod` | string | `delta`, `dollar_price`, `offset`, `percentage` |
| `strikeValue` | number | Numeric value (e.g., 20 for 20-delta, 5 for 5-point offset) |
| `expiry` | string | Expiry selection: "2-DTE", "4-DTE", "weekly", "45-DTE" |
| `quantity` | number | Positive = long, negative = short |

### Reading Legs from Option Omega Screenshots

Each leg row in OO shows colored markers:

```
S  B  C  P  |  QTY  |  Δ/±  |  value  |  DTE
```

- **S** (red background) = Sell → short leg (negative quantity)
- **B** (green background) = Buy → long leg (positive quantity)
- **C** (teal/green) = Call
- **P** (pink/red) = Put
- **Δ** = delta-based strike → `strikeMethod: "delta"`
- **±** = strike offset from Current Price → `strikeMethod: "offset"`
- **↔** = strike offset from a non-default reference price → `strikeMethod: "offset"` — capture the reference price in the `strike` string field

### Strike Offset Reference Prices

When `strikeMethod` is `offset`, OO allows selecting what price the offset is relative to:

| OO Setting | Description | How to Record |
|------------|-------------|---------------|
| Current Price | Offset from current underlying price (default) | `strike: "10-point offset"` |
| Current Day's Open | Offset from the day's opening price | `strike: "10-point offset from Day Open"` |
| Previous Day's Close | Offset from prior day's close | `strike: "10-point offset from Prev Close"` |
| Previous Week's Close | Offset from prior week's close | `strike: "10-point offset from Prev Week Close"` |
| Current Week's Open | Offset from current week's open | `strike: "10-point offset from Week Open"` |

The reference price is visible in the OO strategy editor (dropdown next to "Strike Offset"). In the compact screenshot view, `↔` indicates a non-default reference price — check the strategy subtitle or notes for which one.

### Example: Standard DC (equal quantities, delta strikes)
```
S B C P | 1 QTY | 20 Δ | 2 DTE  → short_put, 20-delta, 2-DTE
S B C P | 1 QTY |  0 ± | 4 DTE  → short_call, 0-offset (ATM), 4-DTE
S B C P | 1 QTY | 20 Δ | 2 DTE  → long_call, 20-delta, 2-DTE
S B C P | 1 QTY |  0 ± | 4 DTE  → long_put, 0-offset (ATM), 4-DTE
```

### Example: Ratio DC with mixed strike methods
```
S B C P | 3 QTY | 10 ↔ | 1 DTE  → short_call, 10-offset from Day Open, 1-DTE, x3
S B C P | 3 QTY | 25 ± | 10 DTE → long_call, 25-offset, 10-DTE, x3
S B C P | 2 QTY | 45 Δ | 1 DTE  → short_put, 45-delta, 1-DTE, x2
S B C P | 2 QTY | -25 ± | 10 DTE → long_put, -25-offset, 10-DTE, x2
```

The highlighted letters indicate which markers are active for that leg.

## Entry Filters

Each filter is tagged with a `source`:

| Source | Meaning | Used In Analysis? |
|--------|---------|-------------------|
| `market` | Testable against market data columns (VIX, RSI, S/L ratio) | Yes — `validate_entry_filters` tests these |
| `execution` | Platform-level rules (time of day, day of week, leg ratios) | No — documented but skipped in analysis |

Common market filters:

| Field | Operator | Example | What It Does |
|-------|----------|---------|--------------|
| `VIX_Close` | `<` | 25 | Only enter when VIX below threshold |
| `VIX_Close` | `between` | [15, 25] | VIX range filter |
| `RSI_14` | `between` | [30, 70] | Avoid extreme RSI |
| `openingSLRatio` | `>` | 0.45 | Minimum short/long premium ratio |
| `Vol_Regime` | `in` | ["low", "below_avg"] | Only trade in specific regimes |

Common execution filters:

| Field | Description |
|-------|-------------|
| Entry time | "Open trades at 3:00 PM" |
| Entry day | "Every Monday" |
| DTE mode | "Use exact DTE" |
| Strike mode | "Use exact strike offsets" |

## Exit Rules

| Type | Trigger Format | Example |
|------|---------------|---------|
| `profit_target` | Percentage of credit received | "50% of credit" |
| `stop_loss` | Dollar, percentage, S/L ratio, or debit % | "200% of credit" |
| `time_exit` | Clock time in ET | "14:45 ET" |
| `conditional` | Leg delta threshold, S/L ratio move | "Sell put delta > 0.70" |

**OO note:** "Exit using leg delta(s)" in Option Omega means delta-based exits are enabled, but the screenshot doesn't show the threshold values. You must ask the user for these.

### Advanced Exit Sub-Fields

Each exit rule can also include:

| Field | Type | Description |
|-------|------|-------------|
| `stopLossType` | enum | Calculation method: `percentage`, `dollar`, `sl_ratio`, `debit_percentage` |
| `stopLossValue` | number | Numeric value for the stop loss |
| `monitoring.granularity` | enum | Price check frequency: `intra_minute`, `candle_close`, `end_of_bar` |
| `monitoring.priceSource` | enum | Which price to use: `nbbo`, `mid`, `last` |
| `slippage` | number | Per-rule slippage override |

## Position Sizing

| Field | Description |
|-------|-------------|
| `method` | `pct_of_portfolio`, `fixed_contracts`, `fixed_dollar`, `discretionary` |
| `allocationPct` | Portfolio allocation % (standalone backtest) |
| `backtestAllocationPct` | Allocation in the backtest context |
| `liveAllocationPct` | Allocation in live trading |
| `maxContracts` | Hard cap on contracts |
| `maxContractsPerTrade` | Per-entry contract limit |
| `maxOpenPositions` | Max concurrent positions |
| `maxAllocationDollar` | Maximum dollar allocation per trade |

**Multi-block note:** The same strategy in a standalone backtest block vs a portfolio block will have different sizing. Use `backtestAllocationPct` and `liveAllocationPct` to distinguish, or clone the profile with different `positionSizing` per block.

## Expected Regimes

VIX-based volatility regimes:

| Regime | VIX Range | Description |
|--------|-----------|-------------|
| `very_low` | < 13 | Extremely calm markets |
| `low` | 13 - 16 | Low volatility |
| `below_avg` | 16 - 20 | Below average, normal-ish |
| `above_avg` | 20 - 25 | Above average, slightly elevated |
| `high` | 25 - 30 | High volatility |
| `extreme` | > 30 | Crisis-level volatility |

If the user doesn't know their expected regimes, offer to run `analyze_regime_performance` on the block data to see where the strategy actually performs best.
