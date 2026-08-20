# Mumbai Indians (2024–2026): A Data-Driven Case Study

A ball-by-ball statistical investigation of Mumbai Indians' IPL performance across the 2024–2026 seasons, covering **bowling root causes**, **batting and role management**, **formal hypothesis testing**, and a **verified recruitment shortlist** for the 2027 auction and trade window.

*This is an independent fan project. Not affiliated with, endorsed by, or produced on behalf of Mumbai Indians, the BCCI, or the IPL.*

---

## Table of Contents

- [Overview](#overview)
- [Research Questions](#research-questions)
- [Dataset Description](#dataset-description)
- [Methodology](#methodology)
- [Key Findings](#key-findings)
- [Project Structure](#project-structure)
- [Requirements](#requirements)
- [How to Run](#how-to-run)
- [Results Summary](#results-summary)
- [Ideas for Extension](#ideas-for-extension)

---

## Overview

This project analyzes **219 Mumbai Indians matches** across 3 seasons (2024–2026) at the ball-by-ball level, splitting every delivery into **powerplay** (overs 1–6), **middle overs** (overs 7–15), and **death overs** (overs 16–20) to test a central question: *why did MI's bowling and batting both peak in 2025 and fall back below 2024 levels in 2026 — and is that story a talent problem, or a role-management problem?*

The project moves through four stages: descriptive phase-split analysis (bowling and batting, separately), individual player case studies grounded in that data, **formal statistical hypothesis testing** that checks which of the descriptively compelling findings actually hold up at conventional significance thresholds, and a **recruitment analysis** that turns the same rigor outward — verifying real prospective signing targets against their own ball-by-ball records in the leagues they actually play in, rather than relying on secondhand reported stats.

---

## Research Questions

- Did MI's 2026 bowling and batting decline trace to a genuine talent shortfall, or to specific, identifiable personnel and role-management decisions?
- Which individual players' role changes (Ashwani Kumar, Naman Dhir, Tilak Varma, Will Jacks, Suryakumar Yadav) show a statistically defensible pattern, versus a compelling-looking but underpowered one?
- Does MI's own three-season bowling record support a venue-specific pace-vs-spin selection rule?
- Is the recurring "excessive bowler rotation hurts MI" narrative supported by a direct correlation test, or only by individual case evidence?
- How much does a descriptive, season-aggregated finding change once it's re-tested at the per-match or per-innings level with real statistical power?
- Do MI's publicly reported recruitment targets hold up once verified against their own real ball-by-ball data, or do secondhand figures overstate (or understate) their actual case?

---

## Dataset Description

### Ball-by-Ball Delivery Data (MI, 2024–2026)

Source: [ritesh-ojha/IPL-DATASET](https://github.com/ritesh-ojha/IPL-DATASET) (GitHub), derived from Cricsheet. One row per legal or illegal delivery, across all matches in scope.

| Column | Description |
|---|---|
| `ID` | Match identifier (joins to match metadata) |
| `Innings` | Innings number within the match |
| `Overs` | Over number (0-indexed: 0–19) |
| `BallNumber` | Ball number within the over |
| `Batter`, `Bowler`, `NonStriker` | Player names |
| `ExtraType` | Wide, no-ball, bye, leg-bye, or none |
| `BatsmanRun`, `ExtrasRun`, `TotalRun` | Runs off the bat, extras, and total for the delivery |
| `IsWicketDelivery` | 1 if a wicket fell on this ball |
| `PlayerOut`, `Kind`, `FieldersInvolved` | Dismissal details |
| `BattingTeam` | Team on strike (bowling team is derived, not provided) |

### Match Metadata (MI, 2024–2026)

One row per match.

| Column | Description |
|---|---|
| `match_number` | Joins to `ID` in the ball-by-ball data |
| `team1`, `team2` | Competing teams |
| `match_date` | Used to derive `season` |
| `toss_winner`, `toss_decision` | Toss outcome |
| `result`, `eliminator`, `winner` | Match outcome |
| `player_of_match` | Match award |
| `venue`, `city` | Location (venue names are normalized in the Bowling notebook — some grounds appear under two labels in the source data) |
| `team1_players`, `team2_players` | Squad lists |

### Player Details (MI, 2024–2026)

One row per player (261 players in the source file). Used for batting/bowling hand in matchup-specific splits (e.g. Will Jacks vs. left-handed batters).

| Column | Description |
|---|---|
| `Name`, `longName`, `battingName`, `fieldingName` | Name variants (Cricsheet naming doesn't always match this file 1:1 — see the Bowling notebook's merge-validation cells) |
| `dob` | Date of birth |
| `battingStyles`, `longBattingStyles` | e.g. `rhb` / right-hand bat |
| `bowlingStyles`, `longBowlingStyles` | e.g. `rm` / right-arm medium |
| `playingRoles` | **Known data-quality issue**: this column duplicates `bowlingStyles` rather than containing an actual role label — flagged and not used for role classification anywhere in this project |
| `espn_url` | ESPNcricinfo profile link |

This file predates three current MI players (Ryan Rickelton, Ashwani Kumar, Corbin Bosch); both analysis notebooks patch their batting/bowling styles in manually after verifying them individually.

### Raw Ball-by-Ball Delivery Data (Prospective Targets, 9 Competitions)

Source: [cricsheet.org](https://cricsheet.org) directly, downloaded live by `Prospective_Players.ipynb` at runtime — SA20, BBL, CPL, T20 Internationals, ILT20, MLC, IPL, The Hundred, and T20 Blast, matching whichever league(s) each shortlisted player actually features in. **Not included in this repository** (see `.gitignore`) — thousands of individual match JSON files, fully re-downloadable via the notebook's own `download_cricsheet_competition()` function rather than redistributed.

---

## Methodology

**Phase-split framework.** Every delivery is tagged by over into powerplay, middle, or death overs. Team- and player-level summary tables (runs, balls faced, wickets, economy, strike rate) are built per season and phase, forming the base for every finding in the two MI analysis notebooks.

**Statistical testing.** The Hypothesis Testing notebook re-examines the ten sharpest descriptive findings with the test appropriate to each data shape, rather than one method throughout:
- **Pearson correlation** for trend questions (e.g. does economy worsen over time)
- **Welch's t-test** for two-group comparisons with unequal variance
- **Mann-Whitney U** where the underlying data (e.g. runs off a single delivery) is too skewed for a t-test's normality assumption
- **Fisher's exact test** for small 2×2 count tables, where chi-square's expected-cell-count assumptions don't hold

Season-level aggregates (n=3) are generally too small for meaningful testing on their own — most tests in this project drop to per-match or per-innings granularity specifically to get real statistical power.

**Recruitment verification.** `Prospective_Players.ipynb` starts from a manually compiled reference table (public reporting on each target's role, contract status, and headline stat), then re-derives real phase-split economy and strike rate for each player directly from Cricsheet — the same rigor standard as the MI-internal notebooks, applied outward to players MI has never fielded.

---

## Key Findings

**Root-cause analysis (MI, 2024–2026):**
- **MI's bowling and batting both followed a "recovery, then relapse" arc** — 2025 was the strongest season across nearly every phase; 2026 fell back below even 2024 in several of them.
- **The 2026 powerplay collapse was a personnel-gap problem**, not a Bumrah/Boult problem — four all-rounders pressed into new-ball overs conceded 13.79 economy, against 9.89 from MI's two genuine specialists (Chahar, Ghazanfar).
- **Ashwani Kumar's death-overs role was cut by 85% in 2026 with no injury involved — and it's statistically confirmed** (Fisher's exact test, *p* = 0.0008), the strongest individual finding in the project.
- **Tilak Varma's death-overs strike rate rose every season** (173.81 → 187.76 → 256.67), independently corroborated by his real-world finisher role at the 2026 T20 World Cup.
- **A venue-level pace-vs-spin selection rule** emerges from MI's own three-season bowling record — spin favored at Wankhede, Chennai, Delhi, and Ahmedabad; pace favored at Hyderabad and Lucknow.
- **Only 2 of 10 formally tested hypotheses were confirmed at *p* < 0.05** (Bumrah's decline; Ashwani Kumar's role cut). Several other descriptively strong patterns — Jacks' economy edge against left-handers, Rickelton's season-over-season jump — did not clear the bar at this sample size and are stated with that caveat throughout.

**Recruitment verification (real Cricsheet data, not secondhand reports):**
- **Otneil Baartman and Jhye Richardson verify as the strongest overseas targets** — both post elite, large-sample economy in *more than one phase*, not just the single skill they were originally scouted for. Richardson in particular: 7.50 powerplay economy across 138 real overs, undercutting the read that his repeated unsold IPL status reflects current form rather than risk-aversion to his fitness/translation history.
- **Gerald Coetzee — the original #3 shortlist priority — has the weakest verified death-overs economy of the entire tier (11.13)** once actually computed, meaningfully demoting him once real data replaced reputation.
- **Gus Atkinson's real white-ball numbers are stronger than his Test-only reputation suggested** — the best death-overs wicket-taking strike rate of anyone verified (7.1 balls/wicket), a genuine finding rather than a hopeful projection.
- **Two originally reported figures didn't survive verification.** Ali Khan's and Ruben Trumpelmann's publicly cited economy figures were both drawn from small or aggregate samples that overstate their real, full-sample death-overs numbers once isolated and computed directly (10.16 and 10.51 respectively, against reported figures of 7.43 and 7.60).
- **Two names remain open items rather than resolved**: Bas de Leede's bowling case is essentially unsupported by the data (a single tracked over), and Brad Evans' name returned no matches at all in the MLC data — both flagged explicitly rather than silently dropped from the shortlist.

---

## Project Structure

```
mi-2024-2026-case-study/
│
├── README.md
├── LICENSE
├── .gitignore
│
├── notebooks/
│   ├── Bowling_Analysis.ipynb        # Phase-split bowling root-cause analysis
│   ├── Batting_Analysis.ipynb        # Phase-split batting and role-management analysis
│   ├── Hypothesis_Testing.ipynb      # Formal significance testing of 10 key findings
│   └── Prospective_Players.ipynb     # Recruitment shortlist, verified against real Cricsheet data
│
├── data/
│   ├── 2024_players_details.csv      # Batting/bowling style reference (see Dataset Description)
│   ├── [ball-by-ball CSV]            # Not included — see Data Source below
│   ├── [match metadata CSV]          # Not included — see Data Source below
│   └── [sa20/, bbl/, cpl/, t20i/, ilt20/, mlc/, ipl/, hundred/, blast/]   # Raw JSON, git-ignored — downloaded at runtime by Prospective_Players.ipynb
│
├── docs/
│   ├── research_plan.md              # Original deliverables-and-hypotheses plan
│   └── build_roadmap.md              # 7-phase build roadmap (data loading → dashboarding)
│
└── presentation/
    └── MI_Stakeholder_Presentation.pptx   # 12-slide summary deck
```

---

## Requirements

```
Python 3.10+
pandas
numpy
scipy
matplotlib
seaborn
requests
jupyter
```

Install dependencies:

```bash
pip install pandas numpy scipy matplotlib seaborn requests jupyter
```

---

## How to Run

1. Clone the repository:
   ```bash
   git clone https://github.com/YOUR-USERNAME/mi-2024-2026-case-study.git
   cd mi-2024-2026-case-study
   ```

2. Install dependencies:
   ```bash
   pip install pandas numpy scipy matplotlib seaborn requests jupyter
   ```

3. Add the ball-by-ball and match-metadata CSVs to `data/` from [ritesh-ojha/IPL-DATASET](https://github.com/ritesh-ojha/IPL-DATASET) (see [Data Source](#data-source) below — `2024_players_details.csv` is already included).

4. Launch Jupyter and run the notebooks:
   ```bash
   jupyter notebook notebooks/Bowling_Analysis.ipynb
   ```
   `Bowling_Analysis.ipynb` and `Batting_Analysis.ipynb` are independent of each other; `Hypothesis_Testing.ipynb` re-derives its own data pipeline and can be run standalone. **`Prospective_Players.ipynb` needs no local data setup at all** — it downloads directly from cricsheet.org when run (nine competition archives, several thousand match files in total — expect it to take a few minutes on first run).

---

## Results Summary

**Hypothesis testing (MI root-cause claims):**

| Hypothesis | Test | *p*-value | Result |
|---|---|---|---|
| Bumrah's economy trended worse over time | Pearson correlation | **0.047** | Confirmed |
| Ashwani Kumar's death-overs share fell 2025→2026 | Fisher's exact | **0.0008** | Confirmed |
| Tilak's 2026 death-overs SR > Hardik's | Mann-Whitney U | 0.055 | Marginal |
| Surya's 2026 powerplay dismissal rate > baseline | Fisher's exact | 0.110 | Suggestive, unconfirmed |
| Jacks concedes more per ball vs. RHB than LHB | Mann-Whitney U | 0.170 | Not confirmed |
| MI's 2026 death economy > 2025 | Welch's t-test | 0.235 | Not confirmed |
| Tilak's death-overs SR trended upward | Pearson correlation | 0.313 | Not confirmed |
| Rickelton's middle-overs SR improved 2025→2026 | Welch's t-test | 0.368 | Not confirmed |
| Bowler churn correlates with worse economy | Pearson correlation | 0.153 | Wrong-signed, not confirmed |
| Batting-position instability correlates with lower SR | Pearson correlation | 0.849 | Not supported |

**Recruitment shortlist (verified against real Cricsheet data):**

| Priority | Name | Category | Why |
|---|---|---|---|
| 1 | Otneil Baartman | Overseas Pace | Elite verified economy in both powerplay (8.12) and death (9.48); free agent |
| 2 | Jhye Richardson | Overseas Pace | Best verified powerplay economy of anyone (7.50, 138-over sample) plus strong death numbers |
| 3 | Gus Atkinson | Overseas Pace | Best death-overs strike rate verified (7.1 balls/wicket) plus genuine batting depth |
| 4 | Alzarri Joseph | Overseas Pace | Deepest, most proven sample (155.7 death overs across 3 competitions) |
| 5 | Praful Hinge | Indian Seamer | Domestic equivalent of Baartman's profile; requires an SRH trade |

Full detail, including why several hypothesis-testing results differ from their season-aggregate headline numbers, and the complete phase-split verification tables for all nine recruitment targets, are in `notebooks/Hypothesis_Testing.ipynb` and `notebooks/Prospective_Players.ipynb` respectively.

---

## Ideas for Extension

The current analysis is a strong foundation. Here are concrete directions to take it further:

### 1. Auction-Value Efficiency Modeling
Merge in real IPL auction prices per player and compute a runs/wickets-per-crore efficiency metric — surfacing which of MI's signings have genuinely outperformed their price, and which haven't.

### 2. League-Wide Comparison
The phase-split framework and hypothesis-testing methodology generalize directly to any IPL franchise. Running the same pipeline across all 10 teams would show whether MI's role-management instability is unusual or league-typical.

### 3. Captaincy Era Comparison
Extend the ball-by-ball window back to 2022–2023 (already partially done for Hardik Pandya's individual trajectory in `Batting_Analysis.ipynb`) to formally compare team-wide tactical patterns under Rohit Sharma's and Hardik Pandya's captaincies.

### 4. A Sharper Positional-Instability Metric
H9 in the Hypothesis Testing notebook found no correlation between raw position-count and strike rate — likely because the metric doesn't distinguish productive flexibility from harmful shuffling. A metric based on *deviation from a player's own best/most-frequent position* would better operationalize the underlying claim.

### 5. Resolve the Two Open Recruitment Items
Brad Evans' name never matched in the MLC data — worth trying alternate name spellings or checking whether his MLC deliveries are covered at all in Cricsheet's current file. Bas de Leede's essentially empty bowling record is worth double-checking against a second competition (he has List A and T20 appearances beyond ILT20 that weren't pulled here).

### 6. Extend Verification to the Full Shortlist
Only 9 of the 20 names compiled in Sections 2–4 of `Prospective_Players.ipynb` have been verified against real data so far (the overseas pace targets, primarily). Extending the same Cricsheet pipeline to the Indian seamer and remaining overseas batting targets would bring the whole notebook to the same standard.

### 7. Interactive Dashboard
Both MI analysis notebooks already export clean summary CSVs (see each notebook's final section) purpose-built for a Tableau or Power BI dashboard — the natural next step for a shareable, non-technical version of these findings.

---

## Data Source

Ball-by-ball and match metadata (MI, 2024–2026): [ritesh-ojha/IPL-DATASET](https://github.com/ritesh-ojha/IPL-DATASET) on GitHub — pre-flattened CSVs derived from [Cricsheet](https://cricsheet.org) (Open Database License — free for any use, including commercial, with attribution), updated daily. Prospective-player verification data: [Cricsheet](https://cricsheet.org) directly, across nine competition archives. Player batting/bowling style reference: compiled from [ESPNcricinfo](https://www.espncricinfo.com) player profiles. Analysis covers Mumbai Indians' IPL seasons 2024–2026, and each recruitment target's own league record as of the research date.
