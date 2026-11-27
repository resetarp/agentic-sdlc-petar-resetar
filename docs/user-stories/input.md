# INPUT Domain: Story Bundles

Domain: **INPUT** ("What happens when I type something wrong?").
Owning modules: `parser` (validation, ADR-005) and `cli` (error handling,
Ch 8.2). Both stories build on the pipeline from GAME-STORY-001.

**How error output is checked in this file.** Ch 8.2 fixes the error line
format: a parser error is printed as `Error: invalid input - <message>.` and
any other exception as `Error: <message>`. The wrong-count message is fully
specified (`<Player> hand must contain exactly 5 cards, got <N>`, Ch 6.3 and
Ch 8.2), so those scenarios check the exact line. For bad tokens, duplicates
and a wrong line shape, Ch 8.2 only says what the message must name, so those
scenarios check:

- that the line starts with `Error: invalid input - `,
- that it contains what Ch 8.2 requires (the bad token, or the duplicate card
  and, across hands, both players),
- that no verdict (`wins.` or `Tie.`) is printed for that line.

Every malformed line below has exactly one problem, so the expected error is
unambiguous. The order in which the parser checks the rules is not specified.

---

## INPUT-STORY-001: Reject malformed input with a clear message

**Priority**: Supporting
**Value**: robustness. Without it, a typo either crashes the program or
reaches `ranker` and produces a wrong verdict with no warning (R-3).
**Status**: Approved, not implemented
**Depends on**: GAME-STORY-001

```
AS A player
I WANT a clear error message when I enter a line the program cannot read
SO THAT I can see what to fix instead of getting a crash or a wrong verdict
```

**Architecture references**

- Ch 1.2 FR-1 (input format), FR-5 (stdin and stdout)
- Ch 2.1 TC-6 (exactly 5 cards), TC-7 (card notation)
- Ch 3.3 Player to Poker Hands (input format and card notation)
- Ch 5.3 `parser`, `cli`
- Ch 6.3 Runtime scenario "Malformed Input"
- Ch 8.2 Error Handling, Ch 8.3 Validation (rules 1 to 4)
- ADR-005 (validate input exclusively in the parser)
- Ch 11.1 R-3 (parser accepts malformed input silently), Ch 11.2 TD-2

**Scope**

- In scope: the four validation rules of Ch 8.3 for a single input line, and
  the `cli` printing one `Error:` line for it.
- Out of scope: processing further lines after a bad line, unexpected
  exceptions, stack traces, blank lines and the exit status (INPUT-STORY-002).

### Acceptance scenarios (end to end, through the CLI)

### SCENARIO 1: Too few cards (Ch 6.3)

**Scenario ID**: INPUT-STORY-001-S1

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: 2H 3D 5S  White: 2C 3H 4S 8C AH` is sent on stdin

**THEN**
* stdout contains exactly `Error: invalid input - Black hand must contain exactly 5 cards, got 3.`

### SCENARIO 2: Too many cards

**Scenario ID**: INPUT-STORY-001-S2

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: 2H 3D 5S 9C KD 7H  White: 2C 3H 4S 8C AH` is sent on stdin

**THEN**
* stdout contains exactly `Error: invalid input - Black hand must contain exactly 5 cards, got 6.`

### SCENARIO 3: Ten written as 10 instead of T

**Scenario ID**: INPUT-STORY-001-S3

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: 2H 3D 5S 9C 10D  White: 2C 3H 4S 8C AH` is sent on stdin

**THEN**
* stdout contains exactly one line
* it starts with `Error: invalid input - `
* it contains the bad token `10D`

### SCENARIO 4: Unknown suit

**Scenario ID**: INPUT-STORY-001-S4

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: 2H 3D 5S 9C KX  White: 2C 3H 4S 8C AH` is sent on stdin

**THEN**
* stdout contains exactly one line
* it starts with `Error: invalid input - `
* it contains the bad token `KX`

