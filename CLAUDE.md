\# Project Context: Clustering-Based Sector Rotation Strategy

\## Course & Submission Requirements

\- Course: Financial Analytics and Trading (University of Alberta)

\- Deliverables: .qmd file, self-contained HTML (embed-resources: true), context.md

\- All code must be in R using tidymodels workflow

\- All data via APIs only — no local files, no CSV, no Excel

\- GitHub public repo, daily commits required

\- Python is not permitted

\- Strategy must demonstrate active management value beyond holding SPY

\## Strategic Framing

This is an active fund management strategy. The core question the

document must answer is: why should an investor pay for this strategy

instead of just buying SPY? Every section should build toward

demonstrating that the clustering + Kalman signal combination generates

superior risk-adjusted returns versus the passive benchmark.

The strategy operates as a fully active manager:

\- Takes long positions in Growth regime ETFs

\- Takes short positions in Defensive regime ETFs

\- Goes flat in Neutral regime

\- Rebalances monthly at end of month

\- Must demonstrate value-add over SPY benchmark

\## Assets

SPY, XLB, XLC, XLE, XLF, XLI, XLK, XLP, XLRE, XLV, XLU, XLY

(12 assets total — SPY used as benchmark only, not traded)

\## Periods

\- Training: June 2000 to December 2024

\- Testing: January 2025 to March 2026 inclusive

\- Data frequency: daily

\- Rebalancing frequency: monthly (end of month only)

\- Use lubridate::rollback() for end of month dates

\## Core Components

\### 1. Clustering (Primary Regime Detection)

\- Method: K-means clustering

\- Optimal k determined by elbow method in document

\- Cluster quality validated by silhouette analysis

\- Cluster stability tested across sub-periods (pre/post COVID)

\- Monthly cluster assignment drives position direction AND sign

\- Cluster labels interpreted as: Growth / Neutral / Defensive

\- Features: rolling returns, volatility, Sharpe ratio, RSI, momentum

\- All features scaled before clustering

\- Use tidymodels workflow: recipe → prep → bake → kmeans

\- Clustering is computed on training window, applied forward monthly

\- Never look ahead — cluster labels only use data available at

rebalancing date

\### 2. Kalman Filter (Signal Smoothing)

\- Applied to each technical indicator per ETF daily

\- Indicators fed into Kalman filter: RSI, MACD, ROC, Bollinger Band %B

\- Kalman filter tracks smoothed state of each indicator

\- State covariance P\_t tracked for adaptive thresholding

\- Use dlm or KFAS package in R

\- State equation: x\_t = x\_(t-1) + w\_t (random walk state)

\- Observation equation: y\_t = x\_t + v\_t

\- w\_t ~ N(0, W) process noise, v\_t ~ N(0, V) observation noise

\- Tune W and V via maximum likelihood on training data only

\- Extract: filtered state, state covariance P\_t, innovations v\_t

\- Apply separately to each indicator for each ETF

\- Reference: Kalman (1960), Harvey (1989)

\### 3. Delta Entry Signal

\- Delta = first difference of Kalman filtered signal

\- delta\_t = filtered\_signal\_t - filtered\_signal\_(t-1)

\- Entry triggered when: cluster != "Neutral" AND |delta\_t| > theta\_adaptive

\- Direction determined by cluster label:

\- cluster == "Growth" → LONG

\- cluster == "Defensive" → SHORT

\- cluster == "Neutral" → FLAT (no position)

\- Theta is adaptive: scaled by current state covariance P\_t

\- theta\_adaptive = theta\_base \* sqrt(P\_t)

\- When filter uncertainty is high, require larger delta to enter

\- Rationale: enter when signal has directional momentum, not just level

\- For short entries: delta must be negative and below -theta\_adaptive

\### 4. Exit Signals (Two Independent Triggers)

\#### Primary Exit: Kalman Innovation Z-Score

\- Innovation v\_t = observed\_value\_t - predicted\_value\_t

\- Under correct model, innovations should be white noise (zero mean)

\- Compute rolling z-score of innovation sequence

\- Exit when |z\_score(v\_t)| > 2 (innovation spike = regime break)

\- Works symmetrically on both long and short positions

\- Rationale: filter is "surprised" — its model of the world is wrong

\- This is the key differentiator — uses internal Kalman mechanics

\- Reference: Durbin & Koopman (2012) Time Series Analysis by

State Space Methods

\#### Secondary Exit: Cluster Flip

\- Exit long when cluster flips from Growth to Neutral or Defensive

\- Exit short when cluster flips from Defensive to Neutral or Growth

\- Independent of innovation exit — either trigger closes position

\- Cluster flips only evaluated at monthly rebalancing dates

\- Provides regime-level risk management orthogonal to signal exit

\### 5. Portfolio Construction & Rebalancing

\- Weighting scheme: inverse volatility weighted within active cluster

\- Long book: ETFs assigned to Growth cluster, inverse vol weighted

\- Short book: ETFs assigned to Defensive cluster, inverse vol weighted

\- Neutral regime: go flat, hold cash (0% market exposure)

\- Monthly rebalancing mandatory at end of every month

\- Use dynamic rebalance\_portfolio() function pattern from course notes

\- Follow quasinotation pattern (!!!, purrr::map, walk2)

\- Track: portfolio value, allocations per ETF, monthly transactions

\- Short positions expressed as negative allocations

\- No margin call constraints assumed per professor confirmation

\- All 11 sector ETFs eligible for both long and short

\- SPY used as benchmark comparison only

\## Active Management Value Demonstration

The document must explicitly address:

\- Cumulative returns vs SPY (both training and test periods)

\- Sharpe ratio vs SPY

\- Maximum drawdown vs SPY

\- Information ratio (active return / tracking error)

\- Win rate of individual trades

