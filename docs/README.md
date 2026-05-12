# Documentation

P-Solt documentation provides a complete picture of the architecture, design rationale, testing strategy, and research methodology.

## Core Documents

### [SYSTEM_DESIGN.md](SYSTEM_DESIGN.md)
Architecture overview, design goals, and the layered structure of P-Solt:
- Session layer (time normalization)
- Data layer (bar building, adapters)
- Feature layer (computation pipeline)
- Strategy layer (signals, positions)
- Execution layer (abstract stubs)
- Data contract patterns
- Relationship to the private system

### [RESEARCH_METHODOLOGY.md](RESEARCH_METHODOLOGY.md)
Principles underlying disciplined quantitative research and validation:
- Walk-forward validation thinking
- Separation of feature computation from interpretation
- Parity as a validation principle
- Why public repos show workflow, not edge
- Clear notes on what is not being published

### [TESTING_STRATEGY.md](TESTING_STRATEGY.md)
Approach to unit testing, determinism, and reproducibility:
- Unit tests for each module
- Parity and shadow replay patterns
- Deterministic test design
- Coverage goals

### [DATA_CONTRACTS.md](DATA_CONTRACTS.md)
Generic type definitions and data schemas used throughout the system:
- Bar
- SessionContext
- FeatureVector
- Signal
- Position
- No proprietary fields; designed for educational clarity

---

**Start with SYSTEM_DESIGN.md for an overview, then explore specific topics as needed.**
