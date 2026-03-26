# TradeBlocks Skills

Agent skills for analyzing Option Omega backtests and options trading portfolios. Works with [Claude Code](https://claude.ai/code), [Claude.ai](https://claude.ai), and other [Agent Skills](https://agentskills.io)-compatible tools.

Requires the [TradeBlocks](https://github.com/davidromeo/tradeblocks) MCP server to be running.

## What's included

Guided workflows that chain TradeBlocks MCP tools together for common analysis tasks:

| Skill | What it does |
|-------|-------------|
| `dc-analysis` | Double calendar health check — exit attribution, S/L ratio analysis, VIX regime fit, edge decay, curve fit detection |
| `health-check` | Strategy health check — core metrics, Monte Carlo stress testing, position sizing |
| `wfa` | Walk-forward analysis — test parameter robustness across rolling time windows |
| `optimize` | Parameter exploration — sweep entry time, DTE, delta, VIX, and other fields to find patterns |
| `portfolio` | Portfolio analysis — correlation, diversification, marginal contribution |
| `risk` | Risk assessment — tail dependence, drawdown attribution, stress scenarios |
| `compare` | Strategy comparison — side-by-side metrics across blocks |
| `market-data` | Market data setup — import daily OHLCV, VIX context, and intraday option bars from API, CSV, or DuckDB |

## Install

### Claude Code (plugin)

First, add the marketplace:

```
/plugin marketplace add davidromeo/tradeblocks-skills
```

Then install the plugin:

```
/plugin install tradeblocks@tradeblocks-skills
```

### Manual skill installation

Copy any skill folder into your `.claude/skills/` directory:

```bash
cp -r skills/dc-analysis ~/.claude/skills/
```

## Prerequisites

- **TradeBlocks MCP server**: Must be installed and running — see [TradeBlocks](https://github.com/davidromeo/tradeblocks)
- **Trade data**: Export your Option Omega backtests as CSV (tradelog format)
- **Market data**: Import SPX/QQQ daily OHLCV and VIX context for regime analysis
- **API key** (optional): Set `MASSIVE_API_KEY` for automatic intraday data fetching during trade replay

## Usage

Once installed, skills are available via `/tradeblocks:<skill>` or Claude will invoke them automatically when relevant.

```
/tradeblocks:dc-analysis
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
