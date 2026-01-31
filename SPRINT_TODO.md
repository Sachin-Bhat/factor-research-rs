# Factor Research & Backtesting Engine - Sprint TODOs (~8 Weeks)

## Sprint 1 (Weeks 1-2): Repo + Data Pipeline + Core Types
- [ ] Project setup
  - [ ] Define crate layout (`src/`, `benches/`, `examples/`, `tests/`)
  - [ ] Establish module boundaries (data, factors, stats, portfolio, backtest, execution, io)
  - [ ] Add basic logging + error handling conventions
- [ ] Data model
  - [ ] Define canonical timestamp type and timezone rules
  - [ ] Define `AssetId`, `Bar` (OHLCV), `Event`, `Panel` types
  - [ ] Define `DataSlice` and `Window` interfaces for rolling access
- [ ] Data ingestion (Parquet/Arrow)
  - [ ] Parse metadata schema (symbol, exchange, calendar)
  - [ ] Implement column mapping + type coercion
  - [ ] Handle corporate actions / adjustment factors
- [ ] Resampling + calendars
  - [ ] Trading calendar utilities (market days, sessions)
  - [ ] Daily resample from intraday or vendor OHLCV
  - [ ] Forward-fill and missing data policy
- [ ] Factor computation skeleton
  - [ ] Factor registry + metadata (name, lookback, frequency)
  - [ ] Factor execution runner (per date, per asset)
  - [ ] Persist factor outputs per day (cross-sectional table)
- [ ] Deliverables
  - [ ] Basic CLI to ingest and compute a dummy factor
  - [ ] Sample dataset + README for local runs

## Sprint 2 (Weeks 3-4): Factors + Statistics Core
- [ ] Implement baseline factors
  - [ ] 12M-1M momentum
  - [ ] Short-horizon mean reversion (e.g., 5D or 10D)
  - [ ] Volatility / variance factor
  - [ ] Optional quality/earnings momentum placeholder
- [ ] Cross-sectional statistics
  - [ ] Rank transforms + winsorization utilities
  - [ ] Spearman/Pearson IC per day
  - [ ] IC lags (1-20d)
  - [ ] IC IR (mean / std)
- [ ] Factor decay curves
  - [ ] Forward return alignment by horizon
  - [ ] Decay curve generator + CSV output
- [ ] Regressions + inference
  - [ ] Cross-sectional regressions (OLS)
  - [ ] t-stats, p-values, standard errors
  - [ ] Factor covariance matrix
- [ ] Deliverables
  - [ ] Metrics report output (CSV/JSON)
  - [ ] Example notebook stub or plotting script placeholder

## Sprint 3 (Weeks 5-6): Portfolio Construction + Backtester
- [ ] Portfolio construction
  - [ ] Equal-weight long-only factor portfolio
  - [ ] Long-short portfolio (top/bottom quantiles)
  - [ ] Risk parity or volatility targeting
  - [ ] Position constraints (max weight, turnover caps)
- [ ] Execution + transaction costs
  - [ ] Market/limit execution models
  - [ ] Linear slippage + spread cost
  - [ ] Transaction cost accounting in PnL
- [ ] Backtester core
  - [ ] Daily event loop
  - [ ] Cash + positions accounting
  - [ ] PnL, drawdown, turnover, capacity metrics
- [ ] Deliverables
  - [ ] Strategy runner + sample config
  - [ ] Backtest results output (CSV/JSON)

## Sprint 4 (Weeks 7-8): Robustness + Signature Extension
- [ ] Robustness toolkit
  - [ ] Rolling-window IC/IR
  - [ ] Bootstrap IC estimates
  - [ ] Outlier sensitivity checks
  - [ ] Residual autocorrelation tests
- [ ] Extension (choose one)
  - [ ] Microstructure: order flow imbalance, microprice, short-term reversal
  - [ ] Alt-data: sentiment factor integration
  - [ ] Stability: rolling hypothesis tests across regimes
  - [ ] Auto-diff: factor optimization (experimental)
- [ ] Performance + scaling
  - [ ] Parallel factor computation (Rayon)
  - [ ] Benchmarks for core loops
  - [ ] Memory profiling notes
- [ ] Deliverables
  - [ ] Research report (summary + plots)
  - [ ] Final README + reproducibility steps

## Stretch Goals (If Time Remains)
- [ ] Almgren-Chriss transaction cost model
- [ ] Multi-asset support (equities + FX + commodities)
- [ ] Model diagnostics (rolling skew/kurtosis)
- [ ] Strategy comparison dashboard
