# 11. Risks and Technical Debts

## 11.1 Risks

| ID   | Risk | Likelihood | Impact | Mitigation |
|------|------|------------|--------|------------|
| R-1  | **Incorrect tiebreaker key construction in `ranker`** - a subtly wrong tuple (e.g. wrong ordering of kickers for Two Pairs) produces the right answer on common cases but fails on edge cases. | Medium | High - silent wrong answers undermine the kata's purpose. | Cover every tiebreaker variant with a dedicated BDD scenario; include cases where the winner is decided by the second or third kicker. |
| R-2  | **Royal Flush misclassified as Straight Flush** - if the ranker checks for Straight Flush before Royal Flush, a 10-J-Q-K-A flush is downgraded silently. | Low | Medium - wrong output for the highest possible hand. | Detection order in `ranker` must check Royal Flush first; enforced by a dedicated unit test (QS-2). |
| R-3  | **Parser accepts malformed input silently** - a gap in validation (e.g. duplicate cards not checked, within a hand or across both hands) lets bad data reach `ranker` and produce a nonsense result. | Low | Medium - incorrect output with no error message. | Validation checklist in `parser` covers token format, card count, and duplicate detection across the whole line; each rule has at least one negative test. |
| R-4  | **Convention-only module boundaries erode over time** - without a framework enforcing separation, a future change might add poker logic to `cli` or formatting logic to `ranker`. | Low | Low - the kata is small and short-lived, but sets a bad precedent. | Code review checks that new logic lands in the correct module; `ruff` rule B (flake8-bugbear) flags some common coupling patterns. |

## 11.2 Technical Debts

| ID   | Debt | Impact | Remediation |
|------|------|--------|-------------|
| TD-1 | **No `pyproject.toml` entry point defined** - the program is run with `python3 -m poker_hands` rather than a named console script. | Minor inconvenience; no functional impact. | Add a `[project.scripts]` entry in `pyproject.toml` when packaging becomes desirable. |
| TD-2 | **No negative / fuzz test coverage for the parser** - only the happy path and the specific error cases listed in Chapter 8.2 are tested. Unexpected input formats may produce unhelpful error messages. | Low for a kata; would matter if the program were exposed to arbitrary user input. | Add property-based tests (e.g. with `hypothesis`) if the input surface grows. |
| TD-3 | **Hand category output strings are not centralised** - if `formatter` constructs reason strings independently of the `HandCategory` enum, renaming a category requires changes in two places. | Low risk now; grows if more categories or output formats are added. | Map `HandCategory` values to their display strings in one place (e.g. a dict in `formatter` or a `label` property on the enum). |
