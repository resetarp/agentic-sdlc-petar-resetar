# 2. Constraints

## 2.1 Technical Constraints

| ID    | Constraint | Rationale |
|-------|------------|-----------|
| TC-1  | Implementation language is **Python 3.12**. | Established in ADR-001. |
| TC-2  | **No third-party runtime dependencies.** Only the Python standard library (`enum`, `dataclasses`, `re`, etc.) may be used in production code. | Keeps the solution self-contained and portable; no package manager required to run the program. |
| TC-3  | Test toolchain is limited to **pytest** and **pytest-bdd**. No other test libraries. | Established in ADR-001. pytest-bdd ties acceptance scenarios directly to Gherkin feature files. |
| TC-4  | The program must run as a **CLI process** (stdin → stdout). No sockets, no HTTP, no GUI. | Established in ADR-001; dictated by the kata's input/output format. |
| TC-5  | **No persistence layer.** All state lives in memory for the duration of a single run. | Established in ADR-001. The kata is stateless by definition. |
| TC-6  | The system must handle exactly **5-card hands**. No other hand sizes are supported. | Defined by the kata specification. |
| TC-7  | Card notation follows the kata spec: values `2 3 4 5 6 7 8 9 T J Q K A`, suits `C D H S`. | Deviating from this notation would break the sample test cases. |

## 2.2 Organisational Constraints

| ID    | Constraint | Rationale |
|-------|------------|-----------|
| OC-1  | The solution must be **fully test-driven** (TDD/BDD). No production code without a failing test first. | The kata is an exercise in disciplined TDD practice, not just in getting a correct answer. |
| OC-2  | All acceptance criteria are expressed as **Gherkin scenarios** in `.feature` files, executed by pytest-bdd. | Keeps requirements human-readable and directly executable. |
| OC-3  | The codebase must pass **CI on every commit** with zero external service calls. | No network access in CI; the test suite must be fully hermetic. |
| OC-4  | Architecture is documented using the **arc42 template** with **C4 PlantUML diagrams**, versioned in Git alongside the code. | Project convention established before development started. |

## 2.3 Conventions

| ID    | Constraint | Rationale |
|-------|------------|-----------|
| CV-1  | Follow **PEP 8** and enforce it with **`ruff`** (linting), **`black`** (formatting, line length 88), and **`isort`** (import ordering) in CI, as configured in `pyproject.toml`. | Tools are already present in the repo; consistent style reduces review friction. |
| CV-2  | Use **`dataclasses`** for value objects (Card, Hand) and **`enum.IntEnum`** for ordered ranks and categories. | Idiomatic Python for domain value types; IntEnum makes rank comparison a simple `>` / `<`. |
| CV-3  | Module names mirror the domain: `card`, `hand`, `ranking`, `parser`, `cli`. No generic names like `utils` or `helpers`. | Keeps the ubiquitous language visible in the file system. |
| CV-4  | The nine hand categories must be named exactly as defined in Chapter 1 FR-2 everywhere in code, tests, and documentation. | A single ubiquitous language prevents translation errors between layers. |
