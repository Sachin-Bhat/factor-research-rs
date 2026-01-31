# Detailed Guide: Building a Rust Factor Research & Backtesting Engine

This guide is a practical, end-to-end blueprint for building a high-performance factor research and backtesting engine in Rust within ~8 weeks. It is structured as a build plan that maps to the sprint TODOs.

## 0) Principles and Scope
- Focus on correctness and reproducibility first; add performance after the logic is validated.
- Design around a clean data model (time, asset, feature), then layer in stats, portfolio construction, and execution.
- Keep the Rust core deterministic and fast; use Python only for exploratory analysis and plotting.
- Prefer explicit, audited assumptions: calendars, missing data handling, and transaction cost rules.

## 1) Core Architecture
### Recommended module boundaries
- `data/` -> ingestion, schema, calendars, resampling, adjustments
- `core/` -> shared types: `AssetId`, `Timestamp`, `Bar`, `Panel`, `Returns`
- `factors/` -> factor definitions, registry, factor runner
- `stats/` -> IC/IR, regressions, covariance, decay curves
- `portfolio/` -> signal transforms, weighting, constraints
- `execution/` -> fills, slippage, spread, cost model
- `backtest/` -> event loop, PnL, accounting
- `io/` -> CSV/JSON output, serialization

### Potential Layout
```
factor-research-rs/
├─ Cargo.toml                 # workspace
├─ README.md
├─ LICENSE
├─ .gitignore
├─ crates/
│  ├─ fr-core/                # shared types + utilities
│  │  └─ src/
│  ├─ fr-data/                # ingestion, calendars, resampling
│  │  └─ src/
│  ├─ fr-factors/             # factor definitions + runner
│  │  └─ src/
│  ├─ fr-stats/               # IC/IR, regressions, decay, covariance
│  │  └─ src/
│  ├─ fr-portfolio/           # signal transforms, weights, constraints
│  │  └─ src/
│  ├─ fr-execution/           # slippage/spread, fills
│  │  └─ src/
│  ├─ fr-backtest/            # event loop, PnL, accounting
│  │  └─ src/
│  ├─ fr-io/                  # CSV/JSON/Parquet output helpers
│  │  └─ src/
│  └─ fr-cli/                 # binaries / CLI entrypoints
│     └─ src/
├─ configs/                   # YAML/TOML run configs
├─ data/                      # (optional) small sample datasets
├─ results/                   # outputs + snapshots
├─ reports/                   # plots + writeups
├─ notebooks/                 # python EDA/plots
├─ examples/                  # end-to-end demos
├─ benches/                   # criterion benches
└─ scripts/                   # helper scripts
```

### Data flow (high-level)
1. Ingest raw market data -> normalized OHLCV + corporate actions
2. Build rolling windows -> compute factors per asset, per date
3. Generate forward returns -> align with factors
4. Compute IC/IR and regressions -> rank factors
5. Construct portfolio -> apply execution + costs
6. Backtest -> report PnL and diagnostics

## 2) Data Pipeline (Rust)
### Data model conventions
- Use a single trading calendar (e.g., NYSE) across all assets for cross-sectional alignment.
- Define a `Timestamp` type that normalizes all inputs to UTC and store session-based timestamps for daily bars.
- Use `AssetId` as a compact integer index for fast table operations.

### Ingestion steps
- Parse Parquet/Arrow into a columnar in-memory representation.
- Validate schema: `timestamp`, `open`, `high`, `low`, `close`, `volume`, `symbol`.
- Apply corporate actions via adjustment factors (split/dividend) at ingestion time.
- Persist a clean, deterministic OHLCV store for downstream calculations.

### Resampling
- Convert intraday to daily OHLCV using standard aggregation rules.
- Use a consistent missing-data policy (drop, forward-fill, or flag as missing) and document it.

### Rolling windows
- Create a rolling window abstraction that yields slices of data without reallocations.
- Favor contiguous data layouts to improve cache behavior.

