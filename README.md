# P-Solt

A **public, sanitized reference architecture** for building disciplined, testable trading systems.

P-Solt extracts architectural patterns from a private quantitative system—showing *workflow and structure*, not proprietary edge. It demonstrates how to organize sessions, data ingestion, feature computation, signal generation, and position management in a clean, deterministic, testable way.

## What This Repo Shows

- **Session & time normalization** – Clean abstraction for trading calendars and intraday session rules (RTH, pre/post, overnight).
- **Deterministic bar aggregation** – Reproducible OHLCV computation from lower to higher timeframes.
- **Feature computation pipeline** – Modular generic features (SMA, volatility, time-of-day factors) with stable, versioned interface.
- **Disciplined signal architecture** – Clean separation of signal intent, position lifecycle, and execution stubs.
- **Testing discipline** – Comprehensive deterministic unit tests validating each layer in isolation.
- **Documentation-driven design** – Type hints, docstrings, and architecture rationale documented throughout.

## What This Repo Intentionally Omits

- **No proprietary strategy logic** – The signal generator is intentionally fake and uses absurd constants to show this is demo-only.
- **No tuned parameters** – Thresholds, windows, and coefficients are reference values, not production edge.
- **No live integration** – No real broker connections, no account details, no execution wiring.
- **No proprietary feature combinations** – Examples use generic, well-known technical indicators.
- **No backtesting results** – No walk-forward sweeps, no performance data, no strategy alpha.

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                         EXECUTION LAYER                      │
│                    (Stub / Abstract only)                    │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│                      STRATEGY LAYER                          │
│  Signal Generator │ Position Manager │ Risk Constraints     │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│                      FEATURE LAYER                           │
│            Feature Pipeline & Computation                   │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│                       DATA LAYER                             │
│  Bar Builder │ Adapter (CSV, Vendor, Broker) │ Types        │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│                     SESSION LAYER                            │
│  Time Normalization │ Trading Calendar │ Deterministic      │
└─────────────────────────────────────────────────────────────┘
```

## Repository Layout

```
P-Solt/
├── README.md                          # This file
├── requirements.txt                   # Minimal dependencies
├── .env.example                       # Template environment variables
├── .gitignore                         # Standard Python ignores
│
├── src/                               # Core reference architecture
│   ├── types.py                       # Data contract types
│   │
│   ├── session/
│   │   └── service.py                 # Session/calendar logic
│   │
│   ├── data/
│   │   ├── bar_builder.py             # Bar aggregation
│   │   └── adapter.py                 # Abstract data adapter
│   │
│   ├── features/
│   │   └── pipeline.py                # Feature computation
│   │
│   └── strategy/
│       ├── signal_generator.py        # Demo signal logic (intentionally fake)
│       └── position_manager.py        # Position lifecycle
│
├── tests/                             # Deterministic unit tests
│   ├── test_session_service.py
│   ├── test_bar_builder.py
│   ├── test_feature_pipeline.py
│   └── test_position_manager.py
│
├── examples/                          # Educational examples
│   ├── README.md                      # Example documentation & disclaimers
│   ├── dummy_signal_engine.py         # End-to-end demo workflow
│   └── data/
│       └── sample_bars.csv            # Synthetic sample data
│
└── docs/                              # Technical documentation
    ├── README.md                      # Documentation index
    ├── SYSTEM_DESIGN.md               # Architecture & design rationale
    ├── RESEARCH_METHODOLOGY.md        # Validation & workflow principles
    ├── TESTING_STRATEGY.md            # Test approach & parity validation
    └── DATA_CONTRACTS.md              # Generic type schemas
```

## Quick Start

### Setup

```bash
# Clone and create a virtual environment
git clone https://github.com/AyokunleOlufosoye/P-Solt.git
cd P-Solt
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### Run Tests

```bash
pytest tests/ -v
```

All tests are deterministic and isolated. They validate each module independently without external I/O.

### Run Example

```bash
python examples/dummy_signal_engine.py
```

This end-to-end example loads synthetic bar data, aggregates to a higher timeframe, computes features, generates fake signals, and prints a readable summary. **All constants and signals are intentionally unrealistic and for demonstration only.**

## Research & Methodology

P-Solt embodies principles from disciplined quantitative research:

- **Walk-forward validation thinking** – The session and bar layers support reproducible, deterministic replay needed for validation.
- **Feature vs. interpretation separation** – Raw features (SMA, volatility) are computed independently; signal interpretation is isolated in the signal generator.
- **Parity as a validation principle** – Deterministic modules enable shadow replay and cross-venue parity checks.
- **Public repos show workflow, not edge** – This repo teaches *how to build*, not *what to trade*.

See [`docs/RESEARCH_METHODOLOGY.md`](docs/RESEARCH_METHODOLOGY.md) for details.

## Intended Audience

- **Recruiters & hiring managers** – Evaluating engineering discipline, testing rigor, and architectural thinking.
- **Quantitative traders & researchers** – Referencing session logic, deterministic data pipelines, and feature engineering patterns.
- **Software engineers exploring quant finance** – Understanding modular design applied to trading systems.

## Important Notes on Numbers & Examples

**All numeric constants, parameters, and signals in this repo are intentionally fake and unrealistic.**

- The dummy signal generator uses constants based on mathematical constants (π, e, φ) to make clear it is not production logic.
- Threshold values, feature windows, and position sizes are reference values only.
- Sample data is synthetic (2025-05-12, 09:30–09:49, ES 1-minute bars).

**If you wonder whether a number is real: it is not.** The repo is designed for educational architecture review, not for running live or paper trading.

## Documentation

Start with:
- [`docs/README.md`](docs/README.md) – Documentation index.
- [`docs/SYSTEM_DESIGN.md`](docs/SYSTEM_DESIGN.md) – Architecture, layers, and design goals.
- [`docs/TESTING_STRATEGY.md`](docs/TESTING_STRATEGY.md) – Testing approach and reproducibility.
- [`docs/RESEARCH_METHODOLOGY.md`](docs/RESEARCH_METHODOLOGY.md) – Validation principles and public-repo philosophy.
- [`docs/DATA_CONTRACTS.md`](docs/DATA_CONTRACTS.md) – Generic type schemas.

## License & Contribution

This is a public portfolio repository. It is not open-source; no license file is provided and pull requests are not accepted. Feel free to reference the architecture or adapt the patterns for your own work.

---

**P-Solt** is extracted from patterns in a private quantitative system. This sanitized version demonstrates **disciplined architecture and testing practice without exposing proprietary strategy logic, live execution details, or sensitive research results.**
