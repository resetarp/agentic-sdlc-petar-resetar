# GAME Domain: Story Bundles

Domain: **GAME** (hand evaluation and comparison).
Owning modules: `card`, `hand`, `ranker`, `comparator`, `formatter`
(see Ch 4.3, Ch 5.3). This bundle also touches `parser` and `cli`, because
the first story has to deliver a working end-to-end path.

---

## GAME-STORY-001: Decide the winner by high card

**Priority**: Core (Pareto 20%)
**Value**: decides about 25% of all random games, and it builds the whole
pipeline that every later story depends on.
**Status**: Approved, not implemented

```
AS A player
I WANT to enter two hands that contain no combination and be told who wins and why
SO THAT I can settle the most common kind of game without checking the cards by hand
```

**Architecture references**

- Ch 1.2 FR-1, FR-2 (High Card only), FR-3, FR-4, FR-5
- Ch 3.3 External interfaces (input and output format)
- Ch 4.3 Decomposition, Ch 4.4 Tiebreaker key
- Ch 6.1 Scenario "one player wins", Ch 6.2 Scenario "tie"
- Ch 10.2 QS-1 (kata samples 1, 3 and 4), QS-3, QS-4 (High Card kicker)
- Ch 11.1 R-1 (tiebreaker key errors)

**Scope**

- In scope: hands whose category is High Card, ties, and the full path
  from stdin to stdout for well-formed input.
- Out of scope: every other category (GAME-STORY-002 to 005) and malformed
  input (INPUT-STORY-001). Until GAME-STORY-002 is built, the scenarios in
  this bundle use only hands with no pair, straight or flush.

### Acceptance scenarios (end to end, through the CLI)

### SCENARIO 1: Higher top card wins (kata sample 1)

**Scenario ID**: GAME-STORY-001-S1

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: 2H 3D 5S 9C KD  White: 2C 3H 4S 8C AH` is sent on stdin

**THEN**
* stdout contains exactly `White wins. - with high card: Ace`

### SCENARIO 2: Equal top card, decided by the next card (kata sample 3)

**Scenario ID**: GAME-STORY-001-S2

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: 2H 3D 5S 9C KD  White: 2C 3H 4S 8C KH` is sent on stdin

**THEN**
* stdout contains exactly `Black wins. - with high card: 9`

### SCENARIO 3: Same ranks in both hands is a tie (kata sample 4)

**Scenario ID**: GAME-STORY-001-S3

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: 2H 3D 5S 9C KD  White: 2D 3H 5C 9S KH` is sent on stdin

**THEN**
* stdout contains exactly `Tie.`

### SCENARIO 4: Decided by the lowest card

**Scenario ID**: GAME-STORY-001-S4

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: 2H 4D 5S 9C KD  White: 3C 4H 5D 9S KH` is sent on stdin

**THEN**
* stdout contains exactly `White wins. - with high card: 3`

---

### INFRA sub-stories

#### GAME-INFRA-001.1: Runnable package skeleton (deployment)

```
AS A developer
I WANT a `poker_hands` package that runs with `python3 -m poker_hands`
SO THAT there is somewhere to put the code and a way to run it
```

**Architecture references**: Ch 7.3 Developer workstation, Ch 7.4 Installation
steps, Ch 7.5 No release artefact, ADR-001, QS-8, TD-1

### SCENARIO 1: Package runs on a clean environment

**Scenario ID**: GAME-INFRA-001.1-S1

**GIVEN**
* a fresh virtual environment with Python 3.12 and only `pytest` and `pytest-bdd` installed

**WHEN**
* `python3 -m poker_hands` is run with empty stdin

**THEN**
* the process exits with status 0
* nothing is written to stdout

### SCENARIO 2: No runtime dependencies

**Scenario ID**: GAME-INFRA-001.1-S2

**GIVEN**
* the `[project]` section of `pyproject.toml`

**WHEN**
* its runtime `dependencies` are inspected

**THEN**
* the list is empty (standard library only)

### SCENARIO 3: Linting and formatting tools are declared in pyproject.toml

**Scenario ID**: GAME-INFRA-001.1-S3

**GIVEN**
* the repository contains a `pyproject.toml` with `ruff`, `black`, and `isort` configured

**WHEN**
* `ruff check .`, `black --check .`, and `isort --check .` are each run against the source tree

**THEN**
* each command exits with code 0

---

#### GAME-INFRA-001.2: Acceptance test harness

```
AS A developer
I WANT pytest-bdd set up with a `features/` folder and step definitions in `tests/`
SO THAT every GIVEN-WHEN-THEN scenario in this bundle can run as a test
```

**Architecture references**: Ch 4.5 Testing strategy, Ch 8.5 Testability,
Ch 12 Testing terms, QS-5

### SCENARIO 1: Feature files are discovered and run

**Scenario ID**: GAME-INFRA-001.2-S1

**GIVEN**
* `features/high_card.feature` contains GAME-STORY-001-S1 to S4

**WHEN**
* `pytest` is run from the repository root

**THEN**
* the four scenarios are collected and reported by their names

### SCENARIO 2: Suite is hermetic and fast

**Scenario ID**: GAME-INFRA-001.2-S2

**GIVEN**
* the full test suite

**WHEN**
* `pytest` is run with no network access

**THEN**
* all tests pass
* no files are created outside pytest's temporary directories
* the run takes less than 5 seconds

---

#### GAME-INFRA-001.3: CI pipeline

```
AS A code reviewer
I WANT every push checked by ruff, black, isort and pytest
SO THAT broken or badly formatted code cannot reach main unnoticed
```

**Architecture references**: Ch 7.3 CI runner, Ch 8.7 Code style
enforcement, QS-9

### SCENARIO 1: All checks run on push

**Scenario ID**: GAME-INFRA-001.3-S1

**GIVEN**
* a GitHub Actions workflow pinned to Python 3.12

**WHEN**
* a commit is pushed

**THEN**
* the steps run in this order: install dev dependencies, ruff, black --check, isort --check, pytest
* the workflow fails if any step fails

### SCENARIO 2: Pipeline is fast

**Scenario ID**: GAME-INFRA-001.3-S2

**GIVEN**
* the workflow from GAME-INFRA-001.3-S1

**WHEN**
* it runs on a fresh GitHub-hosted runner

**THEN**
* it finishes in less than 30 seconds

### SCENARIO 3: CI fails fast on a linting violation

**Scenario ID**: GAME-INFRA-001.3-S3

**GIVEN**
* the CI pipeline from GAME-INFRA-001.3-S1

**WHEN**
* a commit introduces a PEP 8 violation (for example, an unused import)

**THEN**
* `ruff check .` exits with a non-zero code
* the CI run is marked failed
* subsequent steps (including `pytest`) do not run

---

#### GAME-INFRA-001.4: Test report artefact (monitoring)

```
AS A code reviewer
I WANT CI to publish a JUnit XML test report
SO THAT I can see which scenario failed without reading raw logs
```

**Architecture references**: Ch 7.3 CI runner (Artefacts), Ch 8.6 Logging

### SCENARIO 1: Report is attached to every run

**Scenario ID**: GAME-INFRA-001.4-S1

**GIVEN**
* the workflow from GAME-INFRA-001.3

**WHEN**
* pytest finishes, whether it passed or failed

**THEN**
* a JUnit XML file is uploaded as a workflow artefact
* every scenario ID from this bundle appears in it as a test case name

---

#### Data store and event handling: not applicable

The standard INFRA checklist also asks for a data store and event handling.
This system has neither: there is no persistence (Ch 1.2 Out of scope,
ADR-001) and no message bus (Ch 3.1). No sub-stories are created for them.
GAME-INFRA-001.2-S2 checks that nothing gets written to disk.

---

### BE sub-stories

#### GAME-BE-001.1: Card, Rank and Suit value objects

```
AS A developer
I WANT immutable Card objects with ranks I can compare
SO THAT poker rules can compare cards with a plain `>`
```

**Architecture references**: Ch 5.3 `card`, Ch 8.1 Domain model, Ch 8.4
Immutability, ADR-003

### SCENARIO 1: Ranks compare by value

**Scenario ID**: GAME-BE-001.1-S1

**GIVEN**
* `Rank.ACE` and `Rank.KING`

**WHEN**
* they are compared

**THEN**
* `Rank.ACE > Rank.KING` is true
* `int(Rank.TWO) == 2` and `int(Rank.ACE) == 14`

### SCENARIO 2: Cards are immutable and hashable

**Scenario ID**: GAME-BE-001.1-S2

**GIVEN**
* `Card(Rank.KING, Suit.DIAMONDS)`

**WHEN**
* code tries to assign to its `rank`, and the card is put into a set together with an equal card

**THEN**
* the assignment raises `dataclasses.FrozenInstanceError`
* the set contains one element

---

#### GAME-BE-001.2: Hand value object

```
AS A developer
I WANT a Hand that always holds exactly five cards and a player name
SO THAT later stages never need to check the card count
```

**Architecture references**: Ch 5.3 `hand`, Ch 8.1 Domain model, Ch 8.3
Validation

### SCENARIO 1: A valid hand is built

**Scenario ID**: GAME-BE-001.2-S1

**GIVEN**
* the player name "Black" and five distinct cards

**WHEN**
* a `Hand` is constructed

**THEN**
* `hand.player == "Black"`
* `len(hand.cards) == 5`

### SCENARIO 2: Wrong card count is rejected

**Scenario ID**: GAME-BE-001.2-S2

**GIVEN**
* the player name "Black" and four cards

**WHEN**
* a `Hand` is constructed

**THEN**
* a `ValueError` is raised

---

#### GAME-BE-001.3: Parse a well-formed input line

```
AS A developer
I WANT `parse_line` to turn the kata notation into two Hand objects
SO THAT the rest of the pipeline works with domain types instead of text
```

**Architecture references**: Ch 3.3 Player to Poker Hands, Ch 5.3 `parser`,
Ch 12 Card notation. Error cases belong to INPUT-STORY-001 (Ch 8.2, ADR-005).

### SCENARIO 1: Kata line is parsed

**Scenario ID**: GAME-BE-001.3-S1

**GIVEN**
* the line `Black: 2H 3D 5S 9C KD  White: 2C 3H 4S 8C AH`

**WHEN**
* `parse_line` is called

**THEN**
* the first hand belongs to "Black" and its first card is `Card(Rank.TWO, Suit.HEARTS)`
* the second hand belongs to "White" and its last card is `Card(Rank.ACE, Suit.HEARTS)`

### SCENARIO 2: Ten is written as T

**Scenario ID**: GAME-BE-001.3-S2

**GIVEN**
* a line where Black holds `TS`

**WHEN**
* `parse_line` is called

**THEN**
* Black's hand contains `Card(Rank.TEN, Suit.SPADES)`

---

#### GAME-BE-001.4: Classify High Card with a tiebreaker key

```
AS A developer
I WANT `rank_hand` to return HIGH_CARD with the card ranks in descending order as the key
SO THAT comparing two High Card hands is a single tuple comparison
```

**Architecture references**: Ch 4.4 Tiebreaker key, Ch 5.3 `ranker`, Ch 8.1
`HandCategory`, `RankedHand`, ADR-003, QS-6, R-1

### SCENARIO 1: Key is sorted descending

**Scenario ID**: GAME-BE-001.4-S1

**GIVEN**
* a `Hand` built directly (no parser) with 2H 3D 5S 9C KD

**WHEN**
* `rank_hand` is called

**THEN**
* the category is `HandCategory.HIGH_CARD`
* the tiebreaker key is `(13, 9, 5, 3, 2)`

### SCENARIO 2: Card order in the input does not matter

**Scenario ID**: GAME-BE-001.4-S2

**GIVEN**
* the same five cards as GAME-BE-001.4-S1 in the order KD 2H 9C 5S 3D

**WHEN**
* `rank_hand` is called

**THEN**
* the result equals the result of GAME-BE-001.4-S1

---

#### GAME-BE-001.5: Compare two ranked hands

```
AS A developer
I WANT `compare` to return a Result naming the winner and the deciding card, or a tie
SO THAT the formatter has everything it needs to explain the verdict
```

**Architecture references**: Ch 5.3 `comparator`, Ch 8.1 `Result`, ADR-002.
The comparator owns the reason: it records which rank(s) decided the game as
`Rank` values, for High Card the first rank that differs (decided 2026-11-25).
It does not turn ranks into text; that is the formatter's job (Ch 5.3
formatter, TD-3).
High Card has 0 primary ranks, so its reason is always the winner's rank at
the first difference. The full rule for other categories is in README.md
("Which rank(s) the reason shows").

