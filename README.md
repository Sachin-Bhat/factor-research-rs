# Factor Research & Backtesting Engine (Rust)

A Rust-first factor research and backtesting engine focused on cross-sectional alpha research, statistical evaluation (IC/IR, decay, regressions), and realistic portfolio construction with execution and transaction cost modeling.

## Workspace layout
- `crates/fr-core`: canonical data model and shared utilities
- `crates/fr-data`: ingestion, calendars, resampling
- `crates/fr-factors`: factor traits, registry, implementations
- `crates/fr-stats`: IC/IR, regressions, decay curves, covariance
- `crates/fr-portfolio`: signal transforms, weighting, constraints
- `crates/fr-execution`: slippage, spread, fills
- `crates/fr-backtest`: event loop, PnL, accounting
- `crates/fr-io`: CSV/JSON output helpers
- `crates/fr-cli`: CLI entrypoints

## Quick start
```bash
cargo check
cargo run -p fr-cli -- --help
```

## Configs
See `configs/sample.toml` for a placeholder run configuration.

## Notes
This repo is designed for a staged build-out. The engine logic lives in libraries; CLI commands live in `fr-cli`.
