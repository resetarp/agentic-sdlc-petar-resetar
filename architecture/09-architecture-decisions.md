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

## ADR-002: Decompose into a Five-Stage Pipeline of Pure Modules

**Status:** Accepted

**Context:**
The kata requires parsing text input, applying poker rules, comparing two hands,
and formatting output. These are four distinct concerns. Without an explicit
structure, they risk collapsing into one large function that is hard to test and
reason about.

**Decision:**
Decompose the system into five modules forming a linear pipeline:
`parser` → `ranker` → `comparator` → `formatter`, orchestrated by `cli`.
Two domain modules (`card`, `hand`) underpin the pipeline. Dependencies flow
in one direction only; no cycles are permitted.

**Rationale:**
- Each stage has a single, clear responsibility and can be unit-tested in isolation.
- A linear dependency graph makes the system easy to follow top-to-bottom.
- Alternatives considered:
  - *Single-module script*: simple to start but conflates I/O, rules, and formatting — hard to test individual rules.
  - *Class hierarchy per hand category*: adds polymorphism overhead that is not justified for a kata of this size.

**Consequences:**
- Component boundaries are enforced by convention (no framework enforces them).
- Adding a new hand category requires changes only in `ranker`; all other modules are unaffected.

---

## ADR-003: Represent Hand Categories and Card Ranks as IntEnum

**Status:** Accepted

**Context:**
Hand comparison and tiebreaking both require ordered comparisons between hand
categories (e.g. Full House beats Flush) and between card ranks (e.g. Ace beats
King). These comparisons must be correct, readable, and free of magic numbers.

**Decision:**
Represent `HandCategory` (1–9) and `Rank` (2–14) as `IntEnum`. Tiebreakers are
expressed as tuples of integers so that Python's built-in tuple comparison
resolves all ties without special-case branching.

**Rationale:**
- `IntEnum` makes `HandCategory.FULL_HOUSE > HandCategory.FLUSH` a plain `>`
  with no helper function needed.
- Tiebreaker tuples delegate all comparison logic to Python's tuple ordering —
  the comparator needs no poker knowledge.
- Alternatives considered:
  - *Plain integers with constants*: loses type safety and IDE support.
  - *String-based categories*: requires a separate ordering map; error-prone.
  - *Custom `__lt__` on a dataclass*: more code, same result.

**Consequences:**
- `HandCategory` values must be assigned in strict rank order and never changed
  once agreed (changing an integer breaks comparisons silently).
- `ROYAL_FLUSH = 9` with an empty tiebreaker `()` means two Royal Flushes are
  always a tie, which is correct.

---

## ADR-004: Treat Royal Flush as a Distinct Hand Category

**Status:** Accepted

**Context:**
The standard kata on codingdojo.org lists eight hand categories, with Straight
Flush as the highest. The project requirement (Chapter 1, FR-2) adds Royal Flush
(10-J-Q-K-A of the same suit) as a ninth, distinct category ranked above all
Straight Flushes.

**Decision:**
Add `ROYAL_FLUSH = 9` to `HandCategory`. The ranker detects it before checking
for a Straight Flush. Its tiebreaker key is the empty tuple `()` because two
Royal Flushes are always equal.

**Rationale:**
- Makes the category explicit in the domain model and in output messages, rather
  than treating it as a special case of Straight Flush.
- Detection order (Royal Flush checked first) prevents a 10-J-Q-K-A flush from
  being misclassified as a Straight Flush.
- Alternatives considered:
  - *Straight Flush with Ace-high tiebreaker*: technically equivalent at runtime
    but loses the explicit Royal Flush label in output and in the domain model.

**Consequences:**
- Output must use the label "royal flush" for this category, not "straight flush".
- Any future extension (e.g. Texas Hold'em) already has the correct hierarchy.

---

## ADR-005: Validate Input Exclusively in the Parser

**Status:** Accepted

**Context:**
Invalid input (wrong card count, unknown token, duplicate card) must be caught
and reported clearly. The question is where validation lives.

**Decision:**
All input validation happens inside `parser`. Downstream modules (`ranker`,
`comparator`, `formatter`) treat their inputs as guaranteed valid and perform no
defensive checks.

**Rationale:**
- Validates once at the trust boundary; avoids scattered defensive code.
- Keeps domain modules free of error-handling noise, making them easier to read
  and test.
- Alternatives considered:
  - *Validate in each module*: redundant checks, inconsistent error messages.
  - *Validate in `cli`*: couples I/O logic to domain validation rules.

**Consequences:**
- `parser` must be comprehensive; any gap allows bad data to propagate silently.
- Domain modules may use `assert` for internal invariants in debug mode, but
  must not rely on it for user-facing validation.
