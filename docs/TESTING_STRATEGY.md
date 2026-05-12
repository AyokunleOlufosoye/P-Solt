# Testing Strategy

P-Solt emphasizes deterministic, reproducible testing. This document describes the testing approach for each module.

## Principles

### Determinism
All tests are deterministic. Running the same test multiple times produces the same result. No randomness, no network I/O, no luck.

### Isolation
Each module is tested independently. Session tests do not depend on bar builder; feature tests do not depend on strategy logic. This enables quick diagnosis of failures.

### Reproducibility
Test data and expected outputs are baked into test code. Tests can be run on any machine, any time, with identical results.

### Clarity
Tests are readable. Expected behavior is obvious. Test names describe what is being tested, not how.

## Unit Tests by Module

### Session Service Tests (`tests/test_session_service.py`)

**Validates:**
- Session classification: RTH, pre-market, post-market, overnight.
- Boundary conditions: market open, market close, midnight.
- Trading date computation.
- Timezone handling.

**Example:**
```python
def test_rth_classification():
    """Verify RTH is 09:30-16:00 in target timezone."""
    # 09:30 CT -> RTH
    # 09:29 CT -> pre-market
    # 16:01 CT -> post-market
```

**Determinism check:** Running test 100 times produces identical results.

### Bar Builder Tests (`tests/test_bar_builder.py`)

**Validates:**
- OHLCV aggregation correctness (max of highs, min of lows, sum of volume, etc.).
- Timestamp semantics (start time, end time, period alignment).
- Boundary cases (single bar, partial period).
- Determinism across runs.

**Example:**
```python
def test_5min_to_1hour_aggregation():
    """Verify 12 x 5-min bars aggregate to 1 hour correctly."""
    # Input: 12 bars of 5 min each, specific OHLCV
    # Expected: 1 bar spanning the hour, with correct OHLCV
```

**Determinism check:** Aggregating identical bars always produces identical result.

### Feature Pipeline Tests (`tests/test_feature_pipeline.py`)

**Validates:**
- Feature computation correctness (SMA, volatility, time-of-day factors).
- Version tagging.
- Edge cases (insufficient history, no variation).
- Determinism.

**Example:**
```python
def test_sma_computation():
    """Verify 20-bar SMA is computed correctly."""
    # Input: 30 bars with known values
    # Expected: SMA values match manual calculation
```

**Determinism check:** Computing features from identical bars always produces identical vectors.

### Position Manager Tests (`tests/test_position_manager.py`)

**Validates:**
- Opening a LONG or SHORT position.
- Closing a position and computing P&L correctly.
- P&L is correct for both LONG and SHORT.
- Position state transitions.

**Example:**
```python
def test_long_position_profit():
    """Verify P&L for LONG position: entry 100, exit 110 = +10."""
    # Open LONG at 100, size 1
    # Close at 110
    # Expected P&L: +10

def test_short_position_loss():
    """Verify P&L for SHORT position: entry 100, exit 110 = -10."""
    # Open SHORT at 100, size 1
    # Close at 110
    # Expected P&L: -10
```

**Determinism check:** Opening/closing identical positions always produces identical P&L.

## Test Execution

```bash
# Run all tests with verbose output
pytest tests/ -v

# Run a specific test module
pytest tests/test_session_service.py -v

# Run a specific test
pytest tests/test_session_service.py::test_rth_classification -v
```

All tests pass deterministically. No warnings, no skips.

## Parity & Shadow Replay Patterns

While full integration tests are outside this repo's scope, the architecture supports parity validation:

### Conceptual Shadow Replay

In a production system, you might:
1. Record a live market session (bars, timestamps, signals).
2. Replay the session through the research codebase (Python).
3. Compare research signals to production signals.
4. Discrepancies indicate bugs or drift.

P-Solt's deterministic modules enable this:
- Session classification is reproducible.
- Bar aggregation is reproducible.
- Feature computation is reproducible.
- Signal generation is reproducible (if seeded correctly).

### Conceptual Cross-System Validation

If you port logic to a different language (e.g., C++), you can:
1. Run the same bars through Python version.
2. Run the same bars through C++ version.
3. Compare features and signals.
4. Parity means identical logic was implemented correctly.

P-Solt's typed data contracts make this feasible.

## Coverage Goals

- **Unit tests:** 100% coverage of public module interfaces.
- **Edge cases:** Boundary conditions, empty inputs, extreme values.
- **Determinism:** Every test run produces identical output.

## Future Extensibility

As the system grows, you might add:

- **Integration tests:** End-to-end workflows from data ingestion to signal generation.
- **Property-based tests:** Using libraries like Hypothesis to auto-generate test cases.
- **Performance tests:** Ensuring modules run within acceptable time/memory.
- **Golden data tests:** Replaying historical sessions and comparing to archived expected outputs.

But the foundation remains: deterministic, isolated unit tests for each module.
