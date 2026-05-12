# System Design

P-Solt is a reference architecture for disciplined, testable trading systems. This document describes the design goals, layered structure, and key principles.

## Design Goals

### Canonicalization
Normalize time, market sessions, and data formats at entry points. Once canonicalized, all downstream processing operates on a consistent, predictable representation. This eliminates ambiguity and enables reproducible replay.

### Parity
Design deterministic modules such that identical inputs always produce identical outputs. This enables cross-validation: computing the same logic in different codebases or systems should yield identical results. Parity is fundamental to validation and debugging.

### Modularity
Clean layer separation with explicit data contracts. Each layer depends on well-defined input/output types. Modules can be tested independently and substituted without affecting others.

### Testability
Every module is designed to be testable in isolation. Unit tests are deterministic, reproducible, and require no external resources (no network, no real data, no luck). This supports continuous iteration and high confidence in core logic.

### Documentation-Driven Design
Code is self-documenting through clear type hints, docstrings, and architectural diagrams. Comments explain *why*, not *what*. Architecture decisions are recorded in this document and data contracts are explicit.

## Layered Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                      EXECUTION LAYER                         │
│                   (Stub / Abstract Only)                     │
│            Submit orders, manage live positions              │
│     No implementation; all broker integration is here        │
└────────────────────┬─────────────────────────────────────────┘
                     │ Position updates, fills
                     ▼
┌──────────────────────────────────────────────────────────────┐
│                      STRATEGY LAYER                          │
│                                                              │
│  Signal Generator         Position Manager                  │
│  ├─ Interprets features   ├─ Open / Close position         │
│  └─ Generates signals     ├─ Track entry, P&L              │
│                           └─ Lifecycle management           │
│                                                              │
│  Risk Constraints (optional)                                │
│  ├─ Max position size                                       │
│  └─ Drawdown checks                                         │
└────────────────────┬─────────────────────────────────────────┘
                     │ Signals, positions
                     ▼
┌──────────────────────────────────────────────────────────────┐
│                      FEATURE LAYER                           │
│                                                              │
│  Feature Pipeline                                           │
│  ├─ Compute SMA (fast, slow)                               │
│  ├─ Compute volatility (rolling)                           │
│  ├─ Compute time-of-day factors                            │
│  └─ Emit FeatureVector with version tag                    │
│                                                              │
│  Features are deterministic; versioned for reproducibility │
└────────────────────┬─────────────────────────────────────────┘
                     │ Feature vectors
                     ▼
┌──────────────────────────────────────────────────────────────┐
│                       DATA LAYER                             │
│                                                              │
│  Bar Builder              Adapter                           │
│  ├─ Aggregate lower       ├─ CSV source                    │
│    to higher timeframe    ├─ Vendor source (stub)          │
│  ├─ Normalize OHLCV       └─ Broker source (stub)          │
│  └─ Preserve semantics                                     │
│                                                              │
│  SessionContext                                             │
│  ├─ Trading date                                           │
│  └─ Session type (RTH, pre, post, overnight)               │
└────────────────────┬─────────────────────────────────────────┘
                     │ Bars, session info
                     ▼
