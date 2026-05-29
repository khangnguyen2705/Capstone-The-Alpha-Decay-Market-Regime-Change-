# Pair Scan v2 — Snooping-Free Protocol

**Date:** 2026-05-29 · **Selection:** in-sample 2022-2024 ONLY · **Hold-out:** 2025->2026-03 (single look)

## Why v2
v1 selected pairs using out-of-sample Sharpe, which contaminated the hold-out. v2 selects on
in-sample evidence only — including a **statistical-significance trading gate with real L1
spreads** — then looks at 2025-26 exactly once, as a report, never as a filter.

## Funnel
- candidates (corr >= 0.7): **627**
- passed in-sample cointegration gate (p<0.01, half-life<=30d, beta>0): **20**
- passed in-sample trading gate (Sharpe t-stat > 2.0, positive in-sample return, real costs): **2**

## Selected (in-sample qualified) — with honest out-of-sample report

| Pair | IS coint p | β | HL(d) | IS Sharpe | IS t-stat | IS ret | OOS Sharpe±SE | OOS p | OOS ret |
|---|---|---|---|---|---|---|---|---|---|
| CVX/DVN | 0.0045 | 0.234 | 23.8 | 1.24 | **2.14** | 50.3% | -1.82±0.91 | 0.224 | -22.3% |
| EOG/SLB | 0.0092 | 0.299 | 19.6 | 1.35 | **2.34** | 74.9% | 0.71±0.91 | 0.623 | 14.0% |

## Out-of-sample verdict
**0 of 2** selected pairs held up on the untouched hold-out (cointegration still p<0.05 AND OOS Sharpe at least one SE above zero).

Read the OOS columns carefully — passing in-sample selection did **not** mean surviving:
- Every selected pair's **out-of-sample cointegration p-value is well above 0.05** — the
  equilibrium that justified the trade had **broken by 2025-26** in each case.
- Any positive OOS return therefore came **without** a live cointegration to anchor it, and the
  OOS Sharpe error bars (±~0.9) straddle zero — statistically indistinguishable from luck.
- This is the **alpha-decay thesis on a clean hold-out**: even pairs with significant,
  cost-aware, genuinely cointegrated in-sample records decayed out of sample.

Note: SYF/WFC — the v1 'winner' — **did not even reach this stage**, because its in-sample
trading Sharpe was negative. The snooping-free protocol correctly rejected it. That is the
whole point: v1's result was a selection artifact.

**Fundable conclusion: none of these clears the bar for live capital.** At most, EOG/SLB
warrants a forward paper-trading watch *contingent on cointegration re-establishing* (p<0.05
sustained) before any allocation.

## Takeaway
A cointegration p-value is necessary, not sufficient. Demanding *statistically significant,
cost-aware, in-sample profitability* before ever touching the hold-out is the discipline that
separates a real edge from a backtest artifact — even when the honest answer is 'nothing qualifies.'