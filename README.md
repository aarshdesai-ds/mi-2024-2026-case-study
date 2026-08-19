# Mumbai Indians (2024–2026): A Data-Driven Case Study

A ball-by-ball statistical investigation of Mumbai Indians' IPL performance across the 2024–2026 seasons, covering **bowling root causes**, **batting and role management**, and **formal hypothesis testing** of the sharpest findings.

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

The project moves through three stages: descriptive phase-split analysis (bowling and batting, separately), individual player case studies grounded in that data, and a final pass of **formal statistical hypothesis testing** that checks which of the descriptively compelling findings actually hold up at conventional significance thresholds — and which don't.

---

## Research Questions

- Did MI's 2026 bowling and batting decline trace to a genuine talent shortfall, or to specific, identifiable personnel and role-management decisions?
- Which individual players' role changes (Ashwani Kumar, Naman Dhir, Tilak Varma, Will Jacks, Suryakumar Yadav) show a statistically defensible pattern, versus a compelling-looking but underpowered one?
- Does MI's own three-season bowling record support a venue-specific pace-vs-spin selection rule?
- Is the recurring "excessive bowler rotation hurts MI" narrative supported by a direct correlation test, or only by individual case evidence?
- How much does a descriptive, season-aggregated finding change once it's re-tested at the per-match or per-innings level with real statistical power?

---

## Dataset Description

### Ball-by-Ball Delivery Data

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

### Match Metadata

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

### Player Details

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

---

## Methodology

**Phase-split framework.** Every delivery is tagged by over into powerplay, middle, or death overs. Team- and player-level summary tables (runs, balls faced, wickets, economy, strike rate) are built per season and phase, forming the base for every finding in the two analysis notebooks.

**Statistical testing.** The Hypothesis Testing notebook re-examines the ten sharpest descriptive findings with the test appropriate to each data shape, rather than one method throughout:
- **Pearson correlation** for trend questions (e.g. does economy worsen over time)
- **Welch's t-test** for two-group comparisons with unequal variance
- **Mann-Whitney U** where the underlying data (e.g. runs off a single delivery) is too skewed for a t-test's normality assumption
- **Fisher's exact test** for small 2×2 count tables, where chi-square's expected-cell-count assumptions don't hold

Season-level aggregates (n=3) are generally too small for meaningful testing on their own — most tests in this project drop to per-match or per-innings granularity specifically to get real statistical power.

---

## Key Findings

- **MI's bowling and batting both followed a "recovery, then relapse" arc** — 2025 was the strongest season across nearly every phase; 2026 fell back below even 2024 in several of them.
- **The 2026 powerplay collapse was a personnel-gap problem**, not a Bumrah/Boult problem — four all-rounders pressed into new-ball overs conceded 13.79 economy, against 9.89 from MI's two genuine specialists (Chahar, Ghazanfar).
- **Ashwani Kumar's death-overs role was cut by 85% in 2026 with no injury involved — and it's statistically confirmed** (Fisher's exact test, *p* = 0.0008), the strongest individual finding in the project.
- **Tilak Varma's death-overs strike rate rose every season** (173.81 → 187.76 → 256.67), independently corroborated by his real-world finisher role at the 2026 T20 World Cup.
- **Suryakumar Yadav's 2026 dip traces to the powerplay, not the death overs** — an earlier, less careful read of the same data initially pointed the wrong way before a closer look at his actual dismissals corrected it.
- **A venue-level pace-vs-spin selection rule** emerges from MI's own three-season bowling record — spin favored at Wankhede, Chennai, Delhi, and Ahmedabad; pace favored at Hyderabad and Lucknow.
- **Only 2 of 10 formally tested hypotheses were confirmed at *p* < 0.05** (Bumrah's decline; Ashwani Kumar's role cut). Several other descriptively strong patterns — Jacks' economy edge against left-handers, Rickelton's season-over-season jump — did not clear the bar at this sample size and are stated with that caveat throughout.

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
│   ├── Bowling_Analysis.ipynb      # Phase-split bowling root-cause analysis
│   ├── Batting_Analysis.ipynb      # Phase-split batting and role-management analysis
│   └── Hypothesis_Testing.ipynb    # Formal significance testing of 10 key findings
│
├── data/
│   ├── 2024_players_details.csv    # Batting/bowling style reference (see Dataset Description)
│   ├── [ball-by-ball CSV]          # Not included — see Data Source below
│   └── [match metadata CSV]        # Not included — see Data Source below
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
jupyter
```

Install dependencies:

```bash
pip install pandas numpy scipy matplotlib seaborn jupyter
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
   pip install pandas numpy scipy matplotlib seaborn jupyter
   ```

3. Add the ball-by-ball and match-metadata CSVs to `data/` from [ritesh-ojha/IPL-DATASET](https://github.com/ritesh-ojha/IPL-DATASET) (see [Data Source](#data-source) below — `2024_players_details.csv` is already included).

4. Launch Jupyter and run the notebooks in order:
   ```bash
   jupyter notebook notebooks/Bowling_Analysis.ipynb
   ```
   `Bowling_Analysis.ipynb` and `Batting_Analysis.ipynb` are independent of each other; `Hypothesis_Testing.ipynb` re-derives its own data pipeline and can be run standalone.

---

## Results Summary

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

Full detail, including why several of these differ from their season-aggregate headline numbers, is in `notebooks/Hypothesis_Testing.ipynb`.

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

### 5. Interactive Dashboard
Both analysis notebooks already export clean summary CSVs (see each notebook's final section) purpose-built for a Tableau or Power BI dashboard — the natural next step for a shareable, non-technical version of these findings.

### 6. Extend Hypothesis Testing to the Venue Rule
The pace-vs-spin venue findings in `Bowling_Analysis.ipynb` are descriptive only; formally testing them (e.g. per-match economy by bowler type, by venue) would place them on the same statistical footing as the ten hypotheses already tested.

### 7. Batter-Hand Splits for the Full Bowling Attack
The LHB/RHB matchup analysis currently covers Will Jacks only. Extending it to every MI bowler would test whether matchup-specific effects are a broader, exploitable pattern or specific to his case.

---

## Data Source

Ball-by-ball and match metadata: [ritesh-ojha/IPL-DATASET](https://github.com/ritesh-ojha/IPL-DATASET) on GitHub — pre-flattened CSVs derived from [Cricsheet](https://cricsheet.org) (Open Database License — free for any use, including commercial, with attribution), updated daily. Player batting/bowling style reference: compiled from [ESPNcricinfo](https://www.espncricinfo.com) player profiles. Analysis covers Mumbai Indians' IPL seasons 2024–2026.