┌──────────────────────────────────────────────────────────────┐
│                      SESSION LAYER                           │
│                                                              │
│  Session Service (Deterministic)                            │
│  ├─ Classify time into RTH / pre / post / overnight         │
│  ├─ Compute trading date                                   │
│  ├─ Normalize timezones                                    │
│  └─ Support calendar lookups                               │
│                                                              │
│  Provides canonical timestamp semantics for all upstream   │
└──────────────────────────────────────────────────────────────┘
```

## Layer Responsibilities

### Session Layer
**Responsible for:** Time normalization, trading calendar logic, session classification.

- Classify any timestamp into a session type: RTH (regular trading hours), pre-market, post-market, overnight.
- Compute trading date (market date) from a timestamp.
- Handle timezone conversions consistently.
- Provide deterministic lookups for market hours, holidays, etc.

**Key principle:** This layer is the "source of truth" for time. All upstream layers depend on its canonicalization.

### Data Layer
**Responsible for:** Bar building, data ingestion, data type contracts.

- **Bar Builder:** Aggregates lower-timeframe bars into higher-timeframe OHLCV. Preserves boundary semantics and timestamp consistency.
- **Adapter:** Abstract interface for data sources. Implementations handle CSV, vendor APIs, or broker feeds. Public repo includes CSV stub and abstract stubs for external sources.
- **Types:** Standardized dataclasses (Bar, SessionContext, FeatureVector, Signal, Position).

**Key principle:** Data enters the system here. All downstream logic assumes data has been canonicalized and validated.

### Feature Layer
**Responsible for:** Computing features deterministically, versioning features.

- Takes bars and session context.
- Computes generic features: moving averages, volatility, time-of-day factors, etc.
- Emits FeatureVector with a version tag (enables reproducibility across code changes).
- **Does not interpret:** Raw features are facts; meaning is assigned by the strategy layer.

**Key principle:** Feature computation is deterministic and reproducible. Same bars → same features.

### Strategy Layer
**Responsible for:** Signal generation, position lifecycle, optional risk constraints.

- **Signal Generator:** Takes feature vectors and interprets them to emit signals (BUY, SELL, HOLD).
- **Position Manager:** Tracks open positions, computes entry/exit logic, tracks P&L.
- **Risk Constraints (optional):** Enforce max position size, drawdown limits, etc.

**Key principle:** This layer contains *interpretive* logic. Signals are opinions about features, not facts.

### Execution Layer
**Responsible for:** Order submission, live position management, connection to broker/exchange.

- **Public repo:** Abstract stubs only. No real broker wiring, no credentials, no live trading.
- **Production system:** Concrete implementations here handle order submission, fills, position reconciliation.

**Key principle:** This layer is isolated by abstraction. Swapping it does not affect strategy or feature logic.

## Data Contracts

All layers communicate via well-defined types (see [`DATA_CONTRACTS.md`](DATA_CONTRACTS.md)):

- **Bar:** OHLCV + timestamp + symbol.
- **SessionContext:** Timestamp + session type + trading date.
- **FeatureVector:** Dict of computed features + version tag.
- **Signal:** Direction (BUY/SELL/HOLD) + confidence + timestamp.
- **Position:** Side (LONG/SHORT) + entry price + size + entry time + P&L.

These are minimal; production systems may extend with additional fields (e.g., commission, slippage) without breaking this architecture.

## Relationship to Private System

P-Solt extracts *patterns* from a private quantitative system:

- **Session canonicalization** → Core principle from production validated on live markets.
- **Deterministic bar building** → Foundational for reproducibility in production research.
- **Modular feature pipeline** → Decouples feature definition from strategy interpretation.
- **Parity validation** → Critical for validating code ports and cross-venue behavior.

**What is NOT extracted:**
- No proprietary signal logic.
- No tuned thresholds or parameters.
- No feature combinations that reveal edge.
- No backtesting results or alpha estimates.
- No live execution wiring or broker secrets.

The public repo shows *methodology and structure*, not *secrets*.

## Key Design Decisions

### Why Layers?
Layers decouple concerns. Changing signal logic does not break data ingestion. Adding a new data source does not touch feature computation. This reduces risk and enables parallel work.

### Why Determinism?
Deterministic code is debuggable, testable, and reproducible. If a bug occurs during development or validation, identical inputs reveal it every time. No "flaky" bugs, no luck.

### Why Types?
Explicit types (dataclasses + type hints) make contracts clear. IDE tooling catches mistakes. Tests validate that types flow correctly through layers.

### Why Feature Versioning?
Feature code changes frequently during research. Version tags ensure that historical analysis can be reproduced with the same feature definition, even if the code has been updated.

## Testing Strategy

Each layer has deterministic unit tests that validate behavior in isolation:

- **Session tests:** RTH/pre/post/overnight classification, boundary cases, determinism.
- **Bar builder tests:** Aggregation correctness, OHLCV semantics, timestamp consistency.
- **Feature tests:** Feature computation determinism, edge cases, version tagging.
- **Strategy tests:** Signal generation, position lifecycle, P&L tracking.

See [`TESTING_STRATEGY.md`](TESTING_STRATEGY.md) for details.

## Extensibility

This architecture supports extending without breaking:

- **New data sources:** Implement DataAdapter interface.
- **New features:** Add computation to FeaturePipeline with a new version tag.
- **New signal logic:** Write new SignalGenerator or extend existing.
- **New risk constraints:** Add checks to StrategyLayer.
- **New execution backends:** Implement ExecutionLayer interface.

Each extension is isolated and testable independently.
