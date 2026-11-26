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
Represent `HandCategory` and `Rank` (2–14) as `IntEnum`. `HandCategory` assigns
one integer per category in rank order: `HIGH_CARD = 1` through `ROYAL_FLUSH = 10`.
Tiebreakers are expressed as tuples of integers so that Python's built-in tuple
comparison resolves all ties without special-case branching.

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
- `HandCategory` values run 1–10 in strict rank order and must never change
  once agreed (changing an integer breaks comparisons silently).
- `ROYAL_FLUSH = 10` with an empty tiebreaker `()` means two Royal Flushes are
  always a tie, which is correct.

---

## ADR-004: Treat Royal Flush as a Distinct Hand Category

**Status:** Accepted

**Context:**
The standard kata on codingdojo.org lists nine hand categories, with Straight
Flush as the highest. The project requirement (Chapter 1, FR-2) adds Royal Flush
(10-J-Q-K-A of the same suit) as a tenth, distinct category ranked above all
Straight Flushes, giving a clean `STRAIGHT_FLUSH = 9`, `ROYAL_FLUSH = 10`.

**Decision:**
Add `ROYAL_FLUSH = 10` to `HandCategory`. The ranker detects it before checking
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

---

## ADR-006: Treat Ace-Low Straight (A-2-3-4-5) as the Lowest Straight

**Status:** Accepted

**Context:**
`Rank.ACE = 14` in the standard ordering. Without special handling, A-2-3-4-5
would not be detected as a straight (the gap between 14 and 2 is not 1), and
even if detected some other way, the tiebreaker would rank it above 10-J-Q-K, which
is wrong. The kata spec (Chapter 1, FR-2) requires this hand to be valid and to
rank as the lowest straight.

**Decision:**
The ranker checks for the Ace-low straight explicitly: when the sorted ranks are
`[2, 3, 4, 5, 14]`, the hand is classified as a Straight and the tiebreaker key
treats the Ace as rank 1 (i.e. the key is `(5, 4, 3, 2, 1)`) so it sorts below
all other straights.

**Rationale:**
- Explicit detection is safer than re-mapping `Rank.ACE` globally (changing its
  integer value would break all rank comparisons elsewhere).
- Using `1` in the tiebreaker key (not `Rank.ACE = 14`) correctly places the
  wheel below 2-3-4-5-6.
- Alternatives considered:
  - *Add `Rank.LOW_ACE = 1`*: introduces a duplicate Ace concept and complicates
    card construction and display.
  - *Normalize rank list before straight detection*: same effect as the chosen
    approach but less explicit about why the substitution happens.

**Consequences:**
- The ranker must handle `[2, 3, 4, 5, 14]` as a special case before the
  general consecutive-ranks check.
- The tiebreaker key for a wheel is `(5, 4, 3, 2, 1)`, not `(14, 5, 4, 3, 2)`.
- Tests must include at least one Ace-low straight scenario and one that confirms
  it loses to 2-3-4-5-6.
