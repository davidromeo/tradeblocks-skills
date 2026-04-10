# TradeBlocks MCP Installation Guide

Walk the user through these steps ONE AT A TIME. After each step, ask:
"Have you completed that? Did it work?" — and wait for confirmation before moving on.
Never dump all the steps at once.

---

## Before You Start — What Is an MCP?

When the user hasn't encountered this term before, explain it:

> "MCP stands for Model Context Protocol — it's basically a plugin that lets Claude connect
> to external tools and data. Without it, I'm just a chatbot. With the TradeBlocks MCP
> connected, I can directly read and analyze your trade history, run real calculations,
> and explain the results — all through normal conversation. No spreadsheets, no coding."

The TradeBlocks MCP is separate from any TradeBlocks app — it's a small server that
runs on the user's computer and gives Claude access to their trade data.

---

## Step 1: Check Which Platform They're Using

Ask:

> "Are you using the Claude desktop app (something you downloaded and installed on your
> computer), or are you using Claude in a web browser at claude.ai?"

**→ If Claude Desktop:** continue below with Option A or B.
**→ If Claude.ai in a browser:** explain that the MCP needs Claude Desktop to work locally.
  Offer to help them install Claude Desktop first (free download at claude.ai/download),
  then return here to continue.

---

## Step 2: Check for Node.js

The primary installation method uses the terminal and requires Node.js. Ask:

> "Do you have Node.js installed? You can check by opening Terminal (Mac) or
> Command Prompt (Windows) and typing `node --version`. If you see a version number
> like 'v20.x.x', you're good."

**→ If yes:** continue to Option A (npx — recommended).
**→ If no:** direct them to **nodejs.org** to download and install the LTS version,
  then come back here once they see a version number.
**→ If they're not comfortable with the terminal:** skip to Option B (MCPB one-click).

---

## Option A: npx via Terminal (Recommended)

This is the most reliable installation method. Once Node.js is confirmed, we add TradeBlocks
to Claude's config file — this tells Claude where to find the MCP server every time it starts.

Ask: "Do you know where you want to keep your trading data on your computer? It can be
anywhere — something like a 'TradeBlocks' folder on your Desktop, or ~/Trading/backtests.
If you don't have one yet, just create an empty folder now."

Once they have a folder path, guide them to edit Claude's config file:

**On Mac:**
Open this file in any text editor (create it if it doesn't exist):
`~/Library/Application Support/Claude/claude_desktop_config.json`

**On Windows:**
`%APPDATA%\Claude\claude_desktop_config.json`

Add this content — replacing the path with their actual folder:
```json
{
  "mcpServers": {
    "tradeblocks": {
      "command": "npx",
      "args": ["-y", "tradeblocks-mcp", "/path/to/your/trading/folder"]
    }
  }
}
```

The `-y` flag is important — it tells npx to automatically confirm the package download
without waiting for user input. Without it, the connection may silently fail.

If the file already exists with other content, add the `"tradeblocks"` entry inside the
existing `"mcpServers"` block rather than replacing the whole file.

Ask: "Have you saved the config file?"

**Common hiccups to watch for:**
- The file must be valid JSON — no trailing commas, all brackets matched
- The folder path must actually exist on their computer
- On Mac, the `~/Library` folder is hidden by default — in Finder, press
  Cmd+Shift+G and paste `~/Library/Application Support/Claude/` to navigate there

Then: **Fully quit and reopen Claude Desktop** — not just a new window, but a full quit.

Ask: "Have you restarted Claude?"

---

## Option B: MCPB One-Click Installer (No Terminal Required)

If the user isn't comfortable with the terminal, try the one-click installer.

> "There's also a one-click installer that doesn't require the terminal. Head to the
> TradeBlocks GitHub releases page and look for a file ending in `.mcpb`:
> https://github.com/davidromeo/tradeblocks/releases
> Note: if that page isn't loading, the installer may not be available right now —
> in that case we'll need to use the terminal method above."

If they can download the `.mcpb` file:
- Double-click it to run the installer
- It will prompt them to select their trading data folder
- Create an empty folder if they don't have one yet — we'll add data after

Then: **Fully quit and reopen Claude Desktop.**

Ask: "Have you restarted Claude?"

---

## Step 3: Verify the Connection

> "Let me check that everything's working."

Call `list_blocks` silently.

- **If it returns a list (even empty):** "The connection is working — you're all set!"
- **If it errors or says the tool doesn't exist:**
  > "Claude can't reach TradeBlocks yet. Let's check:
  > Did you fully quit Claude and reopen it?
  > Does the folder path in your config actually exist?
  > Is the JSON in the config file valid — no missing commas?"
  Walk them through troubleshooting one step at a time.

---

## Step 4: Get Your Data In

Once the connection is verified, say:

> "The connection is working! Now let's get your trading data loaded.
> Do you have strategy exports from Option Omega as CSV files?"

Then read [importing-data.md] and walk them through it.

---

## Troubleshooting

**"npx: command not found"**
→ Node.js isn't installed or didn't install correctly. Go to nodejs.org, download the LTS
  version, install it, restart the terminal, and try `node --version` again.

**"Claude says it doesn't have TradeBlocks tools"**
→ Make sure Claude was fully quit and reopened (Cmd+Q on Mac, not just closing the window)
→ Double-check the config file is valid JSON — paste it into jsonlint.com to verify
→ Confirm the folder path in the config actually exists

**"The .mcpb file didn't do anything"**
→ Claude Desktop must be installed first
→ On Mac: try right-clicking and choosing "Open"
→ If the GitHub page is down, use Option A (npx) instead

**"I can't find the Library folder on Mac"**
→ In Finder: press Cmd+Shift+G, then type `~/Library/Application Support/Claude/`

**"list_blocks returns empty"**
→ The connection works! You just don't have data yet. Read [importing-data.md].

**"The config file already has stuff in it"**
→ Don't replace it — add the `"tradeblocks"` entry inside the existing `"mcpServers"` block.
