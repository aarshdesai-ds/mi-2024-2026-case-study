# MI Case Study: Deliverables & Testable Hypotheses

A research plan translating the qualitative analysis from this case study into concrete pandas-buildable outputs and falsifiable hypotheses. Built on top of the Cricsheet-based data pipeline from Section 1 (ball-by-ball IPL data, 2023-2026, phase-tagged by over).

**Setup reminder:** load Cricsheet CSVs (via ritesh-ojha/IPL-DATASET or a current Kaggle mirror), engineer a `phase` column (`over<6`=powerplay, `6–15`=middle, `16–19`=death), and you have everything needed for every item below.

---

## Part 1: Core Deliverables (build these first)

These are standalone outputs — tables, charts, or summary stats — that the hypotheses in Part 2 will draw on.

| # | Deliverable | What it shows | Key groupby |
|---|---|---|---|
| D1 | **MI phase-split bowling table, 2023-2026** | Economy & wickets by bowler, season, phase | `bowler, season, phase` |
| D2 | **MI phase-split batting table, 2023-2026** | Strike rate & average by batter, season, batting position | `batter, season, position` |
| D3 | **League-wide XI churn table** | Distinct players used per team per season (tests the "MI rotated more than anyone" claim) | `team, season` → count distinct `batter`/`bowler` |
| D4 | **MI bowling-combination volatility index** | How often MI's XI bowling pair/combination changed match-to-match vs. league average | `team, match_no` → set difference of bowlers used vs. prior match |
| D5 | **Jacks bowling matchup split** | Economy/wickets vs LHB vs RHB, by season | `bowler='Jacks', batter_hand, season` |
| D6 | **Captaincy era comparison table** | Win %, NRR, phase-wise team economy/strike rate under Rohit (2013-23) vs Hardik (2024-26) | `captain, season` |
| D7 | **Bumrah workload-vs-effectiveness trend** | Overs bowled, death economy, home/away split, by season | `bowler='Bumrah', season, venue_type` |
| D8 | **Rickelton vs de Kock opener comparison** | Runs, SR, average, by season and by "Rohit available Y/N" | `batter, season, rohit_played` |
| D9 | **MI death-overs economy vs league average, by season** | MI's overs 16-20 economy relative to the 10-team median each season | `team, season, phase='death'` |
| D10 | **Auction value-efficiency table** | (Runs + wickets×20) per crore spent, by MI signing, by season | Requires merging performance data with auction price data (manual entry from IPL official/ESPNcricinfo, no clean open API for this) |

---

## Part 2: Testable Hypotheses

Each hypothesis below is falsifiable with a specific, simple test — mostly comparisons of means, correlations, or before/after splits. Given your Python background, a groupby + a t-test or a simple correlation (`scipy.stats.ttest_ind`, `.corr()`) is enough for all of these; you don't need anything more complex than that.

### Bowling & death-overs cluster

**H1 — Bumrah dependency.** *MI's death-overs (16-20) economy across the team is significantly worse in matches/seasons where Bumrah bowls fewer death overs than his season average.*
— Test: split matches into "Bumrah bowled ≥2 death overs" vs "<2", compare team death economy (t-test).

**H2 — MI's death-overs economy has structurally worsened relative to the league, not just had a bad year.** *MI's death-overs economy percentile rank (vs. the other 9 teams) has declined year-over-year from 2023 to 2026, not fluctuated randomly.*
— Test: rank MI within the 10-team field each season; check for a monotonic/declining trend rather than noise.

**H3 — Jacks' bowling value is matchup-specific, not general-purpose.** *Jacks' bowling economy and wickets/over are significantly better against left-hand-heavy batting orders (≥3 LHB in the opposition top 6) than right-hand-heavy ones.*
— Test: split his bowling spells by opposition LHB count, compare economy (t-test or simple mean comparison given likely small sample).