### SCENARIO 5: Duplicate card within one hand

**Scenario ID**: INPUT-STORY-001-S5

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: 2H 4S 4C 3D 4S  White: 2C 3H 4H 8C AH` is sent on stdin

**THEN**
* stdout contains exactly one line
* it starts with `Error: invalid input - `
* it contains the duplicate card `4S`

### SCENARIO 6: Same card in both hands

**Scenario ID**: INPUT-STORY-001-S6

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: 2H 3D 5S 9C KD  White: 2C 3H 4S 8C KD` is sent on stdin

**THEN**
* stdout contains exactly one line
* it starts with `Error: invalid input - `
* it contains the duplicate card `KD` and both player names, `Black` and `White`

### SCENARIO 7: Line with only one hand

**Scenario ID**: INPUT-STORY-001-S7

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the line `Black: 2H 3D 5S 9C KD` is sent on stdin

**THEN**
* stdout contains exactly one line
* it starts with `Error: invalid input - `

In every scenario above, no line on stdout contains `wins.` or `Tie.`.

---

### INFRA sub-stories

#### INPUT-INFRA-001.1: Malformed-input acceptance feature file

```
AS A developer
I WANT the INPUT-STORY-001 scenarios in their own feature file
SO THAT they run with the rest of the suite and appear separately in the test report
```

**Architecture references**: Ch 4.5 Testing strategy, Ch 8.5 Testability,
OC-2, QS-5, Ch 7.3 CI runner (Artefacts)

### SCENARIO 1: Malformed-input scenarios are collected alongside the GAME ones

**Scenario ID**: INPUT-INFRA-001.1-S1

**GIVEN**
* `features/malformed_input.feature` contains INPUT-STORY-001-S1 to S7

**WHEN**
* `pytest` is run from the repository root

**THEN**
* the seven scenarios are collected and reported by their names
* all GAME-STORY scenarios still pass

### SCENARIO 2: CI reports the new scenarios without workflow changes

**Scenario ID**: INPUT-INFRA-001.1-S2

**GIVEN**
* the CI workflow from GAME-INFRA-001.3

**WHEN**
* a commit adding `features/malformed_input.feature` is pushed

**THEN**
* the workflow file is unchanged
* the JUnit XML report contains INPUT-STORY-001-S1 to S7

#### Deployment, data store, events and monitoring: no new work

Deployment (GAME-INFRA-001.1), CI (GAME-INFRA-001.3) and the test report
(GAME-INFRA-001.4) are reused unchanged. Data store and events remain not
applicable (Ch 1.2 Out of scope, ADR-001, Ch 3.1). There is no logging
(Ch 8.6): the error line on stdout is the only report of a bad line.

---

### BE sub-stories

All four sub-stories extend `parse_line` from GAME-BE-001.3. Each raises
`ValueError`; the parser never prints (Ch 8.2 "Only `cli` writes to stdout").

#### INPUT-BE-001.1: Reject lines that do not match the input pattern

```
AS A developer
I WANT `parse_line` to reject a line that does not have the shape `<Name>: C1 C2 C3 C4 C5  <Name>: C1 C2 C3 C4 C5`
SO THAT nothing downstream receives a half-parsed line
```

**Architecture references**: Ch 8.3 rule 1, Ch 1.2 FR-1, Ch 3.3, ADR-005

### SCENARIO 1: Second hand missing

**Scenario ID**: INPUT-BE-001.1-S1

**GIVEN**
* the line `Black: 2H 3D 5S 9C KD`

**WHEN**
* `parse_line` is called

**THEN**
* a `ValueError` is raised

### SCENARIO 2: Colon after the player name missing

**Scenario ID**: INPUT-BE-001.1-S2

**GIVEN**
* the line `Black 2H 3D 5S 9C KD  White: 2C 3H 4S 8C AH`

**WHEN**
* `parse_line` is called

**THEN**
* a `ValueError` is raised

