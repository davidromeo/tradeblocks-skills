# Tool Tour Reference

This file contains everything you need to introduce and explain each tool in the tutorial tour.
For each tool: say the "Before" line, run the tool, then use the "After" guidance to explain results.
Keep it conversational — these are starting points, not scripts.

---

## Tool 1: list_blocks

**What it does:** Lists all the trade data you have loaded in TradeBlocks. Every analysis starts here.

**Before running, say something like:**
> "First, let's see what data you have. TradeBlocks organizes your trade history into **blocks** —
> each one is like a folder for a strategy or portfolio. Let me pull up your list."

**Run:** `list_blocks`

**After — explain the results:**
- Tell them how many blocks they have
- Point to the most interesting ones: biggest trade count is usually the main portfolio
- Explain date ranges in plain terms: "this goes back to May 2022, so about 3 years of history"
- Note any that have a daily log or reporting log: "this one has extra data that unlocks deeper analysis"
- Suggest a block to focus on, or ask which one they want to start with

**Jargon to define if it comes up:**
- "block" = a folder of trade history for one strategy or portfolio
- "reporting log" = your actual broker fills, as opposed to the backtest simulation

---

## Tool 2: get_statistics

**What it does:** Pulls the key performance numbers for a block — win rate, total profit, risk metrics, and more.

**Before running, say something like:**
> "Now let's look at how this portfolio has actually performed. This tool pulls the key numbers —
> think of it like a report card for your trading."

**Run:** `get_statistics` on the chosen block

**After — focus on these three first, then add more if they're curious:**

**Win Rate** — "This is how often your trades close as winners. [X]% means roughly [X in 10]
trades are profitable. For options strategies, anywhere from 40–70% is normal depending on the approach."

**Net P&L** — "Your total profit after commissions across the whole history. [Amount] over [time period]."

**Max Drawdown** — "This is the single most important risk number — the worst losing stretch
you experienced from peak to trough. A [X]% drawdown means at the worst point, the portfolio
was down [X]% from its high before recovering."

**If Sharpe is notable (above 2.0 or below 0.5), explain it:**
"Your Sharpe ratio of [X] measures return relative to risk. Above 1.0 is decent,
above 2.0 is excellent, above 3.0 is exceptional. Yours is [assessment]."

**Check in:** "How does this match what you expected?"

---

## Tool 3: portfolio_health_check

**What it does:** A full diagnostic — checks diversification, tail risk, Monte Carlo projections,
and walk-forward efficiency all at once, and returns an overall verdict.

**Before running, say something like:**
> "Now for the full checkup. This is like an annual physical for your portfolio — it looks at
> six different dimensions of health and flags anything that needs attention. It takes a moment to run."

**Run:** `portfolio_health_check`

**After — lead with the verdict, then explain each flag:**

If HEALTHY: "Good news — everything looks solid. Let me walk you through what it checked..."

If ISSUES_DETECTED: "It found [N] things worth knowing about. None of this means the portfolio
is broken — let me explain each one..."

**Common flags explained in plain English:**

*High correlation between strategies:* "Two of your strategies are moving almost in lockstep —
when one wins, the other wins too, and vice versa. That means you're not getting as much
diversification benefit as you might think. It's worth asking whether both are needed."

*Monte Carlo warning:* "The simulation flagged something, but this one often needs context —
let me check whether it's a real concern or a quirk of how the math works with compounding portfolios."
(Note: if the tool itself flags that dollar-mode is inflated and percentage-mode looks fine, say so clearly.)

*Walk-forward efficiency:* "This checks whether the strategy held up on data it wasn't built on.
[X]% efficiency means in out-of-sample testing, it performed at [X]% of its in-sample rate.
Above 70% is healthy."

**If checks were skipped due to missing profiles:**
Mention it briefly and offer to come back to it — don't dive in during the tour:
> "A few checks were skipped because your strategies don't have profiles yet. A profile is just
> a short label for each strategy that tells TradeBlocks what kind of trade it is and what market
> conditions it's designed for — it unlocks some deeper analysis. Setting them up is a separate
> session though, so let's keep going with the tour for now and you can come back to it later."

Do NOT start the profiling flow during the tool tour. Keep moving.

---

## Tool 4: stress_test

**What it does:** Shows how your portfolio performed during specific named market events —
COVID crash, 2022 bear market, major VIX spikes, etc.

**Before running, say something like:**
> "This next one is really useful — it shows how your portfolio held up during some of the
> worst market events of the past few years. Instead of you having to look up dates, it already
> knows when the COVID crash was, when the 2022 bear market was, and so on."

**Run:** `stress_test`

**After — explain the results:**
- For each scenario, say what the market event was in plain English: "This was when COVID hit
  and the market dropped 34% in about a month"
- Give their actual numbers: "You made $X / lost $X during that period"
- Put it in context: "A loss during COVID is expected — what matters is how quickly you recovered
  and whether the loss was within the range your strategy is designed to handle"
