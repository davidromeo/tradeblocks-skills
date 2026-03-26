---
name: profile
description: Create or update a strategy profile from an Option Omega screenshot, verbal description, or block data exploration. Use when profiling a strategy, setting up a new block, importing strategy settings, or when another skill reports a missing profile.
compatibility: Requires TradeBlocks MCP server with trade data loaded
---

# Strategy Profile

Create or update a strategy profile so other skills (DC analysis, health check, portfolio, risk) have the context they need. Profiles capture the trader's intent — structure, entry logic, exit rules, and regime thesis — not just what the data shows.

## When This Skill Triggers

- User says "profile my strategy", "set up a profile", "import my strategy"
- User shares an Option Omega screenshot of a backtest
- Another skill calls `get_strategy_profile` and gets no result
- User wants to update an existing profile after changing settings

## Prerequisites

- TradeBlocks MCP server running
- At least one block with trade data loaded (verify with `list_blocks`)

## Process

### Step 1: Check for Existing Profile

1. **Ask which strategy to profile.** Use `list_blocks` if needed to show available blocks.
2. **Check for an existing profile.** Call `list_profiles` for the target block.
   - If profile exists: load it with `get_strategy_profile`, display the current profile summary, and ask: "Want to update this profile or start fresh?"
   - If updating: go to Step 5 (Update Flow).
   - If no profile: continue to Step 2.

### Step 2: Gather Strategy Details

Ask the user for their strategy details. Lead with the screenshot request — it's the fastest path.

> "Share a screenshot of your strategy from Option Omega — I can pull most of the details from that. You can paste it directly or give me a file path. If you don't have a screenshot, you can describe the strategy instead."

**From an Option Omega screenshot, extract:**

| Field | Where to Find It |
|-------|-------------------|
| `underlying` | Underlying card (e.g., "QQQ", "SPX") |
| `structureType` | Infer from legs — 4 legs with 2 DTE groups = double_calendar, etc. |
| `strategyName` | Title bar (e.g., "Monday 2/4 DC - QQQ") |
| `legs` | Legs card — each row shows S/B/C/P markers, QTY, delta or offset, DTE |
| `exitRules` | Exit card — profit target %, time exit, "Exit using leg delta(s)" |
| `positionSizing` | Entry card — allocation %, max contracts; or Allocation card in portfolio view |
| `keyMetrics` | Performance cards — P/L, CAGR, Sharpe, Sortino, Win%, Max DD, etc. |

**Leg interpretation from the screenshot:**
- `S` (red) = Sell, `B` (green) = Buy — these indicate short/long
- `C` = Call, `P` = Put
- `Delta` column with `Δ` symbol = delta-based strike selection
- `±` symbol = strike offset method
- `DTE` column = days to expiration for that leg

**If no screenshot:** Ask the user to describe:
- Underlying and structure type
- Leg configuration (short/long, put/call, deltas or offsets, DTEs)
- Entry rules (time, day, frequency)
- Exit rules (profit target, stops, time exit)
- Position sizing approach

### Step 3: Targeted Follow-ups

Only ask about fields that are ambiguous or missing from the screenshot. Common gaps:

| Gap | Question |
|-----|----------|
| Exit delta thresholds | "The screenshot says 'Exit using leg delta(s)' — what delta values trigger the exit?" |
| Expected VIX regimes | "What VIX environments do you expect this to perform best in?" (or offer to analyze from data) |
| Strategy thesis | "In one sentence, what's the thesis behind this strategy?" (optional) |
| Block mapping | "What's the block name in TradeBlocks? The test name '[screenshot name]' may differ from the block ID." |

**Do not ask about fields that can be verified from the block data** — handle those in Step 4.

### Step 4: Verify From Block Data

Use the block's trade data to verify and fill fields rather than assuming from structure type.

| Field | How to Verify | Tool |
|-------|---------------|------|
| `greeksBias` | Check actual greeks exposure across trades | `decompose_greeks` |
| `reEntry` | Check if multiple entries occur on the same day | `run_sql`: `SELECT date_opened, COUNT(*) as entries FROM trades.trade_data WHERE block_id = '...' GROUP BY date_opened HAVING COUNT(*) > 1` |
| `capLosses` / `capProfits` | Check actual P/L distribution for structural caps | `get_field_statistics` on `netPl` or `plPct` |
| `closeOnCompletion` | Check if all legs close simultaneously | `run_sql` to examine leg-level close behavior |