### SCENARIO 1: Higher key wins and reason is the first differing rank

**Scenario ID**: GAME-BE-001.5-S1

**GIVEN**
* Black is HIGH_CARD with key `(13, 9, 5, 3, 2)`
* White is HIGH_CARD with key `(14, 8, 4, 3, 2)`

**WHEN**
* `compare(black, white)` is called

**THEN**
* `result.winner == "White"`
* `result.category == HandCategory.HIGH_CARD`
* `result.reason == (Rank.ACE,)`
* `result.kicker is None` (High Card has 0 primary ranks)

### SCENARIO 2: Deciding rank is the first one that differs, not the first card

**Scenario ID**: GAME-BE-001.5-S2

**GIVEN**
* Black is HIGH_CARD with key `(13, 9, 5, 3, 2)`
* White is HIGH_CARD with key `(13, 8, 4, 3, 2)`

**WHEN**
* `compare(black, white)` is called

**THEN**
* `result.winner == "Black"`
* `result.reason == (Rank.NINE,)`
* `result.kicker is None`

### SCENARIO 3: Equal keys produce a tie

**Scenario ID**: GAME-BE-001.5-S3

**GIVEN**
* Black and White are both HIGH_CARD with key `(13, 9, 5, 3, 2)`

**WHEN**
* `compare(black, white)` is called

**THEN**
* `result.is_tie == True`
* `result.winner` is absent or None

---

### FE sub-stories

For a CLI, "frontend" means the text the user sees (`formatter`) and the
terminal interaction (`cli`).

#### GAME-FE-001.1: Format the verdict

```
AS A player
I WANT the result in the kata's exact wording
SO THAT I can read the outcome and its reason at a glance
```

**Architecture references**: Ch 3.3 Poker Hands to Player, Ch 5.3
`formatter`, Ch 1.2 FR-4, TD-3

### SCENARIO 1: Face card is spelled out

**Scenario ID**: GAME-FE-001.1-S1

**GIVEN**
* a Result with winner "White", category HIGH_CARD and reason `(Rank.ACE,)`

**WHEN**
* `format_result` is called

**THEN**
* it returns `White wins. - with high card: Ace`

### SCENARIO 2: Number card is shown as a digit

**Scenario ID**: GAME-FE-001.1-S2

**GIVEN**
* a Result with winner "Black", category HIGH_CARD and reason `(Rank.NINE,)`

**WHEN**
* `format_result` is called

**THEN**
* it returns `Black wins. - with high card: 9`

### SCENARIO 3: Tie

**Scenario ID**: GAME-FE-001.1-S3

**GIVEN**
* a tie Result

**WHEN**
* `format_result` is called

**THEN**
* it returns `Tie.`

### SCENARIO 4: Ten is shown as 10

**Scenario ID**: GAME-FE-001.1-S4

**GIVEN**
* a Result with winner "Black", category HIGH_CARD and reason `(Rank.TEN,)`

**WHEN**
* `format_result` is called

**THEN**
* it returns `Black wins. - with high card: 10`

---

#### GAME-FE-001.2: Command-line entry point

```
AS A player
I WANT to type or pipe games into the program and get one verdict per line
SO THAT I can check one game or many in one run
```

**Architecture references**: Ch 1.2 FR-5, Ch 5.3 `cli`, Ch 8.2 rule "only
cli writes to stdout", Ch 8.5 (cli tested through subprocess or an injected buffer)

### SCENARIO 1: One line in, one verdict out

**Scenario ID**: GAME-FE-001.2-S1

**GIVEN**
* stdin holds the line `Black: 2H 3D 5S 9C KD  White: 2C 3H 4S 8C AH`

**WHEN**
* `main()` runs

**THEN**
* stdout holds exactly one line: `White wins. - with high card: Ace`

### SCENARIO 2: Several lines produce verdicts in order

**Scenario ID**: GAME-FE-001.2-S2

**GIVEN**
* stdin holds kata sample lines 1, 3 and 4 in that order:
  `Black: 2H 3D 5S 9C KD  White: 2C 3H 4S 8C AH`,
  `Black: 2H 3D 5S 9C KD  White: 2C 3H 4S 8C KH`,
  `Black: 2H 3D 5S 9C KD  White: 2D 3H 5C 9S KH`

**WHEN**
* `main()` runs

**THEN**
* stdout holds three lines in the same order:
  `White wins. - with high card: Ace`,
  `Black wins. - with high card: 9`,
  `Tie.`

---

### Implementation order

1. GAME-INFRA-001.1, 001.2, 001.3, 001.4: package, test harness, CI, report
2. GAME-BE-001.1, 001.2, 001.3, 001.4, 001.5: domain types first, then each
   pipeline stage in data-flow order
3. GAME-FE-001.1, 001.2: formatter, then cli
4. GAME-STORY-001-S1 to S4 end to end: the kata samples pass through the real CLI

---

### Traceability check

Every scenario traces back to an architecture section, a parent story,
and a testable THEN clause.

| Scenario | Parent | Architecture | Checks (THEN) |
|---|---|---|---|
| GAME-STORY-001-S1 | GAME-STORY-001 | Ch 6.1, QS-1 sample 1 | exact output line |
| GAME-STORY-001-S2 | GAME-STORY-001 | QS-1 sample 3, QS-4 | exact output line |
| GAME-STORY-001-S3 | GAME-STORY-001 | Ch 6.2, QS-1 sample 4, QS-3 | exact output line |
| GAME-STORY-001-S4 | GAME-STORY-001 | R-1 | exact output line |
| GAME-INFRA-001.1-S1 | GAME-INFRA-001.1 | Ch 7.4, QS-8 | exit code, empty stdout |
| GAME-INFRA-001.1-S2 | GAME-INFRA-001.1 | ADR-001 | empty dependency list |
| GAME-INFRA-001.1-S3 | GAME-INFRA-001.1 | Ch 8.7, CV-1 | each linter exits 0 |
| GAME-INFRA-001.2-S1 | GAME-INFRA-001.2 | Ch 8.5 | scenarios collected |
| GAME-INFRA-001.2-S2 | GAME-INFRA-001.2 | Ch 3.3 Isolation, QS-5 | pass, no files, < 5 s |
| GAME-INFRA-001.3-S1 | GAME-INFRA-001.3 | Ch 7.3, Ch 8.7 | step order, fails on failure |
| GAME-INFRA-001.3-S2 | GAME-INFRA-001.3 | QS-9 | < 30 s |
| GAME-INFRA-001.3-S3 | GAME-INFRA-001.3 | CV-1, OC-3 | ruff non-zero; run failed; pytest skipped |
| GAME-INFRA-001.4-S1 | GAME-INFRA-001.4 | Ch 7.3 Artefacts | XML uploaded, IDs present |
| GAME-BE-001.1-S1 | GAME-BE-001.1 | ADR-003, Ch 8.1 | comparison and int values |
| GAME-BE-001.1-S2 | GAME-BE-001.1 | Ch 8.4 | error raised, set size 1 |
| GAME-BE-001.2-S1 | GAME-BE-001.2 | Ch 5.3 hand | player and length |
| GAME-BE-001.2-S2 | GAME-BE-001.2 | Ch 5.3 hand | ValueError |
| GAME-BE-001.3-S1 | GAME-BE-001.3 | Ch 3.3 | player names and cards |
| GAME-BE-001.3-S2 | GAME-BE-001.3 | Ch 12 Card notation | TEN parsed |
| GAME-BE-001.4-S1 | GAME-BE-001.4 | Ch 4.4 | category and key |
| GAME-BE-001.4-S2 | GAME-BE-001.4 | Ch 4.4, QS-6 | same result in any order |
| GAME-BE-001.5-S1 | GAME-BE-001.5 | Ch 5.3 comparator | winner, category, reason |
| GAME-BE-001.5-S2 | GAME-BE-001.5 | Ch 4.4 | deciding rank, winner |
| GAME-BE-001.5-S3 | GAME-BE-001.5 | QS-3 | is_tie true, no winner |
| GAME-FE-001.1-S1 | GAME-FE-001.1 | Ch 3.3 | exact string |
| GAME-FE-001.1-S2 | GAME-FE-001.1 | Ch 3.3 | exact string |
| GAME-FE-001.1-S3 | GAME-FE-001.1 | Ch 1.2 FR-4 | exact string |
| GAME-FE-001.1-S4 | GAME-FE-001.1 | Ch 3.3 Card notation | exact string with 10 |
| GAME-FE-001.2-S1 | GAME-FE-001.2 | Ch 1.2 FR-5 | one exact line |
| GAME-FE-001.2-S2 | GAME-FE-001.2 | Ch 5.3 cli | three lines in order |

---

## GAME-STORY-002: Decide the winner with pairs

**Priority**: Core (Pareto 20%)
**Value**: Pair and Two Pairs together decide 69.2% of all random games
(60.2% from Pair alone). Without this story, any hand containing a pair
is silently misclassified as High Card.
**Status**: Approved, not implemented

```
AS A player
I WANT to enter two hands where at least one contains a pair or two pairs and be told who wins and why
SO THAT the most common game outcomes are decided correctly
```

**Architecture references**

- Ch 1.2 FR-2 (Pair, Two Pairs), FR-3, FR-4
- Ch 4.4 Tiebreaker key (Pair key, Two Pairs key)
- Ch 5.3 `ranker` (classification), `comparator` (kicker comparison), `formatter` (reason formatting)
- Ch 8.1 `HandCategory` PAIR = 2, TWO_PAIRS = 3
- Ch 10.2 QS-3 (tie within category), QS-4 (kicker cases)
- Ch 11.1 R-1 (wrong tiebreaker key construction)
- ADR-003 (IntEnum and tuple comparison)
- README.md "Decided questions" (reason tuple, kicker field, Two Pairs written as "X and Y")

**Scope**

- In scope: Pair and Two Pairs classification, all tiebreaker and kicker
  cases for those two categories, and their output strings.
- Out of scope: all other categories. Hands in this bundle's scenarios
  contain exactly one pair or exactly two pairs: no straights, flushes
  or higher combinations.

**Two Pairs display format (decided here):** Two primary ranks are written
as `<high> and <low>`, e.g. `two pairs: Jack and 8`. Full House uses "over"
(three over pair); Two Pairs uses "and" (peer relationship). This wording is
settled in GAME-FE-002.1-S3 and GAME-FE-002.1-S4 and must be used consistently
in `formatter` and in all future stories.

---

### Acceptance scenarios (end to end, through the CLI)

### SCENARIO 1: Pair beats High Card

**Scenario ID**: GAME-STORY-002-S1

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: 2H 4S 4C 3D 5H  White: 2C 8D AS QH 3S` is sent on stdin

**THEN**
* stdout contains exactly `Black wins. - with pair: 4`

### SCENARIO 2: Two Pairs beats Pair

**Scenario ID**: GAME-STORY-002-S2

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: 2H 4S 4C 2D KH  White: 3C 3H 8D QS 7H` is sent on stdin

**THEN**
* stdout contains exactly `Black wins. - with two pairs: 4 and 2`

### SCENARIO 3: Higher pair wins

**Scenario ID**: GAME-STORY-002-S3

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: KC KH 3D 5S 8C  White: QS QD 3H 5C 7D` is sent on stdin

**THEN**
* stdout contains exactly `Black wins. - with pair: King`

### SCENARIO 4: Same pair, winner decided by kicker

**Scenario ID**: GAME-STORY-002-S4

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: 5S 5H KD 8C 3H  White: 5C 5D QS 8H 3D` is sent on stdin

**THEN**
* stdout contains exactly `Black wins. - with pair: 5, high card: King`

### SCENARIO 5: Higher top pair wins in Two Pairs

**Scenario ID**: GAME-STORY-002-S5

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: JS JH 3S 3H KC  White: 9S 9H 8S 8H AC` is sent on stdin

**THEN**
* stdout contains exactly `Black wins. - with two pairs: Jack and 3`

### SCENARIO 6: Same two pairs, winner decided by kicker

**Scenario ID**: GAME-STORY-002-S6

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: JS JH 8S 8H 3C  White: JC JD 8C 8D 2H` is sent on stdin

**THEN**
* stdout contains exactly `Black wins. - with two pairs: Jack and 8, high card: 3`

### SCENARIO 7: Identical Two Pairs hands are a tie

**Scenario ID**: GAME-STORY-002-S7

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: 9S 9H 5S 5H KC  White: 9C 9D 5C 5D KH` is sent on stdin

**THEN**
* stdout contains exactly `Tie.`

### SCENARIO 8: Same top pair, decided by the lower pair (not a kicker)

**Scenario ID**: GAME-STORY-002-S8

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: KH KD 5S 5C 3D  White: KC KS 4H 4D AH` is sent on stdin

