---
name: tradeblocks-install
description: >
  Installation and onboarding guide for TradeBlocks. Use this skill when a user says they want
  to install TradeBlocks, asks how to get started with TradeBlocks, types /tradeblocks-install,
  or says they want to learn what TradeBlocks is. This skill does NOT require the TradeBlocks
  MCP to be connected — it guides the user through setup from scratch.
---

# TradeBlocks Installation Skill

This skill guides a brand-new user through everything they need before they can start using
TradeBlocks: understanding what it is, installing the MCP, getting data loaded, and launching
the interactive tutorial. Go slow. One step at a time.

---

## Step 1: What Is TradeBlocks?

Before touching anything technical, give the user a brief plain-English picture of what
they're about to install and why it's worth it:

> "TradeBlocks connects directly to Claude so you can analyze your options trading strategies
> through normal conversation — no spreadsheets, no coding required. It works by reading your
> trade history, which you organize into **blocks** (each block is a folder for one strategy
> or portfolio). Once your data is loaded, you can ask things like 'how is my strategy
> performing?', 'is my edge decaying over time?', or 'how did I do during the 2022 crash?'
> — and I'll run the real analysis and walk you through the results."

Then ask: *"Does that sound like what you're looking for? Ready to get it set up?"*

Wait for confirmation before proceeding.

---

## Step 2: Installation

Read [references/installation.md] and walk the user through setup step by step.

Key rules:
- One step at a time — never dump all the steps at once
- After each step, ask: "Did that work? Any issues?"
- Wait for confirmation before moving forward
- If something goes wrong, troubleshoot patiently before continuing

Don't move to Step 3 until the connection is verified.

---

## Step 3: Get Your Data In

Once the MCP connection is confirmed, help the user get their first trading data loaded.

Read [references/importing-data.md] and walk them through it conversationally.

Once `list_blocks` returns at least one block, say:

> "Your data is in and everything's connected. You're ready to start."

Then continue to Step 4.

---

## Step 4: Install and Launch the Tutorial

The TradeBlocks tutorial skill walks you through the analysis tools one at a time, using
your real data. Here's how to get it:

**Download the tutorial skill:**
> "Head to https://github.com/amyyesand/claude-skills and download the file called
> `tradeblocks-tutorial.zip` from the `tradeblocks-tutorial` folder."

**Install it in Claude Desktop:**
1. Click **Customize** in the left sidebar
2. Click **Skills**
3. Click the **+** button and select **Upload a skill**
4. Select the zip file you just downloaded
5. Once installed, you'll see `tradeblocks-tutorial` in your `/` slash command menu

**Launch it:**
> "Start a new chat and type `/tradeblocks-tutorial` — the tutorial will take it from there
> and walk you through everything one tool at a time."

---

## Tone and Pacing

**Go slow.** This may be the user's first time with an MCP or with any developer-adjacent
tooling. Calm and patient beats fast and complete.

**Celebrate small wins.** "Great — Node.js is installed, you're past the first hurdle!"
goes a long way.

**Never assume comfort with the terminal.** Always offer the MCPB one-click path for users
who hesitate at command-line steps.

**If they get stuck**, troubleshoot one thing at a time. Don't overwhelm them with a list
of five possible causes — start with the most likely one.

---

## Reference Files

- **[references/installation.md]** — Step-by-step MCP installation for both npx and MCPB paths,
  including platform detection, Node.js check, config file editing, and verification.
- **[references/importing-data.md]** — How to export CSVs from Option Omega and get them into
  TradeBlocks, including the manual path for browser users and the `import_csv` tool for
  Desktop users.
