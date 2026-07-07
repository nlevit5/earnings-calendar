# Earnings Calendar — Refresh Playbook

This repo is a single-file earnings calendar for **July 2026** at `index.html`, backed by
`earnings.json`. Each company carries an estimate and, once it reports, actual revenue/EPS
with a beat/miss status. The page computes the scoreboard live from the embedded data.

Run this procedure to update the calendar as companies report (used by both the automated
schedule and any on-demand "refresh earnings" request).

## Steps

1. `cd` into the repo and `git pull` to get the latest.
2. Determine today's date. In `earnings.json`, find every company whose `date` is **on or
   before today** and `reported` is `false` — these are the ones that should have results now.
3. For each such company, use web search (company investor-relations press release first, then
   MarketBeat / Nasdaq / Zacks / Yahoo Finance) to find the **actual reported revenue and EPS**
   for the quarter it just reported. Use the headline adjusted/non-GAAP EPS when that is the
   figure the company and Street lead with.
4. Compare each actual against the `revenueEstimate` / `epsEstimate` already in the record:
   - `beat` if actual > estimate, `miss` if actual < estimate, `inline` if essentially equal.
5. Update the company object in **both** `earnings.json` **and** the embedded `DATA` literal
   inside `index.html` (keep them byte-identical):
   `reported:true, actualRevenue, actualEps, revStatus, epsStatus, reportedDate, surpriseNote`
   (`surpriseNote` = one short phrase, e.g. "Revenue beat, EPS in line; raised FY guidance").
6. **Do not fabricate, and sanity-check magnitudes.** If a company's actuals cannot be confirmed,
   leave `reported:false` and move on. If reported figures fail a plausibility check (e.g. wildly
   outside the company's historical range), do **not** stamp a beat/miss — set `reported:true` but
   leave the numeric fields blank and explain the caveat in `surpriseNote` (see Samsung below).
7. Opportunistically fix any clearly-wrong upcoming date you notice (e.g. a company reporting
   before its quarter could have closed).
8. **Discover major new names.** Also check whether any *genuinely large* company (mega-cap,
   global or US) reported since the last run but is **not yet in the calendar**. If so, add it as
   a new entry (both files) with date, sector, one-line description, and — if already reported —
   actuals + beat/miss. Keep this tight: only true giants, not a long tail of mid-caps.

## Verify before committing
- `"ticker":` count **matches between** `earnings.json` and `index.html` (currently 133; it only
  changes when you add a new name via step 8).
- Both files parse as valid JSON / valid JS literal (no trailing commas, no `undefined`).
- Only fields listed above changed; estimates and descriptions untouched unless correcting an error.

## Commit
- `git add -A && git commit -m "Refresh actuals: <tickers updated> (<date>)" && git push`
- If nothing reported since the last run, make no commit.

## Notes
- Revenue is the headline metric; EPS is secondary.
- Dates for not-yet-reported companies are projected and may shift; correct them when noticed.
- Foreign names use their home listing (e.g. `005930.KS`, `SAP.DE`) and `time:""` (US BMO/AMC
  doesn't apply). Map local reporting metrics sensibly (e.g. Korean "sales" → revenue).
- **Samsung (005930.KS):** currently marked reported for its Jul 7 *preliminary* guidance with
  figures left blank (the ~₩89T operating-profit numbers circulating look erroneous/unverified).
  When Samsung's **detailed Q2 results publish ~Jul 30**, verify against Samsung's official IR and
  fill in the real revenue/operating-profit with a proper beat/miss.