## 3) Factor Computation Framework
### Factor API design
A factor is a pure function of a rolling window for a single asset, returning a scalar per date.

Recommended interface:
- Inputs: `AssetId`, `Date`, `Window<Bar>`
- Output: `Option<f64>` (None for insufficient lookback)

### Factor registry and runner
- A registry maps `FactorId -> FactorDefinition` (name, lookback, frequency, transform).
- The runner iterates across dates, then assets, emitting a cross-sectional table of factor values.

### Baseline factors
- Momentum (12M-1M): log return from t-252 to t-21
- Mean reversion (short horizon): negative short-term return (5D or 10D)
- Volatility factor: rolling std of daily returns
- Optional quality: placeholder factor until fundamentals are integrated

## 4) Statistical Evaluation
### IC/IR
- Compute cross-sectional IC per date (Spearman rank + Pearson value).
- Lag IC by shifting forward returns 1-20 days to observe decay.
- Compute IR as mean(IC) / std(IC).

### Decay curves
- For each factor, compute mean IC at each forward horizon.
- Output as CSV for plotting.

### Regressions
- Perform daily cross-sectional OLS:
  - y: forward returns
  - x: factor value (optionally z-scored)
- Aggregate coefficient distributions and t-stats.

### Covariance matrix
- Use factor returns or IC series to compute covariance for portfolio risk modeling.

## 5) Portfolio Construction
### Signal transformation
- Rank or z-score factor values per date.
- Optional neutralization (demean or industry/sector neutralization when available).

### Weighting schemes
- Equal-weight long-only: top quantile
- Long-short: top minus bottom quantile
- Risk parity or volatility targeting: scale by inverse risk

### Constraints
- Max weight per asset
- Turnover cap
- Gross and net exposure limits

## 6) Execution & Transaction Costs
### Execution models
- Market: fill at close or next open with slippage
- Limit: fill if price crosses limit (optional for daily bars)

### Cost model
- Linear slippage: `cost = k * |trade_value|`
- Spread cost: add half-spread to buys, subtract on sells
- Keep costs explicit and reported separately from raw PnL

## 7) Backtester
### Event loop
- For each date:
  - Load factor values
  - Generate target weights
  - Convert to trades
  - Apply execution + costs
  - Update cash + positions

### Metrics
- PnL, volatility, drawdown, Sharpe
- Turnover and capacity
- Transaction cost attribution

### Output
- Per-date performance table
- Summary JSON for dashboards or reports

## 8) Robustness Toolkit
### Rolling windows
- Compute rolling IC/IR to detect regime instability.

### Bootstrap IC
- Resample daily ICs to estimate confidence intervals.

### Outlier sensitivity
- Compare factor performance with and without winsorization.

### Residual checks
- Test for autocorrelation in regression residuals.

## 9) Signature Extension Options
### Option A: Microstructure alpha (intraday)
- Order flow imbalance
- Microprice
- Short-term reversal / liquidity provision signals

### Option B: Alternative data factor
- Local embedding model -> sentiment factor
- Latent semantic momentum

### Option C: Stability and robustness study
- Rolling hypothesis tests
- Sensitivity to parameter choices

### Option D: Auto-diff factor optimization
- Implement reverse-mode autodiff for factor tuning

Pick one extension and implement it end-to-end (data -> factor -> stats -> backtest).

## 10) Performance Engineering
- Use columnar storage for speed.
- Parallelize factor computation and IC calculations with Rayon.
- Profile hotspots; optimize only after correctness is validated.

## 11) Reproducibility and Research Artifacts
- Provide a deterministic config (dataset, dates, parameters).
- Store outputs in `results/` with versioned config snapshots.
- Add a short report with charts and interpretation.

## 12) Suggested Milestones
- End of Week 2: data pipeline + factor skeleton
- End of Week 4: 3-4 factors + IC/IR + decay curves
- End of Week 6: portfolio + backtester
- End of Week 8: robustness + signature extension