- Highlight any surprisingly strong periods: "Interestingly, you did well during [X] — that's
  because [VIX was elevated / the market was trending / etc.]"

---

## Tool 5: analyze_edge_decay

**What it does:** Checks whether a strategy is still working as well as it used to.
Combines five signals into one report: period trends, rolling performance, walk-forward testing,
regime comparison, and (if available) live vs backtest alignment.

**Before running, say something like:**
> "This is one of the most important questions in trading: is this strategy still working, or
> is the edge fading? Markets change over time, and strategies that worked in 2022 don't always
> work the same way in 2025. This tool checks for signs of decay using five different angles."

**Run:** `analyze_edge_decay`

**After — synthesize the five signals:**
- Period metrics: "Looking year by year, is performance improving, stable, or declining?"
- Rolling metrics: "In the most recent stretch of trades, how does performance compare to the historical average?"
- Walk-forward: "When tested on data it hadn't seen, did the strategy hold up?"
- Regime comparison: "Does it perform differently in different market environments?"
- Live alignment: "If you're trading this live, how closely does reality match the backtest?"

Don't just list the five signals — give an overall read: "Taken together, this looks [healthy /
worth watching / concerning], because..."

---

## Tool 6: get_correlation_matrix

**What it does:** Shows how much your strategies move together.
A score of 1.0 means they're essentially the same trade; 0.0 means they're completely independent.

**Before running, say something like:**
> "When you run multiple strategies, you want them to be somewhat independent — so a bad day for
> one isn't automatically a bad day for all of them. This tool measures that. Think of it like
> checking whether your eggs are actually in different baskets."

**Run:** `get_correlation_matrix`

**After — explain the matrix:**
- For pairs with high correlation (above 0.7): "These two are moving together most of the time.
  It's worth asking whether they're actually different strategies or just variations of the same trade."
- For pairs with low correlation (below 0.3): "These are genuinely independent — great for diversification."
- Give the overall picture: "Most of your strategies are [well-diversified / somewhat correlated / highly correlated]"

---

## Tool 7: run_monte_carlo

**What it does:** Runs thousands of simulations of possible futures, based on your historical
trade patterns shuffled into different orders. Shows the range of likely outcomes — best case,
worst case, and most likely.

**Before running, say something like:**
> "Here's a fun one. This tool takes your historical trades and shuffles them into thousands of
> different possible sequences — kind of like asking 'what if the future looks like the past,
> but in a different order?' It shows you the range of where things might end up."

**Run:** `run_monte_carlo`

**After — focus on three things:**
- Probability of profit: "In [X]% of the simulated futures, the portfolio ends up profitable."
- Median outcome: "The middle-of-the-road projection is [amount/return] over [timeframe]."
- Worst-case drawdown: "Even in the bad scenarios, the simulated max drawdown is around [X]%.
  That gives you a sense of what to mentally prepare for."

If the Monte Carlo flagged unusual results due to dollar vs percentage mode, explain it:
"This result needs a small asterisk — the dollar-based simulation gets skewed by position
sizing growth over time. The percentage-based view is more meaningful here, and that shows [X]."

---

## Tool 8: compare_blocks

**What it does:** Puts multiple blocks side by side so you can compare their key stats directly.

**Before running, say something like:**
> "You have several blocks, and this tool lets you compare them head to head.
> It's useful for seeing at a glance which strategies or portfolios are performing best."

**Run:** `compare_blocks` with 2–4 of their blocks

**After — highlight the most interesting differences:**
- Which has the best Sharpe (best risk-adjusted return)?
- Which has the highest net P&L?
- Which has the lowest drawdown (best risk control)?
- "You can see that [Block A] has a higher win rate, but [Block B] actually makes more money
  because its average winner is much larger."

---

## Tool 9: what_if_scaling

**What it does:** Simulates what would have happened if you'd traded a strategy at a different size.
"What if I'd only run this at half position size?" or "What if I'd doubled this one?"

**Before running, say something like:**
> "The last tool in the tour is a 'what if' machine. It lets you replay history with different
> position sizes — so you can see how the numbers would have changed if you'd been more or less
> aggressive with a particular strategy."

**Run:** `what_if_scaling` — pick a strategy and try 0.5x or 2x

**After:**
- Show the before/after comparison
- "Halving the size would have reduced your max drawdown from [X] to [Y] — but also cut your
  profit from [A] to [B]. It's a tradeoff: less risk, but also less reward."
- "This is useful for calibrating position sizes going forward."

---

## After the Tour

When you've finished the tools the user wants to see, wrap up warmly:

> "That's the core toolkit. You can ask me to run any of these analyses any time —
> just describe what you want to know and I'll figure out which tool to use.
> You can also go deeper on any of these: for example, if the health check flagged something,
> we could investigate that further. What would you like to explore?"
