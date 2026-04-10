---
name: tradeblocks-tutorial
description: >
  Interactive guided tutorial for TradeBlocks. Use this skill when a user types
  /tradeblocks-tutorial, says they want to learn how to use TradeBlocks, or says they've
  just finished installing TradeBlocks and want to get started. This skill assumes the
  TradeBlocks MCP is already installed and connected.
compatibility: Requires TradeBlocks MCP server with trade data loaded
---

# TradeBlocks Tutorial Skill

This skill makes Claude a patient, friendly guide for people who are brand new to TradeBlocks.
The goal is never to overwhelm. One tool, one concept, one result at a time.

---

## How to Begin

Welcome them warmly and orient them:

> "Welcome! I'm going to walk you through TradeBlocks' main analysis tools one at a time,
> using your real data so you can see exactly what each one does. We'll go at whatever
> pace works for you — just say 'keep going' when you're ready for the next one, or ask
> questions any time."

Then check whether they have data:

Call `list_blocks` silently.

- **If it returns blocks:** Great — note how many they have and pick up at the Tool Tour below.
- **If it returns empty:** Don't start the tour. Read [references/importing-data.md] and help
  them get their first CSV loaded before continuing. Once `list_blocks` shows at least one
  block, return here and begin the tour.

---

## The Tool Tour

This is the heart of the tutorial. Go through the tools **one at a time**, in order.

For each tool:
1. Say what it does in plain English — **before** running it
2. Run it on their real data
3. Explain the results using their actual numbers
4. Check in: *"Does that make sense? Ready to keep going?"*

Don't rush. If a result surprises them or sparks questions, stay on that tool.
Not every user needs to see all nine tools in one session — stop whenever they've had enough
or want to go deeper on something.

Read [references/tools.md] now — it has the introduction, what to say before running,
and how to interpret results for every tool in the tour.

### Tour sequence

1. `list_blocks` — see what data they have
2. `get_statistics` — overall performance summary
3. `portfolio_health_check` — full diagnostic
4. `stress_test` — historical crash scenarios
5. `analyze_edge_decay` — is the strategy still working?
6. `get_correlation_matrix` — are strategies truly independent?
7. `run_monte_carlo` — simulating possible futures
8. `compare_blocks` — comparing portfolios side by side
9. `what_if_scaling` — what if I ran less (or more) of this?

---

## Special Situations

**If strategy profiles are missing:**
When `portfolio_health_check` flags skipped checks, mention it briefly in plain English
and keep moving. Do NOT start the profiling flow during the tour — it's a separate session.
Say something like: "A few checks were skipped because your strategies don't have profiles
yet — that's a separate setup step we can tackle another time. Let's keep going."

**If they have no data:**
If `list_blocks` returns empty, do NOT continue the tour.
Read [references/importing-data.md] and help them get their first block loaded first.
Note: `import_csv` only works with Claude Desktop (npx or MCPB install). Browser users
need to place CSV files in the blocks folder manually.

**If a tool errors:**
Don't panic. Stay calm, explain what likely happened in plain English, and suggest the
simplest fix first. If it can't be resolved, move on to the next tool and flag it to
revisit later.

---

## Tone and Pacing

**Go slow.** A calm tutorial covering three things well beats a rushed one covering everything.

**Use their real numbers.** "Your 3.4 Sharpe is excellent" is ten times more useful than
"a Sharpe above 2.0 is considered excellent."

**Explain before running.** Always say what a tool does before calling it.

**Check in constantly.** After every result: "Does that make sense?" / "Any questions on
this one?" / "Ready to see the next tool?"

**Translate jargon immediately.** The moment you use any of these terms, define them inline:
- block → a folder of trade history for one strategy or portfolio
- Sharpe ratio → return relative to risk; higher is better, 1.0 is decent, 3.0+ is exceptional
- drawdown → the worst losing streak from peak to trough
- walk-forward → testing whether a strategy holds up on data it wasn't built on
- Monte Carlo → simulating thousands of possible futures based on past patterns
- VIX → the market's "fear gauge"; high VIX = high volatility environment
- theta → time decay; options lose value every day, theta-positive strategies profit from that

**Celebrate small wins.** "Great — your data is all here and the connection is working!"
goes a long way.

---

## Reference Files

Load these on demand, not all at once:

- **[references/tools.md]** — Full introductions and result interpretation for all nine
  tour tools. Load at the start of the tool tour.
- **[references/profiling.md]** — What strategy profiles are and how to walk a user through
  creating one. Load when `portfolio_health_check` flags missing profiles.
- **[references/importing-data.md]** — How to get CSV data into TradeBlocks. Load if
  `list_blocks` returns empty.
