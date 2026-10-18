# Erste Earnings Volatility

An event study of how the share-price volatility of **Erste Group (`EBS.VI`)**
behaves around its quarterly results, benchmarked against the Austrian index
**ATX (`^ATX`)**.

> **Question.** Does the volatility of Erste Group increase around quarterly
> results, how fast does it return to normal, and is the effect specific to
> Erste or does the ATX move too?

The motivation is a market-making one: results dates are known in advance, so if
volatility reliably spikes around them, quotes and spreads should be widened
ahead of the event.

## Data & scope

| Item | Value |
|---|---|
| Stock | Erste Group, `EBS.VI` (Wiener Börse) |
| Benchmark | ATX, `^ATX` |
| Data | Daily OHLCV (adjusted close, via `yfinance`) |
| Period | 2023-01-01 → present (~11 results dates in range) |
| Events | Quarterly results (Q1, H1, Q3, full year) |

Event dates come from the [Erste Investor Relations financial
calendar](https://www.erstegroup.com/en/investors/events/financial_calendar),
not only from Yahoo, and are snapped to the nearest trading day (Erste publishes
before the open, so day 0 = publication day).

## Method

The analysis is an event study, done entirely in [`analysis.ipynb`](analysis.ipynb):

1. **Returns.** Daily log returns per ticker, `r_t = ln(C_t / C_{t-1})`. Garman–Klass
   volatility is computed too, but close-to-close is the main measure (results
   land before the open, which GK cannot see).
2. **Event window.** For each event, take days `k = -10 … +10` relative to day 0.
3. **Volatility profile.** For each `k`, take the root-mean-square of returns across
   all events (`sqrt(mean(r²))`) — a per-day volatility level. (A short-window
   standard deviation is deliberately avoided: two points do not estimate a std.)
4. **Baseline.** The same RMS measure over all days *outside* any ±10-day window.
5. **Ratio.** Profile ÷ baseline. A ratio of 2.0 on day 0 means volatility is twice
   its normal level.
6. **Market adjustment.** The same profile is built for the ATX on the same dates.
   If the ATX does not move, the effect is specific to Erste.
7. **Significance (placebo test).** Many random non-event date sets are drawn and
   their day-0 ratio compared to the observed one, giving an empirical p-value.
8. **Robustness.** The profile is recomputed with the median of `|r|` to check the
   result is not driven by a single large surprise.

## Charts

- **Rolling 20-day annualized volatility** of EBS and ATX, with vertical lines on
  the results dates.
- **Volatility profile** (days −10 to +10) as a ratio to baseline, EBS and ATX on
  one chart — the main result.

## Running

```bash
python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter lab analysis.ipynb         # or open in VS Code
```

The notebook reads `data/ticker_data.csv` (the saved raw download), so it runs
without hitting the network. Re-run the first download cell only to refresh the
data.

## Repository

```
analysis.ipynb          the full event study
data/ticker_data.csv    raw daily OHLCV, saved before any transformation
requirements.txt        Python dependencies
```

## Caveats

- Small sample (~11 events); no data on surprise size or guidance.
- Daily data only — says nothing about the intraday speed of the move.
- Volatility is measured, not spreads; the market-making implication is an
  inference, not a result.
- Event windows can overlap with broader stress periods (e.g. the March 2023
  banking stress).
