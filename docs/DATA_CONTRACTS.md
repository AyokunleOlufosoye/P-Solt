# Data Contracts

This document defines the generic type schemas used throughout P-Solt. These types are the "contract" between layers: each layer depends on well-defined input and output types.

## Overview

All types are defined in `src/types.py` using Python dataclasses. Types are:
- **Immutable where possible** (frozen dataclasses).
- **Type-hinted** for IDE and static analysis support.
- **Minimal** (no fields beyond what is needed for the generic reference architecture).
- **Production-extensible** (fields can be added without breaking this schema).

## Type Definitions

### Bar

```python
from enum import Enum
from dataclasses import dataclass
from datetime import datetime

class BarPeriod(Enum):
    """Standard bar periods."""
    MINUTE_1 = "1m"
    MINUTE_5 = "5m"
    MINUTE_15 = "15m"
    MINUTE_60 = "60m"
    DAILY = "D"

@dataclass(frozen=True)
class Bar:
    """
    OHLCV bar (candlestick).
    
    Attributes:
        symbol: Instrument identifier (e.g., "ES", "NQ").
        timestamp: Bar close time (UTC or localized).
        open: Opening price.
        high: Highest price in the period.
        low: Lowest price in the period.
        close: Closing price.
        volume: Shares or contracts traded.
        period: Bar period (e.g., 1m, 5m, 60m).
    """
    symbol: str
    timestamp: datetime
    open: float
    high: float
    low: float
    close: float
    volume: int
    period: BarPeriod
```

**Usage:** Data layer emits bars; feature and strategy layers consume bars.

---

### SessionContext

```python
class SessionType(Enum):
    """Market session types."""
    RTH = "rth"  # Regular Trading Hours (09:30-16:00 ET)
    PRE = "pre"  # Pre-market (04:00-09:30 ET)
    POST = "post"  # Post-market (16:00-20:00 ET)
    OVERNIGHT = "overnight"  # Overnight (20:00-04:00 ET)

@dataclass(frozen=True)
class SessionContext:
    """
    Trading session information for a given timestamp.
    
    Attributes:
        timestamp: The timestamp being classified.
        session_type: RTH, pre-market, post-market, or overnight.
        trading_date: The "market date" (trading day). Used for overnight sessions.
        timezone: Timezone string (e.g., "America/Chicago").
    """
    timestamp: datetime
    session_type: SessionType
    trading_date: datetime
    timezone: str
```

**Usage:** Session layer emits; data and strategy layers consume for context.

---

### FeatureVector

```python
from typing import Dict, Any

@dataclass(frozen=True)
class FeatureVector:
    """
    Computed features at a point in time.
    
    Attributes:
        timestamp: Time at which features were computed.
        features: Dictionary of feature names to values.
        version: Version tag for reproducibility (e.g., "v1", "v2").
    
    Notes:
        - Generic features dict allows extensibility.
        - Version tag enables historical analysis to specify exact feature definition.
    """
    timestamp: datetime
    features: Dict[str, Any]  # e.g., {"sma_fast": 100.5, "volatility": 2.3, "hour_of_day": 10}
    version: str  # e.g., "v1"
```

**Usage:** Feature layer emits; strategy layer consumes.

---

### Signal

```python
class SignalDirection(Enum):
    """Signal intent."""
    BUY = "buy"
    SELL = "sell"
    HOLD = "hold"
    EXIT = "exit"  # Close an existing position

@dataclass(frozen=True)
class Signal:
    """
    Trading signal (intent) from the strategy layer.
    
    Attributes:
        timestamp: Time at which signal was generated.
        direction: BUY, SELL, HOLD, or EXIT.
        confidence: Confidence in signal (0.0 to 1.0). Optional hint for position sizing.
        reason: Human-readable reason (e.g., "SMA crossover"). For logging/debugging.
    """
    timestamp: datetime
    direction: SignalDirection
    confidence: float  # 0.0 to 1.0
    reason: str
```

**Usage:** Strategy layer emits; execution layer consumes (or risk/position manager).

---

### Position

```python
class PositionSide(Enum):
    """Position direction."""
    LONG = "long"
    SHORT = "short"

@dataclass
class Position:
    """
    Open position.
    
    Attributes:
        side: LONG or SHORT.
        entry_price: Price at which position was opened.
        size: Number of shares/contracts.
        entry_time: Timestamp when position was opened.
        symbol: Instrument identifier.
        pnl: Current unrealized P&L. Can be None if not computed.
    
    Notes:
        - Position is mutable (updated as market price changes).
        - pnl can be computed as (current_price - entry_price) * size for LONG,
          or (entry_price - current_price) * size for SHORT.
    """
    side: PositionSide
    entry_price: float
    size: float
    entry_time: datetime
    symbol: str
    pnl: float = None
```

**Usage:** Position manager maintains; strategy layer queries for P&L, exit logic.

---

## Extension & Compatibility

Production systems often extend these types:

- **Bar:** Add `bid`, `ask`, `bid_size`, `ask_size` for quote data.
- **SessionContext:** Add `is_holiday`, `is_earnings_day` for special logic.
- **FeatureVector:** Add any domain-specific features.
- **Signal:** Add `target_price`, `stop_loss` for limit order hints.
- **Position:** Add `commission`, `slippage`, `bid_ask_spread` for accounting.

Extensions do not break this schema. Old code continues to work; new code uses new fields as needed.

## Relationships

```
Bar (input) → BarBuilder → Bar (output)
   ↓
SessionService → SessionContext (input)
   ↓
FeaturePipeline (Bar + SessionContext) → FeatureVector
   ↓
SignalGenerator (FeatureVector) → Signal
   ↓
PositionManager (Signal + price data) → Position
   ↓
ExecutionLayer (Position) → Orders (output)
```

Each layer transforms its input type to an output type according to clear contracts.