#### INPUT-BE-001.2: Reject unknown card tokens

```
AS A developer
I WANT `parse_line` to reject any token that is not a value from `2 3 4 5 6 7 8 9 T J Q K A` followed by a suit from `C D H S`
SO THAT only real cards reach the domain model
```

**Architecture references**: Ch 8.3 rule 2, Ch 8.2 "Unrecognised rank or
suit token", Ch 2.1 TC-7, Ch 12 Card Notation, R-3

### SCENARIO 1: Ten written as 10

**Scenario ID**: INPUT-BE-001.2-S1

**GIVEN**
* the line `Black: 2H 3D 5S 9C 10D  White: 2C 3H 4S 8C AH`

**WHEN**
* `parse_line` is called

**THEN**
* a `ValueError` is raised
* its message contains `10D`

### SCENARIO 2: Unknown suit letter

**Scenario ID**: INPUT-BE-001.2-S2

**GIVEN**
* the line `Black: 2H 3D 5S 9C KX  White: 2C 3H 4S 8C AH`

**WHEN**
* `parse_line` is called

**THEN**
* a `ValueError` is raised
* its message contains `KX`

### SCENARIO 3: Lowercase card is not in the notation

**Scenario ID**: INPUT-BE-001.2-S3

**GIVEN**
* the line `Black: 2H 3D 5S 9C kd  White: 2C 3H 4S 8C AH`

**WHEN**
* `parse_line` is called

**THEN**
* a `ValueError` is raised
* its message contains `kd`

#### INPUT-BE-001.3: Reject hands without exactly five cards

```
AS A developer
I WANT `parse_line` to check the card count of each hand and name the player in the error
SO THAT the player knows which hand to fix
```

**Architecture references**: Ch 8.3 rule 3, Ch 8.2 "Wrong number of cards
in a hand", Ch 6.3, Ch 2.1 TC-6. `Hand` also rejects a wrong count
(GAME-BE-001.2), but only the parser knows the player's name in the input
line, so the parser checks first.

### SCENARIO 1: Three cards (Ch 6.3)

**Scenario ID**: INPUT-BE-001.3-S1

**GIVEN**
* the line `Black: 2H 3D 5S  White: 2C 3H 4S 8C AH`

**WHEN**
* `parse_line` is called

**THEN**
* a `ValueError` is raised
* its message is `Black hand must contain exactly 5 cards, got 3`

### SCENARIO 2: Six cards in the second hand

**Scenario ID**: INPUT-BE-001.3-S2

**GIVEN**
* the line `Black: 2H 3D 5S 9C KD  White: 2C 3H 4S 8C AH 7D`

**WHEN**
* `parse_line` is called

**THEN**
* a `ValueError` is raised
* its message is `White hand must contain exactly 5 cards, got 6`

#### INPUT-BE-001.4: Reject duplicate cards on the line

```
AS A developer
I WANT `parse_line` to reject a card that appears twice on the line, within one hand or across both hands
SO THAT impossible deals from a single 52-card deck never reach `ranker`
```

**Architecture references**: Ch 8.3 rule 4, Ch 8.2 "Duplicate card, within
one hand or shared by both hands", ADR-005, R-3

### SCENARIO 1: Duplicate within one hand

**Scenario ID**: INPUT-BE-001.4-S1

**GIVEN**
* the line `Black: 2H 4S 4C 3D 4S  White: 2C 3H 4H 8C AH`

**WHEN**
* `parse_line` is called

**THEN**
* a `ValueError` is raised
* its message contains `4S`

### SCENARIO 2: Same card in both hands

**Scenario ID**: INPUT-BE-001.4-S2

**GIVEN**
* the line `Black: 2H 3D 5S 9C KD  White: 2C 3H 4S 8C KD`

**WHEN**
* `parse_line` is called

**THEN**
* a `ValueError` is raised
* its message contains `KD`, `Black` and `White`

