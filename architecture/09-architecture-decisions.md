# 9. Architecture Decisions

## ADR-001: Use a Python Standard-Library CLI with No Persistence

**Status:** Accepted

**Context:**
The Poker Hands kata needs to parse two 5-card hands, rank them, and report the winner.
It is a stateless calculation with no network, multi-user, or storage requirements.

**Decision:**
Use Python 3.12 with no application framework. The system is a command-line program
built on the standard library, keeps all data in memory, and is tested with pytest and pytest-bdd.

**Rationale:**
- Project will be simplest to implement with Python
- Problem is pure computation, so no framework or database is needed
- `enum` and `dataclasses` model ordered card ranks and hand categories cleanly
- No persistence keeps every run deterministic and easy to test
- Text in, text out is simple to drive from tests and CI

**Consequences:**
- No history of past games is kept
- No graphical or web interface
- Component boundaries must be kept by convention, not a framework
