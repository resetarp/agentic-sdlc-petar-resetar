# 10. Quality Requirements

Quality goals were established in Chapter 1.3. This chapter makes them
measurable with concrete scenarios.

## 10.1 Quality Tree

```
Quality
├── Correctness (highest priority)
│   ├── All ten hand categories classified correctly
│   ├── All tie-breaking rules applied correctly
│   └── All kata sample cases produce the expected output
├── Testability
│   ├── Every module testable in isolation
│   └── Acceptance criteria expressed as executable Gherkin scenarios
├── Readability
│   ├── Domain vocabulary visible in module and type names
│   └── Code style enforced automatically
└── Simplicity
    ├── No runtime dependencies beyond stdlib
    └── No more components than the problem requires
```

## 10.2 Quality Scenarios

| ID   | Quality attribute | Stimulus | Response | Measurable criterion |
|------|-------------------|----------|----------|----------------------|
| QS-1 | Correctness | The four kata sample inputs are fed to the program. | The program produces the four expected output lines verbatim. | `pytest` suite green; all four sample cases pass as BDD scenarios. |
| QS-2 | Correctness | A hand containing `TH JH QH KH AH` is submitted. | The program reports `royal flush`, not `straight flush`. | Dedicated BDD scenario passes. |
| QS-3 | Correctness | Two hands in the same category with identical tiebreaker cards are submitted. | The program outputs `Tie.` | BDD scenario for each of the ten categories passes. |
| QS-4 | Correctness | Two hands where the winner is decided by a kicker (not the primary rank) are submitted. | The correct winner is reported. | BDD scenarios for Pair, Two Pairs, and High Card kicker cases all pass. |
| QS-5 | Testability | A developer adds a new BDD scenario for an untested edge case. | The scenario can be wired to existing step definitions or new ones without touching production code outside `ranker`. | New scenario is runnable with `pytest` in under 5 seconds. |
| QS-6 | Testability | `ranker` is unit-tested without invoking `parser` or `cli`. | Tests construct `Hand` objects directly and call `rank_hand()`. | All `ranker` unit tests pass with no I/O or subprocess calls. |
| QS-7 | Readability | A new developer reads the codebase for the first time. | They can identify where hand classification rules live within 2 minutes. | Verified by code review; module named `ranker` contains all poker rules. |
| QS-8 | Simplicity | The project is checked out on a clean machine with only Python 3.12 installed. | `pip install pytest pytest-bdd && pytest` is the complete setup; the program runs with `python3 -m poker_hands`. | No other installation steps are needed; verified in CI from a fresh runner. |
| QS-9 | Simplicity | CI pipeline runs the full test suite. | All checks complete in under 30 seconds. | CI wall-clock time measured on GitHub Actions. |