**H4 — Ghazanfar's powerplay wicket-taking is a repeatable skill signal, not a small-sample artifact.** *His powerplay strike rate (balls per wicket) is meaningfully better than the league-average spinner's powerplay strike rate, and holds up across different opposition teams (not concentrated in 1-2 matches).*
— Test: compute his powerplay wickets/balls across matches; check variance/concentration (is >50% of his powerplay success from one game?).

**H5 — Santner's small 2026 sample (5 games) doesn't support a reliable form conclusion either way.** *His 2026 economy (8.92) falls within his normal season-to-season variance from prior IPL seasons, i.e., it's not distinguishable from noise given n=5.*
— Test: compare his 2026 economy against the distribution of his own 4-5 game rolling economy in prior seasons — is 8.92 inside or outside that historical spread?

### Batting order & role-management cluster

**H6 — Tilak Varma performs better at No. 3 than at Nos. 4-5.** *His strike rate and average at No. 3 are meaningfully higher than at Nos. 4-5, across 2023-2026.*
— Test: groupby position, compare SR/average; this is the most direct data test of the case study's central batting-order critique.

**H7 — MI's batting-order instability (frequent position changes for the same player) correlates with worse individual output.** *For MI's top 6, seasons/stretches with more distinct batting positions used per player correlate with lower SR/average for that player.*
— Test: for each player-season, compute (a) number of distinct positions batted, (b) SR/average; check correlation across all player-seasons.

