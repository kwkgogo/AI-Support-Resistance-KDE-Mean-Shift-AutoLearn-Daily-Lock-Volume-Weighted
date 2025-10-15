# AI Support/Resistance (KDE + Mean-Shift) — AutoLearn · Daily-Lock · Volume-Weighted

**Version:** `stable-fix4`  
**License:** MIT License

A production-grade TradingView indicator that models support/resistance via **Kernel Density Estimation** and **Mean-Shift clustering**, with **automatic parameter search**, **daily-timeframe learning lock**, **volume-aware weighting**, and strict **time-window curation** to avoid "ancient price" artifacts.

This project is designed for traders and quants who want robust, interpretable S/R levels derived from statistically grounded density clustering rather than ad-hoc heuristics.

---

## Why this Indicator?

- **Statistically grounded**: KDE + Mean-Shift finds **modes** of price interaction (pivot density).
- **Volume-aware**: pivots with higher relative volume are weighted more.
- **Time-window strictness**: hard cutoff (e.g., last 365 days) eliminates obsolete levels.
- **Auto-learn**: small grid search tunes `K`, kernel bandwidth (as ATR multiple), and temporal half-life.
- **Daily-lock**: learn pivots from the **daily** timeframe even on intraday charts (once-per-day pivot insertion).
- **Performance-safe**: constant-time per-bar consistency checks, recycled line pool, defensive bounds.

---

## Method (High-Level)

1. **Pivot extraction**: classic `ta.pivothigh/ta.pivotlow` with `(lenLeft, lenRight)`.
2. **Windowing**: enforce max capacity and a strict **lookbackDays** cutoff on pivot timestamps.
3. **Weighting**:
   - **KDE kernel**: Gaussian over price distance with adaptive bandwidth `bw = ATR * bwATR`.
   - **Temporal decay**: exponential half-life in **bars** or **days** (when daily-locked).
   - **Volume weight**: `(Volume / EMA(Volume)) ^ volPow`, clamped to `[volFloor, volCap]`.
4. **Clustering**: **Mean-Shift** iteratively moves seeds to density modes; dedup near-duplicates.
5. **Auto-learning** (optional): grid over `{K} × {bwATR} × {halfLife}`, using
   - **Density score**: sum of kernel weights at each center,
   - **Backtest score**: touch-within-band followed by rebound-to-ATR-target within `btLookahead`.
6. **Rendering**: S/R lines, optionally width-scaled by normalized density scores.

---

## Core Parameters

- **Structure**
  - `Pivot Left/Right`: pivot span for local extrema.
  - `Max pivots kept`: cap for buffers (default 600).

- **Learning Scope**
  - `Limit learning by time window`: on/off.
  - `Time window (days)`: e.g., 365 (recommended 180–365).
  - `Lock learning to daily timeframe`: **ON** to learn only from daily pivots (strongly recommended).

- **Volume Weighting**
  - `Use volume weighting`: on/off.
  - `Volume EMA length`: default 20.
  - `Volume weight power`: default 1.2.
  - `Floor/Cap`: guards against outliers.

- **Auto-Learn (Grid Search)**
  - `Enable Auto-Learn`: on/off.
  - `Re-tune every N bars`: default 50.
  - `Mean-Shift iterations`: default 3.
  - Backtest knobs: `btLookahead`, `btBandATR`, `btReboundATR`, and weights `wDensity`, `wBounce`.

- **Visualization**
  - `Support/Resistance colors`.
  - `Base/Max line width`.
  - `Style by score (width)`.

- **Optional UX**
  - `Auto-reduce K when samples are few`: helpful for "first screen" on sparse assets.

---

## Installation (TradingView)

1. Open **TradingView** → **Pine Editor**.  
2. Paste the contents of `indicator.pine` (this repository's script).  
3. Click **Add to chart**.  
4. Configure inputs:
   - **Turn ON**: `Lock learning to daily timeframe`, `Use volume weighting`.
   - Set **Time window** to `180–365` days (typical).
   - Optionally enable **Auto-Learn** for adaptive tuning.

---

## Practical Guidance

- **Daily-lock ON**: avoids multi-intraday duplication of the same daily pivot; stability improves dramatically.
- **Time window**: avoid >365d on assets with regime shifts; stale levels degrade quality.
- **Score-width ON**: thicker lines ≈ stronger historical reaction density.
- **Auto-learn cadence**: set `Re-tune every N bars` to `50–100` for intraday, larger for higher timeframes.
- **First-load sparse data**: enable `Auto-reduce K` if levels don't appear immediately.

---

## Performance Notes

- Per-bar backtest is **constant-time**, using a tiny lookahead and the current centers (not full pivot buffers).
- Line pool is **recycled and pruned** to avoid visual artifacts.
- Defensive checks handle empty arrays, small samples, and changing K.

---

## Limitations & Scope

- This indicator **does not output buy/sell signals**; it models price structure (S/R).  
  For trade execution logic, combine with trend/volatility/flow models or build a signal layer on top.
- Grid search is deliberately **small** to keep real-time speed acceptable.

---

## FAQ

**Q: Why daily-lock if I trade intraday?**  
Daily pivots reduce noise, prevent duplicate insertions within the same day, and produce institutional-grade levels.

**Q: Why KDE+Mean-Shift vs k-means?**  
Mean-Shift finds **modes** without assuming spherical clusters or pre-defined centers; KDE captures non-Gaussian price interaction structure.

**Q: Can I backtest this as a standalone strategy?**  
This is an **indicator**. For a strategy, layer entries/exits around bands or centers with risk rules.

---

## Contributing

This repository is published under an **MIT license**; contributions and forks are welcome.  
Please open an issue to propose improvements, bugs, or feature requests.

---

## License

**MIT License**  
See LICENSE file for details.
