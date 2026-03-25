# Double Calendar Mechanics

Reference material for understanding DC behavior and analysis approaches.

## How a DC Works

A double calendar (DC) is a 4-leg options structure:
- **Short put + Short call** at the NEAR expiration (front month)
- **Long put + Long call** at the FURTHER expiration (back month), same strikes

The trade profits from:
1. **Theta decay differential** — short legs decay faster than long legs
2. **Underlying staying in range** — the "tent" between the two strikes
3. **Stable or slightly rising IV** — long legs have more vega exposure

## Key Variables That Change DC Behavior

### DTE Spread
- **Narrow (2/4, 2/3, 1/2):** Fast theta decay, high gamma risk, very sensitive to single-day moves. Charm decay is significant — deltas get sensitive close to expiry.
- **Mid (5/7, 6/7, 8/10):** Balance of theta and vega. Most variable in response to delta selection. The 5/7 in particular is very sensitive to which deltas you choose.
- **Wide (9/23, 14/21, 21/28):** More vega-driven, less sensitive to single-day moves. The "valley of death" (mid-trade sag between the peaks) is more pronounced. Need movement to reach one of the profit peaks.

### Delta Selection
- **Low delta (10-20):** Wide tent, more room for the underlying, but the valley of death is worse when range-bound. Benefits most from S/L ratio entry filtering.
- **Mid delta (25-35):** Sweet spot for many DCs. Narrower tent but better theta and less sag.
- **High delta (40-50):** Very narrow tent, fast theta, but more sensitive to fast vega shifts. When IV expands and then crushes, the front date legs can explode. S/L ratio filtering has less impact.

### S/L Ratio (Short/Long Ratio)
The ratio of short leg premium to long leg premium at entry. Key health metric for DCs.

**Entry filter:**
- Below 0.4: Generally poor entries across most DCs — not enough short premium
- 0.5-0.7: Sweet spot for many 5/7 DCs at lower deltas (PF 4-5 in Amy's testing)
- Above 0.7-0.75: Edge disappears for some DCs — both sides priced high, mean reversion likely
- The optimal range varies by DTE and delta — always test for your specific DC

**Exit indicator:**
- S/L ratio declining = spread value deteriorating
- S/L approaching 1.0 = one side likely deep ITM
- S/L ratio move down exit (e.g., -100% from entry) catches structural deterioration

**Correlation with VIX9D/VIX:** S/L ratio and VIX9D/VIX ratio are correlated because both measure aspects of term structure. Using both as filters is redundant — pick one.

## Exit Hierarchy (from community experience)

**Level 1 — Must have:**
- Leg delta exits: Protect against deep ITM moves
- S/L ratio exit: Catch structural deterioration (sag)
- Time exit: Don't let DCs expire — assignment risk and impossible-to-model slippage

**Level 2 — Recommended:**
- Profit target: Lock in gains (50-100% of debit typical)
- Time action: Add stop loss at 0DTE if trade is losing

**Level 3 — Optional:**
- VIX move exits: Noisy but can catch vol regime shifts
- Some longer-DTE DCs benefit from VIX move down exits
- Some shorter-DTE DCs benefit from VIX move up exits

## Curve Fitting Warning Signs

From community experience:
- If $0.40 S/L filter works but $0.43 doesn't — you're fitting to specific paths
- If one minute of entry time makes a big difference — that's noise
- Test with "the opposite" filter: if removing the filter barely changes results, it may not be doing much
- Filters should have EXPLANATORY POWER — you should know WHY they work
- A baseline DC should be profitable WITHOUT filters. Filters improve, not create, edge.

## Curve Fitting in DCs

From the OO community (DC Workshop, March 2026):

**Amy's principle:** "All backtests are technically fitted to the past, but real curve fitting happens when parameters are tuned to maximize backtest performance on a specific dataset. If exiting on 65 looks great but 60 doesn't, then you've wandered into curve fitting territory."

**The opposite test:** Run the backtest with the OPPOSITE filter. If the results are also good, the filter isn't doing much. If the filtered version is amazing and the opposite is terrible, the filter might be finding real signal — or it might be fitting to a specific market period.

**Filter dependency test:** Remove one filter at a time. If removing a single filter collapses the strategy, that filter is doing all the heavy lifting. That's a yellow flag — the edge is narrow and filter-dependent.

**Entry time sensitivity:** If 2:45 PM entry is great but 2:40 PM is terrible, that's noise. Test every 30 minutes around your chosen entry — the results should be roughly similar. Big differences in adjacent time slots = overfitting.

**The debit/premium test:** Higher debits in a DC can mean you're entering in unfavorable term structure. If your filters are selecting for specific premium levels, you might be fitting to a pricing anomaly rather than a structural edge.

**Trade count rule of thumb:** For weekly DCs (~44 trades/year), you need at least 2-3 full years of data (100+ trades) for any statistical claim. Filters that reduce to <25 trades/year should have very strong explanatory power to justify the thin sample.

## Common Failure Modes

1. **Vol crush:** VIX elevated but grinding down. Longs decay faster than expected because vol is declining. Average S/L ratio has been declining across many DCs since 2022.
2. **Event vol:** Pre-event IV pumps up the longs, improving S/L ratio at entry. But if the event causes mean reversion, the trade gets crushed on both sides.
3. **Range-bound + low delta:** The valley of death — underlying stays in the middle of the tent where both peaks are too far away. Low-delta DCs with narrow DTE spreads are most vulnerable.
4. **Deep ITM at expiry:** Slippage can be $2-8 wide on options that are 100+ points ITM. This is why delta exits and S/L ratio exits exist — to avoid this scenario. Hard to model in backtest.
