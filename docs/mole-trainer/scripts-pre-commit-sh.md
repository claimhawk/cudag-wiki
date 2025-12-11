```markdown
1. **Purpose**  
This script runs lightweight pre-commit quality gates on staged or all Python files in a Git repository. It enforces linting via Ruff, validates docstrings, and performs type checking with mypy (ignoring missing imports for external dependencies).

2. **Key Components**  
- `PYRUNNER`: Determines whether to use `uvx python` or fallback to `python`.  
- `PY_FILES`: Collects target `.py` files — either staged (`git diff --cached`) or all (`git ls-files`).  
- `ruff check`: Lints Python files for style and syntax issues.  
- `utils/check_docstrings.py`: Validates docstring formatting and presence.  
- `mypy --config-file mypy.ini`: Type-checks code with configurable rules.

3. **Algorithm Notes**  
Uses `git` to determine context and file targets. Falls back gracefully if no files are found. Pipes `git ls-files` or `git diff --cached` output into arrays for processing. Uses `set -euo pipefail` for strict error handling.

4. **Dependencies**  
- `git` (for repository context and file listing)  
- `ruff` (Python linter)  
- `mypy` (type checker)  
- `uvx` (optional Python runner; fallback to `python`)  
- `utils/check_docstrings.py` (local utility for docstring validation)

5. **Usage Example**  
```bash
# Run on staged files
./scripts/pre-commit.sh

# Run on all Python files
./scripts/pre-commit.sh --all
```
```