---

### FE sub-stories

#### INPUT-FE-001.1: Print parser errors as one error line

```
AS A player
I WANT a rejected line to produce one readable error line instead of a verdict
SO THAT I can see at once what was wrong with my input
```

**Architecture references**: Ch 5.3 `cli`, Ch 6.3, Ch 8.2 (rules "Only
`cli` writes to stdout" and the error line format `Error: invalid input - <message>.`)

### SCENARIO 1: Parser error becomes an error line

**Scenario ID**: INPUT-FE-001.1-S1

**GIVEN**
* stdin holds the line `Black: 2H 3D 5S  White: 2C 3H 4S 8C AH`

**WHEN**
* `main()` runs

**THEN**
* stdout holds exactly one line: `Error: invalid input - Black hand must contain exactly 5 cards, got 3.`
  (the parser's message, wrapped as Ch 8.2 specifies)

### SCENARIO 2: Valid lines are unaffected

**Scenario ID**: INPUT-FE-001.1-S2

**GIVEN**
* stdin holds the line `Black: 2H 3D 5S 9C KD  White: 2C 3H 4S 8C AH`

**WHEN**
* `main()` runs

**THEN**
* stdout holds exactly one line: `White wins. - with high card: Ace`

---

### Implementation order

1. INPUT-INFRA-001.1: add `features/malformed_input.feature`
2. INPUT-BE-001.1 to 001.4: parser rules 1 to 4 (Ch 8.3), in that order
3. INPUT-FE-001.1: error line in `cli`
4. INPUT-STORY-001-S1 to S7 end to end, plus all GAME-STORY scenarios as regression

### Traceability check

| Scenario | Parent | Architecture | Checks (THEN) |
|---|---|---|---|
| INPUT-STORY-001-S1 | INPUT-STORY-001 | Ch 6.3, Ch 8.2, Ch 8.3 rule 3 | exact error line |
| INPUT-STORY-001-S2 | INPUT-STORY-001 | Ch 8.2, Ch 8.3 rule 3, TC-6 | exact error line |
| INPUT-STORY-001-S3 | INPUT-STORY-001 | Ch 8.3 rule 2, TC-7 | one line; prefix; token named |
| INPUT-STORY-001-S4 | INPUT-STORY-001 | Ch 8.3 rule 2, TC-7 | one line; prefix; token named |
| INPUT-STORY-001-S5 | INPUT-STORY-001 | Ch 8.3 rule 4, R-3 | one line; prefix; card named |
| INPUT-STORY-001-S6 | INPUT-STORY-001 | Ch 8.3 rule 4, Ch 8.2 | one line; prefix; card and both players named |
| INPUT-STORY-001-S7 | INPUT-STORY-001 | Ch 8.3 rule 1, FR-1 | one line; prefix |
| INPUT-INFRA-001.1-S1 | INPUT-INFRA-001.1 | Ch 8.5, OC-2 | scenarios collected; GAME still passes |
| INPUT-INFRA-001.1-S2 | INPUT-INFRA-001.1 | Ch 7.3 | workflow unchanged; IDs in report |
| INPUT-BE-001.1-S1 | INPUT-BE-001.1 | Ch 8.3 rule 1 | ValueError |
| INPUT-BE-001.1-S2 | INPUT-BE-001.1 | Ch 8.3 rule 1, Ch 3.3 | ValueError |
| INPUT-BE-001.2-S1 | INPUT-BE-001.2 | Ch 8.2, TC-7 | ValueError naming `10D` |
| INPUT-BE-001.2-S2 | INPUT-BE-001.2 | Ch 8.2, TC-7 | ValueError naming `KX` |
| INPUT-BE-001.2-S3 | INPUT-BE-001.2 | Ch 8.2, Ch 12 Card Notation | ValueError naming `kd` |
| INPUT-BE-001.3-S1 | INPUT-BE-001.3 | Ch 6.3, Ch 8.2 | exact ValueError message, count 3 |
| INPUT-BE-001.3-S2 | INPUT-BE-001.3 | Ch 8.2, TC-6 | exact ValueError message, count 6 |
| INPUT-BE-001.4-S1 | INPUT-BE-001.4 | Ch 8.3 rule 4 | ValueError naming `4S` |
| INPUT-BE-001.4-S2 | INPUT-BE-001.4 | Ch 8.3 rule 4, Ch 8.2 | ValueError naming `KD`, Black, White |
| INPUT-FE-001.1-S1 | INPUT-FE-001.1 | Ch 6.3, Ch 8.2 | exact error line |
| INPUT-FE-001.1-S2 | INPUT-FE-001.1 | Ch 5.3 cli | verdict unchanged |

Result: all 20 scenarios have an architecture reference, a parent and a
concrete assertion. Each malformed line has exactly one problem.

---

## INPUT-STORY-002: Keep processing after a bad line

**Priority**: Supporting
**Value**: robustness for piped input. Without it, one bad line in a batch
stops every game after it.
**Status**: Approved, not implemented
**Depends on**: GAME-STORY-001, INPUT-STORY-001

```
AS A player piping many games into the program
I WANT a bad line to be reported and skipped
SO THAT every other game in the batch still gets its verdict
```

**Architecture references**

- Ch 1.2 FR-5 (stdin, interactive or piped)
- Ch 5.3 `cli` ("printing an error message and continuing", "Skips blank
  lines; exits with status 0 after the last line")
- Ch 8.2 Error Handling (row "Any other unexpected exception"; rules
  "Errors are reported per line", "No stack traces are shown to the user",
  "Blank or whitespace-only lines are skipped" and the exit status rule)
- Ch 8.5 Testability (dependencies are passed in; `cli` tested by
  subprocess or an injected buffer)

**Scope**

- In scope: continuing with the next line after any error, catching
  unexpected exceptions in `cli`, never showing a stack trace, skipping blank
  lines, and exiting with status 0.
- Out of scope: what counts as malformed (INPUT-STORY-001).

### Acceptance scenarios (end to end, through the CLI)

### SCENARIO 1: Bad line between two good lines

**Scenario ID**: INPUT-STORY-002-S1

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* these three lines are sent on stdin:
  `Black: 2H 3D 5S 9C KD  White: 2C 3H 4S 8C AH`,
  `Black: 2H 3D 5S  White: 2C 3H 4S 8C AH`,
  `Black: 2H 3D 5S 9C KD  White: 2D 3H 5C 9S KH`

**THEN**
* stdout holds three lines, in this order:
  `White wins. - with high card: Ace`,
  `Error: invalid input - Black hand must contain exactly 5 cards, got 3.`,
  `Tie.`

### SCENARIO 2: Several bad lines in a row

**Scenario ID**: INPUT-STORY-002-S2

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* these three lines are sent on stdin:
  `Black: 2H 3D 5S 9C KD  White: 2C 3H 4S 8C KD`,
  `Black: 2H 3D 5S 9C KX  White: 2C 3H 4S 8C AH`,
  `Black: 2H 3D 5S 9C KD  White: 2C 3H 4S 8C KH`

**THEN**
* stdout holds three lines, in this order:
  a line starting with `Error: invalid input - ` that contains `KD`,
  a line starting with `Error: invalid input - ` that contains `KX`,
  `Black wins. - with high card: 9`

### SCENARIO 3: Bad last line

**Scenario ID**: INPUT-STORY-002-S3

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* these two lines are sent on stdin:
  `Black: 2H 3D 5S 9C KD  White: 2C 3H 4S 8C AH`,
  `Black: 2H 3D 5S 9C KD`

**THEN**
* stdout holds two lines, in this order:
  `White wins. - with high card: Ace`,
  a line starting with `Error: invalid input - `
* the program ends after reading all input

### SCENARIO 4: No stack trace for bad input

**Scenario ID**: INPUT-STORY-002-S4

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the lines from INPUT-STORY-002-S2 are sent on stdin

**THEN**
* neither stdout nor stderr contains `Traceback`

### SCENARIO 5: Blank lines are skipped

**Scenario ID**: INPUT-STORY-002-S5

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* these four lines are sent on stdin:
  `Black: 2H 3D 5S 9C KD  White: 2C 3H 4S 8C AH`,
  an empty line,
  a line holding only spaces,
  `Black: 2H 3D 5S 9C KD  White: 2D 3H 5C 9S KH`

**THEN**
* stdout holds exactly two lines, in this order:
  `White wins. - with high card: Ace`,
  `Tie.`

### SCENARIO 6: Exit status is 0 even when a line had an error

**Scenario ID**: INPUT-STORY-002-S6

**GIVEN**
* the program is started with `python3 -m poker_hands`

**WHEN**
* the lines from INPUT-STORY-002-S1 are sent on stdin and stdin is closed

**THEN**
* the process exits with status 0

---

### INFRA sub-stories

#### INPUT-INFRA-002.1: Error-recovery acceptance feature file

```
AS A developer
I WANT the INPUT-STORY-002 scenarios in their own feature file
SO THAT they run with the rest of the suite and appear separately in the test report
```

**Architecture references**: Ch 4.5 Testing strategy, Ch 8.5 Testability,
OC-2, QS-5, Ch 7.3 CI runner (Artefacts)

### SCENARIO 1: Error-recovery scenarios are collected

**Scenario ID**: INPUT-INFRA-002.1-S1

**GIVEN**
* `features/error_recovery.feature` contains INPUT-STORY-002-S1 to S6

**WHEN**
* `pytest` is run from the repository root

**THEN**
* the six scenarios are collected and reported by their names
* all GAME-STORY and INPUT-STORY-001 scenarios still pass

### SCENARIO 2: CI reports the new scenarios without workflow changes

**Scenario ID**: INPUT-INFRA-002.1-S2

**GIVEN**
* the CI workflow from GAME-INFRA-001.3

**WHEN**
* a commit adding `features/error_recovery.feature` is pushed

**THEN**
* the workflow file is unchanged
* the JUnit XML report contains INPUT-STORY-002-S1 to S6

#### Deployment, data store, events and monitoring: no new work

Reused unchanged from GAME-STORY-001. Data store and events remain not
applicable (Ch 1.2, ADR-001, Ch 3.1). No logging (Ch 8.6).

---

### BE sub-stories

None. The parser already raises a `ValueError` for every malformed line
(INPUT-STORY-001), and this story only changes how `cli` reacts to errors.

---

### FE sub-stories

#### INPUT-FE-002.1: Continue with the next line after an error

```
AS A player
I WANT the program to move on to the next line after reporting an error
SO THAT one bad game does not hide the results of the others
```

**Architecture references**: Ch 5.3 `cli`, Ch 8.2 rules "Errors are reported
per line; a bad line does not abort processing of subsequent lines" and
"Blank or whitespace-only lines are skipped without any output"

### SCENARIO 1: Error line, then a verdict

**Scenario ID**: INPUT-FE-002.1-S1

**GIVEN**
* stdin holds two lines:
  `Black: 2H 3D 5S  White: 2C 3H 4S 8C AH`,
  `Black: 2H 3D 5S 9C KD  White: 2C 3H 4S 8C AH`

**WHEN**
* `main()` runs

**THEN**
* stdout holds two lines, in this order:
  `Error: invalid input - Black hand must contain exactly 5 cards, got 3.`,
  `White wins. - with high card: Ace`

### SCENARIO 2: Blank line produces no output

**Scenario ID**: INPUT-FE-002.1-S2

**GIVEN**
* stdin holds three lines:
  an empty line,
  `Black: 2H 3D 5S 9C KD  White: 2C 3H 4S 8C AH`,
  an empty line

**WHEN**
* `main()` runs

**THEN**
* stdout holds exactly one line: `White wins. - with high card: Ace`

#### INPUT-FE-002.2: Catch unexpected exceptions

```
AS A player
I WANT an unexpected failure on one line to be reported like any other error
SO THAT the program never stops with a stack trace
```

**Architecture references**: Ch 8.2 row "Any other unexpected exception"
(prints `Error: <message>` and continues), Ch 8.2 rule "No stack traces are
shown to the user", Ch 8.5 (dependencies are passed in)

No valid or malformed input line triggers an unexpected exception when the
code is correct, so this sub-story is the one place that uses a test double:
`main()` is given a ranker that raises, as Ch 8.5 allows. The handler exists
because Ch 8.2 requires it.

### SCENARIO 1: Unexpected exception becomes an error line

**Scenario ID**: INPUT-FE-002.2-S1

**GIVEN**
* `main()` is given a ranker that raises `RuntimeError("boom")` for the first line only
* stdin holds two lines:
  `Black: 2H 3D 5S 9C KD  White: 2C 3H 4S 8C AH`,
  `Black: 2H 3D 5S 9C KD  White: 2D 3H 5C 9S KH`

**WHEN**
* `main()` runs

**THEN**
* stdout holds two lines, in this order:
  `Error: boom`,
  `Tie.`

### SCENARIO 2: No stack trace for an unexpected exception

**Scenario ID**: INPUT-FE-002.2-S2

**GIVEN**
* the setup from INPUT-FE-002.2-S1

**WHEN**
* `main()` runs

**THEN**
* neither stdout nor stderr contains `Traceback`

---

### Implementation order

1. INPUT-INFRA-002.1: add `features/error_recovery.feature`
2. INPUT-FE-002.1: per-line error handling and blank-line skipping in `cli`
3. INPUT-FE-002.2: top-level handler for unexpected exceptions in `cli`
4. INPUT-STORY-002-S1 to S6 end to end, plus all earlier scenarios as regression

### Traceability check

| Scenario | Parent | Architecture | Checks (THEN) |
|---|---|---|---|
| INPUT-STORY-002-S1 | INPUT-STORY-002 | Ch 8.2, Ch 5.3 cli | three lines in order |
| INPUT-STORY-002-S2 | INPUT-STORY-002 | Ch 8.2 | three lines in order; tokens named |
| INPUT-STORY-002-S3 | INPUT-STORY-002 | Ch 8.2, FR-5 | two lines in order; program ends |
| INPUT-STORY-002-S4 | INPUT-STORY-002 | Ch 8.2 | no `Traceback` |
| INPUT-STORY-002-S5 | INPUT-STORY-002 | Ch 8.2, Ch 5.3 cli | blank lines skipped; two lines |
| INPUT-STORY-002-S6 | INPUT-STORY-002 | Ch 8.2, Ch 5.3 cli | exit status 0 |
| INPUT-INFRA-002.1-S1 | INPUT-INFRA-002.1 | Ch 8.5, OC-2 | scenarios collected; earlier pass |
| INPUT-INFRA-002.1-S2 | INPUT-INFRA-002.1 | Ch 7.3 | workflow unchanged; IDs in report |
| INPUT-FE-002.1-S1 | INPUT-FE-002.1 | Ch 8.2 | two lines in order |
| INPUT-FE-002.1-S2 | INPUT-FE-002.1 | Ch 8.2 | blank lines skipped |
| INPUT-FE-002.2-S1 | INPUT-FE-002.2 | Ch 8.2, Ch 8.5 | `Error: boom` then `Tie.` |
| INPUT-FE-002.2-S2 | INPUT-FE-002.2 | Ch 8.2 | no `Traceback` |

Result: all 12 scenarios have an architecture reference, a parent and a
concrete assertion.
