```markdown
## Purpose

`CODING_STEPS.md` defines the development workflow and working agreements for the `mole-trainer` project, emphasizing small, atomic commits with automated quality gates. It ensures code is typed, tested, documented, and version-controlled, enabling reliable time-travel through git history via feature branches and tags.

## Key Components

- **Workflow Loop**: A 8-step iterative process for committing atomic changes with safety checks.
- **Branch & Tag**: Convention for creating feature branches and maintaining matching tags.
- **Quality Gates**: Pre-commit hooks and `uvx` commands enforcing Ruff, MyPy, and docstring checks.
- **Output Verification**: Guidance to regenerate and visually inspect artifacts after rendering changes.
- **Repo Tooling**: Scripts (`pre-commit.sh`, `check_docstrings.py`) and config (`mypy.ini`) for automated enforcement.

## Algorithm Notes

- **Atomic Commit Strategy**: Each step must be independently shippable — buildable, typed, lint-clean, and testable.
- **Progressive Refinement**: Changes are split into smallest units; growing steps are refactored into smaller commits.
- **Reproducibility**: Manual data extraction or asset generation must be scripted to prevent drift.

## Dependencies

- `uvx` (for Python tool invocations)
- `Ruff` (Python linter)
- `Mypy` (static type checker)
- `scripts/pre-commit.sh` (local hook script)
- `scripts/check_docstrings.py` (docstring validator)
- `mypy.ini` (type checker config)

## Usage Example

```bash
# Create feature branch and tag
git checkout -b feature/data-grid-generation
git tag -f feature/data-grid-generation

# Make atomic change, then run quality gates
uvx python -m ruff check src tests
uvx python -m mypy src
./scripts/pre-commit.sh --all

# Verify output visually
./scripts/generate.sh --dataset-name sanity --test

# Commit and update tag
git commit -m "Add data grid generation logic"
git tag -f feature/data-grid-generation
```
```