# Poker Hands - User Story Inventory

## Domains

| Domain | Modules | What the user experiences |
|--------|---------|--------------------------|
| GAME | `card`, `hand`, `ranker`, `comparator`, `formatter`, `parser`, `cli` | "Who wins, and why?" - entering two hands and getting a verdict |
| INPUT | `parser`, `cli` | "What happens when I type something wrong?" - clear error messages, no crashes |

---

## Pareto Analysis

### Probability calculation

A game is decided at the level of the higher hand's category. For two independently
drawn hands, the probability that a game is decided at category c is:

```
P(decided at c) = F(c)^2 - F(c-1)^2
```

where F(c) is the probability a random 5-card hand from a 52-card deck is
category c or lower, and F(c-1) is the same for the category just below c.
Hand counts from combinatorics (total = 2,598,960). The Straight and
Straight Flush counts include the Ace-low straight, as ADR-006 requires.

| Category | Count | Cumulative F(c) | P(decided at c) |
|----------|-------|-----------------|-----------------|
| High Card | 1,302,540 | 0.501177 | 25.1179% |
| Pair | 1,098,240 | 0.923746 | 60.2129% |
| Two Pairs | 123,552 | 0.971285 | 9.0088% |
| Three of a Kind | 54,912 | 0.992414 | 4.1490% |
| Straight | 10,200 | 0.996339 | 0.7805% |
| Flush | 5,108 | 0.998304 | 0.3920% |
| Full House | 3,744 | 0.999745 | 0.2878% |
| Four of a Kind | 624 | 0.999985 | 0.0480% |
| Straight Flush | 36 | 0.999998 | 0.0028% |
| Royal Flush | 4 | 1.000000 | 0.0003% |

A story's value is the sum of P(decided at c) over its categories. Stories
003 and 004 are not contiguous ranges (Straight and Flush rank between Three
of a Kind and Full House), so their values are sums of separate rows, not a
single F^2 difference.

| Story | Categories covered | Calculation | P(game decided here) | Cumulative |
|-------|--------------------|-------------|----------------------|------------|
| GAME-STORY-001 | High Card | 25.1179 | 25.1% | 25.1% |
| GAME-STORY-002 | Pair, Two Pairs | 60.2129 + 9.0088 | 69.2% | 94.3% |
| GAME-STORY-003 | Three of a Kind, Full House, Four of a Kind | 4.1490 + 0.2878 + 0.0480 | 4.5% | 98.8% |
| GAME-STORY-004 | Straight, Flush | 0.7805 + 0.3920 | 1.2% | 100.0% |
| GAME-STORY-005 | Straight Flush, Royal Flush | 0.0028 + 0.0003 | < 0.01% | 100.0% |

**Core stories: GAME-STORY-001 + GAME-STORY-002 together decide 94.3% of all random games.**
That is 2 of 7 stories (29%) delivering 94.3% of measurable game-outcome value.
A strict 20% of 7 stories would be 1.4 stories. GAME-STORY-002 is included
in the core because Pair alone decides 60.2% of games.

### Four core-story questions per Core story

**GAME-STORY-001 - Decide the winner by high card**
1. MVP? Yes - the kata requires a working end-to-end pipeline; this story builds it.
2. Blocks others? Yes - every later GAME story extends the ranker that this story introduces; INPUT stories depend on the cli this story introduces.
3. User value? 25.1% of all random games are decided at High Card level.
4. Architecture components exercised? All seven: `card`, `hand`, `parser`, `ranker`, `comparator`, `formatter`, `cli` (Ch 4.3, Ch 5.2, Ch 5.3).

