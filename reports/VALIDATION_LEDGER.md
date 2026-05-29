# Validation Ledger — every headline claim vs. the evidence
**Date:** 2026-05-29 · **Purpose:** put the verification gate *in front of* each claim. No result is
stated as "genuine/real"; each is qualified by the kill-tests it actually survived (and those it did not).

## The kill-test checklist (run before reporting any positive result)
1. **Lookahead** — signals/params/loadings lagged ≥1 bar
2. **Out-of-sample** — tested on an untouched hold-out; selection not done on the test window
3. **Multiple testing** — DSR / honest trial count incl. researcher degrees of freedom
4. **Costs** — real spread/impact, not a flat proxy
5. **Capacity/turnover** — survives realistic turnover and size
6. **Microstructure** — not a mid-price bid-ask-bounce artifact (lag-decay + AC1)
7. **Neutrality** — dollar *and* beta neutral if "neutral" is claimed
8. **Data integrity** — splits/glitches handled; survivorship; external validity
9. **Significance** — t-stat with autocorrelation; not one-window luck

Legend: ✅ pass · ❌ fail · ⚠️ partial / caveated · — not applicable · ❓ not tested

---

## Part A — Daily NVDA/TSLA pairs bot

**Claim A1 (original):** "A profitable pairs bot that decayed."
| 1 LA | 2 OOS | 3 MT | 4 Cost | 5 Cap | 6 Micro | 7 Neut | 8 Data | 9 Sig |
|---|---|---|---|---|---|---|---|---|
| ✅ | ✅ | — | ✅ | ❓ | ❓ | — | ✅ | ⚠️ |
**Verdict: SUPPORTED as a decay story, not as a profitable strategy.** Full-sample Sharpe 0.40, but a
−52% drawdown and post-break Sharpe ~0; the +38.5% is front-loaded in 2022. Lagged positions (no
lookahead), real L1 costs, NVDA 10:1 / TSLA 3:1 splits correctly adjusted. Daily frequency ⇒ bounce
not material but untested.

**Claim A2:** "The pair was never truly cointegrated; decommission (structural break, not drift)."
**Verdict: SUPPORTED.** Rolling Engle-Granger p 0.67→0.44, never durably <0.05; Chow break 2024-01-10;
correlation 0.68→0.42. This is the strongest, best-evidenced conclusion in the project.

---

## Part B — Daily pair scan

**Claim B1 (v1):** "SYF/WFC is a fundable replacement (OOS Sharpe 0.94)."
**Verdict: ❌ RETRACTED — data snooping.** Selection used the OOS window. Failed kill-test 2. Also failed
the later in-sample trading gate (its IS Sharpe was negative). A textbook false positive.

**Claim B2 (v2):** "No fundable daily pair exists in the universe."
| 1 LA | 2 OOS | 3 MT | 4 Cost | 5 Cap | 6 Micro | 7 Neut | 8 Data | 9 Sig |
|---|---|---|---|---|---|---|---|---|
| ✅ | ✅ | ⚠️ | ✅ | ❓ | ❓ | — | ✅ | ⚠️ |
**Verdict: SUPPORTED.** 627→20→2; both survivors (CVX/DVN, EOG/SLB) failed the untouched hold-out
(OOS cointegration p 0.22 / 0.62). Caveats: IS t-stat inflated by warmup zeros (kill-test 9 ⚠️) and
multiple-testing only partially priced (Bonferroni noted, not enforced). Neither caveat resurrects a
pair, so the conclusion holds.

---

## Part C — Intraday Factor-Residual Stat-Arb (IFR)

**Claim C1 (original):** "A genuine, OOS-stable gross edge, Sharpe 1.61, DSR 0.97."
| 1 LA | 2 OOS | 3 MT | 4 Cost | 5 Cap | 6 Micro | 7 Neut | 8 Data | 9 Sig |
|---|---|---|---|---|---|---|---|---|
| ✅ | ✅ | ⚠️ | ✅ | ❌ | ❌ | ❌ | ⚠️ | ⚠️ |
**Verdict: ❌ RETRACTED.** Failed the microstructure test decisively: gross Sharpe collapses 1.93→−0.23
with one extra bar of execution lag, 97% of names show negative lag-1 residual autocorrelation. The
"edge" is **bid-ask bounce**, not capturable reversion. (DSR confirmed the *pattern* is statistically
real — but a real pattern can still be uncapturable bounce; DSR can't tell the difference, which is
why kill-test 6 exists.)

**Claim C2:** "Net Sharpe −8.9; the edge is eaten by transaction costs."
**Verdict: ⚠️ TRUE BUT REFRAMED.** Net is negative, but not because real alpha was taxed — the apparent
gross edge ≈ the spread (it *is* the bounce). "No capturable alpha" is the honest statement.

**Claim C3:** "Factor-neutral / market-neutral residual book."
**Verdict: ❌ FAILED neutrality.** Dollar-neutral (Σw≈0) but net market beta ~0.19 on gross 1.0 —
~19% unhedged directional exposure. Should be called "dollar-neutral," not "factor-neutral."

**Claim C4 (regime-aware bot):** "Drawdown halved (−30.5%→−14%), OOS Sharpe 0.63→0.94."
**Verdict: ⚠️ HEAVILY CAVEATED / SUPERSEDED.** Drawdown reduction is real, but it sits on SYF/WFC — itself
a snooping pick (B1) — and the OOS Sharpe lift was on only ~19 active days (small-sample). Treat as a
demonstration of the kill-switch mechanism, not evidence of an edge.

---

## Data & infrastructure

**Claim D1:** "Found and fixed a split-adjustment bug."
**Verdict: ✅ SUPPORTED.** Naive detector mis-classified earnings crashes (NFLX −37.5%, etc.) as splits;
hardened `_snap_ratio` now adjusts 36 genuine splits, rejects 15 crashes/glitches. Verified clean.

**Claim D2 (infra honesty):** "Institutional level."
**Verdict: ⚠️ ONLY THE REASONING.** Discipline (OOS, DSR, de-bounce, audit) is institutional-grade; the
build is research-prototype (30-min not tick, single factor, no matching engine, possibly-synthetic data).

---

## What remains UNTESTED across the project (honest gaps)
- **Capacity / market impact at size** (kill-test 5) — never modeled; moot only because nothing is fundable.
- **Multiple-testing across ALL researcher DOF** (kill-test 3) — DSR counted 12; true count is larger ⇒ DSR overstated.
- **External validity** (kill-test 8) — data shows synthetic fingerprints (imbalance often exactly 0, scripted 2026 break); treat all numbers as "on this dataset."
- **Daily-pair microstructure** (kill-test 6) — bounce not checked for daily strategies (low materiality at daily frequency).

## Bottom line
After running the checklist against every headline: **no claim of a fundable edge survives.** The
surviving claims are all *negative or diagnostic* — decommission NVDA/TSLA, no fundable daily pair,
intraday signal is bounce, the split bug is fixed. The previously over-stated positives (C1, C3, B1)
are retracted or downgraded. Claims and evidence now line up.
