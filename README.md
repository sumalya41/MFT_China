# MFT China Futures

**An out-of-sample, cost-aware research pipeline for 39 Chinese futures products
(2005-02 → 2021-10), on 5m / 15m / 30m / 60m / daily bars.**

235 recorded cells · 130 figures · 418 truncation/causality checks, 0 failed · report built 2026-11-07

> Most ideas fail out of sample. This repository documents what survives **and what
> doesn't**. Nothing here clears every robustness threshold — the output is a watch
> list, not an order list.

Full slide-by-slide findings (every figure with a *what it is / what it explains*
note): [`report/MFT_China_research_presentation.pdf`](report/MFT_China_research_presentation.pdf)

---

## Executive summary

- **Volatility is forecastable; direction mostly is not.** HAR-RV wins in every
  regime and time frame; linear mean models explain a median **0.0000** of
  out-of-sample return.
- **Dependence is heavy-tailed.** Student-t copula (ν = 12) beats Gaussian;
  lower-tail dependence 0.14 → strategies fail together. One market factor
  explains 41% of daily co-movement.
- **Relationships:** 27 Granger links replicate out of sample; **0 of 10**
  fundamental chains do.
- **One live candidate:** the daily Stage 2 breakout, **long only** —
  83 / 72 / 71 bp net per trade (train / validation / test), test t = 2.3.
  It still fails the full robustness test (see below).
- **Everything faster than ~30 minutes fails costs** (6 bp round trip).
  RSI filters add nothing.
- **Regime at sample end (2021-10-18):** HIGH volatility, BULL breadth
  (Markov state CALM).

## Headline numbers

| Item | Result |
|---|---|
| Best volatility model | HAR-RV, all frames & regimes (walk-forward QLIKE 0.473 vs 0.496 fixed winner) |
| Night-session variance term | −11.5% day-session QLIKE in HIGH vol (t = 4.9) vs −5.9% in LOW — supported |
| Linear mean models | median test R² = 0.0000 |
| Granger replication | 27 links OOS; fundamental chains 0/10 |
| Stage 2 daily long (L20) | 83 / 72 / 71 bp net per trade; test t = 2.3 |
| Stage 2 portfolio | test Sharpe 4.10 vs 1.16 trend benchmark — **not robust** vs EMA200 filter |
| Best ex-ante portfolio recipe | band 4.0 + HRP: test Sharpe +0.46, max DD −21.5% |
| Selection risk | PBO 0.16; deflated-Sharpe hurdle 2.03 over 45 trials |
| VaR (best vol winner) | 1.0% exceedance vs 1.0% nominal |

## Data

- 39 products with sufficient history, 2005-02-01 → 2021-10-18.
- Cleaning (Steps 1–2f): removed 99 calendar-gap rows, 7,858 stale/zero-volume
  bars, 678 short-day rows, 8,040 duplicate timestamps. Kept as real (clipped
  only): 176 large daily moves, 214 one-minute moves over the daily limit.
- Contract rolls back-adjusted; roll jumps removed from returns.
- Every data-quality flag has an explicit rule; **no rule uses future data**.

## Method — how the research is kept honest

- **One calendar split for all frequencies:** train ≤ 2015-02-09 ·
  validation 2015-03-17 → 2018-06-11 · test ≥ 2018-07-11, with a 20-day embargo.
  The test is 792 trading days and is **read once**.
- **Regime-balanced splits** (train_R / val_R): purged interleaved blocks;
  thresholds learned per regime on train_R, never assumed.
- **"Supported"** = same sign in all three periods, |t| above the Bonferroni bar
  for the number of searches, and (for gated rules) better than the parent rule
  in train, validation *and* test.
- **Costs:** 6 bp round trip in every net number; overlapping trades clustered
  into 10-day blocks so they are not counted as independent.
- **Causality:** 418 truncation checks on the real builders, 0 failed; structures
  re-run on negated series; drift removed.
- **Execution model:** bar-based half-spreads (median 1.84 bp) + square-root
  impact law. No order-book data.

## Pipeline (Parts A–H)

| Part | Steps | Content |
|---|---|---|
| A | 1–6 | Data cleaning, coverage, rolls, EDA, intraday shape, correlation, feature checks |
| B | 9–13 | Stationarity (d* ≈ 0.3), mean/vol models, distributions, EVT, copulas, overnight effect |
| C | 14–17 | Granger lead-lag, cointegration, tail dependence, fundamental chains, relationship library |
| D | 18–19 | Factor library, price-volume studies (18c–18v), Stage 2, RSI, opportunity ranks |
| E | 20 | Regimes, structural breaks, regime-dependent factors |
| F | 21–22 | Factor/sector dynamics, drift & decay monitoring (CUSUM alarms) |
| G | 23–25 | Execution model, strategy library, portfolio construction, investor profiles, VaR/ES |
| H | 26 | Opportunity board |

## The one live candidate — daily Stage 2 breakout (long)

Net of 6 bp round trip:

| Cohort | train_R | val_R | test |
|---|---|---|---|
| L20, per trade | 83 bp | 72 bp | 71 bp (t 2.3) |
| L50, per trade | 89 bp | 89 bp | 69 bp (t 2.0) |
| S2 L20 long | 105 bp | 77 bp | 115 bp |
| S2 L20 short | 65 bp | 71 bp | **−24 bp** |

- Portfolio test Sharpe **4.10** vs 1.16 for a 60-day trend benchmark; holds in
  7 of 8 sectors; stays positive dropping any one sector.
- **Why it is not deployable yet:** it does not beat the plain above-EMA200 trend
  filter in all three periods; test t = 2.3 barely clears the 2.24 bar; shorts
  fail on test.
- RSI (18v) adds nothing: 0/8 exit rules, 0/4 entry markers, 0/2 pullback entries
  beat the plain trade.
- **Build target: long-only, range-regime version.**

## Selected negative results (what does not work)

- Direction at any horizon from 5 minutes to 20 days: 380 cells, 2 stable,
  **0 beat the 6 bp cost**.
- 5m close-location reversal: ~0.66 bp break-even cost (mostly bid-ask bounce).
- Learned thresholds: 204 findings pass |t| > 4.15 — 189 about the next *range*,
  only 15 about signed return.
- VCP vs plain Stage 2: beats its parent in 0 of 6 cells.
- Sector rules: 245 tested, 11 stable, **all reversion**; 0/66 sector×regime
  direction cells hold in all periods.
- Topology/Mapper, flow indicators (PMF, QFF, Wasserstein), ECHO: nothing beyond
  trailing volatility / 20-day momentum; the flow trio ≈ 1.98 independent signals.
- Overnight & opening-reversal trades decay on test (opening reversal Sharpe
  1.80 → 1.28 → 0.21); no overnight variant positive on train+validation.
- Tangency portfolio: validation 2.59 → test **0.11** (selection risk in one number).
- Investor profiles: 6/12 admitted on validation, **none profitable on test**.

## Opportunity board (as of 2021-10-18)

- Regime: HIGH volatility · CALM Markov state · BULL breadth.
- Watch-list mix: B carry XS, C2 pair CU~PB, B breadth TS, F2 sector rotation,
  D regime-switching multi-factor.
- Bull candidates: JM, ZN, J, V, FU, P, CF, AL, PP, ZC.
  Bear candidates: M, TF, RM, T, TS, FG, B, RB, AU.
- **A watch list, not an order list.**

## Repository layout

<!-- TODO: adjust to your actual repo -->