**THEN**
* stdout contains exactly `Black wins. - with two pairs: King and 5`
* stdout does not contain `high card` (the lower pair is a primary rank, and White's Ace kicker never counts)

### SCENARIO 9: Same pair, decided by the last kicker

**Scenario ID**: GAME-STORY-002-S9

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: 5H 5D KS 9C 3D  White: 5C 5S KH 9H 2H` is sent on stdin

**THEN**
* stdout contains exactly `Black wins. - with pair: 5, high card: 3`

---

### INFRA sub-stories

#### GAME-INFRA-002.1: Pair acceptance feature file

```
AS A developer
I WANT the GAME-STORY-002 scenarios in their own feature file
SO THAT they run with the rest of the suite and show up separately in the test report
```

**Architecture references**: Ch 4.5 Testing strategy, Ch 8.5 Testability,
OC-2, QS-5, Ch 7.3 CI runner (Artefacts)

### SCENARIO 1: Pair scenarios are collected next to the High Card ones

**Scenario ID**: GAME-INFRA-002.1-S1

**GIVEN**
* `features/pairs.feature` contains GAME-STORY-002-S1 to S9

**WHEN**
* `pytest` is run from the repository root

**THEN**
* the nine scenarios are collected and reported by their names
* the GAME-STORY-001 scenarios in `features/high_card.feature` are still collected and pass

### SCENARIO 2: CI reports the new scenarios without workflow changes

**Scenario ID**: GAME-INFRA-002.1-S2

**GIVEN**
* the CI workflow from GAME-INFRA-001.3

**WHEN**
* a commit adding `features/pairs.feature` is pushed

**THEN**
* the workflow file is unchanged
* the JUnit XML report from GAME-INFRA-001.4 contains GAME-STORY-002-S1 to S9

#### Deployment, data store, events and monitoring: no new work

Deployment (GAME-INFRA-001.1), CI (GAME-INFRA-001.3) and the test report
(GAME-INFRA-001.4) are reused unchanged. Data store and events remain not
applicable (Ch 1.2 Out of scope, ADR-001, Ch 3.1), and there is still no
logging (Ch 8.6).

---

### BE sub-stories

#### GAME-BE-002.1: Classify Pair with a tiebreaker key

```
AS A developer
I WANT `rank_hand` to return PAIR with key (pair_rank, kicker3, kicker2, kicker1)
SO THAT two Pair hands can be compared by a single tuple comparison with correct kicker ordering
```

**Architecture references**: Ch 4.4 Tiebreaker key (Pair key), Ch 5.3 `ranker`,
Ch 8.1 `HandCategory` PAIR = 2, ADR-003, QS-6, R-1

### SCENARIO 1: Pair classified with correct key

**Scenario ID**: GAME-BE-002.1-S1

**GIVEN**
* a `Hand` built directly with 5S 5H KD 8C 3H (pair of 5s, kickers K=13, 8, 3)

**WHEN**
* `rank_hand` is called

**THEN**
* `ranked.category == HandCategory.PAIR`
* `ranked.tiebreaker_key == (5, 13, 8, 3)`

### SCENARIO 2: Kickers are ordered highest to lowest

**Scenario ID**: GAME-BE-002.1-S2

**GIVEN**
* a `Hand` built directly with 7H 7D 2S JC 9H (pair of 7s, kickers J=11, 9, 2)

**WHEN**
* `rank_hand` is called

**THEN**
* `ranked.category == HandCategory.PAIR`
* `ranked.tiebreaker_key == (7, 11, 9, 2)`

### SCENARIO 3: Pair is not misclassified as Two Pairs or High Card

**Scenario ID**: GAME-BE-002.1-S3

**GIVEN**
* a `Hand` built directly with KC KH 3D 5S 8C (exactly one pair of Kings)

**WHEN**
* `rank_hand` is called

**THEN**
* `ranked.category == HandCategory.PAIR`
* `ranked.tiebreaker_key == (13, 8, 5, 3)`

---

#### GAME-BE-002.2: Classify Two Pairs with a tiebreaker key

```
AS A developer
I WANT `rank_hand` to return TWO_PAIRS with key (high_pair_rank, low_pair_rank, kicker)
SO THAT two Two-Pairs hands can be compared by a single tuple comparison
```

**Architecture references**: Ch 4.4 Tiebreaker key (Two Pairs key), Ch 5.3 `ranker`,
Ch 8.1 `HandCategory` TWO_PAIRS = 3, ADR-003, QS-6, R-1

### SCENARIO 1: Two Pairs classified with correct key

**Scenario ID**: GAME-BE-002.2-S1

**GIVEN**
* a `Hand` built directly with JS JH 8S 8H 3C (Jacks and 8s, kicker 3)

**WHEN**
* `rank_hand` is called

**THEN**
* `ranked.category == HandCategory.TWO_PAIRS`
* `ranked.tiebreaker_key == (11, 8, 3)`

### SCENARIO 2: High pair comes first in the key regardless of card order

**Scenario ID**: GAME-BE-002.2-S2

**GIVEN**
* a `Hand` built directly with 3C 3D JH JS KC (3s and Jacks, kicker K, low pair dealt first)

**WHEN**
* `rank_hand` is called

**THEN**
* `ranked.category == HandCategory.TWO_PAIRS`
* `ranked.tiebreaker_key == (11, 3, 13)`

---

#### GAME-BE-002.3: Primary-rank table

```
AS A developer
I WANT `ranker` to state how many primary ranks start each category's key
SO THAT the comparator can tell a primary rank from a kicker without knowing poker rules
```

**Architecture references**: Ch 4.4 Tiebreaker key, Ch 5.3 `ranker` ("contains
all poker rules") and `comparator` ("no poker rules live here"), README.md
"Which rank(s) the reason shows"

### SCENARIO 1: Table covers the categories built so far

**Scenario ID**: GAME-BE-002.3-S1

**GIVEN**
* the primary-rank table in `ranker`

**WHEN**
* it is read for HIGH_CARD, PAIR and TWO_PAIRS

**THEN**
* the values are 0, 1 and 2
* entries for higher categories are added by the stories that classify them (GAME-STORY-003 to 005)

---

#### GAME-BE-002.4: Comparator resolves Pair and Two Pairs results with reason and kicker

```
AS A developer
I WANT `compare` to populate `result.reason` and `result.kicker` correctly for Pair and Two Pairs
SO THAT the formatter can produce the exact output string without any poker knowledge
```

**Architecture references**: Ch 5.3 `comparator`, Ch 8.1 `Result`, ADR-002,
ADR-003, Ch 4.4, README.md "Decided questions" (reason tuple, kicker field,
primary-rank table from GAME-BE-002.3: Pair has 1 primary rank, Two Pairs has 2)

### SCENARIO 1: Pair win decided by the pair rank itself

**Scenario ID**: GAME-BE-002.4-S1

**GIVEN**
* Black is PAIR with key `(13, 8, 5, 3)` (pair of Kings)
* White is PAIR with key `(12, 7, 5, 3)` (pair of Queens)

**WHEN**
* `compare(black, white)` is called

**THEN**
* `result.winner == "Black"`
* `result.category == HandCategory.PAIR`
* `result.reason == (Rank.KING,)`
* `result.kicker is None`

### SCENARIO 2: Pair win decided by a kicker

**Scenario ID**: GAME-BE-002.4-S2

**GIVEN**
* Black is PAIR with key `(5, 13, 8, 3)` (pair of 5s, kickers K, 8, 3)
* White is PAIR with key `(5, 12, 8, 3)` (pair of 5s, kickers Q, 8, 3)

**WHEN**
* `compare(black, white)` is called

**THEN**
* `result.winner == "Black"`
* `result.reason == (Rank.FIVE,)`
* `result.kicker == Rank.KING`

### SCENARIO 3: Two Pairs win decided by the high pair

**Scenario ID**: GAME-BE-002.4-S3

**GIVEN**
* Black is TWO_PAIRS with key `(11, 3, 13)` (Jacks and 3s, kicker K)
* White is TWO_PAIRS with key `(9, 8, 14)` (9s and 8s, kicker A)

**WHEN**
* `compare(black, white)` is called

**THEN**
* `result.winner == "Black"`
* `result.reason == (Rank.JACK, Rank.THREE)`
* `result.kicker is None`

### SCENARIO 4: Two Pairs win decided by a kicker

**Scenario ID**: GAME-BE-002.4-S4

**GIVEN**
* Black is TWO_PAIRS with key `(11, 8, 3)` (Jacks and 8s, kicker 3)
* White is TWO_PAIRS with key `(11, 8, 2)` (Jacks and 8s, kicker 2)

**WHEN**
* `compare(black, white)` is called

**THEN**
* `result.winner == "Black"`
* `result.reason == (Rank.JACK, Rank.EIGHT)`
* `result.kicker == Rank.THREE`

### SCENARIO 5: Identical Two Pairs hands are a tie

**Scenario ID**: GAME-BE-002.4-S5

**GIVEN**
* Black is TWO_PAIRS with key `(9, 5, 13)`
* White is TWO_PAIRS with key `(9, 5, 13)`

**WHEN**
* `compare(black, white)` is called

**THEN**
* `result.is_tie == True`
* `result.winner` is absent or None

### SCENARIO 6: Higher category wins, reason is the winner's pair

**Scenario ID**: GAME-BE-002.4-S6

**GIVEN**
* Black is HIGH_CARD with key `(13, 9, 5, 3, 2)`
* White is PAIR with key `(8, 4, 3, 2)`

**WHEN**
* `compare(black, white)` is called

**THEN**
* `result.winner == "White"`
* `result.category == HandCategory.PAIR`
* `result.reason == (Rank.EIGHT,)`
* `result.kicker is None`

### SCENARIO 7: Two Pairs decided by the lower pair is not a kicker

**Scenario ID**: GAME-BE-002.4-S7

**GIVEN**
* Black is TWO_PAIRS with key `(13, 5, 3)` (Kings and 5s, kicker 3)
* White is TWO_PAIRS with key `(13, 4, 14)` (Kings and 4s, kicker A)

**WHEN**
* `compare(black, white)` is called

**THEN**
* `result.winner == "Black"`
* `result.reason == (Rank.KING, Rank.FIVE)`
* `result.kicker is None` (the difference is at index 1, inside the 2 primary ranks)

---

### FE sub-stories

#### GAME-FE-002.1: Format Pair and Two Pairs verdicts

```
AS A player
I WANT Pair and Two Pairs results formatted in the kata's wording
SO THAT I can read the pair rank and any deciding kicker at a glance
```

**Architecture references**: Ch 3.3 Poker Hands to Player, Ch 5.3 `formatter`,
Ch 1.2 FR-4, TD-3, README.md "Decided questions" (Two Pairs written as "X and Y",
Ten displays as "10", face cards spelled out)

### SCENARIO 1: Pair win, decided by the pair

**Scenario ID**: GAME-FE-002.1-S1

**GIVEN**
* a Result with winner "Black", category PAIR, reason `(Rank.KING,)`, kicker `None`

**WHEN**
* `format_result` is called

**THEN**
* it returns `Black wins. - with pair: King`

### SCENARIO 2: Pair win, decided by a kicker

**Scenario ID**: GAME-FE-002.1-S2

**GIVEN**
* a Result with winner "Black", category PAIR, reason `(Rank.FIVE,)`, kicker `Rank.KING`

**WHEN**
* `format_result` is called

**THEN**
* it returns `Black wins. - with pair: 5, high card: King`

### SCENARIO 3: Two Pairs win, decided by the pairs

**Scenario ID**: GAME-FE-002.1-S3

**GIVEN**
* a Result with winner "Black", category TWO_PAIRS, reason `(Rank.JACK, Rank.THREE)`, kicker `None`

**WHEN**
* `format_result` is called

**THEN**
* it returns `Black wins. - with two pairs: Jack and 3`

### SCENARIO 4: Two Pairs win, decided by a kicker

**Scenario ID**: GAME-FE-002.1-S4

**GIVEN**
* a Result with winner "Black", category TWO_PAIRS, reason `(Rank.JACK, Rank.EIGHT)`, kicker `Rank.THREE`

**WHEN**
* `format_result` is called

**THEN**
* it returns `Black wins. - with two pairs: Jack and 8, high card: 3`

### SCENARIO 5: Two Pairs with a Ten rank

**Scenario ID**: GAME-FE-002.1-S5

**GIVEN**
* a Result with winner "White", category TWO_PAIRS, reason `(Rank.TEN, Rank.FOUR)`, kicker `None`

**WHEN**
* `format_result` is called

**THEN**
* it returns `White wins. - with two pairs: 10 and 4`

---

### Implementation order

1. GAME-INFRA-002.1: add `features/pairs.feature` (its scenarios fail until the code exists)
2. GAME-BE-002.1: extend `ranker` to classify Pair
3. GAME-BE-002.2: extend `ranker` to classify Two Pairs
4. GAME-BE-002.3: add the primary-rank table to `ranker`
5. GAME-BE-002.4: extend `comparator` to populate reason and kicker for Pair and Two Pairs
6. GAME-FE-002.1: extend `formatter` to render Pair and Two Pairs strings
7. GAME-STORY-002-S1 to S9 end to end, plus GAME-STORY-001-S1 to S4 as regression

---

### Traceability check

| Scenario | Parent | Architecture | Checks (THEN) |
|---|---|---|---|
| GAME-STORY-002-S1 | GAME-STORY-002 | Ch 1.2 FR-2, FR-4 | exact output line |
| GAME-STORY-002-S2 | GAME-STORY-002 | Ch 1.2 FR-2, FR-4 | exact output line |
| GAME-STORY-002-S3 | GAME-STORY-002 | Ch 4.4, QS-4 | exact output line |
| GAME-STORY-002-S4 | GAME-STORY-002 | Ch 4.4, QS-4, R-1 | exact output line |
| GAME-STORY-002-S5 | GAME-STORY-002 | Ch 4.4, R-1 | exact output line |
| GAME-STORY-002-S6 | GAME-STORY-002 | Ch 4.4, QS-4, R-1 | exact output line |
| GAME-STORY-002-S7 | GAME-STORY-002 | QS-3 | exact output line `Tie.` |
| GAME-STORY-002-S8 | GAME-STORY-002 | Ch 4.4 Two Pairs key, README reason rule | exact output line, no kicker |
| GAME-STORY-002-S9 | GAME-STORY-002 | QS-4, R-1 | exact output line |
| GAME-INFRA-002.1-S1 | GAME-INFRA-002.1 | Ch 8.5, OC-2 | scenarios collected, 001 still passes |
| GAME-INFRA-002.1-S2 | GAME-INFRA-002.1 | Ch 7.3 | workflow unchanged, IDs in report |
| GAME-BE-002.1-S1 | GAME-BE-002.1 | Ch 4.4, Ch 8.1 | category PAIR; key (5,13,8,3) |
| GAME-BE-002.1-S2 | GAME-BE-002.1 | Ch 4.4, R-1 | category PAIR; kickers descending |
| GAME-BE-002.1-S3 | GAME-BE-002.1 | Ch 4.4, Ch 8.1 | category PAIR; not TWO_PAIRS |
| GAME-BE-002.2-S1 | GAME-BE-002.2 | Ch 4.4, Ch 8.1 | category TWO_PAIRS; key (11,8,3) |
| GAME-BE-002.2-S2 | GAME-BE-002.2 | Ch 4.4, R-1 | high pair first regardless of card order |
| GAME-BE-002.3-S1 | GAME-BE-002.3 | Ch 4.4, Ch 5.3 ranker | table values 0, 1, 2 |
| GAME-BE-002.4-S1 | GAME-BE-002.4 | Ch 5.3, ADR-002 | winner, reason (KING,), kicker None |
| GAME-BE-002.4-S2 | GAME-BE-002.4 | Ch 4.4, ADR-003, R-1 | winner, reason (FIVE,), kicker KING |
| GAME-BE-002.4-S3 | GAME-BE-002.4 | Ch 5.3, ADR-002 | winner, reason (JACK,THREE), kicker None |
| GAME-BE-002.4-S4 | GAME-BE-002.4 | Ch 4.4, ADR-003, R-1 | winner, reason (JACK,EIGHT), kicker THREE |
| GAME-BE-002.4-S5 | GAME-BE-002.4 | QS-3 | is_tie True, no winner |
| GAME-BE-002.4-S6 | GAME-BE-002.4 | Ch 8.1 Result | winner, category, reason (EIGHT,), kicker None |
| GAME-BE-002.4-S7 | GAME-BE-002.4 | Ch 4.4, README reason rule | reason (KING,FIVE), kicker None |
| GAME-FE-002.1-S1 | GAME-FE-002.1 | Ch 3.3, FR-4 | `Black wins. - with pair: King` |
| GAME-FE-002.1-S2 | GAME-FE-002.1 | Ch 3.3, FR-4, QS-4 | `Black wins. - with pair: 5, high card: King` |
| GAME-FE-002.1-S3 | GAME-FE-002.1 | Ch 3.3, FR-4 | `Black wins. - with two pairs: Jack and 3` |
| GAME-FE-002.1-S4 | GAME-FE-002.1 | Ch 3.3, FR-4, QS-4 | `Black wins. - with two pairs: Jack and 8, high card: 3` |
| GAME-FE-002.1-S5 | GAME-FE-002.1 | Ch 3.3, Ch 12 Card notation | `White wins. - with two pairs: 10 and 4` |

---

## GAME-STORY-003: Sets of equal ranks

**Priority**: Supporting
**Value**: Three of a Kind, Full House, and Four of a Kind together decide 4.5% of
all random games (4.149% + 0.288% + 0.048%). Includes kata sample 2.
**Status**: Approved, not implemented

```
AS A player
I WANT to enter two hands where at least one contains three of a kind, a full house, or four of a kind and be told who wins and why
SO THAT games decided by repeated ranks above a pair are settled correctly
```

**Architecture references**

- Ch 1.2 FR-2 (Three of a Kind, Full House, Four of a Kind), FR-3, FR-4
- Ch 4.4 Tiebreaker key (Full House key)
- Ch 5.3 `ranker`, `comparator`, `formatter`
- Ch 6.1 Runtime scenario (kata sample 2: full house beats flush)
- Ch 8.1 `HandCategory` THREE_OF_A_KIND = 4, FULL_HOUSE = 7, FOUR_OF_A_KIND = 8
- Ch 10.2 QS-1 (kata sample 2), QS-3
- Ch 11.1 R-1 (tiebreaker key errors)
- Ch 12 Glossary (Three of a Kind: "Ranked by that rank"; Four of a Kind: "Ranked by that rank")
- ADR-003 (IntEnum and tuple comparison)
- README.md "Decided questions" (Full House reason format "X over Y", primary-rank table)

**Scope**

- In scope: Three of a Kind, Full House, and Four of a Kind classification,
  their tiebreaker keys, reason population, and output strings.
- Out of scope: Straight and Flush (GAME-STORY-004), Straight Flush and Royal
  Flush (GAME-STORY-005).
- S1 deliberately places a Flush on the losing side. Until GAME-STORY-004 is
  built, the ranker treats White's hand as High Card; a Full House beats High
  Card just as it beats a Flush, so S1 passes at both stages of development.

**Tiebreaker keys for this story:**
- Full House: `(three_rank, pair_rank)` as defined in Ch 4.4.
- Three of a Kind: `(three_rank,)`. Ch 12 states "Ranked by that rank." With
  one 52-card deck, two hands can never share the same three-rank (only four
  cards of each rank exist; one hand takes three of them, leaving at most one
  for the other), so a kicker position in the key is never needed.
- Four of a Kind: `(quad_rank,)`. Ch 12 states "Ranked by that rank." The
  same deck constraint makes a shared quad rank impossible.

**Full House reason format (confirmed here):** reason = `(three_rank, pair_rank)`,
both primary ranks. The formatter writes them as `<three> over <pair>`,
e.g. `full house: 4 over 2` (kata sample 2). Three of a Kind reason = `(three_rank,)`,
one primary rank. Four of a Kind reason = `(quad_rank,)`, one primary rank.

---

### Acceptance scenarios (end to end, through the CLI)

### SCENARIO 1: Full House beats Flush (kata sample 2)

**Scenario ID**: GAME-STORY-003-S1

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: 2H 4S 4C 2D 4H  White: 2S 8S AS QS 3S` is sent on stdin

**THEN**
* stdout contains exactly `Black wins. - with full house: 4 over 2`

### SCENARIO 2: Three of a Kind beats Two Pairs

**Scenario ID**: GAME-STORY-003-S2

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: 7H 7D 7S KH 3C  White: AS AH 9S 9H QC` is sent on stdin

**THEN**
* stdout contains exactly `Black wins. - with three of a kind: 7`

### SCENARIO 3: Four of a Kind beats Full House

**Scenario ID**: GAME-STORY-003-S3

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: 9S 9H 9C 9D KH  White: 3H 3D 3C KS KD` is sent on stdin

**THEN**
* stdout contains exactly `Black wins. - with four of a kind: 9`

### SCENARIO 4: Higher three-rank decides between Full Houses

**Scenario ID**: GAME-STORY-003-S4

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: KH KD KC 2S 2H  White: QS QH QC AH AD` is sent on stdin

**THEN**
* stdout contains exactly `Black wins. - with full house: King over 2`

### SCENARIO 5: Higher three-rank decides between Three of a Kind hands

**Scenario ID**: GAME-STORY-003-S5

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: 8H 8D 8S KH 3C  White: 6C 6S 6H QD 2S` is sent on stdin

**THEN**
* stdout contains exactly `Black wins. - with three of a kind: 8`

### SCENARIO 6: Higher quad rank decides between Four of a Kind hands

**Scenario ID**: GAME-STORY-003-S6

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: 9S 9H 9C 9D KH  White: 6S 6H 6C 6D AS` is sent on stdin

**THEN**
* stdout contains exactly `Black wins. - with four of a kind: 9`

---

### INFRA sub-stories

#### GAME-INFRA-003.1: Sets-of-ranks acceptance feature file

```
AS A developer
I WANT the GAME-STORY-003 scenarios in their own feature file
SO THAT they run with the rest of the suite and appear separately in the test report
```

**Architecture references**: Ch 4.5 Testing strategy, Ch 8.5 Testability,
OC-2, QS-5, Ch 7.3 CI runner (Artefacts)

### SCENARIO 1: Sets-of-ranks scenarios are collected alongside earlier ones

**Scenario ID**: GAME-INFRA-003.1-S1

**GIVEN**
* `features/sets_of_ranks.feature` contains GAME-STORY-003-S1 to S6

**WHEN**
* `pytest` is run from the repository root

**THEN**
* the six scenarios are collected and reported by their names
* all GAME-STORY-001 and GAME-STORY-002 scenarios still pass

### SCENARIO 2: CI reports the new scenarios without workflow changes

**Scenario ID**: GAME-INFRA-003.1-S2

**GIVEN**
* the CI workflow from GAME-INFRA-001.3

**WHEN**
* a commit adding `features/sets_of_ranks.feature` is pushed

**THEN**
* the workflow file is unchanged
* the JUnit XML report contains GAME-STORY-003-S1 to S6

#### Deployment, data store, events and monitoring: no new work

All reused from GAME-STORY-001 (GAME-INFRA-001.1, 001.3, 001.4). Data store
and events remain not applicable (Ch 1.2, ADR-001, Ch 3.1). No logging
(Ch 8.6).

---

### BE sub-stories

#### GAME-BE-003.1: Classify Three of a Kind with a tiebreaker key

```
AS A developer
I WANT `rank_hand` to return THREE_OF_A_KIND with key (three_rank,)
SO THAT three-of-a-kind hands compare correctly with one tuple comparison
```

**Architecture references**: Ch 5.3 `ranker`, Ch 8.1 THREE_OF_A_KIND = 4,
Ch 12 Glossary ("Ranked by that rank"), ADR-003, QS-6

### SCENARIO 1: Three of a Kind classified with correct key

**Scenario ID**: GAME-BE-003.1-S1

**GIVEN**
* a `Hand` built directly with 7H 7D 7S KH 3C (three 7s)

**WHEN**
* `rank_hand` is called

**THEN**
* `ranked.category == HandCategory.THREE_OF_A_KIND`
* `ranked.tiebreaker_key == (7,)`

---

#### GAME-BE-003.2: Classify Full House with a tiebreaker key

```
AS A developer
I WANT `rank_hand` to return FULL_HOUSE with key (three_rank, pair_rank)
SO THAT full house hands compare correctly with one tuple comparison
```

**Architecture references**: Ch 4.4 Tiebreaker key (Full House key),
Ch 5.3 `ranker`, Ch 8.1 FULL_HOUSE = 7, ADR-003, QS-6, R-1

### SCENARIO 1: Full House classified with correct key

**Scenario ID**: GAME-BE-003.2-S1

**GIVEN**
* a `Hand` built directly with 2H 4S 4C 2D 4H (three 4s, pair 2s)

**WHEN**
* `rank_hand` is called

**THEN**
* `ranked.category == HandCategory.FULL_HOUSE`
* `ranked.tiebreaker_key == (4, 2)`

### SCENARIO 2: Three-rank comes first regardless of card order

**Scenario ID**: GAME-BE-003.2-S2

**GIVEN**
* a `Hand` built directly with KS KC 2H KD 2S (pair 2s dealt before the third King)

**WHEN**
* `rank_hand` is called

**THEN**
* `ranked.category == HandCategory.FULL_HOUSE`
* `ranked.tiebreaker_key == (13, 2)`

---

#### GAME-BE-003.3: Classify Four of a Kind with a tiebreaker key

```
AS A developer
I WANT `rank_hand` to return FOUR_OF_A_KIND with key (quad_rank,)
SO THAT four-of-a-kind hands compare correctly with one tuple comparison
```

**Architecture references**: Ch 5.3 `ranker`, Ch 8.1 FOUR_OF_A_KIND = 8,
Ch 12 Glossary ("Ranked by that rank"), ADR-003, QS-6

### SCENARIO 1: Four of a Kind classified with correct key

**Scenario ID**: GAME-BE-003.3-S1

**GIVEN**
* a `Hand` built directly with 9S 9H 9C 9D KH (four 9s)

**WHEN**
* `rank_hand` is called

**THEN**
* `ranked.category == HandCategory.FOUR_OF_A_KIND`
* `ranked.tiebreaker_key == (9,)`

---

#### GAME-BE-003.4: Primary-rank table extended for Three of a Kind, Full House, Four of a Kind

```
AS A developer
I WANT the primary-rank table in `ranker` to include entries for the three new categories
SO THAT the comparator correctly distinguishes primary ranks from kickers for these categories
```

**Architecture references**: Ch 4.4 Tiebreaker key, Ch 5.3 `ranker`,
README.md "Which rank(s) the reason shows"

### SCENARIO 1: Table entries for new categories are correct

**Scenario ID**: GAME-BE-003.4-S1

**GIVEN**
* the primary-rank table in `ranker`

**WHEN**
* it is read for THREE_OF_A_KIND, FULL_HOUSE and FOUR_OF_A_KIND

**THEN**
* THREE_OF_A_KIND maps to 1
* FULL_HOUSE maps to 2
* FOUR_OF_A_KIND maps to 1

---

#### GAME-BE-003.5: Comparator resolves Three of a Kind, Full House, and Four of a Kind results

```
AS A developer
I WANT `compare` to populate `result.reason` and `result.kicker` correctly for the three new categories
SO THAT the formatter produces the exact output string without poker knowledge
```

**Architecture references**: Ch 5.3 `comparator`, Ch 8.1 `Result`, ADR-002,
ADR-003, Ch 4.4, Ch 12, README.md "Decided questions"

### SCENARIO 1: Full House win over Flush, reason is both primary ranks

**Scenario ID**: GAME-BE-003.5-S1

**GIVEN**
* Black is FULL_HOUSE with key `(4, 2)` (valid input: `Black: 2H 4S 4C 2D 4H  White: 2S 8S AS QS 3S`)
* White is FLUSH with key `(14, 12, 8, 3, 2)`

**WHEN**
* `compare(black, white)` is called

**THEN**
* `result.winner == "Black"`
* `result.category == HandCategory.FULL_HOUSE`
* `result.reason == (Rank.FOUR, Rank.TWO)`
* `result.kicker is None`

### SCENARIO 2: Three of a Kind win over Two Pairs, reason is the three-rank

**Scenario ID**: GAME-BE-003.5-S2

**GIVEN**
* Black is THREE_OF_A_KIND with key `(7,)` (valid input: `Black: 7H 7D 7S KH 3C  White: AS AH 9S 9H QC`)
* White is TWO_PAIRS with key `(14, 9, 12)`

**WHEN**
* `compare(black, white)` is called

**THEN**
* `result.winner == "Black"`
* `result.category == HandCategory.THREE_OF_A_KIND`
* `result.reason == (Rank.SEVEN,)`
* `result.kicker is None`

### SCENARIO 3: Four of a Kind win over Full House, reason is the quad rank

**Scenario ID**: GAME-BE-003.5-S3

**GIVEN**
* Black is FOUR_OF_A_KIND with key `(9,)` (valid input: `Black: 9S 9H 9C 9D KH  White: 3H 3D 3C KS KD`)
* White is FULL_HOUSE with key `(3, 13)`

**WHEN**
* `compare(black, white)` is called

**THEN**
* `result.winner == "Black"`
* `result.category == HandCategory.FOUR_OF_A_KIND`
* `result.reason == (Rank.NINE,)`
* `result.kicker is None`

---

### FE sub-stories

#### GAME-FE-003.1: Format Three of a Kind, Full House, and Four of a Kind verdicts

```
AS A player
I WANT Three of a Kind, Full House, and Four of a Kind results formatted in the kata's wording
SO THAT I can read the decisive rank(s) at a glance
```

**Architecture references**: Ch 3.3 Poker Hands to Player, Ch 5.3 `formatter`,
Ch 1.2 FR-4, TD-3, README.md "Decided questions" (Full House "X over Y",
Ten displays as "10", face cards spelled out)

### SCENARIO 1: Full House formatted with "over" (kata sample 2)

**Scenario ID**: GAME-FE-003.1-S1

**GIVEN**
* a Result with winner "Black", category FULL_HOUSE, reason `(Rank.FOUR, Rank.TWO)`, kicker `None`
  (produced by `Black: 2H 4S 4C 2D 4H  White: 2S 8S AS QS 3S`)

**WHEN**
* `format_result` is called

**THEN**
* it returns `Black wins. - with full house: 4 over 2`

### SCENARIO 2: Full House with face card ranks

**Scenario ID**: GAME-FE-003.1-S2

**GIVEN**
* a Result with winner "Black", category FULL_HOUSE, reason `(Rank.KING, Rank.TWO)`, kicker `None`
  (produced by `Black: KH KD KC 2S 2H  White: QS QH QC AH AD`)

**WHEN**
* `format_result` is called

**THEN**
* it returns `Black wins. - with full house: King over 2`

### SCENARIO 3: Three of a Kind formatted with single rank

**Scenario ID**: GAME-FE-003.1-S3

**GIVEN**
* a Result with winner "Black", category THREE_OF_A_KIND, reason `(Rank.SEVEN,)`, kicker `None`
  (produced by `Black: 7H 7D 7S KH 3C  White: AS AH 9S 9H QC`)

**WHEN**
* `format_result` is called

**THEN**
* it returns `Black wins. - with three of a kind: 7`

### SCENARIO 4: Four of a Kind formatted with single rank

**Scenario ID**: GAME-FE-003.1-S4

**GIVEN**
* a Result with winner "Black", category FOUR_OF_A_KIND, reason `(Rank.NINE,)`, kicker `None`
  (produced by `Black: 9S 9H 9C 9D KH  White: 3H 3D 3C KS KD`)

**WHEN**
* `format_result` is called

**THEN**
* it returns `Black wins. - with four of a kind: 9`

---

### Implementation order

1. GAME-INFRA-003.1: add `features/sets_of_ranks.feature`
2. GAME-BE-003.1: extend `ranker` to classify Three of a Kind
3. GAME-BE-003.2: extend `ranker` to classify Full House
4. GAME-BE-003.3: extend `ranker` to classify Four of a Kind
5. GAME-BE-003.4: extend the primary-rank table in `ranker`
6. GAME-BE-003.5: extend `comparator` for the three new categories
7. GAME-FE-003.1: extend `formatter` for the three new categories
8. GAME-STORY-003-S1 to S6 end to end, plus earlier stories as regression

---

### Traceability check

| Scenario | Parent | Architecture | Checks (THEN) |
|---|---|---|---|
| GAME-STORY-003-S1 | GAME-STORY-003 | Ch 6.1, QS-1 sample 2 | exact output line |
| GAME-STORY-003-S2 | GAME-STORY-003 | Ch 1.2 FR-2, FR-4 | exact output line |
| GAME-STORY-003-S3 | GAME-STORY-003 | Ch 1.2 FR-2, FR-4 | exact output line |
| GAME-STORY-003-S4 | GAME-STORY-003 | Ch 4.4 Full House key, R-1 | exact output line |
| GAME-STORY-003-S5 | GAME-STORY-003 | Ch 12, R-1 | exact output line |
| GAME-STORY-003-S6 | GAME-STORY-003 | Ch 12, R-1 | exact output line |
| GAME-INFRA-003.1-S1 | GAME-INFRA-003.1 | Ch 8.5, OC-2 | scenarios collected; earlier pass |
| GAME-INFRA-003.1-S2 | GAME-INFRA-003.1 | Ch 7.3 | workflow unchanged; IDs in report |
| GAME-BE-003.1-S1 | GAME-BE-003.1 | Ch 12, Ch 8.1 | THREE_OF_A_KIND; key (7,) |
| GAME-BE-003.2-S1 | GAME-BE-003.2 | Ch 4.4, Ch 8.1 | FULL_HOUSE; key (4,2) |
| GAME-BE-003.2-S2 | GAME-BE-003.2 | Ch 4.4, R-1 | three-rank first regardless of order |
| GAME-BE-003.3-S1 | GAME-BE-003.3 | Ch 12, Ch 8.1 | FOUR_OF_A_KIND; key (9,) |
| GAME-BE-003.4-S1 | GAME-BE-003.4 | Ch 4.4, Ch 5.3 ranker | table values 1, 2, 1 |
| GAME-BE-003.5-S1 | GAME-BE-003.5 | Ch 5.3, ADR-002, QS-1 | winner; reason (FOUR,TWO); kicker None |
| GAME-BE-003.5-S2 | GAME-BE-003.5 | Ch 5.3, ADR-002 | winner; reason (SEVEN,); kicker None |
| GAME-BE-003.5-S3 | GAME-BE-003.5 | Ch 8.1, ADR-002 | winner; reason (NINE,); kicker None |
| GAME-FE-003.1-S1 | GAME-FE-003.1 | Ch 3.3, FR-4, QS-1 | `Black wins. - with full house: 4 over 2` |
| GAME-FE-003.1-S2 | GAME-FE-003.1 | Ch 3.3, FR-4 | `Black wins. - with full house: King over 2` |
| GAME-FE-003.1-S3 | GAME-FE-003.1 | Ch 3.3, FR-4 | `Black wins. - with three of a kind: 7` |
| GAME-FE-003.1-S4 | GAME-FE-003.1 | Ch 3.3, FR-4 | `Black wins. - with four of a kind: 9` |

---

## GAME-STORY-004: Straights and flushes

**Priority**: Supporting
**Value**: Straight and Flush together decide 1.2% of all random games
(0.781% + 0.392%). Includes the Ace-low straight (ADR-006).
**Status**: Approved, not implemented

```
AS A player
I WANT to enter two hands where at least one contains a straight or a flush and be told who wins and why
SO THAT games decided by consecutive ranks or a single suit are settled correctly
```

**Architecture references**

- Ch 1.2 FR-2 (Straight, Flush), FR-3, FR-4
- Ch 4.4 Tiebreaker key (High Card / Flush key: five descending ranks)
- Ch 5.3 `ranker`, `comparator`, `formatter`
- Ch 8.1 `HandCategory` STRAIGHT = 5, FLUSH = 6
- Ch 10.2 QS-3 (tie within category)
- Ch 11.1 R-1 (tiebreaker key errors), R-2 (Royal Flush misclassified as Straight Flush)
- Ch 12 Glossary (Straight: "Ranked by the highest card"; Flush: "Ranked using High Card rules")
- ADR-003 (IntEnum and tuple comparison)
- ADR-006 (Ace-low straight: detected as `[2,3,4,5,14]`, key `(5,4,3,2,1)`)
- README.md "Decided questions" (primary-rank table: Straight = 1, Flush = 0;
  Ace-low straight follows ADR-006)

**Scope**

- In scope: Straight and Flush classification, their tiebreaker keys,
  reason/kicker population, and output strings, including the Ace-low straight.
- Out of scope: Straight Flush and Royal Flush (GAME-STORY-005). No hand in
  this bundle's scenarios is both a straight and a flush.

**Tiebreaker keys for this story:**

Both Straight and Flush use a five-element descending-rank key.

- Straight: `(r5, r4, r3, r2, r1)` descending. ADR-006 defines the wheel key
  as `(5, 4, 3, 2, 1)` (Ace treated as 1). All straight keys use the same
  five-element shape so that any two straight keys compare uniformly. Ch 12
  confirms the straight is "Ranked by the highest card", which is the first
  element of the key.
- Flush: `(r5, r4, r3, r2, r1)` descending, identical structure to High Card
  per Ch 4.4 and Ch 12 ("Ranked using High Card rules").

Primary ranks: Straight = 1 (the high card), Flush = 0 (every card is a
kicker). Reason for a Straight win is `(high_card_rank,)`. Reason for a Flush
win is the rank at the first difference, same as High Card.

---

### Acceptance scenarios (end to end, through the CLI)

### SCENARIO 1: Straight beats Three of a Kind

**Scenario ID**: GAME-STORY-004-S1

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: 4S 5H 6C 7D 8S  White: 9H 9D 9C KS 2H` is sent on stdin

**THEN**
* stdout contains exactly `Black wins. - with straight: 8`

### SCENARIO 2: Flush beats Straight

**Scenario ID**: GAME-STORY-004-S2

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: 2H 8H JH QH AH  White: 3S 4D 5C 6H 7S` is sent on stdin

**THEN**
* stdout contains exactly `Black wins. - with flush: Ace`

### SCENARIO 3: Ace-low straight (wheel) beats a Pair

**Scenario ID**: GAME-STORY-004-S3

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: AH 2D 3S 4C 5H  White: KC KH 7D 3D 9S` is sent on stdin

**THEN**
* stdout contains exactly `Black wins. - with straight: 5`

### SCENARIO 4: Wheel loses to the lowest regular straight (ADR-006)

**Scenario ID**: GAME-STORY-004-S4

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: 2C 3D 4H 5S 6H  White: AH 2D 3S 4C 5H` is sent on stdin

**THEN**
* stdout contains exactly `Black wins. - with straight: 6`

### SCENARIO 5: Higher straight wins

**Scenario ID**: GAME-STORY-004-S5

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: 9S TH JC QD KH  White: 4D 5H 6S 7C 8D` is sent on stdin

**THEN**
* stdout contains exactly `Black wins. - with straight: King`

### SCENARIO 6: Flush decided by second card

**Scenario ID**: GAME-STORY-004-S6

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: AH 9H 6H 4H 2H  White: AD 8D 6D 4D 2D` is sent on stdin

**THEN**
* stdout contains exactly `Black wins. - with flush: 9`

### SCENARIO 7: Two equal straights are a tie (QS-3)

**Scenario ID**: GAME-STORY-004-S7

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: 4S 5H 6C 7D 8S  White: 4D 5C 6H 7S 8H` is sent on stdin

**THEN**
* stdout contains exactly `Tie.`

### SCENARIO 8: Wrap-around K-A-2-3-4 is not a straight

**Scenario ID**: GAME-STORY-004-S8

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: KH AD 2S 3C 4H  White: QS JD 9C 7H 5D` is sent on stdin

**THEN**
* stdout contains exactly `Black wins. - with high card: Ace`

### SCENARIO 9: Ace-high straight ranks Ace as the top card

**Scenario ID**: GAME-STORY-004-S9

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: TS JH QC KD AH  White: 9D TC JS QH KS` is sent on stdin

**THEN**
* stdout contains exactly `Black wins. - with straight: Ace`

---

### INFRA sub-stories

#### GAME-INFRA-004.1: Straights and flushes acceptance feature file

```
AS A developer
I WANT the GAME-STORY-004 scenarios in their own feature file
SO THAT they run with the rest of the suite and appear separately in the test report
```

**Architecture references**: Ch 4.5 Testing strategy, Ch 8.5 Testability,
OC-2, QS-5, Ch 7.3 CI runner (Artefacts)

### SCENARIO 1: Straight and flush scenarios are collected alongside earlier ones

**Scenario ID**: GAME-INFRA-004.1-S1

**GIVEN**
* `features/straights_and_flushes.feature` contains GAME-STORY-004-S1 to S9

**WHEN**
* `pytest` is run from the repository root

**THEN**
* the nine scenarios are collected and reported by their names
* all earlier GAME-STORY scenarios still pass

### SCENARIO 2: CI reports the new scenarios without workflow changes

**Scenario ID**: GAME-INFRA-004.1-S2

**GIVEN**
* the CI workflow from GAME-INFRA-001.3

**WHEN**
* a commit adding `features/straights_and_flushes.feature` is pushed

**THEN**
* the workflow file is unchanged
* the JUnit XML report contains GAME-STORY-004-S1 to S9

#### Deployment, data store, events and monitoring: no new work

All reused from GAME-STORY-001 (GAME-INFRA-001.1, 001.3, 001.4). Data store
and events remain not applicable (Ch 1.2, ADR-001, Ch 3.1). No logging
(Ch 8.6).

---

### BE sub-stories

#### GAME-BE-004.1: Classify Straight with a tiebreaker key, including the wheel

```
AS A developer
I WANT `rank_hand` to return STRAIGHT with a five-element descending key, using (5,4,3,2,1) for the wheel, and HIGH_CARD for wrap-around sequences
SO THAT straights compare correctly and K-A-2-3-4 is not mistaken for a straight
```

**Architecture references**: Ch 5.3 `ranker`, Ch 8.1 STRAIGHT = 5,
Ch 12 Glossary ("Ranked by the highest card"), ADR-003, ADR-006, QS-6, R-1

### SCENARIO 1: Regular straight classified with correct key

**Scenario ID**: GAME-BE-004.1-S1

**GIVEN**
* a `Hand` built directly with 4S 5H 6C 7D 8S (straight 4 to 8)

**WHEN**
* `rank_hand` is called

**THEN**
* `ranked.category == HandCategory.STRAIGHT`
* `ranked.tiebreaker_key == (8, 7, 6, 5, 4)`

### SCENARIO 2: Ace-low straight classified with key (5,4,3,2,1)

**Scenario ID**: GAME-BE-004.1-S2

**GIVEN**
* a `Hand` built directly with AH 2D 3S 4C 5H (ranks `[14, 2, 3, 4, 5]`, detected as wheel per ADR-006)

**WHEN**
* `rank_hand` is called

**THEN**
* `ranked.category == HandCategory.STRAIGHT`
* `ranked.tiebreaker_key == (5, 4, 3, 2, 1)`

### SCENARIO 3: Wrap-around K-A-2-3-4 is classified as High Card, not a Straight

**Scenario ID**: GAME-BE-004.1-S3

**GIVEN**
* a `Hand` built directly with KH AD 2S 3C 4H (ranks sorted: `[2, 3, 4, 13, 14]`)

**WHEN**
* `rank_hand` is called

**THEN**
* `ranked.category == HandCategory.HIGH_CARD`
* `ranked.tiebreaker_key == (14, 13, 4, 3, 2)`

### SCENARIO 4: Ace-high straight classified with Ace as the high card

**Scenario ID**: GAME-BE-004.1-S4

**GIVEN**
* a `Hand` built directly with TS JH QC KD AH (ranks `[10, 11, 12, 13, 14]`)

**WHEN**
* `rank_hand` is called

**THEN**
* `ranked.category == HandCategory.STRAIGHT`
* `ranked.tiebreaker_key == (14, 13, 12, 11, 10)`

---

#### GAME-BE-004.2: Classify Flush with a tiebreaker key

```
AS A developer
I WANT `rank_hand` to return FLUSH with a five-element descending key
SO THAT flushes compare correctly with one tuple comparison, using the same logic as High Card
```

**Architecture references**: Ch 4.4 Tiebreaker key (High Card / Flush key),
Ch 5.3 `ranker`, Ch 8.1 FLUSH = 6, Ch 12 Glossary, ADR-003, QS-6

### SCENARIO 1: Flush classified with correct key

**Scenario ID**: GAME-BE-004.2-S1

**GIVEN**
* a `Hand` built directly with 2H 4H 8H JH AH (all Hearts, non-consecutive)

**WHEN**
* `rank_hand` is called

**THEN**
* `ranked.category == HandCategory.FLUSH`
* `ranked.tiebreaker_key == (14, 11, 8, 4, 2)`

### SCENARIO 2: Card order in input does not affect the key

**Scenario ID**: GAME-BE-004.2-S2

**GIVEN**
* the same five cards as GAME-BE-004.2-S1 in the order AH 2H JH 8H 4H

**WHEN**
* `rank_hand` is called

**THEN**
* `ranked.tiebreaker_key == (14, 11, 8, 4, 2)`

---

#### GAME-BE-004.3: Primary-rank table extended for Straight and Flush

```
AS A developer
I WANT the primary-rank table in `ranker` to include entries for Straight and Flush
SO THAT the comparator correctly identifies the reason and kicker for these categories
```

**Architecture references**: Ch 4.4 Tiebreaker key, Ch 5.3 `ranker`,
README.md "Which rank(s) the reason shows"

### SCENARIO 1: Table entries for Straight and Flush are correct

**Scenario ID**: GAME-BE-004.3-S1

**GIVEN**
* the primary-rank table in `ranker`

**WHEN**
* it is read for STRAIGHT and FLUSH

**THEN**
* STRAIGHT maps to 1
* FLUSH maps to 0

---

#### GAME-BE-004.4: Comparator resolves Straight and Flush results

```
AS A developer
I WANT `compare` to populate `result.reason` and `result.kicker` correctly for Straight and Flush
SO THAT the formatter produces the exact output string without poker knowledge
```

**Architecture references**: Ch 5.3 `comparator`, Ch 8.1 `Result`, ADR-002,
ADR-003, Ch 4.4, ADR-006, README.md "Decided questions"

### SCENARIO 1: Straight beats Three of a Kind, reason is the high card

**Scenario ID**: GAME-BE-004.4-S1

**GIVEN**
* Black is STRAIGHT with key `(8, 7, 6, 5, 4)`
  (valid input: `Black: 4S 5H 6C 7D 8S  White: 9H 9D 9C KS 2H`)
* White is THREE_OF_A_KIND with key `(9,)`

**WHEN**
* `compare(black, white)` is called

**THEN**
* `result.winner == "Black"`
* `result.category == HandCategory.STRAIGHT`
* `result.reason == (Rank.EIGHT,)`
* `result.kicker is None`

### SCENARIO 2: Wheel beats Pair, reason is 5 (high card of the wheel)

**Scenario ID**: GAME-BE-004.4-S2

**GIVEN**
* Black is STRAIGHT with key `(5, 4, 3, 2, 1)`
  (valid input: `Black: AH 2D 3S 4C 5H  White: KC KH 7D 3D 9S`)
* White is PAIR with key `(13, 9, 7, 3)`

**WHEN**
* `compare(black, white)` is called

**THEN**
* `result.winner == "Black"`
* `result.category == HandCategory.STRAIGHT`
* `result.reason == (Rank.FIVE,)`
* `result.kicker is None`

### SCENARIO 3: Flush beats Straight, reason is the flush's highest card

**Scenario ID**: GAME-BE-004.4-S3

**GIVEN**
* Black is FLUSH with key `(14, 12, 11, 8, 2)`
  (valid input: `Black: 2H 8H JH QH AH  White: 3S 4D 5C 6H 7S`)
* White is STRAIGHT with key `(7, 6, 5, 4, 3)`

**WHEN**
* `compare(black, white)` is called

**THEN**
* `result.winner == "Black"`
* `result.category == HandCategory.FLUSH`
* `result.reason == (Rank.ACE,)`
* `result.kicker is None`

### SCENARIO 4: Flush decided by second card, reason is that card

**Scenario ID**: GAME-BE-004.4-S4

**GIVEN**
* Black is FLUSH with key `(14, 9, 6, 4, 2)`
  (valid input: `Black: AH 9H 6H 4H 2H  White: AD 8D 6D 4D 2D`)
* White is FLUSH with key `(14, 8, 6, 4, 2)`

**WHEN**
* `compare(black, white)` is called

**THEN**
* `result.winner == "Black"`
* `result.category == HandCategory.FLUSH`
* `result.reason == (Rank.NINE,)`
* `result.kicker is None`

### SCENARIO 5: Identical flush hands are a tie

**Scenario ID**: GAME-BE-004.4-S5

**GIVEN**
* Black is FLUSH with key `(14, 9, 6, 4, 2)`
  (valid input: `Black: AH 9H 6H 4H 2H  White: AS 9S 6S 4S 2S`)
* White is FLUSH with key `(14, 9, 6, 4, 2)`

**WHEN**
* `compare(black, white)` is called

**THEN**
* `result.is_tie == True`
* `result.winner` is absent or None

---

### FE sub-stories

#### GAME-FE-004.1: Format Straight and Flush verdicts

```
AS A player
I WANT Straight and Flush results formatted in the kata's wording
SO THAT I can read the decisive rank at a glance
```

**Architecture references**: Ch 3.3 Poker Hands to Player, Ch 5.3 `formatter`,
Ch 1.2 FR-4, TD-3, README.md "Decided questions" (Ten displays as "10",
face cards spelled out)

### SCENARIO 1: Straight with a face card high

**Scenario ID**: GAME-FE-004.1-S1

**GIVEN**
* a Result with winner "Black", category STRAIGHT, reason `(Rank.KING,)`, kicker `None`
  (produced by `Black: 9S TH JC QD KH  White: 4D 5H 6S 7C 8D`)

**WHEN**
* `format_result` is called

**THEN**
* it returns `Black wins. - with straight: King`

### SCENARIO 2: Straight with a number high card (wheel)

**Scenario ID**: GAME-FE-004.1-S2

**GIVEN**
* a Result with winner "Black", category STRAIGHT, reason `(Rank.FIVE,)`, kicker `None`
  (produced by `Black: AH 2D 3S 4C 5H  White: KC KH 7D 3D 9S`)

**WHEN**
* `format_result` is called

**THEN**
* it returns `Black wins. - with straight: 5`

### SCENARIO 3: Flush with an Ace high

**Scenario ID**: GAME-FE-004.1-S3

**GIVEN**
* a Result with winner "Black", category FLUSH, reason `(Rank.ACE,)`, kicker `None`
  (produced by `Black: 2H 8H JH QH AH  White: 3S 4D 5C 6H 7S`)

**WHEN**
* `format_result` is called

**THEN**
* it returns `Black wins. - with flush: Ace`

### SCENARIO 4: Flush decided by a mid-rank card

**Scenario ID**: GAME-FE-004.1-S4

**GIVEN**
* a Result with winner "Black", category FLUSH, reason `(Rank.NINE,)`, kicker `None`
  (produced by `Black: AH 9H 6H 4H 2H  White: AD 8D 6D 4D 2D`)

**WHEN**
* `format_result` is called

**THEN**
* it returns `Black wins. - with flush: 9`

---

### Implementation order

1. GAME-INFRA-004.1: add `features/straights_and_flushes.feature`
2. GAME-BE-004.1: extend `ranker` to classify Straight, detect the wheel, and reject wrap-arounds
3. GAME-BE-004.2: extend `ranker` to classify Flush
4. GAME-BE-004.3: extend the primary-rank table in `ranker`
5. GAME-BE-004.4: extend `comparator` for Straight and Flush
6. GAME-FE-004.1: extend `formatter` for Straight and Flush
7. GAME-STORY-004-S1 to S9 end to end, plus earlier stories as regression

---

### Traceability check

| Scenario | Parent | Architecture | Checks (THEN) |
|---|---|---|---|
| GAME-STORY-004-S1 | GAME-STORY-004 | Ch 1.2 FR-2, FR-4 | exact output line |
| GAME-STORY-004-S2 | GAME-STORY-004 | Ch 1.2 FR-2, FR-4 | exact output line |
| GAME-STORY-004-S3 | GAME-STORY-004 | ADR-006, FR-2, FR-4 | exact output line |
| GAME-STORY-004-S4 | GAME-STORY-004 | ADR-006, R-1 | exact output line |
| GAME-STORY-004-S5 | GAME-STORY-004 | Ch 12 | exact output line |
| GAME-STORY-004-S6 | GAME-STORY-004 | Ch 4.4, R-1 | exact output line |
| GAME-STORY-004-S7 | GAME-STORY-004 | QS-3 | exact output line `Tie.` |
| GAME-STORY-004-S8 | GAME-STORY-004 | ADR-006, R-1 | exact output line; Ace wins as high card |
| GAME-STORY-004-S9 | GAME-STORY-004 | Ch 12, ADR-006 | exact output line; Ace is high not low |
| GAME-INFRA-004.1-S1 | GAME-INFRA-004.1 | Ch 8.5, OC-2 | scenarios collected; earlier pass |
| GAME-INFRA-004.1-S2 | GAME-INFRA-004.1 | Ch 7.3 | workflow unchanged; IDs in report |
| GAME-BE-004.1-S1 | GAME-BE-004.1 | Ch 12, Ch 8.1 | STRAIGHT; key (8,7,6,5,4) |
| GAME-BE-004.1-S2 | GAME-BE-004.1 | ADR-006, Ch 8.1 | STRAIGHT; key (5,4,3,2,1) |
| GAME-BE-004.1-S3 | GAME-BE-004.1 | ADR-006, R-1 | HIGH_CARD; key (14,13,4,3,2) |
| GAME-BE-004.1-S4 | GAME-BE-004.1 | Ch 12, ADR-006 | STRAIGHT; key (14,13,12,11,10) |
| GAME-BE-004.2-S1 | GAME-BE-004.2 | Ch 4.4, Ch 8.1 | FLUSH; key (14,11,8,4,2) |
| GAME-BE-004.2-S2 | GAME-BE-004.2 | Ch 4.4, QS-6 | same key regardless of card order |
| GAME-BE-004.3-S1 | GAME-BE-004.3 | Ch 4.4, Ch 5.3 ranker | STRAIGHT=1, FLUSH=0 |
| GAME-BE-004.4-S1 | GAME-BE-004.4 | Ch 5.3, ADR-002 | winner; reason (EIGHT,); kicker None |
| GAME-BE-004.4-S2 | GAME-BE-004.4 | ADR-006, ADR-002 | winner; reason (FIVE,); kicker None |
| GAME-BE-004.4-S3 | GAME-BE-004.4 | Ch 5.3, ADR-002 | winner; reason (ACE,); kicker None |
| GAME-BE-004.4-S4 | GAME-BE-004.4 | Ch 4.4, R-1 | winner; reason (NINE,); kicker None |
| GAME-BE-004.4-S5 | GAME-BE-004.4 | QS-3 | is_tie True; no winner |
| GAME-FE-004.1-S1 | GAME-FE-004.1 | Ch 3.3, FR-4 | `Black wins. - with straight: King` |
| GAME-FE-004.1-S2 | GAME-FE-004.1 | Ch 3.3, FR-4, ADR-006 | `Black wins. - with straight: 5` |
| GAME-FE-004.1-S3 | GAME-FE-004.1 | Ch 3.3, FR-4 | `Black wins. - with flush: Ace` |
| GAME-FE-004.1-S4 | GAME-FE-004.1 | Ch 3.3, FR-4 | `Black wins. - with flush: 9` |

---

## GAME-STORY-005: Straight flushes and royal flush

**Priority**: Supporting
**Value**: Straight Flush and Royal Flush together decide less than 0.01% of all
random games (0.0028% + 0.0003%). Settles the Royal Flush output format and
the detection-order rule from ADR-004.
**Status**: Approved, not implemented

```
AS A player
I WANT to enter two hands where at least one contains a straight flush or a royal flush and be told who wins and why
SO THAT the rarest and highest hands are recognised and reported correctly
```

**Architecture references**

- Ch 1.2 FR-2 (Straight Flush, Royal Flush), FR-3, FR-4
- Ch 5.3 `ranker` (detection order: Royal Flush checked before Straight Flush),
  `comparator`, `formatter`
- Ch 8.1 `HandCategory` STRAIGHT_FLUSH = 9, ROYAL_FLUSH = 10
- Ch 10.2 QS-2 (Royal Flush not misclassified as Straight Flush), QS-3 (tie)
- Ch 11.1 R-2 (Royal Flush misclassified as Straight Flush if detection order wrong)
- Ch 12 Glossary (Straight Flush: "Five consecutive ranks all of the same suit,
  other than 10-J-Q-K-A ... Includes the Ace-low A-2-3-4-5 of one suit"; Royal Flush:
  "A Straight Flush with ranks 10-J-Q-K-A. The highest possible hand; two Royal
  Flushes are always a tie.")
- ADR-003 (IntEnum and tuple comparison)
- ADR-004 (Royal Flush as a distinct category, `ROYAL_FLUSH = 10`, key `()`,
  detection order)
- README.md "Decided questions" (primary-rank table: Straight Flush = 1,
  Royal Flush = 0; Royal Flush output format settled here)

**Scope**

- In scope: Straight Flush and Royal Flush classification, detection order,
  tiebreaker keys, reason population, and output strings. This includes the
  suited wheel (A-2-3-4-5 of one suit), which is the lowest Straight Flush
  with key `(5, 4, 3, 2, 1)` per ADR-006. The ranker detects the suited wheel
  the same way as the unsuited wheel but, finding all five cards in the same
  suit, classifies it as STRAIGHT_FLUSH rather than STRAIGHT.
- Out of scope: classifying unsuited straights and plain flushes
  (GAME-STORY-004). S1 deliberately has a plain Flush on the losing side,
  which GAME-STORY-004 already classifies.
- Ch 12 defines a Straight Flush as any same-suit straight other than
  10-J-Q-K-A, explicitly including the suited wheel (ADR-006). The 36
  Straight Flushes in the README Pareto table include the 4 suited wheels.

**Tiebreaker keys:**
- Straight Flush: same five-element descending key as Straight, `(r5,r4,r3,r2,r1)`.
  Ch 12 "Ranked by the highest card"; all straight-family keys share the same shape.
  The suited wheel uses `(5, 4, 3, 2, 1)`, as in ADR-006.
- Royal Flush: `()` empty tuple per ADR-004. Two Royal Flushes are always a tie.

**Royal Flush output format (decided here):** `Result.reason` is `()` for a
Royal Flush. The formatter omits the colon and reason entirely:
`<Name> wins. - with royal flush`. This is recorded in the README decided
questions and must be used in all future references to Royal Flush output.

---

### Acceptance scenarios (end to end, through the CLI)

### SCENARIO 1: Straight Flush beats Flush

**Scenario ID**: GAME-STORY-005-S1

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: 5H 6H 7H 8H 9H  White: 2D 5D 8D JD KD` is sent on stdin

**THEN**
* stdout contains exactly `Black wins. - with straight flush: 9`

### SCENARIO 2: Royal Flush beats Straight Flush (QS-2)

**Scenario ID**: GAME-STORY-005-S2

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: TH JH QH KH AH  White: 9S TS JS QS KS` is sent on stdin

**THEN**
* stdout contains exactly `Black wins. - with royal flush`

### SCENARIO 3: Higher Straight Flush wins

**Scenario ID**: GAME-STORY-005-S3

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: 9H TH JH QH KH  White: 4D 5D 6D 7D 8D` is sent on stdin

**THEN**
* stdout contains exactly `Black wins. - with straight flush: King`

### SCENARIO 4: Two Royal Flushes are a tie (QS-3)

**Scenario ID**: GAME-STORY-005-S4

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: TH JH QH KH AH  White: TC JC QC KC AC` is sent on stdin

**THEN**
* stdout contains exactly `Tie.`

### SCENARIO 5: Two equal Straight Flushes are a tie (QS-3)

**Scenario ID**: GAME-STORY-005-S5

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: 5H 6H 7H 8H 9H  White: 5D 6D 7D 8D 9D` is sent on stdin

**THEN**
* stdout contains exactly `Tie.`

### SCENARIO 6: Suited wheel beats Four of a Kind

**Scenario ID**: GAME-STORY-005-S6

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: AH 2H 3H 4H 5H  White: 9S 9D 9C 9H KD` is sent on stdin

**THEN**
* stdout contains exactly `Black wins. - with straight flush: 5`

### SCENARIO 7: Suited wheel is the lowest Straight Flush

**Scenario ID**: GAME-STORY-005-S7

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: 2D 3D 4D 5D 6D  White: AH 2H 3H 4H 5H` is sent on stdin

**THEN**
* stdout contains exactly `Black wins. - with straight flush: 6`

---

### INFRA sub-stories

#### GAME-INFRA-005.1: Straight flushes and royal flush acceptance feature file

```
AS A developer
I WANT the GAME-STORY-005 scenarios in their own feature file
SO THAT they run with the rest of the suite and appear separately in the test report
```

**Architecture references**: Ch 4.5 Testing strategy, Ch 8.5 Testability,
OC-2, QS-5, Ch 7.3 CI runner (Artefacts)

### SCENARIO 1: Straight flush and royal flush scenarios are collected alongside earlier ones

**Scenario ID**: GAME-INFRA-005.1-S1

**GIVEN**
* `features/straight_flushes.feature` contains GAME-STORY-005-S1 to S7

**WHEN**
* `pytest` is run from the repository root

**THEN**
* the seven scenarios are collected and reported by their names
* all earlier GAME-STORY scenarios still pass

### SCENARIO 2: CI reports the new scenarios without workflow changes

**Scenario ID**: GAME-INFRA-005.1-S2

**GIVEN**
* the CI workflow from GAME-INFRA-001.3

**WHEN**
* a commit adding `features/straight_flushes.feature` is pushed

**THEN**
* the workflow file is unchanged
* the JUnit XML report contains GAME-STORY-005-S1 to S7

#### Deployment, data store, events and monitoring: no new work

All reused from GAME-STORY-001 (GAME-INFRA-001.1, 001.3, 001.4). Data store
and events remain not applicable (Ch 1.2, ADR-001, Ch 3.1). No logging
(Ch 8.6).

---

### BE sub-stories

#### GAME-BE-005.1: Classify Straight Flush and Royal Flush, with correct detection order

```
AS A developer
I WANT `rank_hand` to return ROYAL_FLUSH for 10-J-Q-K-A of a suit and STRAIGHT_FLUSH for all other same-suit straights, including the suited wheel, checking Royal Flush first
SO THAT a Royal Flush is never misclassified as a Straight Flush (ADR-004, R-2)
```

**Architecture references**: Ch 5.3 `ranker`, Ch 8.1 STRAIGHT_FLUSH = 9,
ROYAL_FLUSH = 10, Ch 12 Glossary, ADR-003, ADR-004, ADR-006, QS-2, R-2

### SCENARIO 1: Straight Flush classified with correct key

**Scenario ID**: GAME-BE-005.1-S1

**GIVEN**
* a `Hand` built directly with 5H 6H 7H 8H 9H (all Hearts, consecutive 5-9)

**WHEN**
* `rank_hand` is called

**THEN**
* `ranked.category == HandCategory.STRAIGHT_FLUSH`
* `ranked.tiebreaker_key == (9, 8, 7, 6, 5)`

### SCENARIO 2: Royal Flush classified with empty key, not Straight Flush (QS-2)

**Scenario ID**: GAME-BE-005.1-S2

**GIVEN**
* a `Hand` built directly with TH JH QH KH AH (all Hearts, 10-J-Q-K-A)

**WHEN**
* `rank_hand` is called

**THEN**
* `ranked.category == HandCategory.ROYAL_FLUSH`
* `ranked.tiebreaker_key == ()`

### SCENARIO 3: King-high same-suit straight is Straight Flush, not Royal Flush

**Scenario ID**: GAME-BE-005.1-S3

**GIVEN**
* a `Hand` built directly with 9H TH JH QH KH (all Hearts, 9-10-J-Q-K)

**WHEN**
* `rank_hand` is called

**THEN**
* `ranked.category == HandCategory.STRAIGHT_FLUSH`
* `ranked.tiebreaker_key == (13, 12, 11, 10, 9)`

### SCENARIO 4: Suited wheel is a Straight Flush with key (5,4,3,2,1)

**Scenario ID**: GAME-BE-005.1-S4

**GIVEN**
* a `Hand` built directly with AH 2H 3H 4H 5H (all Hearts, A-2-3-4-5)

**WHEN**
* `rank_hand` is called

**THEN**
* `ranked.category == HandCategory.STRAIGHT_FLUSH`
* `ranked.tiebreaker_key == (5, 4, 3, 2, 1)`

---

#### GAME-BE-005.2: Primary-rank table extended for Straight Flush and Royal Flush

```
AS A developer
I WANT the primary-rank table in `ranker` to include entries for Straight Flush and Royal Flush
SO THAT the comparator correctly identifies the reason for these categories
```

**Architecture references**: Ch 4.4 Tiebreaker key, Ch 5.3 `ranker`,
ADR-004, README.md "Which rank(s) the reason shows"

### SCENARIO 1: Table entries for Straight Flush and Royal Flush are correct

**Scenario ID**: GAME-BE-005.2-S1

**GIVEN**
* the primary-rank table in `ranker`

**WHEN**
* it is read for STRAIGHT_FLUSH and ROYAL_FLUSH

**THEN**
* STRAIGHT_FLUSH maps to 1
* ROYAL_FLUSH maps to 0

---

#### GAME-BE-005.3: Comparator resolves Straight Flush and Royal Flush results

```
AS A developer
I WANT `compare` to populate `result.reason` and `result.kicker` correctly for Straight Flush and Royal Flush
SO THAT the formatter produces the exact output string without poker knowledge
```

**Architecture references**: Ch 5.3 `comparator`, Ch 8.1 `Result`, ADR-002,
ADR-003, ADR-004, README.md "Decided questions"

### SCENARIO 1: Straight Flush beats Flush, reason is the high card

**Scenario ID**: GAME-BE-005.3-S1

**GIVEN**
* Black is STRAIGHT_FLUSH with key `(9, 8, 7, 6, 5)`
  (valid input: `Black: 5H 6H 7H 8H 9H  White: 2D 5D 8D JD KD`)
* White is FLUSH with key `(13, 11, 8, 5, 2)`

**WHEN**
* `compare(black, white)` is called

**THEN**
* `result.winner == "Black"`
* `result.category == HandCategory.STRAIGHT_FLUSH`
* `result.reason == (Rank.NINE,)`
* `result.kicker is None`

### SCENARIO 2: Royal Flush beats Straight Flush, reason is empty

**Scenario ID**: GAME-BE-005.3-S2

**GIVEN**
* Black is ROYAL_FLUSH with key `()`
  (valid input: `Black: TH JH QH KH AH  White: 9S TS JS QS KS`)
* White is STRAIGHT_FLUSH with key `(13, 12, 11, 10, 9)`

**WHEN**
* `compare(black, white)` is called

**THEN**
* `result.winner == "Black"`
* `result.category == HandCategory.ROYAL_FLUSH`
* `result.reason == ()`
* `result.kicker is None`

### SCENARIO 3: Two Royal Flushes are a tie

**Scenario ID**: GAME-BE-005.3-S3

**GIVEN**
* Black is ROYAL_FLUSH with key `()`
  (valid input: `Black: TH JH QH KH AH  White: TC JC QC KC AC`)
* White is ROYAL_FLUSH with key `()`

**WHEN**
* `compare(black, white)` is called

**THEN**
* `result.is_tie == True`
* `result.winner` is absent or None

### SCENARIO 4: Suited wheel beats Four of a Kind, reason is 5

**Scenario ID**: GAME-BE-005.3-S4

**GIVEN**
* Black is STRAIGHT_FLUSH with key `(5, 4, 3, 2, 1)`
  (valid input: `Black: AH 2H 3H 4H 5H  White: 9S 9D 9C 9H KD`)
* White is FOUR_OF_A_KIND with key `(9,)`

**WHEN**
* `compare(black, white)` is called

**THEN**
* `result.winner == "Black"`
* `result.category == HandCategory.STRAIGHT_FLUSH`
* `result.reason == (Rank.FIVE,)`
* `result.kicker is None`

---

### FE sub-stories

#### GAME-FE-005.1: Format Straight Flush and Royal Flush verdicts

```
AS A player
I WANT Straight Flush and Royal Flush results formatted in the kata's wording
SO THAT the highest hands are reported clearly, with no spurious reason for a Royal Flush
```

**Architecture references**: Ch 3.3 Poker Hands to Player, Ch 5.3 `formatter`,
Ch 1.2 FR-4, ADR-004, TD-3, README.md "Decided questions" (Royal Flush output
format: no colon or reason; face cards and Ten spelled out per existing rules)

### SCENARIO 1: Straight Flush with a number high card

**Scenario ID**: GAME-FE-005.1-S1

**GIVEN**
* a Result with winner "Black", category STRAIGHT_FLUSH, reason `(Rank.NINE,)`, kicker `None`
  (produced by `Black: 5H 6H 7H 8H 9H  White: 2D 5D 8D JD KD`)

**WHEN**
* `format_result` is called

**THEN**
* it returns `Black wins. - with straight flush: 9`

### SCENARIO 2: Straight Flush with a face card high

**Scenario ID**: GAME-FE-005.1-S2

**GIVEN**
* a Result with winner "Black", category STRAIGHT_FLUSH, reason `(Rank.KING,)`, kicker `None`
  (produced by `Black: 9H TH JH QH KH  White: 4D 5D 6D 7D 8D`)

**WHEN**
* `format_result` is called

**THEN**
* it returns `Black wins. - with straight flush: King`

### SCENARIO 3: Royal Flush produces no reason after the category

**Scenario ID**: GAME-FE-005.1-S3

**GIVEN**
* a Result with winner "Black", category ROYAL_FLUSH, reason `()`, kicker `None`
  (produced by `Black: TH JH QH KH AH  White: 9S TS JS QS KS`)

**WHEN**
* `format_result` is called

**THEN**
* it returns `Black wins. - with royal flush`

---

### Implementation order

1. GAME-INFRA-005.1: add `features/straight_flushes.feature`
2. GAME-BE-005.1: extend `ranker` to classify Straight Flush and Royal Flush, Royal Flush checked first
3. GAME-BE-005.2: extend the primary-rank table in `ranker`
4. GAME-BE-005.3: extend `comparator` for Straight Flush and Royal Flush
5. GAME-FE-005.1: extend `formatter` for Straight Flush and Royal Flush
6. GAME-STORY-005-S1 to S7 end to end, plus earlier stories as regression

---

### Traceability check

| Scenario | Parent | Architecture | Checks (THEN) |
|---|---|---|---|
| GAME-STORY-005-S1 | GAME-STORY-005 | Ch 1.2 FR-2, FR-4 | exact output line |
| GAME-STORY-005-S2 | GAME-STORY-005 | QS-2, ADR-004 | exact output line; royal flush label |
| GAME-STORY-005-S3 | GAME-STORY-005 | Ch 12, R-2 | exact output line |
| GAME-STORY-005-S4 | GAME-STORY-005 | QS-3, ADR-004 | exact output line `Tie.` |
| GAME-STORY-005-S5 | GAME-STORY-005 | QS-3 | exact output line `Tie.` |
| GAME-STORY-005-S6 | GAME-STORY-005 | ADR-006, Ch 12 | exact output line |
| GAME-STORY-005-S7 | GAME-STORY-005 | ADR-006 | exact output line |
| GAME-INFRA-005.1-S1 | GAME-INFRA-005.1 | Ch 8.5, OC-2 | scenarios collected; earlier pass |
| GAME-INFRA-005.1-S2 | GAME-INFRA-005.1 | Ch 7.3 | workflow unchanged; IDs in report |
| GAME-BE-005.1-S1 | GAME-BE-005.1 | Ch 12, Ch 8.1 | STRAIGHT_FLUSH; key (9,8,7,6,5) |
| GAME-BE-005.1-S2 | GAME-BE-005.1 | ADR-004, QS-2 | ROYAL_FLUSH; key () |
| GAME-BE-005.1-S3 | GAME-BE-005.1 | ADR-004, R-2 | STRAIGHT_FLUSH not ROYAL_FLUSH; key (13,12,11,10,9) |
| GAME-BE-005.1-S4 | GAME-BE-005.1 | ADR-006 | STRAIGHT_FLUSH; key (5,4,3,2,1) |
| GAME-BE-005.2-S1 | GAME-BE-005.2 | ADR-004, Ch 5.3 ranker | STRAIGHT_FLUSH=1, ROYAL_FLUSH=0 |
| GAME-BE-005.3-S1 | GAME-BE-005.3 | Ch 5.3, ADR-002 | winner; reason (NINE,); kicker None |
| GAME-BE-005.3-S2 | GAME-BE-005.3 | ADR-004, ADR-002 | winner; reason (); kicker None |
| GAME-BE-005.3-S3 | GAME-BE-005.3 | QS-3, ADR-004 | is_tie True; no winner |
| GAME-BE-005.3-S4 | GAME-BE-005.3 | ADR-006, ADR-002 | winner; reason (FIVE,); kicker None |
| GAME-FE-005.1-S1 | GAME-FE-005.1 | Ch 3.3, FR-4 | `Black wins. - with straight flush: 9` |
| GAME-FE-005.1-S2 | GAME-FE-005.1 | Ch 3.3, FR-4 | `Black wins. - with straight flush: King` |
| GAME-FE-005.1-S3 | GAME-FE-005.1 | Ch 3.3, FR-4, ADR-004 | `Black wins. - with royal flush` |
