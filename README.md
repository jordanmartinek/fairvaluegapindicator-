# Ultimate FVG & Order Block Engine

A Pine Script **v6** indicator for TradingView that detects Fair Value Gaps (FVGs) and Order Blocks (OBs), **scores every zone 0-100 from measurable conditions**, and shows only the zones that clear a quality threshold.

> File: [`UltimateFVG.pine`](./UltimateFVG.pine) - copy the whole file into TradingView's Pine Editor and click *Add to chart*.

The score is a **rule-based quality rating**, not a statistical probability or win rate.

> **Status:** written and reviewed carefully against Pine v6 rules, but it has not yet been compiled in TradingView. If the editor reports an error on first paste, send me the exact message and line and I'll fix it.

## What it does

| Area | Implementation |
|---|---|
| **FVG detection** | 3-candle gaps. Height / ATR(14) is tiered Weak / Moderate / Strong / Very strong (all thresholds adjustable, < 0.10 ATR rejected). ATR is built from *clamped* true range so one news bar can't distort ratios. Optional higher-timeframe ATR normalization. |
| **Displacement** | Body/ATR, body/range, close position, opposing wick, consecutive candles, range expansion vs 20-bar average. |
| **Structure** | Internal and external swing breaks (close-based). Classified BOS / CHoCH / MSS (CHoCH + displacement). External breaks weigh more than internal. |
| **Liquidity** | SSL/BSL sweeps of previous day / week high-low, equal highs/lows, external and internal swings. Sweep + MSS earns full credit. |
| **Premium / discount** | External, internal and higher-timeframe dealing ranges (any combination). Range only updates on confirmed, significant swings. Equilibrium tolerance (default 5%) and best-location band (default 30%) adjustable. Discount on several ranges adds confluence. |
| **Order blocks** | Last opposing candle before a displacement leg that broke structure. Rejected if no displacement, short leg, too small/large, closed through before the break, or choppy (efficiency ratio). Full-range / body / hybrid boundaries. |
| **Confluence** | FVG inside OB, OB inside FVG, partial overlap, nearby (ATR distance). Optional merged drawing. |
| **Volume** | Displacement RVOL, quiet pre-move volume, 3-bar confirmation. Can be disabled; its weight is then removed and the score re-normalized. |
| **Lifecycle** | FVG: untouched, first touch, partial, 50%+, filled, invalidated. OB: fresh, first retest, partial, deep, mitigated, invalidated. Invalidation rule selectable (close / wick / body / structure). Zones are never removed on first touch. Optional 50% (consequent encroachment) line. |
| **Multi-timeframe** | Three HTF slots. Each contributes: inside HTF FVG, inside HTF OB, HTF discount/premium, HTF structure. Averaged, so not every timeframe has to agree. |
| **Dashboard** | Structure, HTF bias, dealing range, location, nearest bull/bear FVG + OB with scores, confluence, high-quality count, statistics. |
| **Alerts** | New FVG/OB, confluence, approaching, entering, invalidation, mitigation, 90+ zones - each with its own minimum score. |
| **Diagnostics** | Hover any zone label for the full score breakdown (PASS/FAIL per component, points, location, structure, liquidity, HTF, status). Optional inspector table. |

## Scoring

Weights are configurable and the result is normalized to 0-100 over the components that apply to each zone.

| FVG component | Default | OB component | Default |
|---|---:|---|---:|
| ATR-normalized size | 15 | ATR characteristics | 10 |
| Displacement | 20 | Displacement | 20 |
| Premium / discount | 20 | Premium / discount | 20 |
| BOS / MSS | 15 | BOS / MSS | 15 |
| Liquidity sweep | 10 | Liquidity sweep | 10 |
| Volume | 5 | FVG confluence | 10 |
| HTF alignment | 10 | HTF alignment | 10 |
| Freshness | 5 | Freshness | 5 |

Notes:
- FVGs have no confluence slot in the table, so OB confluence is an extra **bonus** (default +8, capped at 100). OBs also have an optional volume weight (default 0).
- Bullish zones in discount score up to full credit; premium is penalized and equilibrium is penalized more (as specified). Bearish is mirrored.
- Quality classes: 90+ Exceptional, 80-89 High, 70-79 Good, 60-69 Moderate, below 60 hidden by default.

## Presets

Presets set swing lengths, minimum score, max distance, sweep look-back, displacement threshold, HTF slots, and emphasis on certain score components. Any override that is not `0` / `Preset` wins.

| Preset | Run on chart | HTF slots | Emphasis |
|---|---|---|---|
| Scalping | 1m / 3m / 5m | 15m, 1H | Displacement, local structure, tight proximity |
| Intraday | 5m / 15m | 1H, 4H | Premium/discount, liquidity sweeps |
| Swing | 1H / 4H | D, W | Structure, HTF alignment, wide proximity |

Zone detection always runs on the **chart** timeframe; the dashboard warns if the chart doesn't match the preset.

## Non-repainting

- All state (zones, pivots, structure, lifecycle, statistics) changes only on **confirmed** bars; drawings are rebuilt from that state.
- Pivots are only used after their right-hand bars have closed.
- HTF data in the default *confirmed only* mode uses `[1]` with `lookahead_on`, which returns the last **completed** HTF bar. Turning the option off uses the developing HTF bar (it can repaint).
- Previous day/week levels use the same completed-period technique.
- Each zone stores formation time (the candle that created it) and confirmation time (the bar that closed it into existence).

## Alerts

Either create one alert with **Any alert() function call** (detailed messages), or use the individual `alertcondition` entries. Don't enable both for the same event.

## Statistics (not win rates)

For each tracked zone, from its first touch the script measures, over a configurable horizon and normalized by ATR at formation:
- % of zones reached within the horizon,
- % of reached zones whose maximum favorable excursion hit a target,
- average MFE and MAE.

The entry reference is the zone's near edge; there is no stop, target or execution model, so these are descriptive, not win rates.

## Known limitations

- HTF order blocks are a simplified approximation (lowest-low bearish / highest-high bullish candle before an HTF structure break).
- Zones are selected for display on the last bar, so very old zones beyond the stored 150 are dropped.
- TradingView can't click-inspect boxes; use label tooltips (hover) or the inspector table.
- Thresholds are starting points, not universal truths - tune them per symbol (defaults were chosen with NQ/ES-style futures in mind).