**GAME-STORY-002 - Decide the winner with pairs**
1. MVP? Yes - Pair is the most common deciding category (69.2% of games). Without it, every hand containing a pair would be classified as High Card, so about 69% of games would get a wrong verdict, not an error.
2. Blocks others? No direct blocks, but covers the dominant code path through the ranker.
3. User value? 69.2% of all random games are decided by Pair or Two Pairs.
4. Architecture components exercised? `ranker` (Pair, Two Pairs classification and tiebreaker keys), `comparator` (kicker comparison), `formatter` (new reason formats). Ch 4.4, Ch 5.3, ADR-003, R-1.

---

## Prioritized Story List

| ID | Title | Value (% games decided) | Depends on | Priority |
|----|-------|------------------------|------------|----------|
| GAME-STORY-001 | Decide the winner by high card | 25.1% | none | **Core** |
| GAME-STORY-002 | Decide the winner with pairs | 69.2% | GAME-STORY-001 | **Core** |
| GAME-STORY-003 | Sets of equal ranks (three of a kind, full house, four of a kind) | 4.5% | GAME-STORY-001 | Supporting |
| GAME-STORY-004 | Straights and flushes, including ace-low straight (ADR-006) | 1.2% | GAME-STORY-001 | Supporting |
| GAME-STORY-005 | Straight flushes and royal flush (ADR-004) | < 0.01% | GAME-STORY-001, GAME-STORY-004 | Supporting |
| INPUT-STORY-001 | Reject malformed input with a clear message | n/a | GAME-STORY-001 | Supporting |
| INPUT-STORY-002 | Keep processing after a bad line; no stack traces | n/a | GAME-STORY-001, INPUT-STORY-001 | Supporting |

**Kata sample coverage (QS-1):** samples 1, 3 and 4 are covered by
GAME-STORY-001 (S1, S2, S3). Sample 2
(`Black: 2H 4S 4C 2D 4H  White: 2S 8S AS QS 3S` -> `Black wins. - with full house: 4 over 2`)
is a Full House, so GAME-STORY-003 must include it as an acceptance scenario.

