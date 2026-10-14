# Ultimate Fair Value Gap (FVG) Indicator

A Pine Script **v6** indicator for TradingView that detects 3-candle Fair Value Gaps, **scores** each gap by institutional confluence, and only draws gaps that meet a minimum score — so your chart stays clean. Mitigation state drives both the **color** and the **score**.

> File: [`UltimateFVG.pine`](./UltimateFVG.pine)

## What makes it different

Most FVG indicators draw every gap and clutter the chart. This one treats each gap as a scored object and filters out the noise.

### Scoring engine (0–100)

Each gap earns points from configurable factors:

| Factor | What it checks | Default weight |
|---|---|---|
| **Daily bias alignment** | Prior daily close vs. prior-day midpoint. Bullish FVG in bullish bias (and vice versa) scores. | 25 |
| **Order / Breaker block origin** | The gap-forming displacement came out of an opposite-color candle (last-down-before-up / last-up-before-down). | 20 |
| **Liquidity sweep** | A key level (PDH, PDL, PWH, PWL, or recent swing) was swept just before the gap formed (wick pierces, close returns). | 25 |
| **Displacement strength** | The displacement candle's range vs. ATR. | 15 |
| **Gap size** | Gap height vs. ATR (filters tiny noise gaps). | 15 |

Only gaps scoring **≥ Minimum score** (default 50) are drawn. Raise the threshold to show only the highest-conviction gaps.

Each box shows a **letter grade** (A+/A/B/C/D) and/or the numeric score.

### Mitigation = color + score

| State | Meaning | Visual | Score effect |
|---|---|---|---|
| **Unmitigated** | Price hasn't entered the gap | Full-color bull/bear box | Full score |
| **Partially mitigated** | Price tapped in but hasn't filled through | Dimmed; remaining zone redrawn | Unchanged |
| **Fully mitigated ("tested")** | Price filled through the gap | Muted grey box, stops extending | **Penalty applied** (default −30) |

Optionally **auto-hide** fully mitigated gaps entirely.

### Anti-clutter design

- Score threshold filters out low-conviction gaps.
- Boxes **snap and stop extending** once fully mitigated (no infinite boxes).
- Hard cap on tracked gaps — oldest are recycled.
- Compact **top-right dashboard**: daily bias, active FVG count, A-grade count, current min score.

## Inputs

- **FVG Detection** — min gap size (× ATR), max tracked gaps, extend-right, ATR length.
- **Scoring Engine** — minimum score to draw, per-factor weights, mitigation penalty, displacement/size calibration.
- **Structure** — swing lookback (for OB/sweep), sweep window (bars before gap).
- **Mitigation & Display** — hide mitigated, show score/grade, dashboard, key levels.
- **Colors** — bull, bear, partial, tested, label text.

## Alerts

- `New Bullish FVG`
- `New Bearish FVG`

## Install

1. Open **TradingView → Pine Editor**.
2. Paste the contents of [`UltimateFVG.pine`](./UltimateFVG.pine).
3. **Add to chart**.

## Notes & tuning

- **Daily bias** uses prior daily close vs. prior-day midpoint (as agreed). FVG detection runs on the chart timeframe; bias is always daily.
- Order-block and sweep detection are built in — no second indicator needed. They use lightweight proxies (opposite-color origin candle; wick-pierce-and-return against key levels), which keeps the script fast and self-contained. If you want stricter structure (e.g. confirmed Market-Structure-Shift before a breaker), that can be layered on.
- Start with the defaults, then raise **Minimum score** if you still see too many gaps, or lower it to see more.

## Disclaimer

This is a charting/analysis tool for educational purposes. It is not financial advice. Markets involve risk; test any tool before trading with it.
