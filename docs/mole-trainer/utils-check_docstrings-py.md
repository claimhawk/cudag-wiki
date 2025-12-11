```markdown
1. **Purpose**  
This utility enforces docstring requirements on Python modules and public top-level symbols (classes/functions). It scans source files for missing module docstrings and missing docstrings on public classes/functions (those not prefixed with `_`), reporting violations via CLI output.

2. **Key Components**  
- `has_docstring(node: ast.AST)` → Checks if an AST node has a docstring.  
- `iter_public_toplevel(nodes: Iterable[ast.stmt])` → Yields public top-level classes/functions (excluding those starting with `_`).  
- `check_file(path: Path)` → Validates docstrings in a single file and returns error messages.  
- `main(argv: list[str] | None = None)` → CLI entrypoint that processes files and exits with 1 if errors found.

3. **Algorithm Notes**  
Uses Python’s `ast` module to parse source files into an abstract syntax tree (AST). Iterates over top-level nodes, filters public symbols, and checks for docstrings via `ast.get_docstring()`. Errors are collected and printed to stderr.

4. **Dependencies**  
- `argparse` — for CLI argument parsing  
- `ast` — for AST traversal and docstring extraction  
- `sys` — for stderr output and exit codes  
- `pathlib.Path` — for file path handling  
- `typing.Iterable` — for type annotations

5. **Usage Example**  
```bash
python -m utils.check_docstrings src/module1.py src/module2.py
```
Outputs errors to stderr if any docstrings are missing; exits with code 1 if violations exist.
```