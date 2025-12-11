```markdown
# CODE_QUALITY.md — Python Code Quality Guidelines

## Purpose
This document defines **non-negotiable guardrails** for Python code in the `mole-trainer` project, ensuring code is readable, maintainable, and resistant to accidental breakage. It enforces strict typing, functional design, complexity limits, and idiomatic Python practices, treating every engineer as a “good ancestor” who must write code that future selves and peers can understand without friction.

## Key Components
- **Core Philosophy**: Prioritizes correctness, functional design, strict typing, and low complexity.
- **Language Subset**: Enforces Python 3, dataclasses, enums, comprehensions, and context managers; forbids deep inheritance, metaclasses, global state, and bare `except`.
- **Functional Paradigms**: Promotes pure functions, side-effect isolation, and immutability bias.
- **Strict Typing**: Requires full type hints, typed collections, and zero type errors in CI.
- **Complexity Limits**: Enforces max cyclomatic complexity (10), nesting depth (3), function length (60 lines), and parameter count (5).
- **Idiomatic Python**: Mandates naming conventions, early returns, and standard library preference.
- **Project Structure**: Layers code by responsibility (domain → infrastructure → interfaces).
- **Error Handling & Logging**: Enforces explicit exceptions, structured logging, and no sensitive data logging.
- **Testing & Docs**: Requires unit/integration tests and docstrings for all public APIs.
- **Pre-Build Pipeline**: Enforces formatting (Black), linting (Ruff), type checking (mypy), complexity analysis, security scans, and test coverage.
- **Code Review Standards**: Reviewers must validate correctness, simplicity, typing, complexity, style, and test coverage.
- **Cultural Rules**: Rejects “quick hacks”; favors simplicity over premature optimization.

## Algorithm Notes
- **Functional Design**: Functions are pure (no side effects) except at well-defined boundaries (I/O, DB, etc.).
- **Immutability Bias**: Prefer immutable data structures; mutate only locally and explicitly.
- **Early Returns**: Control flow favors early returns to reduce nesting.
- **Extract & Delegate**: Complex logic is extracted into small, named functions or services.
- **Side-Effect Boundaries**: Centralize I/O in `infrastructure` or `adapters` layers.

## Dependencies
- **Static Analysis**: `mypy`, `pyright`, `ruff`, `bandit`, `radon`, `xenon`
- **Formatting**: `black`
- **Testing**: `pytest`, `coverage`
- **Dev Tooling**: `uvx`, `pre-commit`, `scripts/check_docstrings.py`
- **Project Setup**: `requirements-dev.txt`

## Usage Example
```bash
# Run pre-commit hooks locally
pip install -r requirements-dev.txt
scripts/pre-commit.sh

# Run full build check (including all Python files)
make build

# Check only staged files
uvx python -m ruff check src scripts
uvx python -m mypy src scripts
```

> **Note**: This is a *guideline document*, not a module or executable. It governs how code must be written, reviewed, and built.
```