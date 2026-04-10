# Getting Your Trade Data Into TradeBlocks

Read this file when the user says they have trading data, or after the MCP connection is
verified. Walk them through it conversationally — check in at each step.

---

## Step 0: Check What's Already There

Before asking the user anything, call `list_blocks` silently.

- **If it returns blocks:** Their data is already loaded — no importing needed.
  Say: "Your data is already there — I can see [X blocks]. You're all set!"
  Return to the main tutorial flow.

- **If it returns empty:** Continue below to help them get data in.

---

## Important: How You Import Depends on How You Installed

Before diving in, check how the user installed TradeBlocks:

**If they used the MCPB installer or npx (Claude Desktop):**
Claude can import CSV files directly using the `import_csv` tool. This is the easy path.

**If they're using Claude.ai in a browser (not Claude Desktop):**
The `import_csv` tool is NOT available — it requires local filesystem access that the
browser version doesn't have. Instead, they need to place their CSV files directly into
their blocks folder manually, then Claude can read them. Explain this:

> "Since you're using Claude in the browser, I can't import files directly for you.
> But it's easy to do manually — you just need to create a folder for each strategy
> inside your TradeBlocks blocks directory and drop your CSV file in there."

Guide them to:
1. Find their TradeBlocks blocks folder (set during installation — often something like `~/Trading/backtests/`)
2. Create a subfolder named after their strategy (e.g., `iron-condor-spx`)
3. Copy their tradelog CSV into that subfolder and name it `tradelog.csv`
4. Call `list_blocks` — it should now appear

---

## Step 1: Export from Option Omega

If they haven't exported yet, help them do that first:

1. Open Option Omega and find the backtest or strategy they want to analyze
2. Look for **File → Export** or a download/export button in the results view
3. Choose **CSV** format and save somewhere easy to find (Desktop or Downloads)

The exported file is their **tradelog** — a row-by-row record of every trade the backtest ran.

If they're exporting a **portfolio** (multiple strategies), Option Omega may export a single
combined CSV or separate files per strategy — both formats work with TradeBlocks.

Ask: "Do you have a CSV file exported and saved on your computer?"

---

## Step 2: Set Up the Folder Structure

TradeBlocks expects one subfolder per strategy inside the main blocks directory:

```
your-blocks-folder/
  my-iron-condor/
    tradelog.csv
  my-calendar-spread/
    tradelog.csv
```

For users importing via Claude (MCPB/npx), this is handled automatically. For manual imports,
they'll need to create the subfolder themselves.

---

## Step 3: Import the Data

### If import_csv is available (Claude Desktop users):

Ask: "What's the filename, and roughly where did you save it?
Something like 'iron-condor.csv' in your Downloads folder."

Then call `import_csv`:
- `blockName`: what they want to call this strategy/portfolio (e.g., "Iron Condor SPX")
- `csvPath`: the filename or full path they gave you
- `csvType`: "tradelog" for trade history (the default)

TradeBlocks automatically searches Downloads, Desktop, and Documents if only a filename is given.

If the import succeeds, `list_blocks` will now show the new block.

### If import_csv is not available (browser users):

Guide them to place files manually as described above, then call `list_blocks` to confirm
the block appears.

---

## Optional: Daily Log

Some Option Omega exports include a **daily log** — a day-by-day record of portfolio value.
This unlocks more accurate drawdown charts and some additional analysis. If they have one,
import it to the same block:

- `blockName`: same name as above
- `csvPath`: path to the daily log file
- `csvType`: "dailylog"

Or manually: place it in the same subfolder as `dailylog.csv`.

---

## Optional: Reporting Log (Live Trades)

If they've been trading this strategy live and have a broker export, they can import that too
as a **reporting log**. This lets Claude compare their backtest predictions against what
actually happened in live trading — one of TradeBlocks' most valuable analyses.

- `csvType`: "reportinglog"

Or manually: place it in the subfolder as `reportinglog.csv`.

---

## After Importing

Once data is in, call `list_blocks` to confirm the block appears, then return to the
main tutorial flow and continue with the tool tour.