**Duplicate cards (INPUT-STORY-001):** Ch 8.2 and 8.3 rule 4 reject a card
that appears twice anywhere on the input line, within one hand or across both
hands (single 52-card deck). INPUT-STORY-001 must include an acceptance
scenario for each case, for example:
- within a hand: `Black: 2H 4S 4C 3D 4S  White: 2C 3H 4H 8C AH` (4S twice in Black's hand)
- across hands: `Black: 2H 3D 5S 9C KD  White: 2C 3H 4S 8C KD` (KD in both hands)

Both must print an `Error:` line naming the duplicate card, and processing
must continue with the next line (Ch 8.2).

---

## Decided questions (not open)

- **The comparator computes the reason; the formatter displays it.** `Result.reason` is a tuple of `Rank` values naming the rank(s) that decided the game. For High Card it is the first rank that differs between the two tiebreaker keys, e.g. `(Rank.NINE,)`; for a Full House it will be `(Rank.FOUR, Rank.TWO)` ("4 over 2"). The formatter turns ranks into text, in one place (TD-3).
- **Which rank(s) the reason shows.** The reason always describes the winner's hand. If the game was decided by a kicker, the deciding kicker is added after it as `high card: <kicker>`.
  Each category's tiebreaker key starts with its *primary* ranks (the ranks that make the combination), followed by kickers:

  | Category | Primary ranks at the start of the key |
  |---|---|
  | High Card, Flush | 0 (every card is a kicker) |
  | Pair, Three of a Kind, Four of a Kind, Straight, Straight Flush | 1 |
  | Two Pairs, Full House | 2 |
  | Royal Flush | 0 (empty key) |

  `Result` carries two fields for this, both as `Rank` values (no text):
  - `reason`: the winner's primary ranks. For a category with 0 primary ranks, it is the winner's rank at the first difference, or the winner's highest card when the categories differ.
  - `kicker`: the winner's rank at the first difference when that difference falls after the primary ranks; otherwise `None`. Always `None` for categories with 0 primary ranks, because their deciding card is already the reason.

  The comparator applies one rule:
  1. Tie: no reason, no kicker.
  2. Categories differ, or the first difference falls inside the primary ranks: `reason` as above, `kicker = None`.
  3. Otherwise (decided by a kicker): `reason` as above, `kicker` = the winner's rank at the first difference.

  The formatter writes `<category>: <reason>`, then `, high card: <kicker>` when `kicker` is set.

  Examples:
  - Full House beats Flush (kata sample 2): reason `(Rank.FOUR, Rank.TWO)`, kicker `None` -> `Black wins. - with full house: 4 over 2`
  - Full House `4 over 2` vs `3 over King`: decided by the primary 4 -> `full house: 4 over 2`
  - Both a pair of 5s, decided by a King kicker: reason `(Rank.FIVE,)`, kicker `Rank.KING` -> `Black wins. - with pair: 5, high card: King`
  - High Card decided by the second card (kata sample 3): reason `(Rank.NINE,)`, kicker `None` -> `Black wins. - with high card: 9`

  The primary-rank table lives in `ranker` next to the key definitions, because it describes the key layout (Ch 4.4). The comparator only reads it, so it holds no poker rules of its own (Ch 5.3).
  GAME-STORY-002 settles the Two Pairs wording: two primary ranks are written as `<high> and <low>`, e.g. `two pairs: Jack and 8` (see GAME-FE-002.1-S3). Full House uses "over" (three over pair); Two Pairs uses "and". GAME-STORY-003 must include a scenario decided by primary ranks. The wording for Royal Flush (empty reason) is settled in GAME-STORY-005.
- **Ten displays as `10`.** Jack, Queen, King, and Ace are spelled out in full. All other ranks display as their integer value.
- **Ace-low straight follows ADR-006.** The ranker detects `[2, 3, 4, 5, 14]` explicitly and uses tiebreaker key `(5, 4, 3, 2, 1)`.

---

## Open Questions

None. Architecture consistency was checked on 2026-11-26 against commit 40873a8:

- `HandCategory` range: Ch 5.3 (05-building-block-view.md line 55) reads "a `HandCategory` (IntEnum, 1-10)" (en dash normalised to a hyphen) and line 58 reads "`ROYAL_FLUSH` is category 10", matching Ch 4.4, Ch 8.1, ADR-003 and ADR-004.
- Module names: CV-3 (02-constraints.md line 30) reads "`card`, `hand`, `parser`, `ranker`, `comparator`, `formatter`, `cli`", matching Ch 4.3 and Ch 5.3.

---

## Pareto Progress

- **Pareto Progress: 1/2 core stories (50% of 20% core stories)**
  - Calculation: 1 approved story (GAME-STORY-001) / 2 core stories = 50%.
- **Core functionality coverage: 25.1% of the 94.3% target**
  - Calculation: GAME-STORY-001 approved = 25.1%. GAME-STORY-002 pending approval would add 69.2%, reaching 94.3%.

---

## Story Bundles - Status

| Story ID | Title | File | Status |
|----------|-------|------|--------|
| GAME-STORY-001 | Decide the winner by high card | `game.md` | Approved |
| GAME-STORY-002 | Decide the winner with pairs | `game.md` | Bundle written, awaiting approval |
| GAME-STORY-003 | Sets of equal ranks | `game.md` | Pending |
| GAME-STORY-004 | Straights and flushes | `game.md` | Pending |
| GAME-STORY-005 | Straight flushes and royal flush | `game.md` | Pending |
| INPUT-STORY-001 | Reject malformed input with a clear message | `input.md` | Pending |
| INPUT-STORY-002 | Keep processing after a bad line | `input.md` | Pending |

---

## Traceability Index

| Domain file | Stories |
|-------------|---------|
| `docs/user-stories/game.md` | GAME-STORY-001, GAME-STORY-002 |

`input.md` does not exist yet. It is created with INPUT-STORY-001. Stories
GAME-STORY-003 to 005 are added to `game.md` as their bundles are written.
