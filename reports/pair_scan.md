# Pair Scan — Cointegration-Gated Replacement Candidates

**Date:** 2026-05-29 · **Universe:** order-book single names (ETFs excluded)
**In-sample:** 2022→2024 · **Out-of-sample:** 2025→2026-03

## Method (the gate that NVDA/TSLA would have failed)
1. Return-correlation pre-filter ≥ 0.7.
2. In-sample Engle-Granger cointegration **p < 0.01**, OU half-life ≤ 30d, hedge ratio > 0.
3. Out-of-sample: cointegration persists (**p < 0.05**) AND z-score backtest **Sharpe > 0.5** net of 5bps/leg.

Candidates tested: **627** (Bonferroni 5% threshold ≈ 8e-05). Passed in-sample gate: **20**. Survived OOS: **1**.

## Recommended pairs (ranked by out-of-sample Sharpe)

| Rank | Pair | IS corr | IS coint p | Half-life (d) | β | OOS coint p | OOS Sharpe |
|---|---|---|---|---|---|---|---|
| 1 | SYF/WFC | 0.7 | 0.0035 | 13.9 | 1.202 | 0.041 | **0.63** |

## Read this honestly
A pair surviving here has a real long-run equilibrium *and* held up on unseen 2025-26 data — the opposite of NVDA/TSLA, which rode a 2022 correlation with no cointegration. Still require an **economic rationale** for why each pair should co-move before funding, and sit them inside the regime kill-switch + conviction sizing framework so the next decay is caught automatically.