# Intraday Factor-Residual Stat-Arb (IFR) — Results

**Date:** 2026-05-29 · **Frequency:** 30-min bars · **Universe:** 36 liquid names · **Hold-out:** 2025→2026-03

## Headline

- Apparent **Gross Sharpe (pre-cost): 1.61** (IS 1.75 → OOS 1.24), DSR ~0.97, stable across folds.
- **Net Sharpe (after real L1 spread cost): -8.93** (OOS -9.19).
- Annualized turnover ≈ **1177×**. Best net Sharpe across *all* turnover-reduction configs: **-2.15** — never positive.
- ⚠️ **`src/debounce_test.py` shows the gross number is predominantly BID-ASK BOUNCE, not real reversion**: gross Sharpe collapses 1.93→-0.23 with one extra bar of execution lag, and 97% of names have negative lag-1 residual autocorrelation. Honest conclusion: **no capturable alpha.**

## Is the gross edge real? (validation)
- **Deflated Sharpe Ratio = 0.9672** (per-bar SR 0.0325 vs expected-max-under-null 0.0172 across 12 trials, 14,784 bars). DSR near 1.0 ⇒ the gross edge is **not** a multiple-testing artifact.
- **Walk-forward gross Sharpe:** mean 1.86, std 1.11, fraction-positive 1.0 across folds [3.69, 2.5, 1.06, 0.91, 2.46, 0.52]. Stable across time.

![IFR](../figures/intraday_strategy.png)

## What this means (the senior-quant read)
Both apparent edges in this project were illusions — for different reasons:

| | Daily pairs (NVDA/TSLA) | Intraday residual (IFR) |
|---|---|---|
| Apparent edge | weak | strong-looking (1.6) |
| Why it isn't real | relationship **decayed** out-of-sample | **bid-ask bounce**, dies on a 1-bar lag |
| Fundable? | **no** | **no** |

The intraday signal does not *decay*, but that stability was a red herring: it is the ever-present microstructure bounce, not a persistent inefficiency. A spread-*earning* market-maker could harvest the bounce as a market-making rebate, but that is a liquidity-provision business, not the statistical-arbitrage signal this strategy claimed. As a tradable signal, there is nothing capturable here.

## Limitations
- 30-min bars approximate intraday dynamics; true execution alpha needs message-level data + a matching engine.
- Cost = L1 half-spread + small linear impact; a real fill model could be better (passive) or worse (adverse selection).
- Liquid-universe + ≥85% coverage filter imposes survivorship; winsorized returns cap tail/earnings effects.