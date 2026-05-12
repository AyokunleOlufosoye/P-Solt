# Examples

## Overview

This directory contains educational examples demonstrating P-Solt's architecture in action. **All examples are for demonstration only.** Constants, parameters, and signals are intentionally unrealistic.

## Important Disclaimers

### All Constants Are Fake

Every numeric value in this directory is invented:
- Feature windows (SMA, volatility).
- Signal thresholds.
- Position sizes.
- Confidence levels.
- P&L figures in printed output.

**If you wonder whether a number is real: it is not.** Do not copy these constants into production systems. Do not use them as a starting point for research. They are chosen to demonstrate the architecture, not to be useful for trading.

### Sample Data Is Synthetic

`examples/data/sample_bars.csv` contains synthetic OHLCV data:
- Date: 2025-05-12 (May 12, 2025).
- Instrument: ES (E-mini S&P 500 index futures).
- Timeframe: 1-minute bars, 09:30–09:49 (20 bars).
- Values: Invented; not representative of real market behavior.

This data exists only to show how `dummy_signal_engine.py` reads and processes bars.

### Signal Generation Is Illustrative Only

The signal generator in `dummy_signal_engine.py` is intentionally fake:
- Logic is absurd (e.g., triggered by mathematical constants like π and φ).
- Signals do not resemble real strategy alpha.
- Confidence values are arbitrary.
- The goal is to demonstrate code organization, not to explain a real trading idea.

## Running the Example

### Setup

```bash
# From the P-Solt root directory
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### Run

```bash
python examples/dummy_signal_engine.py
```

### Expected Output

The script will:
1. Load synthetic bars from `examples/data/sample_bars.csv`.
2. Print the loaded bars.
3. Aggregate to 5-minute timeframe.
4. Print aggregated bars.
5. Compute features (SMA, volatility, time-of-day).
6. Generate fake signals using absurd thresholds.
7. Manage positions (open, close, compute P&L).
8. Print a summary of signals and positions.

Output is text-based and human-readable.

## Code Organization

### `dummy_signal_engine.py`

End-to-end example workflow:

```
Load bars (CSV)
    ↓
Aggregation (1m → 5m)
    ↓
Feature computation (SMA, volatility, time-of-day)
    ↓
Signal generation (absurd demo logic)
    ↓
Position management (open LONG, close, track P&L)
    ↓
Print summary
```

Each step is clearly commented. Comments explain *why* things are organized this way, not *what* the code does.

### `data/sample_bars.csv`

CSV format:
```
symbol,timestamp,open,high,low,close,volume,period
ES,2025-05-12 09:30:00,5000.00,5001.50,4999.00,5000.50,10000,1m
ES,2025-05-12 09:31:00,5000.50,5002.00,5000.00,5001.00,9500,1m
...
```

20 bars total, covering 09:30–09:49 on 2025-05-12.

## Educational Goals

This example teaches:

1. **How to organize code around layers** (data → features → signals → positions).
2. **How to use deterministic, testable modules** (each step is reproducible).
3. **How to structure a minimal end-to-end workflow** (from raw data to trading decision).
4. **How to keep code readable and maintainable** (clear types, good naming, helpful comments).

## What This Example Is NOT

- Not a working trading strategy.
- Not a backtesting system.
- Not a real data source or broker integration.
- Not a starting point for production code.
- Not a source of trading alpha.

## Next Steps

If you're learning from P-Solt:

1. **Read the architecture** – See `docs/SYSTEM_DESIGN.md` for layers and design rationale.
2. **Read the code** – Start with `src/types.py` (types), then `src/session/service.py` (logic).
3. **Read the tests** – See `tests/` to understand how modules are validated.
4. **Modify the example** – Try changing feature thresholds or signal logic. See how the output changes. (Remember: all constants are demo-only.)
5. **Build your own** – Use P-Solt as a template for your own modular, testable trading system.
