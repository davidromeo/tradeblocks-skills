# TradeBlocks Skills

Agent skills and MCP server for analyzing Option Omega backtests and options trading portfolios. Works with [Claude Code](https://claude.ai/code), [Claude.ai](https://claude.ai), and other [Agent Skills](https://agentskills.io)-compatible tools.

## What's included

**MCP Server** — 50+ tools for querying trade data, running simulations, and analyzing performance. Replays trades with minute-level greeks, tests exit policies, segments by market regime, and more.

**Skills** — Guided workflows that chain the tools together for common analysis tasks:

| Skill | What it does |
|-------|-------------|
| `tradeblocks-dc-analysis` | Double calendar health check — exit attribution, S/L ratio analysis, VIX regime fit, edge decay, curve fit detection |
| `tradeblocks-health-check` | Strategy health check — core metrics, Monte Carlo stress testing, position sizing |
| `tradeblocks-wfa` | Walk-forward analysis — test parameter robustness across rolling time windows |
| `tradeblocks-optimize` | Parameter exploration — sweep entry time, DTE, delta, VIX, and other fields to find patterns |
| `tradeblocks-portfolio` | Portfolio analysis — correlation, diversification, marginal contribution |
| `tradeblocks-risk` | Risk assessment — tail dependence, drawdown attribution, stress scenarios |
| `tradeblocks-compare` | Strategy comparison — side-by-side metrics across blocks |
| `tradeblocks-market-data` | Market data setup — import daily OHLCV, VIX context, and intraday option bars from API, CSV, or DuckDB |

## Install

### Claude Code (plugin)

```bash
/plugin marketplace add davidromeo/tradeblocks-skills
/plugin install tradeblocks@tradeblocks-skills
```

### Claude Code (manual)

```bash
# Clone the repo
git clone https://github.com/davidromeo/tradeblocks-skills.git

# Install the MCP server
cd tradeblocks-skills && npm install

# Load as a plugin
claude --plugin-dir ./tradeblocks-skills
```

### Manual skill installation

Copy any skill folder into your `.claude/skills/` directory:

```bash
cp -r skills/tradeblocks-dc-analysis ~/.claude/skills/
```

## Prerequisites

- **Trade data**: Export your Option Omega backtests as CSV (tradelog format)
- **Market data**: Import SPX/QQQ daily OHLCV and VIX context for regime analysis
- **API key** (optional): Set `MASSIVE_API_KEY` for automatic intraday data fetching during trade replay

## Usage

Once installed, skills are available via `/tradeblocks:skill-name` or Claude will invoke them automatically when relevant.

```
/tradeblocks:tradeblocks-dc-analysis
```

The DC analysis skill will ask which block to analyze, load the strategy profile, and run through exit attribution, regime performance, predictive fields, edge decay, and curve fit detection.

## Data flow

```
Option Omega CSV --> import_csv --> DuckDB --> Tools --> Skills --> Analysis
                                      ^
                                      |
                              Market data (daily/intraday)
                              via import_from_api or import_market_csv
```

## Links

- [TradeBlocks](https://github.com/davidromeo/tradeblocks) — Main application
- [Option Omega](https://optionomega.com) — Options backtesting platform
- [Agent Skills spec](https://agentskills.io) — Open standard for agent skills
