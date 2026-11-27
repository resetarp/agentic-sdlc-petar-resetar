# 4. Solution Strategy

## 4.1 Approach

The problem is pure computation: parse text, apply poker rules, compare results,
print a verdict. The solution strategy follows directly from that:

1. **Model the domain first.** Cards, hands, and hand categories are the core
   concepts. Getting their representation right makes every other layer simple.
2. **Separate concerns into a small pipeline.** Input text flows through four
   distinct stages - parse → classify → compare → format - each testable in
   isolation.
3. **Keep the technology boring.** Python's standard library (`dataclasses`,
   `enum`, `re`) is sufficient. No framework is needed and none is used.
4. **Drive everything with tests.** Gherkin scenarios (pytest-bdd) express
   acceptance criteria; unit tests cover individual components.

## 4.2 Key Technology Decisions

| Decision | Choice | Reference |
|----------|--------|-----------|
| Language | Python 3.12 | ADR-001 |
| Runtime dependencies | None (stdlib only) | ADR-001 |
| Test framework | pytest + pytest-bdd | ADR-001 |
| Domain value types | `dataclasses` + `enum.IntEnum` | ADR-001, CV-2 |
| Linting / formatting | ruff, black, isort | CV-1 |

## 4.3 Decomposition Approach

The system is decomposed into five modules that form a linear pipeline:

```
         stdin
           │
           ▼
    ┌─────────────┐
    │   parser    │
    └──────┬──────┘
           │  Hand
           ▼
    ┌─────────────┐     ┌──────────────┐
    │   ranker    │◀──  │  card / hand │
    └──────┬──────┘     │  (domain)    │
           │  RankedHand└──────────────┘
           ▼
    ┌─────────────┐
    │  comparator │
    └──────┬──────┘
           │  Result
           ▼
    ┌─────────────┐
    │  formatter  │
    └──────┬──────┘
           │
           ▼
         stdout
```

| Module | Responsibility |
|--------|----------------|
| `card` | `Card` value object (rank + suit); `Rank` and `Suit` as `IntEnum` |
| `hand` | `Hand` value object (5 cards + player name); construction from a list of `Card` |
| `parser` | Converts a raw input line into two `Hand` instances; raises on malformed input |
| `ranker` | Classifies a `Hand` into a `HandRank` (category + ordered tiebreaker key); contains all poker rules |
| `comparator` | Accepts two `RankedHand` values and returns a `Result` (winner name or tie) |
| `formatter` | Converts a `Result` into the kata's output string |
| `cli` | Entry point: reads stdin, calls parser → ranker → comparator → formatter, writes stdout |

## 4.4 Top-Level Patterns

### Value objects and IntEnum for domain types

`Card` and `Hand` are immutable `dataclasses`. `Rank` (2-A) and `Suit` (C/D/H/S)
are `IntEnum`, so rank comparison is a plain `>` / `<`. `HandCategory` is also
`IntEnum`, so category comparison is likewise a plain `>` / `<`.

`HandCategory` runs from `HIGH_CARD = 1` to `ROYAL_FLUSH = 10`, one integer per
category in rank order: High Card, Pair, Two Pairs, Three of a Kind, Straight,
Flush, Full House, Four of a Kind, Straight Flush, Royal Flush.

### Tiebreaker key

`ranker` produces a tiebreaker key - a tuple of integers - alongside the
category. Python tuple comparison handles all tie-breaking automatically:
the first differing element decides the winner, with no special-case branching
in the comparator.

For example:
- **Pair** key: `(pair_rank, kicker3, kicker2, kicker1)` - highest pair first,
  then kickers in descending order.
- **Two Pairs** key: `(high_pair_rank, low_pair_rank, kicker)`.
- **Full House** key: `(three_rank, pair_rank)`.
- **High Card / Flush** key: `(card5, card4, card3, card2, card1)` descending.

### Royal Flush as a distinct category

A Royal Flush (10-J-Q-K-A of the same suit) is classified as `ROYAL_FLUSH = 10`,
one step above `STRAIGHT_FLUSH = 9`. Because it is always the highest possible
hand, its tiebreaker key is the empty tuple `()` - two Royal Flushes are always
a tie.

### Ace-low straight

An Ace may act as a low card (value 1) to form the straight A-2-3-4-5. This is
the lowest-ranked straight and its tiebreaker key treats the Ace as rank 1, not
14, so it sorts below 2-3-4-5-6. The ranker must check for this case explicitly
when detecting straights, because `Rank.ACE = 14` in the standard ordering.

### Pipeline, not layers

The five modules form a pipeline, not a layered architecture. Each module
depends only on the modules to its left (domain types) and nothing downstream.
This keeps the dependency graph acyclic and each module independently testable.

## 4.5 Testing Strategy

| Level | Tool | What it covers |
|-------|------|----------------|
| Acceptance | pytest-bdd (Gherkin) | All ten hand categories, all tie-breaking rules, the kata's sample cases, and Ace-low straight |
| Unit | pytest | Each module in isolation: `parser`, `ranker`, `comparator`, `formatter` |
| Integration | pytest | Full pipeline from raw string to output string |

All tests are hermetic: no I/O side-effects, no network, no filesystem writes.