**H8 — De Kock outperforms Rickelton specifically when deployed as an injury-triggered substitute, not as a rolling rotation.** *De Kock's SR/average when brought in for an unavailable Rohit is higher than his SR/average in any other selection context.*
— Test: split de Kock's innings by "Rohit unavailable" vs other, compare — this directly tests whether MI's official "backup, activated by Rohit's fitness" model is actually the right usage pattern (as the coach's quote implies) or just a convenient narrative.

**H9 — Rickelton's mid-season "cold spell" (entering double figures only once in a stretch) was a real dip, not noise.** *His rolling 5-innings strike rate shows a statistically identifiable trough in that stretch relative to his season baseline, rather than being within normal innings-to-innings variance.*
— Test: rolling average SR/runs, visually and via a simple z-score check against his season mean/std.

### Team-structure & recruitment cluster

**H10 — MI used more distinct players across the 2026 season than any other team (the "XI churn" claim from Section 2).** *MI ranks 1st or near-1st among the 10 teams in distinct players fielded across the season.*
— Test: D3 above, direct ranking — this is a clean, fully verifiable test of a claim that was previously sourced only from journalism, not your own data.

**H11 — MI's auction spending efficiency has declined 2023→2026.** *The runs+wickets-per-crore-spent for MI's signings has trended downward across seasons, suggesting a recruitment problem compounding the on-field one.*
— Test: D10 above, trend over time; caveat that this needs manually compiled price data since there's no clean open API for historical auction prices.

**H12 — Captaincy transition (Rohit→Hardik) coincides with a structural, not just record-based, change in tactical patterns.** *Bowling-change timing (e.g., how many overs into an opposition batter's innings before a bowling change is made) and death-over allocation patterns differ measurably between the Rohit and Hardik eras, independent of match result.*
— Test: this is the hardest one to operationalize cleanly from ball-by-ball data alone (captaincy "quality" isn't directly labeled) — flag it as a stretch goal requiring more inference/proxy-metric design rather than a straightforward groupby.

---

## Suggested build order

1. **D1-D3, D5, D9** first — these are pure groupby/aggregation, no cross-referencing needed, and directly support H1, H3, H4, H10.
2. **D6-D8** next — require a small manual mapping table (captain-by-season, Rohit-availability-by-match) that you'll need to hand-build since it's not in the raw ball-by-ball data.
3. **D10 and H11-H12 last** — these need auction price data merged in manually and are the most labor-intensive; treat them as a stretch phase once the core bowling/batting-order story (H1-H10) is done.

## Caveat

These hypotheses are framed for **exploratory, descriptive testing** appropriate for a personal case study — not for publication-grade statistical inference. Several (H1, H3, H5) will have small sample sizes (a season of IPL cricket is ~14 matches), so treat p-values loosely and lean on effect size and visual inspection (plot the rolling averages) rather than hard significance thresholds.

---

## Part 3: Exact Data Sources (verified download links)

### Primary: Cricsheet (ball-by-ball, everything you need for Part 1 & 2)

Direct download, no signup, no scraping:

- **IPL, JSON format (recommended):** `https://cricsheet.org/downloads/ipl_json.zip` — 1,243 matches, covers 2008 through the current 2026 season, updated within days of each match.
- **IPL, YAML format (legacy):** `https://cricsheet.org/downloads/ipl.zip`
- **Full downloads index (all competitions, all years):** `https://cricsheet.org/downloads/`

Each zip is one JSON file per match — write a short pandas/json loop to flatten every delivery into rows (`match_id, season, over, ball, batter, bowler, runs, wicket, ...`). That flattened frame is your base for every deliverable in Part 1.

If you also want the overseas-league data we used for scouting (Baartman/SA20, de Leede/ILT20, etc.), the same downloads page has per-competition zips — e.g. `sat_json.zip` (SA20), `ilt_json.zip` (ILT20), `bbl_json.zip` (Big Bash), `cpl_json.zip` (Caribbean Premier League), `mlc_json.zip` (Major League Cricket), `sma_json.zip` (Syed Mushtaq Ali Trophy, useful for the Indian-seamer scouting hypotheses).

### Ready-made CSV mirror (skip the JSON-parsing step)

- **GitHub:** `https://github.com/ritesh-ojha/IPL-DATASET` — same Cricsheet data, pre-flattened into CSVs, updated daily. Good if you'd rather `pd.read_csv()` straight away than write a JSON flattener.

### Kaggle (browser-based, zero local setup)

- **jamiewelsh2/ball-by-ball-ipl** — `https://www.kaggle.com/datasets/jamiewelsh2/ball-by-ball-ipl` — covers 2008-2023 (check freshness before relying on it for 2024-2026; you'll likely need to supplement with Cricsheet directly for the most recent seasons).
- **veeralakrishna/cricsheet-a-retrosheet-for-cricket** — `https://www.kaggle.com/datasets/veeralakrishna/cricsheet-a-retrosheet-for-cricket` — a general Cricsheet mirror across formats, useful if you want Kaggle's in-browser notebook environment rather than downloading locally.
- Search Kaggle directly for "IPL Complete Dataset" for newer, actively-updated 2025/2026-inclusive uploads — several exist but get re-uploaded under different names each season, so check the "last updated" date before trusting one for 2026 data.

### For validation and anything not in ball-by-ball data (prices, retentions, injuries, trades)

These don't have a clean bulk-download API — use them as manual/reference lookups for D6, D10, and the news-driven facts (injuries, trade talk, auction prices) that came up throughout our conversation:

- **ESPNcricinfo Statsguru** — search "ESPNcricinfo Statsguru IPL" for the query-builder tool. Best for spot-checking any aggregate stat (phase splits, career figures) against what you compute from Cricsheet.
- **IPL official site** — `https://www.iplt20.com/` — points tables, squad lists, and individual player profile pages (e.g. `iplt20.com/players/<name>/<id>`) carry official auction prices and season stats, needed for D10's cost-efficiency deliverable.

### Suggested local setup

```python
import pandas as pd
import json, zipfile, urllib.request

# one-time download
urllib.request.urlretrieve("https://cricsheet.org/downloads/ipl_json.zip", "ipl_json.zip")
with zipfile.ZipFile("ipl_json.zip") as z:
    z.extractall("ipl_json")

# flatten one match file as a starting template
with open("ipl_json/<some_match_id>.json") as f:
    match = json.load(f)
# match["innings"] -> list of innings -> "overs" -> list of overs -> "deliveries"
# loop through and build one row per delivery; this is the core of your D1/D2 pipeline
```

From there, everything in Part 1 and Part 2 is `groupby` + `.mean()`/`.sum()` work in pandas — no scraping, no API keys, no cost.