After running these checks, ask the user:

> "Want me to run additional exploratory analysis on the block to fill in any remaining details? I can check regime performance, entry patterns, and other characteristics."

If yes, run:
- `analyze_regime_performance` with `segmentBy: "volRegime"` to suggest `expectedRegimes`
- `get_field_statistics` on entry fields to detect implicit filters
- `find_predictive_fields` to surface fields that correlate with outcomes

### Step 5: Update Flow

When updating an existing profile:

1. **Show current profile** — Display a readable summary of what's stored.
2. **Accept new input** — User provides an updated screenshot, describes changes, or says what they modified.
3. **Diff old vs new** — Show what changed:
   ```
   Profit target: 50% → 75%
   Allocation: 11% → 8%
   Added exit rule: delta > 70
   ```
4. **Confirm** — User approves the diff before saving.
5. **Save** — Call `profile_strategy` with the full updated profile (upsert).

### Step 6: Build and Save Profile

Assemble the full profile and present it for confirmation before saving.

**Display the draft profile:**
```
Strategy: [strategyName]
Block: [blockId]
Underlying: [underlying]
Structure: [structureType]
Greeks Bias: [greeksBias]

Legs:
  1. [type] [strike] [expiry] x[quantity]
  2. ...

Entry Filters:
  - [each filter with source tag: market/execution]

Exit Rules:
  - [each rule with type and trigger]

Position Sizing:
  Method: [method]
  Allocation: [allocationPct]%
  Max Contracts: [maxContracts]

Expected Regimes: [list]
Thesis: [thesis]

Key Metrics:
  Win Rate: [value]
  Profit Target: [value]
  ...
```

After user confirms, call `profile_strategy` with all fields.

### Step 7: Multi-Block Cloning

After saving, ask: "Does this strategy exist in other blocks with different sizing or allocation?"

If yes:
1. Help identify the other block(s) — use `list_blocks` if needed.
2. Retrieve the just-created profile with `get_strategy_profile`.
3. Ask what differs — typically `positionSizing` fields:
   - `method`: pct_of_portfolio vs fixed_contracts
   - `allocationPct` / `maxContracts` / `maxContractsPerTrade`
   - `backtestAllocationPct` vs `liveAllocationPct`
4. Clone to each additional block via `profile_strategy`, updating only the block-specific fields.
5. Confirm each clone was saved.

### Step 8: Confirm and Next Steps

After all profiles are saved:

1. Verify by calling `get_strategy_profile` for each block and displaying the stored result.
2. Suggest next steps based on the strategy:
   - Double calendar? → `/tradeblocks:dc-analysis`
   - Want a health check? → `/tradeblocks:health-check`
   - Multiple strategies? → `/tradeblocks:portfolio`
   - Concerned about overfitting? → `/tradeblocks:wfa`

## Interpretation Reference

For details on profile field definitions, Option Omega screenshot parsing, and common structure types, see [references/profile-fields.md](references/profile-fields.md).

## Related Skills

- `/tradeblocks:dc-analysis` - Deep analysis for double calendar strategies (uses profile extensively)
- `/tradeblocks:health-check` - Performance metrics and stress testing
- `/tradeblocks:portfolio` - Multi-strategy correlation analysis (uses profiles via `portfolio_structure_map`)
- `/tradeblocks:risk` - Kelly criterion and tail risk (uses profile for sizing context)
- `/tradeblocks:market-data` - Import market data if regime analysis is needed during profiling

## What NOT to Do

- Don't assume greeks bias from structure type — verify with `decompose_greeks`
- Don't guess exit delta thresholds — "Exit using leg delta(s)" in OO doesn't show the values, ask the user
- Don't skip the block mapping question — test names and block IDs often differ
- Don't create a profile without confirming with the user first
- Don't ask 10 questions when a screenshot answers 8 of them
