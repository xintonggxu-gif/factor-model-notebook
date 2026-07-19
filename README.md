# Monthly Multi-Factor Equity Model

A monthly cross-sectional equity strategy built around two established signals: 12-month price momentum and 60-day low volatility. The model ranks a fixed universe of U.S. large-cap stocks at each month-end, combines the two signals into a single score, and holds an equally weighted portfolio of the highest-ranked names.

The project demonstrates a complete and auditable quantitative research workflow—from price preparation and factor construction to portfolio formation, transaction-cost modeling, and out-of-sample evaluation.

## Strategy Overview

The current universe contains 30 U.S. large-cap stocks, with data beginning in January 2015. Prices are downloaded through `yfinance`. The backtest uses the `Adj Close` field so that returns reflect stock splits and distributions.

Two factors are calculated at each month-end:

- **12-month momentum:** `P(t) / P(t-12) - 1`. Stocks with stronger performance over the previous year receive higher scores.
- **60-day low volatility:** The negative of the rolling 60-trading-day standard deviation of daily returns. Taking the negative value means that lower-volatility stocks receive higher scores.

Because the factors have different units and distributions, they are normalized cross-sectionally each month:

1. Values are winsorized at the 5th and 95th percentiles to reduce the influence of outliers.
2. The winsorized observations are converted into Z-scores.
3. The two standardized factors are combined into a single score:

```text
Composite score = alpha × Low-volatility score
                + (1 - alpha) × Momentum score
```

The default configuration uses `alpha = 0.5`, giving equal weight to both factors.

At each rebalance, the model selects the eight stocks with the highest composite scores and assigns each a weight of `1/8` for the following month.

## Backtest Methodology

Signals and returns are aligned to avoid look-ahead bias. Portfolio weights formed using information available at month-end `t` are applied to returns in month `t+1`.

Gross portfolio return is calculated as the weighted sum of the selected stocks’ forward monthly returns. Transaction costs are deducted based on changes in portfolio weights:

```text
Turnover(t)   = Σ |w(t) - w(t-1)|
Net return(t) = Gross return(t) - Cost rate × Turnover(t)
```

The default cost rate is `0.001`, equivalent to 10 basis points per unit of turnover.

The backtest calculates:

- Annualized return
- Annualized volatility
- Sharpe ratio
- Maximum drawdown
- Monthly turnover
- Cumulative net asset value

The current Sharpe ratio is defined as annualized return divided by annualized volatility. No risk-free rate is deducted.

## Out-of-Sample Evaluation

The final 24 months of available data are reserved as the out-of-sample period.

In the default `main()` workflow, the strategy uses fixed parameters:

```text
alpha = 0.5
N = 8
transaction cost = 0.001
```

Results are reported for both the full sample and the out-of-sample window.

The notebook also includes an optional `findsignal()` function that tests a deliberately small parameter grid using only the in-sample period:

- Factor weight `alpha`: 0.25, 0.50, or 0.75
- Number of holdings `N`: 5, 10, or 15

The parameter pair with the highest in-sample Sharpe ratio is then used to evaluate the full history and the out-of-sample period.

This function is not currently called by `main()` and should be treated as a separate research module rather than part of the default run. The search space is kept small to limit data-snooping and selection bias.

## Example Results

One completed run stored in the notebook produced the following results:

| Metric | Result |
| --- | ---: |
| Out-of-sample start | 2024-02-29 |
| Full-sample Sharpe ratio | 0.973 |
| Full-sample maximum drawdown | -15.36% |
| Out-of-sample annualized return | 13.51% |
| Out-of-sample annualized volatility | 12.63% |
| Out-of-sample Sharpe ratio | 1.070 |
| Out-of-sample maximum drawdown | -7.66% |

These figures are provided as a record of one completed run rather than fixed expected output.

Downloading the data again may change the sample end date, out-of-sample split, and reported metrics because of newly available observations or revisions by the data provider.

## Installation

Python 3.10 or later is recommended.

Install the required packages:

```bash
pip install pandas numpy matplotlib yfinance jupyter
```

## Usage

Launch the notebook:

```bash
jupyter notebook newmultifactormodel.ipynb
```

Run the cells in order. The `main()` function will:

1. Download historical stock prices.
2. Convert daily prices into monthly observations.
3. Calculate the momentum and low-volatility factors.
4. Standardize and combine the factor scores.
5. Construct the monthly portfolio.
6. Apply transaction costs.
7. Report full-sample and out-of-sample performance.

An internet connection is required to retrieve data from Yahoo Finance.

## Notebook Structure

| Function | Purpose |
| --- | --- |
| `make_monthly()` | Converts daily prices into month-end prices, monthly returns, and forward returns |
| `compute_factor()` | Calculates the 12-month momentum and 60-day low-volatility factors |
| `countzscore()` | Winsorizes, standardizes, and combines the factor observations |
| `build_weights()` | Selects the Top N stocks and creates an equally weighted portfolio |
| `backtest()` | Applies transaction costs and calculates portfolio performance |
| `oosperiod()` | Reserves the final 24 months for out-of-sample evaluation |
| `findsignal()` | Searches a limited grid of factor weights and portfolio sizes |
| `main()` | Runs the default fixed-parameter backtest |

## Limitations

This project is a research prototype, not a production trading system. Several limitations should be considered when interpreting the results:

- The universe is a manually selected list of 30 current large-cap companies.
- Historical index membership and delisted stocks are not included, creating potential survivorship and selection bias.
- No benchmark is included, so the backtest does not separate factor-driven excess returns from broad market exposure.
- Transaction costs are modeled as a fixed percentage of turnover.
- Bid–ask spreads, market impact, liquidity constraints, and execution delays are not simulated.
- The Sharpe ratio does not account for the risk-free rate.
- The momentum signal uses the full previous 12 months and does not skip the most recent month.
- A 24-month out-of-sample period is relatively short and may be sensitive to the prevailing market regime.
- The optional parameter search does not yet use rolling retraining, walk-forward validation, or corrections for multiple testing.

Useful next steps include adding a benchmark portfolio, introducing rolling out-of-sample tests, controlling for sector exposure, and refining the transaction-cost model.

These improvements would provide stronger evidence of the strategy’s robustness before additional factors or parameters are introduced.

## Disclaimer

This project is provided for educational and quantitative research purposes only. It does not constitute investment advice.

Historical backtest results do not guarantee future performance. Live results may differ materially because of data quality, trading costs, liquidity, and execution conditions.
