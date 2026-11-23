# 1. Introduction and Goals

## 1.1 Purpose

Poker Hands is a command-line program that solves the
[Poker Hands coding kata](https://codingdojo.org/kata/PokerHands/).

Given two named 5-card poker hands on a single line of text, the program
determines which hand wins (or whether the game is a tie) and prints the
result with the reason. For example:

```
Input:  Black: 2H 3D 5S 9C KD  White: 2C 3H 4S 8C AH
Output: White wins. - with high card: Ace
```

The kata is a vehicle for practising clean domain modelling, test-driven
development with pytest-bdd, and a well-structured Python codebase — not for
building production infrastructure.

## 1.2 Requirements Overview

### Functional requirements

| ID   | Requirement |
|------|-------------|
| FR-1 | Parse a line of input containing two named 5-card hands in the format `<Name>: C1 C2 C3 C4 C5`. |
| FR-2 | Recognise all nine hand categories, ranked lowest to highest: High Card, Pair, Two Pairs, Three of a Kind, Straight, Flush, Full House, Four of a Kind, Straight Flush, Royal Flush. A Royal Flush is a Straight Flush topped by an Ace (10-J-Q-K-A of the same suit) and ranks above all other hands. |
| FR-3 | Compare two hands and determine the winner, applying tie-breaking rules within the same category (kicker cards, highest pair, etc.). |
| FR-4 | Report the result in natural language: `<Name> wins. - with <category>: <reason>` or `Tie.` |
| FR-5 | Accept input from stdin (interactive or piped) and write output to stdout. |
| FR-6 | A test harness (pytest-bdd) exercises all sample cases from the kata and additional edge cases. |

### Out of scope

- Persistence of game history
- Multi-player or networked play
- Wild cards or any non-standard card variants
- Texas Hold'em or any variant beyond 5-card hand comparison
- A graphical or web interface

## 1.3 Quality Goals

The top quality goals, in priority order:

| Priority | Quality attribute | Motivation |
|----------|-------------------|------------|
| 1 | **Correctness** | Every hand category and tie-breaking rule must produce the right answer. A wrong result defeats the entire purpose of the kata. |
| 2 | **Testability** | All logic must be exercisable in isolation. BDD scenarios must map directly to acceptance criteria. |
| 3 | **Readability** | Code and architecture should communicate intent clearly: a new reader should understand the domain model in minutes. |
| 4 | **Simplicity** | No dependencies beyond the Python standard library and the test toolchain. The architecture must not be more complex than the problem demands. |

## 1.4 Stakeholders

| Role | Expectation |
|------|-------------|
| **Developer / kata practitioner** | A clean, working solution that demonstrates good TDD and domain-modelling habits. |
| **Code reviewer** | Clear separation between parsing, domain logic, and I/O; readable test scenarios. |
| **CI pipeline** | `pytest` exits 0 on a green suite; runs in under a few seconds with no external dependencies. |
