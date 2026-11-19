# 8. Cross-Cutting Concepts

Cross-cutting concepts are patterns and rules that apply to more than one
component. Because Poker Hands is a small, single-process CLI with no network
or persistence, the set is deliberately minimal.

## 8.1 Domain Model

The shared vocabulary and type system used by every module:

| Type | Kind | Description |
|------|------|-------------|
| `Rank` | `IntEnum` (2–14) | Card face value; `Rank.TWO = 2` through `Rank.ACE = 14`. Integer value enables plain `>` / `<` comparison. |
| `Suit` | `IntEnum` | One of `CLUBS`, `DIAMONDS`, `HEARTS`, `SPADES`. Integer ordering is arbitrary; suits are only compared for equality (flush detection). |
| `Card` | frozen `dataclass` | A `(Rank, Suit)` pair. Immutable; hashable. |
| `Hand` | frozen `dataclass` | A player name and a `tuple[Card, ...]` of exactly 5 cards. |
| `HandCategory` | `IntEnum` (1–9) | Ordered hand categories: `HIGH_CARD = 1` … `ROYAL_FLUSH = 9`. |
| `RankedHand` | frozen `dataclass` | A `Hand` annotated with its `HandCategory` and a tiebreaker key `tuple[int, ...]`. |
| `Result` | frozen `dataclass` | Holds the winner's name and the winning `HandCategory` with reason, or a tie flag. |

All domain types live in `card` and `hand`; `HandCategory`, `RankedHand`, and
`Result` live in `ranker` and `comparator` respectively. No module outside the
domain layer re-defines these types.

## 8.2 Error Handling

The system has one meaningful error path: invalid input from the user.

| Situation | Where detected | Behaviour |
|-----------|---------------|-----------|
| Wrong number of cards in a hand | `parser` | Raises `ValueError` with a message naming the player and the actual count. |
| Unrecognised rank or suit token | `parser` | Raises `ValueError` naming the bad token. |
| Duplicate cards within a hand | `parser` | Raises `ValueError` naming the duplicate. |
| Any other unexpected exception | `cli` | Caught by the top-level handler; prints `Error: <message>` to stdout and continues to the next line. |

Rules:
- Only `cli` writes to stdout; all inner components raise exceptions and never
  print directly.
- Errors are reported per line; a bad line does not abort processing of
  subsequent lines.
- No stack traces are shown to the user.

## 8.3 Validation

Input is validated once, at the system boundary, inside `parser`. No downstream
component re-validates:

1. The input line matches the pattern `<Name>: C1 C2 C3 C4 C5  <Name>: C1 C2 C3 C4 C5`.
2. Each token is a valid `(Rank, Suit)` pair from the kata's notation.
3. Each hand contains exactly 5 cards.
4. No card appears twice in the same hand.

After `parser` returns two `Hand` instances, all subsequent components can
treat their input as valid without defensive checks.

## 8.4 Immutability

All domain value objects (`Card`, `Hand`, `RankedHand`, `Result`) are frozen
`dataclass` instances. They are:

- Created once and never mutated.
- Safe to pass between functions without defensive copying.
- Hashable, so they can be used as dict keys or set members in tests.

## 8.5 Testability

Every component is independently testable because:

- Pure functions only: no global state, no I/O inside domain modules.
- Dependencies are passed in, not imported as singletons.
- `cli` is the only module that touches stdin/stdout; it is tested via subprocess
  or by dependency-injecting a string buffer.
- Gherkin feature files in `features/` express acceptance criteria; step
  definitions in `tests/` wire them to the public API.

## 8.6 Logging

No logging framework is used. The program is a short-lived CLI process; the
only output it produces is the result line (or an error message) written to
stdout. Adding a logging framework would exceed the complexity budget of the
kata.

## 8.7 Code Style Enforcement

Enforced automatically in CI on every commit:

| Tool | Purpose | Configuration |
|------|---------|---------------|
| `ruff` | Linting (style, bugs, complexity) | `pyproject.toml` — rules E, F, W, I, N, UP, B, SIM |
| `black` | Formatting | `pyproject.toml` — line length 88, target Python 3.12 |
| `isort` | Import ordering | `pyproject.toml` — black-compatible profile |
