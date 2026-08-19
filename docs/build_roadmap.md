# MI Case Study: 7-Phase Build & Deployment Roadmap

A project plan taking this from raw data to a live, shareable dashboard — sized for a Python-only skillset, building from scratch, with the end goal of a polished link you'd be comfortable sending in a LinkedIn DM to Mumbai Indians.

**One honest framing note before you start:** a franchise's analytics/comms team gets a lot of unsolicited outreach. The thing that makes this worth their 30 seconds isn't volume of analysis — it's one or two genuinely sharp, data-backed insights presented cleanly. Phase 4 (synthesis) and Phase 7 (the message itself) matter more to the outcome than how sophisticated the dashboard is. Build for clarity, not for impressiveness.

---

## Phase 1: Data Audit & Project Setup

**Goal:** since you already have the CSV directly, this phase is really about confirming what you have and setting up structure — not downloading anything.

- **Audit the CSV first, before writing any analysis code:**
  - What columns does it actually have? (`match_id, season, over, ball, batter, bowler, runs, extras, wicket, batting_team, bowling_team` is the ideal set — check what's missing.)
  - What seasons does it cover? Confirm 2026 is actually in there — a lot of the CSV mirrors floating around (Kaggle especially) stop at 2023 or 2024, and that's a silent failure mode that would quietly break every "2023-2026" comparison in your hypotheses.
  - Row count sanity check: an IPL season is ~70-74 matches × ~240 deliveries ≈ ~17,000 rows per season. If your total row count doesn't roughly match `74 × 240 × (number of seasons)`, something's missing or duplicated.
  - If it's missing 2025/2026, or missing columns you need (batter hand, for instance), that's the moment to go back to the Cricsheet `ipl_json.zip` for just the gap-filling piece, rather than redoing everything from scratch.
- Set up a simple folder structure:
  ```
  mi-case-study/
    data/raw/          # your CSV, untouched, as-received
    data/processed/    # your cleaned/feature-engineered version from Phase 2
    notebooks/         # exploration, one notebook per hypothesis cluster
    src/                # reusable functions (metrics, feature engineering)
    dashboard/          # Phase 6 Streamlit app
    reports/            # Phase 4 written outputs
  ```
- Initialize a **public GitHub repo** now, even though you won't deploy until Phase 7 — Streamlit Community Cloud deploys directly from GitHub, so starting here saves friction later.
- If you still want the supplementary league data for the recruitment section (SA20, ILT20, BBL, SMAT — for scouting Baartman, Hinge, etc.), those aren't in your IPL CSV and still need a separate pull from Cricsheet's per-competition zips (`sat_json.zip`, `ilt_json.zip`, `bbl_json.zip`, `sma_json.zip`).

**Tools:** `pandas` (for the audit), `git`
**Deliverable:** a short data-audit note (what seasons/columns you actually have, what's missing) + an initialized repo.

---

## Phase 2: Cleaning & Feature Engineering

**Goal:** turn your existing CSV into the analysis-ready tables the rest of the plan depends on. No JSON parsing needed — this phase is pure `pandas`.

- Load the CSV and standardize dtypes (dates as datetime, season as a clean categorical, team names consistent across years — franchises occasionally get renamed/rebranded in raw data and that silently breaks groupbys).
- Engineer the derived columns everything downstream depends on:
  - `phase` (powerplay/middle/death from the over number)
  - `batting_position` (running count of batters per innings, if not already a column)
  - `batter_hand` / `bowler_hand` — check if your CSV already has this; if not, you'll need a small manual lookup table for MI's and key opposition players, since this is essential for the Jacks-vs-LHB hypothesis specifically.
- Hand-build the two small reference tables that can't come from ball-by-ball data at all: a **captain-by-match** table (Rohit vs Hardik eras) and a **Rohit-availability-by-match** flag — both needed for the H6/H8 hypotheses from your research plan.
- Save the cleaned, feature-engineered version to `data/processed/` — keep your raw CSV in `data/raw/` untouched so you can always regenerate from scratch if you change your mind about a feature.

**Tools:** `pandas`
**Deliverable:** `ball_by_ball_clean.csv`, `match_meta.csv`, `player_meta.csv` — the foundation for every subsequent phase.

---

## Phase 3: Exploratory Analysis & Hypothesis Testing

**Goal:** actually run the H1-H12 hypotheses and D1-D10 deliverables from your research plan.

- Work through them cluster by cluster (bowling/death-overs → batting-order → team-structure), one notebook per cluster.
- For each hypothesis: state it, run the test (mostly `groupby` + `.mean()`, a handful with `scipy.stats.ttest_ind`), and **write a one-line verdict** immediately below the code — supported, not supported, or inconclusive given sample size. This running log is what Phase 4 gets built from, so don't skip it.
- Flag anything that overturns or complicates a claim we made earlier in this conversation from journalism alone (e.g., if the data shows Tilak's No. 3 advantage is smaller than expected, that's worth knowing now, not after you've written the report).

**Tools:** `pandas`, `scipy.stats`, `matplotlib`/`seaborn` for quick visual sanity checks
**Deliverable:** a set of notebooks with results + a running verdict log per hypothesis.

---

## Phase 4: Insight Synthesis & Root-Cause Report

**Goal:** turn Phase 3's scattered findings into the single tightest version of the argument.

- This is the highest-leverage phase for your actual goal. Rewrite the case study's narrative (root causes, personnel recommendations) using your **own computed numbers** in place of the journalism-sourced ones wherever they overlap — that's what turns this from "a fan's summary of articles" into "independent data analysis," which is the actual credibility signal for outreach.
- Structure it short: a 250-word executive summary up top, then 3-4 headline findings, each with one supporting chart, not a wall of tables.
- Be explicit about what your data confirmed, what it complicated, and what it couldn't test (small samples, missing auction-price data, etc.) — a franchise analyst will trust a report more if it shows its own limits.

**Tools:** Markdown, your Phase 3 notebooks, one or two cleaned-up charts
**Deliverable:** a single polished report (`reports/mi_findings.md`) — this becomes both the dashboard's backbone and a standalone PDF you could attach if asked for more detail.

---

## Phase 5: Recruitment & Auction-Value Modeling

**Goal:** turn the qualitative scouting work (Baartman, Hinge, Mhatre, etc.) into a small, reusable scoring layer.

- Build a simple composite scorer for candidate bowlers: something like `(death economy vs league average) + (recent-form weighting) + (availability: free agent vs trade-needed)`. Keep it transparent and simple — a weighted sum you can explain in one sentence beats a black-box model here, both for your own trust in it and for anyone else reading it.
- Since auction prices/contract status aren't in the ball-by-ball data, this phase leans on the manual research table you'd build from IPL official/ESPNcricinfo — keep it as its own small CSV (`data/processed/recruitment_targets.csv`) so it's easy to update as news changes.
- This is explicitly a **stretch/optional phase** — skip or simplify it if time is tight, since Phases 1-4 are what carries the actual analytical weight.

**Tools:** `pandas`, manually compiled CSV
**Deliverable:** a ranked shortlist table, reusable each auction cycle.

---

## Phase 6: Dashboard Build

**Goal:** a live, interactive version of Phase 4's findings — this is the actual thing you'll link to.

**Recommendation: use Streamlit, not a custom web app.** Given you're Python-only with limited general dev experience, Streamlit is the right tool specifically because it needs no HTML/CSS/JS — you write plain Python and it renders as a web app. A minimal working app is genuinely about 15 lines:

```python
import streamlit as st
import pandas as pd

st.title("Mumbai Indians: A Data-Driven Look at 2023-2026")
df = pd.read_csv("data/processed/ball_by_ball.csv")

st.header("Death-overs economy by season")
death = df[df.phase == "death"].groupby("season").runs.mean() * 6
st.line_chart(death)

st.header("Findings")
st.markdown(open("reports/mi_findings.md").read())
```

- Structure it as 3-4 tabs/pages: **Overview** (the headline findings from Phase 4), **Bowling** (death-overs/Bumrah-dependency charts), **Batting Order** (Tilak/Jacks position analysis), and optionally **Recruitment** (Phase 5's shortlist, if you built it).
- Use `st.plotly_chart` or `st.altair_chart` instead of static matplotlib images if you want hover-interactivity — worth the small extra effort since it's the difference between "report" and "dashboard."
- Test locally with `streamlit run dashboard/app.py` before moving to Phase 7.

**Tools:** `streamlit`, `plotly` or `altair`
**Deliverable:** a working local dashboard.

---

## Phase 7: Deployment & Outreach Packaging

**Goal:** a public URL, and a message worth sending.

**Deployment:**
- Push your repo to GitHub (public), then deploy free via **Streamlit Community Cloud** (streamlit.io/cloud) — connect your GitHub repo, point it at `dashboard/app.py`, and it builds and hosts automatically. No server management needed, which matters given your background.
- Add a short `README.md` to the repo explaining what it is in 2-3 sentences — some readers will click through to the code, not just the dashboard.

**Packaging for outreach:**
- Write a one-paragraph, non-technical summary for the top of the dashboard itself (your Phase 4 executive summary works well here) — assume whoever opens the link has 20 seconds before deciding whether to keep reading.
- **The LinkedIn message itself matters more than any technical phase.** A few things worth building in deliberately:
  - Lead with the link and one concrete, specific finding — not a generic "I made a dashboard about MI." E.g., "Built an independent analysis of MI's 2023-2026 death-overs decline using ball-by-ball data — [link]. The Bumrah-dependency numbers surprised me."
  - Keep it short. Two to three sentences plus the link, not a cover letter.
  - Frame it as independent fan analysis, not advice they need — avoid language that implies you know better than their scouting staff. "Thought this might be an interesting outside perspective" lands better than "here's what you're getting wrong."
  - Set your own expectations realistically: large franchises get a lot of this, and a reply is a genuine long shot regardless of quality. The value of doing this well is the finished portfolio piece itself, independent of whether MI responds — worth remembering so the outcome doesn't feel like the only measure of success.

**Deliverable:** a live Streamlit URL, a public GitHub repo, and a drafted (not yet sent) outreach message.

---

## Suggested pacing

This is a real project, not a weekend build — Phases 1-2 are mechanical and fast (a day or two), Phase 3 is where the real time goes (this is genuinely a 1-2 week phase if done carefully), Phase 4 is short but high-value (don't rush it), Phase 5 is optional/skippable under time pressure, and Phases 6-7 are usually faster than people expect once the data and findings already exist — Streamlit deployment in particular is often a same-day task once the app runs locally.
