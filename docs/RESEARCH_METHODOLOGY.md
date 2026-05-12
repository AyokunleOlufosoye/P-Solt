# Research Methodology

This document describes the principles underlying disciplined quantitative research and validation as embodied in P-Solt.

## Walk-Forward Validation Thinking

In quantitative trading research, the goal is to find and validate strategies that generalize to unseen data. The primary validation tool is **walk-forward analysis**: dividing historical data into training and test windows, training on the training window, and evaluating on the test window, repeating this process forward in time.

P-Solt is designed to support walk-forward workflows:

- **Deterministic session layer** → Reproducible market session classification across any date range.
- **Deterministic bar aggregation** → Identical OHLCV computation, enabling identical feature definitions across windows.
- **Feature versioning** → Historical analysis can specify exactly which feature code was used.
- **Parity principle** → Cross-validating research across code bases or systems ensures results are reliable, not accidental.

The architecture does not implement walk-forward backtesting (that is outside this repo's scope), but it provides the foundation—deterministic, reproducible modules—on which such tools are built.

## Separation of Feature Computation from Feature Interpretation

One of the most powerful principles in quantitative research is separating **what we compute** from **what we think it means**.

### Features Are Facts

A feature is a computed artifact. Examples:
- Simple Moving Average (SMA) over the last 20 bars.
- Realized volatility over the last 20 bars.
- Time of day (e.g., pre-market vs. RTH).

These are deterministic and objective. Given the same bars, computing a feature should always yield the same result.

### Interpretation Is Opinion

A signal is an interpretation: "SMA is above threshold → buy." This is a hypothesis. It may be wrong. Research is about testing hypotheses.

P-Solt enforces this separation:

- **Feature Layer** computes features, emits a FeatureVector.
- **Strategy Layer** consumes FeatureVector and emits a Signal.

Benefits:
- Features can be reused by different signal logic.
- Features can be backtested against many signal hypotheses without recomputation.
- Bugs in signal logic do not corrupt feature definitions.
- Researchers can focus on signal logic independently.

## Parity as a Validation Principle

**Parity** means: identical inputs should produce identical outputs, regardless of implementation language, codebase, or system.

### Why Parity Matters

1. **Debugging:** If two implementations disagree, parity testing isolates the discrepancy.
2. **Code porting:** Moving logic from research (Python) to production (C++) is risky. Parity tests ensure correctness.
3. **Cross-validation:** Running the same strategy logic on multiple backtesting systems should yield identical results. Disagreement signals a bug.
4. **Collaboration:** Multiple researchers can work on different modules and validate they integrate correctly via parity.

### P-Solt Enables Parity

Deterministic modules with explicit data contracts make parity tests feasible:

- **Session Service:** Test that RTH classification is identical across implementations.
- **Bar Builder:** Test that aggregation produces identical OHLCV.
- **Feature Pipeline:** Test that feature computation is identical.
- **Signal Generator:** Test that signal generation is identical.

Each module can be tested in isolation. If parity fails, the failed module is pinpointed.

## Why Public Repos Show Workflow, Not Edge

Quantitative traders often ask: "Can I release code without revealing my edge?"

Yes—by publishing **architecture and methodology**, not **strategy parameters and alpha sources**.

### What We Publish

- How sessions are normalized.
- How bars are aggregated.
- How features are versioned.
- How modules are tested and validated.
- How code is organized for reproducibility.

### What We Don't Publish

- Which feature combinations are predictive.
- What thresholds generate alpha.
- Which market regimes our logic exploits.
- Walk-forward backtesting results.
- Live performance metrics or P&L.
- Anything that hints at our edge.

The public repo teaches *how to build rigorously*, not *what to trade*.

### Analogy: Academic Papers vs. Trade Secrets

Academic papers publish methodology, not reproduction. A researcher might publish, "We discovered that combining mean-reversion and trend-following improves Sharpe ratio," without publishing the exact thresholds, feature weights, or market conditions under which this works. The intellectual contribution is the *methodology*. The edge is in the *parameters and interpretation*.

P-Solt is analogous: we publish methodology (session layer, deterministic bar building, feature versioning, testing approach) and demonstration code (using obviously fake parameters), but we keep the actual alpha source private.

## Determinism as Foundation

All of the above principles depend on **deterministic code**:

- Walk-forward validation requires reproducible feature computation.
- Separation of features and interpretation is only useful if each is deterministic.
- Parity testing is only possible if modules are deterministic.

P-Solt prioritizes determinism:

- No randomness in session classification, bar building, or feature computation.
- No external I/O or network calls in core logic (data is passed as arguments).
- All tests are deterministic and reproducible.
- Pseudo-random number generation (if needed) is seeded for reproducibility.

## Limitations of This Approach

Publishing architecture without publishing edge has limits:

1. **Not a full system:** Readers see a skeleton, not a working strategy. They must fill in signal logic themselves.
2. **Not a research platform:** This repo is not designed for systematic backtesting or parameter optimization. That is beyond scope.
3. **Not a trading engine:** This repo does not connect to brokers, manage live positions, or handle execution.

These limitations are intentional. P-Solt is a *reference*, not a product.

## Further Reading

- See [`TESTING_STRATEGY.md`](TESTING_STRATEGY.md) for unit test design and parity validation patterns.
- See [`SYSTEM_DESIGN.md`](SYSTEM_DESIGN.md) for architecture and design decisions.
- See [`DATA_CONTRACTS.md`](DATA_CONTRACTS.md) for type definitions.
