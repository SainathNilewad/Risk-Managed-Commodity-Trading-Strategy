# Risk-Managed Commodity Trading Strategy

## 1. Project Overview

This project tests two simple, well-known trading ideas on commodity price data using Python, and asks one question:

> **Can these strategies make money after trading costs once strict risk rules are applied, and were they tested honestly?**

| Strategy | Assets | Idea |
|---|---|---|
| Trend following | WTI crude, Brent crude, Gold | Hold the asset when its 20-day average price is above its 60-day average; otherwise stay in cash |
| Spread mean reversion | Brent minus WTI price gap | When the gap is unusually wide or narrow (60-day z-score beyond +/-2), bet it returns to normal |

**Risk rules:** volatility-targeted position sizing (10% annual risk, capped at 1.5x), a drawdown circuit breaker (halve the position below -15% drawdown, restore above -7.5%), a one-day execution delay (no look-ahead bias), and transaction costs (0.05% per unit traded, doubled for the two-leg spread).

**Honest testing:** settings were fixed in advance. Results are judged on **2021 onward (out-of-sample)**; 2014-2020 is reported separately as a check. Extra checks cover cost sensitivity, other moving-average windows (in-sample only), extra execution delay, and a US crude inventory test.

> **Important:** This is an educational historical analysis, not investment advice. The prices are spot or continuous price series, **not tradable futures contracts**: there is no modelling of contract expiry, rolling, margin, slippage or real execution. A backtest does not guarantee future performance.

## 2. Data

Place these files in a `data/` folder next to the notebook.

| File | Content | Source |
|---|---|---|
| `wti.csv` | WTI spot price, USD per barrel (`observation_date`, `DCOILWTICO`) | FRED series `DCOILWTICO`: https://fred.stlouisfed.org/series/DCOILWTICO |
| `brent.csv` | Brent spot price, USD per barrel (`observation_date`, `DCOILBRENTEU`) | FRED series `DCOILBRENTEU`: https://fred.stlouisfed.org/series/DCOILBRENTEU |
| `gold.csv` | Daily gold price, USD per ounce (`date`, `price`) | **TODO: state the exact source and whether it is spot, futures or an ETF** |
| `us_inventories.csv` | Weekly US crude oil inventories (`date`, `stocks_thousand_bbl`, `inventory_change_million_bbl`) | EIA weekly data. **TODO: state the exact series and whether it excludes the Strategic Petroleum Reserve** |

FRED marks missing days with a dot (`.`); the notebook converts these to missing values and drops them. All dates are aligned on days present in WTI, Brent and gold (2,950 trading days, October 2014 to October 2026).

## 3. Tools

Python, Jupyter Notebook, pandas, NumPy, Matplotlib, seaborn.

```bash
pip install pandas numpy matplotlib seaborn
```

## 4. Workflow

1. **Load and clean** the price files (missing values, sorting, duplicate dates).
2. **Handle the 20 April 2020 negative WTI price:** WTI and spread returns are zeroed on that day and the next, because percentage returns are undefined across a negative price (documented in the notebook).
3. **Calculate returns**, including a return series for the Brent-WTI spread.
4. **Split** the data: in-sample before 2021, out-of-sample from 2021.
5. **Build signals:** moving-average crossover and spread z-score.
6. **Size positions:** one-day execution delay, then volatility targeting.
7. **Backtest** with costs and the net-equity drawdown circuit breaker (one shared engine for every strategy).
8. **Evaluate:** total return, annualized volatility, Sharpe ratio, maximum drawdown, position changes, turnover.
9. **Stress-test:** cost sensitivity, in-sample window grid, extra execution delay.
10. **Fundamental check:** US crude inventory changes vs WTI (same-week and release-day reaction).
11. **Write findings** from the actual output, including limitations.

## 5. Results Snapshot (out-of-sample, 2021 onward, after costs)

Data through October 2026. Re-run the notebook to reproduce; do not quote these without checking.

| Strategy | Total Return | Ann. Volatility | Sharpe | Max Drawdown |
|---|---|---|---|---|
| WTI trend, raw | 51.1% | 32.6% | 0.39 | -53.0% |
| WTI trend, risk-managed | 13.1% | 6.6% | 0.36 | -15.7% |
| Brent trend, raw | 49.6% | 33.0% | 0.38 | -61.5% |
| Brent trend, risk-managed | 13.0% | 6.8% | 0.36 | -18.6% |
| Gold trend, raw | 85.5% | 15.9% | 0.77 | -20.1% |
| Gold trend, risk-managed | 46.3% | 9.0% | 0.80 | -13.7% |
| Brent-WTI spread, raw | 135.8% | 18.6% | 0.92 | -21.0% |
| Brent-WTI spread, risk-managed | 71.1% | 6.9% | 1.43 | -3.6% |
| WTI buy & hold | 110.8% | 41.0% | 0.53 | -55.2% |
| Brent buy & hold | 147.1% | 42.1% | 0.59 | -55.0% |
| Gold buy & hold | 116.8% | 18.9% | 0.82 | -24.9% |

Main takeaways (details and caveats in the notebook):

- The oil trend strategies did **not** beat buy-and-hold and made money mainly in two strong years.
- The risk rules cut drawdowns sharply for every strategy but did not improve the Sharpe ratio for the oil trend strategies.
- The spread strategy looks best, but its edge roughly halves with one extra day of execution delay, so it is treated as **unvalidated**.

## 6. Project Structure

```text
Risk-Managed-Commodity-Strategy/
├── data/
│   ├── wti.csv
│   ├── brent.csv
│   ├── gold.csv
│   └── us_inventories.csv
├── Risk-Managed_Commodity_Trading_Strategy.ipynb
└── README.md
```

## 7. How to Run

1. Put the four CSV files in `data/`.
2. Open the notebook in Jupyter or VS Code and choose **Run All**.
3. All settings (windows, thresholds, costs, split date) are in one cell near the top (Step 2).

## 8. Limitations

- Spot or continuous prices, not futures; no roll, margin, financing or slippage.
- **April 2020 is a modelling assumption:** WTI and spread returns are set to zero on 20-21 April 2020 (percentage returns are undefined across a negative price). This is not the actual market P&L for those days; out-of-sample results (2021 onward) are not affected.
- One market regime in the test period; results may differ in other periods.
- Simple fixed cost model; real costs, especially for spreads, are usually higher.
- Settings are conventional defaults, neither optimised nor proven; no walk-forward re-estimation.
- Brent and WTI are priced at different times of day, which may flatter the spread result.
- Inventory tests use the raw change, not a surprise versus forecasts.

## 9. Possible Next Steps

- Test on futures data with a documented roll method.
- Add a bond ETF for fixed-income exposure.
- Re-run the spread test with synchronized prices.
- Replace the single split with walk-forward testing.
