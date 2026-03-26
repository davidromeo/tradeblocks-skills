# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Claude Code plugin that bundles an MCP server (`tradeblocks-mcp`) with guided analysis skills for Option Omega backtests and options trading portfolios. The plugin is distributed via the Agent Skills marketplace.

## Architecture

```
.claude-plugin/       Plugin metadata (plugin.json, marketplace.json)
.mcp.json             MCP server config — launches tradeblocks-mcp from node_modules
skills/               8 skill directories, each with SKILL.md + references/
package.json          Minimal wrapper — only dependency is tradeblocks-mcp
```

**Skills are workflow choreographers, not implementations.** Each SKILL.md describes a multi-step analysis workflow that invokes MCP tools in sequence. The actual logic lives in the `tradeblocks-mcp` npm package (50+ tools for trade queries, simulations, and analysis).

**Reference files are interpretation guides.** Each `references/*.md` explains how to read analysis results — thresholds, tables, domain-specific nuance. Skills link to them contextually, not as prerequisites.

## Setup

```bash
npm install   # installs tradeblocks-mcp dependency
```

No build, lint, or test steps — skills are static markdown.

## Skill Structure

Every skill follows this pattern in its SKILL.md:

```yaml
---
name: skill-id
description: one-liner + trigger conditions
compatibility: MCP server requirements
metadata: author, version
---
```

Followed by: Prerequisites → Process (numbered steps with specific MCP tool calls) → Interpretation Reference → Related Skills.

## Plugin Distribution

- `.claude-plugin/plugin.json` — name, version, author for the plugin itself
- `.claude-plugin/marketplace.json` — lists all skills, sets `strict: false` (skills work without full MCP), defines marketplace entry
- Install path: `/plugin marketplace add davidromeo/tradeblocks-skills` then `/plugin install tradeblocks@tradeblocks-skills`

## MCP Server

Configured in `.mcp.json`. Runs `tradeblocks-mcp/server/index.js` from node_modules with `TRADEBLOCKS_DATA_DIR` set to `~/tradeblocks-data`. The server provides DuckDB-backed tools for importing, querying, and analyzing trade data.

## Domain Concepts

- **Blocks** — named strategy containers in the DuckDB database. Most tools require a `blockId` from `list_blocks`.
- **Strategy profiles** — persistent metadata about a strategy's structure, entry filters, and expected regimes. Created by `profile_strategy`, consumed by analysis skills.
- **Curve-fit detection** — a first-class concern, especially in DC analysis. Multiple steps dedicated to identifying overfitting via walk-forward efficiency, parameter sensitivity, and out-of-sample degradation.
- **Tail correlation** — portfolio/risk skills distinguish normal vs tail correlation (Kendall's tau, joint tail dependence) because trading returns violate Gaussian assumptions.
