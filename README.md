# NIFTY 50 Statistical Pairs Trading

A systematic cointegration-based pairs trading pipeline built on the NIFTY 50 universe.

## Strategy

This pipeline identifies and backtests mean-reverting stock pairs using:

- **Engle-Granger cointegration test** (conservative — run both directions, take max p-value)
- **Rolling OLS hedge ratio** (60-day window, adapts to regime changes)
- **Z-score signals** (entry ±2σ, exit 0.5σ)
- **Pair filters** — beta stability (rolling std < 0.2) and half-life of mean reversion

## Results

| Metric | Best Pair (EICHERMOT/SHRIRAMFIN) |
|---|---|
| Sharpe Ratio | 0.40 |
| Sortino Ratio | 0.92 |
| CAGR | 14% |
| Max Drawdown | -42% |
| Total Return | 2.95x |

**Universe:** NIFTY 50 (46 stocks after liquidity filter)  
**Period:** 2018-01-01 to 2026-03-10  
**Pairs scanned:** 1,035  
**Cointegrated pairs:** 62  
**Pairs passing all filters:** 18  

## Key Findings

- NIFTY 50 large-cap pairs have long mean-reversion half-lives (120–480 days) — the market is efficient for large caps on EOD data
- Only pairs with stable rolling beta (std < 0.2) produce reliable signals
- Transaction costs (10bps/leg) significantly impact Sharpe — EOD pairs trading requires high signal quality
- Smallcap universe likely has faster mean reversion — next research direction

## Pipeline

```
Data Pull → Cointegration Scan → Pair Filtering → Signal Generation → Backtest → Metrics
```

1. **Data** — `get_prices()` downloads adjusted close prices, drops tickers with insufficient history, forward-fills gaps
2. **Cointegration** — `test_cointegration()` runs Engle-Granger both ways, takes conservative p-value
3. **Filtering** — `filter_pairs()` applies rolling beta stability and half-life filters
4. **Signals** — `compute_zscore()` + `generate_signals()` with persistent position logic
5. **Backtest** — `backtest_pair()` with rolling hedge ratio, no lookahead bias
6. **Metrics** — Sharpe, Sortino, CAGR, Max Drawdown (net of 10bps transaction costs, 6% RFR)

## Setup

```bash
# Install uv if needed
curl -LsSf https://astral.sh/uv/install.sh | sh

# Create environment and install dependencies
uv venv --python 3.13
source .venv/bin/activate
uv add yfinance pandas numpy scipy statsmodels matplotlib jupyter
```

## Usage

```bash
jupyter notebook pairs_trading_clean.ipynb
```

Run all cells in order. The full pipeline takes ~5 minutes (1,035 cointegration tests).

## Disclaimer

This is a research project built for educational purposes. Past backtest performance does not guarantee future results. Not financial advice.
