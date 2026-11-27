# 12. Glossary

Domain terms used in this documentation, the codebase, and the Gherkin feature
files. The same term always means the same thing across all three.

## Card Notation

| Term | Definition |
|------|------------|
| **Rank** | The face value of a card. One of `2 3 4 5 6 7 8 9 T J Q K A`, ordered lowest to highest. `T` = Ten, `J` = Jack, `Q` = Queen, `K` = King, `A` = Ace. |
| **Suit** | The symbol on a card. One of `C` (Clubs), `D` (Diamonds), `H` (Hearts), `S` (Spades). Suits are unordered; they are only compared for equality. |
| **Card** | A single playing card, identified by a Rank and a Suit. Written as a two-character token, e.g. `AH` = Ace of Hearts, `TC` = Ten of Clubs. |
| **Hand** | A set of exactly 5 cards dealt to one player, together with that player's name. |
| **Deck** | A standard 52-card deck (13 ranks × 4 suits). No jokers or wild cards. |

## Hand Categories

Listed in ascending rank order (lowest to highest):

| Term | Definition |
|------|------------|
| **High Card** | No other category applies. The hand is ranked by its highest card; if equal, by the next highest, and so on. |
| **Pair** | Exactly two cards share the same rank. Ranked by the pair's rank, then by kicker cards in descending order. |
| **Two Pairs** | Two distinct pairs. Ranked by the highest pair, then the lower pair, then the remaining kicker card. |
| **Three of a Kind** | Exactly three cards share the same rank. Ranked by that rank. |
| **Straight** | Five cards with consecutive ranks (suits may differ). Ranked by the highest card. |
| **Flush** | Five cards of the same suit (ranks need not be consecutive). Ranked using High Card rules. |
| **Full House** | Three cards of one rank and two cards of another rank. Ranked by the three-of-a-kind rank, then the pair rank. |
| **Four of a Kind** | Four cards sharing the same rank. Ranked by that rank. |
| **Straight Flush** | Five consecutive ranks all of the same suit, other than 10-J-Q-K-A (that is a Royal Flush). Includes the Ace-low A-2-3-4-5 of one suit, where the Ace counts as 1 (ADR-006). Ranked by the highest card. |
| **Royal Flush** | A Straight Flush with ranks 10-J-Q-K-A. The highest possible hand; two Royal Flushes are always a tie. |

## Pipeline Terms

| Term | Definition |
|------|------------|
| **Parser** | The module (`parser.py`) that converts a raw input line into two `Hand` instances. The only place where user input is validated. |
| **Ranker** | The module (`ranker.py`) that classifies a `Hand` into a `HandCategory` and computes its tiebreaker key. Contains all poker rules. |
| **Comparator** | The module (`comparator.py`) that accepts two `RankedHand` values and returns a `Result`. Contains no poker rules. |
| **Formatter** | The module (`formatter.py`) that converts a `Result` into the kata's required output string. |
| **CLI** | The entry point module (`cli.py`) that reads stdin, orchestrates the pipeline, and writes stdout. |
| **Tiebreaker key** | A `tuple[int, ...]` produced by the ranker alongside the `HandCategory`. Python tuple comparison resolves all ties without special-case branching in the comparator. |
| **RankedHand** | A `Hand` annotated with its `HandCategory` and tiebreaker key. The unit of comparison. |
| **Result** | The output of the comparator: either the winning player's name and the reason, or a tie indicator. |

## Testing Terms

| Term | Definition |
|------|------------|
| **BDD** | Behaviour-Driven Development. Acceptance criteria written as Gherkin scenarios (`Given` / `When` / `Then`) and executed by pytest-bdd. |
| **Gherkin** | The plain-language format used to write BDD scenarios in `.feature` files. |
| **Step definition** | A Python function decorated with `@given`, `@when`, or `@then` that maps a Gherkin step to executable code. |
| **Hermetic test** | A test that has no I/O side-effects, no network calls, and no filesystem writes. All tests in this project must be hermetic. |