\- Performance during known stress periods (2008, 2020, 2022)

\- Whether shorting the Defensive cluster added value vs long-only

\## Document Structure (QMD Sections)

1\. Problem Framing

\- Why active management over passive SPY

\- Markets as ecosystems mental model

\- Investment mandate and strategy objectives

2\. Mental Model

\- Mermaid flowchart of full decision logic

\- Symmetric long/short structure clearly shown

\- All entry and exit conditions labeled

\- Written decision rules in plain English below diagram

3\. Data

\- API pull via tq\_get() for all 12 assets

\- Training vs test period clearly delineated

\- Price chart indexed to 100 (plotly interactive)

\- Summary statistics table per ETF

4\. Feature Engineering

\- RSI, MACD, ROC, Bollinger Band %B computed daily

\- Each indicator run through Kalman filter per ETF

\- Filtered vs raw signal chart for one ETF illustration

\- Delta computation demonstrated

\- Adaptive threshold theta explained with P\_t

5\. Clustering

\- Elbow plot for optimal k

\- K-means via tidymodels workflow

\- Silhouette analysis for quality

\- Cluster label interpretation (Growth/Neutral/Defensive)

\- Static ggplot cluster visualization

\- gganimate animated cluster membership over training period

\- Stability analysis: pre vs post COVID sub-periods

6\. Signal Generation

\- Delta entry logic with adaptive threshold

\- Innovation sequence and z-score exit

\- Time series chart: filtered signal + delta + entry/exit points

\- Show both long and short signal examples

7\. Decision Rules (Formal)

\- Numbered mathematical statement of every rule

\- Entry conditions for long and short separately

\- Exit conditions for both innovation and cluster flip

\- Weighting formula

\- Rebalancing timing rule

8\. Training Period Backtest (June 2000 - Dec 2024)

\- Cumulative returns vs SPY

\- Long/short contribution breakdown

\- Sharpe ratio, max drawdown, win rate

\- Monthly returns heatmap

\- patchwork combined chart layout

9\. Out of Sample Test (Jan 2025 - Mar 2026)

\- Same metrics as training period

\- Honest assessment of performance

\- Explain any underperformance using cluster stability analysis

\- Compare long-only vs long-short versions

10\. Circle Back

\- What did the strategy learn

\- When does clustering fail (VIX > 40, silhouette < 0.3)

\- Limitations and what would be changed

\- Inversion: what would cause this strategy to fail

\- Connection back to active management value question

\## Above and Beyond Elements

\- Kalman innovation z-score exit (uses internal filter mechanics)

\- Adaptive entry threshold scaled by state covariance P\_t

\- Symmetric long/short with thoughtful position management

\- gganimate animated cluster membership chart across time

\- plotly interactive charts in self-contained HTML

\- Formal decision rules section with mathematical notation

\- Stability analysis comparing pre/post COVID cluster structure

\- Silhouette analysis for cluster quality validation

\- Information ratio and active management metrics vs SPY

\- Monthly returns heatmap using ggplot2

\- Long vs short contribution breakdown in performance section

\## Key R Packages Required

tidyverse, tidyquant, tidymodels, ggplot2, plotly, patchwork,

gganimate, cluster, factoextra, moments, dlm, KFAS,

gt, scales, lubridate, rlang, purrr, zoo, TTR

\## Academic References to Cite

\- Kalman, R.E. (1960) — original Kalman filter paper

\- Harvey, A.C. (1989) — Time Series Models, state space in econometrics

\- Durbin & Koopman (2012) — State Space Methods, innovation sequences

\- Hartigan & Wong (1979) — K-means algorithm

\- Avellaneda & Lee (2010) — stat arb, state space models in finance

\## Grading Risk Areas to Avoid

\- Never load local data files — everything via tq\_get() or API

\- HTML must render charts without re-running — embed-resources: true

\- Commit to GitHub every day you work, not just submission day

\- No Python anywhere in the document

\- Decision rules must be explicit and clear — professor checks this

\- Mental model must be visualized — Mermaid diagram in QMD

\- Must demonstrate active management value vs SPY benchmark

\- Shorting logic must be clearly reasoned — professor grades on thinking

\- No look-ahead bias — never use future data in training signals

\- Kalman filter must be fit on training data only

\## Look-Ahead Bias Rules (Critical)

\- Cluster labels computed using only data up to rebalancing date

\- Kalman filter parameters (W, V) estimated on training period only

\- Applied forward to test period without refitting

\- Delta threshold theta\_base estimated on training period only

\- Innovation z-score rolling window uses only past observations

\- All feature scaling (mean, sd) computed on training data only

then applied to test data using training parameters

\## File Structure for GitHub Repo

project/

├── context.md

├── strategy.qmd

├── strategy.html

└── README.md

\## Notes for Claude Code

\- Always use namespace notation: dplyr::filter(), ggplot2::ggplot()

\- Follow tidymodels workflow: recipe() %>% step\_\*() %>% prep() %>% bake()

\- Monthly rebalancing: use lubridate::rollback() for end of month dates

\- Use purrr::map() for iterating over ETFs, not for loops

\- Delta: dplyr::mutate(delta = value - dplyr::lag(value))

\- Short positions: negative weights in portfolio construction

\- All ggplot charts: theme\_minimal() with professional labeling

\- plotly charts via plotly::ggplotly() for interactivity

\- gganimate for cluster membership animation

\- Use gt package for formatted summary tables

\- Kalman filter: fit on training data only, apply to test separately

\- TTR package for RSI, MACD, ROC, BBands computation

\- Inverse vol weighting: weight\_i = (1/vol\_i) / sum(1/vol\_j)

\- For short book: apply negative sign to inverse vol weights

\- patchwork for combining multiple ggplot charts

\- Namespace everything — no bare function calls