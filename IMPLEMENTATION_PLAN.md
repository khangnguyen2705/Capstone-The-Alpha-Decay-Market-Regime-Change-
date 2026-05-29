# Week 9 Capstone — Alpha Decay (Market Regime Change)
### Implementation Plan v3 — delivered system, from a senior Quant (Jane Street) perspective

> **Arc of the project:** start from one decaying daily pairs bot, prove *why* it died, search the
> universe for a robust replacement under snooping-free discipline, audit our own data for bugs, then
> graduate to an intraday factor-residual strategy that uses the full order book. The capstone theme —
> *nothing lasts forever* — turns out to have **two** distinct failure modes, and we demonstrate both.

---

## 0. Data foundation
- **Source of truth:** `orderbook.parquet` (4.4 GB, 213M rows, L1–L3 minute quotes, 2022-01-03 →
  2026-03-19), covering NVDA, TSLA and ~525 other names.
- **Split handling (hardened):** a >35% one-day drop is adjusted **only** if its implied factor snaps
  to a clean ratio within 8% (plus a multi-ticker glitch guard). This was a *fixed bug* — the naive
  detector wrongly "split-adjusted" real crashes (NFLX −37.5%, etc.). Shared `_snap_ratio` in
  `src/universe.py`, used by every loader. Intraday detection is inherently more robust (real splits
  are overnight gaps; crashes dilute across bars).

## 1. Part A — Decay diagnosis & decommission (daily NVDA/TSLA)
| Component | Module | Result |
|---|---|---|
| Clean daily pair series | `src/load.py` | split-adjusted, 1,056 days |
| Signal + backtest (real L1 costs) | `src/signals.py`, `src/backtest.py` | +38.5% total, Sharpe 0.40, −52% DD |
| Decay metrics + regime diagnostics | `src/metrics.py`, `src/regime.py`, `src/analysis.py` | break dated 2024-01-10 |

**Finding:** the pair was **never durably cointegrated** (Engle-Granger p 0.67→0.44); the "edge" rode
shared AI/growth correlation (0.68→0.42). **Verdict: decommission** — a structural break, not drift.
→ `reports/decay_analysis_report.md`, `reports/decision_memo.md`.

## 2. Part B — Replacement search under snooping-free discipline
| Component | Module | Result |
|---|---|---|
| Universe daily matrix (526×1,056) | `src/universe.py` | cached |
| v1 scanner (selected on OOS — flawed) | `src/scanner.py` | 1 "winner" SYF/WFC — a snooping artifact |
| **v2 scanner (selection in-sample only)** | `src/scanner_v2.py` | 113k→627→20→**2**; both **failed the untouched hold-out** |
| Regime-aware kill-switch bot | `src/regime_bot.py` | drawdown −30.5%→−14.0%, but no real edge |

**Finding:** even significant, cointegrated, profitable-in-sample pairs (CVX/DVN, EOG/SLB) **decayed
out-of-sample**. Clean conclusion: **no fundable daily pair exists** in this universe/timeframe.
→ `reports/pair_scan_v2.md`.

## 3. Part C — Intraday Factor-Residual Stat-Arb (IFR) — the upgrade
Combines the three improvement directions into one strategy (see `STRATEGY_SPEC.md`): use the **full
order book**, trade **factor-neutral residuals** intraday, inside a **risk + validation harness**.

| Layer | Module | Note |
|---|---|---|
| 30-min panel (mid + imbalance + half-spread) | `src/intraday_data.py` | 14,784 bars × 526 |
| Kalman (steady-state/EWMA) factor residuals | `src/factors.py` | adaptive market-beta neutralization |
| Cross-sectional residual-reversal portfolio | `src/portfolio.py` | dollar-neutral, inverse-vol, vol-target, kill-switch, real costs |
| Deflated Sharpe + purged walk-forward | `src/validate.py` | snooping-aware significance |
| De-bounce test (lag-decay + AC1) | `src/debounce_test.py` | distinguishes bounce from reversion |
| Orchestration, figures, report | `src/run_strategy.py` | → `reports/intraday_strategy_report.md` |

**Finding:** an *apparent* gross edge — Sharpe **1.61**, **DSR 0.97**, all folds positive — but the
de-bounce test **rules it out as real alpha**: gross Sharpe collapses **1.93 → −0.23** with one extra
bar of execution lag, and **97% of names show negative lag-1 residual autocorrelation** (the bid-ask
bounce signature). The "edge" is microstructure bounce you cannot capture; net Sharpe is −8.9 because
the apparent edge ≈ the spread. **Honest conclusion: no capturable intraday alpha either.**

## 4. The two failure modes of alpha (the capstone payoff)
| | Daily pairs (NVDA/TSLA) | Intraday residual (IFR) |
|---|---|---|
| Gross edge | weak, **decayed** OOS | strong, **stable** OOS |
| Killed by | **regime change** | **transaction-cost floor** |
| Fundable? | no (signal died) | no (for a cost-paying participant) |

## 5. Engineering notes / known limitations
- All signals lagged ≥1 bar; Kalman loadings lagged → **no lookahead** anywhere.
- IFR uses 30-min bars (research-grade approximation, not message-level HFT); cost = L1 half-spread +
  linear impact proxy; ≥85%-coverage liquid universe imposes survivorship; returns winsorized at ±15%.
- Validation prices in multiple testing via DSR; the daily search uses a strict in-sample-only gate.

## 6. Reproduce
```
python3 src/load.py            # daily pair          -> data/clean/pair_daily.parquet
python3 src/analysis.py        # decay report + figs
python3 src/universe.py        # daily universe
python3 src/scanner_v2.py      # snooping-free pair scan
python3 src/intraday_data.py   # 30-min panel        -> data/clean/intraday_*.parquet
python3 src/run_strategy.py    # IFR: validate + figures + report
```
