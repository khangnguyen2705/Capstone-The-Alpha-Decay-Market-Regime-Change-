# DECISION MEMO — NVDA/TSLA Pairs Bot

**To:** Head of Statistical Arbitrage / Risk Committee
**From:** Quant Research, StatArb desk
**Date:** 2026-05-29
**Re:** Tweak vs. Decommission — NVDA/TSLA mean-reversion bot
**Recommendation:** **DECOMMISSION. Halt new entries today; flatten the residual book into month-end.**

---

## The call

Kill the bot. This is a **structural regime change**, not a parameter problem, and you do not
re-tune your way out of a regime change.

## Why (the trigger that fired)

The strategy assumes NVDA and TSLA mean-revert around a stable long-run relationship. That premise
has failed three independent tests at once:

1. **No cointegration to revert to.** Rolling Engle-Granger p-value averaged 0.67 pre-break and 0.44
   post-break — it was *never* durably below 0.05. The trade was riding shared AI/growth correlation,
   not a genuine equilibrium.
2. **The correlation that powered it has halved** — 0.68 (2022) → 0.42 (2025+). NVDA's
   AI/datacenter cycle has decoupled from TSLA's EV/autonomy-plus-regulation story. That divergence
   is *fundamental and persistent*, not a transient dislocation.
3. **The edge is gone in the P&L.** Rolling Sharpe sits at **~0.0**, the bot is in a **−52%**
   drawdown, equity peaked in **early 2023**, and 2025 was net negative. A Chow test dates the break
   to **2024-01-10 (F = 2,518)**.

## Why not tweak?

Recalibration — wider bands, shorter lookback, re-estimated β — is the right tool for *parameter
drift*: a stable relationship that shifted to a new level. That is not what we have. The hedge ratio
is **non-stationary** (it swings to −1.5, turning the hedge into a second long), and there is **no
stable equilibrium to recalibrate toward.** Tuning parameters on a dead relationship just curve-fits
noise and manufactures false confidence. We would be paying spread and risking capital to harvest a
mean reversion the market no longer offers.

## Cost of being wrong, both ways

- **Keep a dead bot:** continue bleeding the −52% drawdown, carry unhedged single-name tail risk
  (β is unstable), and tie up risk budget. **High, ongoing cost.**
- **Kill a merely-drifting bot:** forfeit recoverable alpha. But the evidence says there is no
  durable alpha to recover — so this risk is low. **The asymmetry favors decommissioning.**

## Actions

1. **Halt new entries immediately.** No fresh `|z|>2` positions.
2. **Flatten the residual book** into month-end on liquidity; do not wait for a `|z|<0.5` mean-revert
   exit that may not come.
3. **Redeploy** the freed risk budget to pairs that still pass a hard cointegration gate.
4. **Archive** the model, code, and this analysis for the post-mortem.

## Re-entry conditions (what would change our mind)

Only reconsider NVDA/TSLA if **all** of the following hold for **3+ consecutive months**:
rolling cointegration p-value **< 0.05**, return correlation back **> 0.6**, mean-reversion
half-life **< 30 days**, and a **stable** (sign-consistent) hedge ratio. Absent that, the pair stays
retired.

## Process upgrade

Adopt a **hard cointegration gate** at allocation time for every pairs strategy (Engle-Granger
p < 0.05 sustained, half-life < ~30d). Had that gate been in place, this bot would never have been
funded on the strength of a 2022 correlation alone — and we would have avoided the three-year bleed.
