# Strategy Specification — Intraday Factor-Residual Statistical Arbitrage (IFR)
**Version:** 1.0 · **Author:** Quant Research (StatArb) · **Date:** 2026-05-29
**Status:** Specification → reference implementation

---

## 1. Motivation
The Week-9 capstone showed that a single daily pairs bot (NVDA/TSLA) decays: the relationship it
bet on stopped existing, and a clean search of the universe found no fundable daily pair. The three
root weaknesses were (a) we discarded 99% of the data by collapsing minute L1–L3 quotes to daily
closes, (b) we bet on one fragile pairwise relationship, and (c) the backtest lacked sizing, a real
cost model, and snooping-aware validation.

**IFR fixes all three in one design.** We change *what we trade*: not a price spread between two
names, but the **factor-neutral idiosyncratic residual of every name**, traded **intraday** on
order-book information, inside a **risk-and-validation harness**.

> **Thesis:** Strip each name to its residual after removing common factors; that residual
> mean-reverts on a short (intraday) horizon; harvest the reversion cross-sectionally across a
> diversified, dollar-neutral, vol-targeted book, timed and de-risked with the order book.

---

## 2. Universe & data
- **Universe:** order-book single names with ≥95% coverage in both train and test windows (~477
  names). ETFs excluded from the tradable set (may be used later as factor proxies).
- **Source:** `orderbook.parquet` (L1–L3 minute quotes, 2022-01-03 → 2026-03-19), split-hardened
  via the shared `_snap_ratio` logic.
- **Frequency:** **30-minute bars** during Regular Trading Hours. This is the tractable intraday
  frequency that still captures intraday reversion (half-lives of bars, not the 14–24 *days* of the
  daily bot) without requiring a message-level matching engine. ~13 bars/day × ~1,056 days.
- **Fields per bar:** mid = (L1_bid+L1_ask)/2; **order-book imbalance** = mean over the bar of
  (L1_bid_sz − L1_ask_sz)/(L1_bid_sz + L1_ask_sz); L1 sizes as a liquidity/impact proxy.

## 3. The tradable object — factor-neutral residual (Improvement #2)
For each name *i* and bar *t*:
1. **Market factor** `f_t` = cross-sectional (equal-weight) mean bar return of the universe.
2. **Time-varying loading** `β_{i,t}` estimated by a **Kalman filter** with a random-walk state
   (`β_{i,t} = β_{i,t-1} + w`, observation `r_{i,t} = β_{i,t} f_t + v`). This is the adaptive hedge
   ratio — it tracks regime change instead of going stale, the direct cure for alpha decay.
3. **Residual return** `e_{i,t} = r_{i,t} − β_{i,t-1} f_t` (loading lagged → no lookahead).

We trade `e`, not prices. A name's residual is what's left after the common move is hedged out, so
the book is structurally market-neutral and diversified across ~477 idiosyncratic bets.

## 4. Signal stack (Improvement #1 — intraday + microstructure)
1. **Statistical reversion (core):** short-horizon residual *reversal* — the cross-sectional
   z-score of each name's trailing-k-bar cumulative residual return. Rich residual (z>0) → short;
   cheap residual (z<0) → long. Raw signal `s_{i,t} = −z_{i,t}`.
2. **Microstructure confirmation (filter):** only take/keep a position when **order-book imbalance
   agrees** with the reversion direction (long only if imbalance ≥ 0, short only if ≤ 0). Filters
   reversions the book is leaning against.
3. **Execution treatment:** at 30-min frequency we model crossing the **L1 half-spread** plus a
   linear **impact** term scaled by trade size / available depth. (True passive-fill execution alpha
   requires message-level data + a matching engine and is out of scope for this backtest; we model
   cost conservatively rather than claim queue rebates.)

## 5. Portfolio construction & risk (Improvement #3)
- **Weights:** `w_{i,t} ∝ s_{i,t}`, lagged one bar. Cross-sectionally **demeaned → dollar-neutral**
  (Σw = 0). Inverse-vol scaled per name. Gross exposure normalized to 1 before vol targeting.
- **Vol targeting:** scale the whole book each bar so trailing realized vol ≈ target (e.g. 10%
  annualized). Caps single-name and gross exposure.
- **Drawdown kill-switch:** reuse the regime overlay concept — when rolling drawdown breaches a
  threshold, scale gross toward zero until recovery. No three-year bleed.

## 6. Cost & P&L model
- Per-bar P&L = `Σ_i w_{i,t-1} · r_{i,t} − costs`.
- Cost = turnover × (L1 half-spread/mid + λ · turnover/depth). λ calibrated, conservative.
- All signals/weights lagged ≥1 bar; Kalman loadings lagged. **No lookahead anywhere.**

## 7. Validation protocol (the fundable gate)
A strategy is accepted **only** if it clears:
1. **Purged & embargoed walk-forward:** train/test splits with a purge gap so train info can't leak
   into test; Sharpe must be **stable across folds**, not driven by one window.
2. **Deflated Sharpe Ratio (DSR):** corrects the observed Sharpe for the **number of trials**,
   non-normal skew/kurtosis, and sample length. Require **DSR > 0** with high confidence — this
   prices in the multiple-testing that produced false winners in the pair-scan v1.
3. **Out-of-sample hold-out:** final single look at 2025→2026-03, never used in selection/tuning.

## 8. Targets & limits
- **Target:** positive, *statistically deflated* Sharpe on the untouched hold-out, low net exposure,
  controlled drawdown. We explicitly accept that the honest outcome may be "no robust edge" — that is
  a valid, fundable conclusion, not a failure.
- **Hard limits:** dollar-neutral (|net| small), gross cap, per-name cap, drawdown kill.

## 9. Key risks & honest limitations
- **Factor misspecification** replaces single-pair fragility — diversified and adaptive, but real.
- **30-min frequency** understates both the intraday alpha *and* the execution difficulty of true
  HFT; this is a research-grade approximation, not a production HFT system.
- **Cost model is a proxy**, not a matching engine; results are indicative, validated by DSR rather
  than taken at face value.
- **Survivorship:** the ≥95%-coverage filter biases toward names that existed throughout.

## 10. Implementation map
| Layer | Module |
|---|---|
| Intraday panel (mids + imbalance) | `src/intraday_data.py` |
| Kalman factor residuals | `src/factors.py` |
| Signal + portfolio + risk + cost | `src/portfolio.py` |
| DSR + purged walk-forward | `src/validate.py` |
| Orchestration, figures, report | `src/run_strategy.py` |

**Build order:** panel → residuals → portfolio backtest → validation → report.
