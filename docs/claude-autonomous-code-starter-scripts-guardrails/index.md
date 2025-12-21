# Project Overview: `claude-autonomous-code-starter/scripts/guardrails`

---

## 1. Overview

This project, located at `claude-autonomous-code-starter/scripts/guardrails`, implements a **guardrail system** designed to enforce safety, compliance, and correctness during autonomous code generation and modification workflows powered by Claude (or similar LLMs). It acts as a **pre- and post-execution validation layer** for code operations, ensuring that generated or edited code adheres to predefined constraints — such as secret handling, command safety, edit scope, and configuration validity.

The system is designed to be integrated into an autonomous code agent pipeline, where it intercepts code generation or modification events and validates them before proceeding. It is particularly useful in environments where untrusted or semi-trusted AI-generated code must be audited before deployment or execution.

> **Problem Solved**: Prevents security leaks (e.g., hardcoded secrets), unsafe commands, invalid edits, or misconfigurations introduced by autonomous code agents — especially when those agents are not fully supervised or audited.

---

## 2. Architecture

The project is organized as a **script-based validation suite**, with no complex frameworks or external dependencies beyond standard Python libraries. All logic is contained within a single directory, with modular files handling specific validation concerns.

### Directory Structure:

```
guardrails/
├── validate_secrets.py         # Validates presence/absence of secrets in code
├── post_write.py              # Validates code after write (e.g., file integrity, formatting)
├── validate_edit.py           # Validates edits to existing code (e.g., scope, impact)
├── validate_command.py        # Validates commands issued by the agent (e.g., shell, system calls)
├── utils.py                   # Shared utilities (e.g., config loading, logging, error formatting)
├── config.yaml                # Configuration file for guardrails (e.g., secret patterns, allowed commands)
```

> **Note**: All files are standalone scripts, designed to be imported or invoked as functions. They do not rely on a main entry point or web framework — they are meant to be called programmatically by the agent orchestrator.

---

## 3. Key Components

### 3.1 `validate_secrets.py`

- **Purpose**: Scans generated or edited code for hardcoded secrets (e.g., API keys, passwords, tokens).
- **Mechanism**: Uses regex patterns defined in `config.yaml` to match suspicious strings. Optionally integrates with external secret detection libraries if available.
- **Output**: Returns `True` if no secrets found, `False` otherwise, with optional detailed report.

### 3.2 `post_write.py`

- **Purpose**: Validates code after it has been written to disk — checks for formatting, syntax, or structural integrity.
- **Mechanism**: May use `ast` or `black` for syntax validation, or custom rules for file structure (e.g., no empty files, no trailing whitespace).
- **Output**: Boolean pass/fail, optionally with line-by-line diagnostics.

### 3.3 `validate_edit.py`

- **Purpose**: Validates proposed edits to existing code (e.g., diff patches, inline changes).
- **Mechanism**: Compares proposed changes against a baseline (e.g., original file) and checks:
  - Whether edits are within allowed scope (e.g., only modifying function bodies, not class definitions)
  - Whether edits introduce new secrets or unsafe patterns
  - Whether edits preserve code structure (e.g., no deletion of required imports)
- **Output**: Boolean pass/fail, with edit diff summary.

### 3.4 `validate_command.py`

- **Purpose**: Validates system commands issued by the agent (e.g., `git commit`, `pip install`, `docker build`).
- **Mechanism**: Uses a whitelist/blacklist from `config.yaml` to allow only safe commands. May also validate command arguments for dangerous flags (e.g., `rm -rf`, `sudo` without password).
- **Output**: Boolean pass/fail, with command audit log.

### 3.5 `utils.py`

- **Purpose**: Provides shared utilities across all validators.
- **Key Functions**:
  - `load_config(config_path: str)` — Loads `config.yaml` and returns parsed config dict.
  - `log(message: str, level: str = "INFO")` — Simple logging wrapper.
  - `get_secret_patterns()` — Returns list of regex patterns for secret detection.
  - `is_safe_command(cmd: str)` — Checks if command is in allowed list.
  - `diff_summary(old: str, new: str)` — Generates human-readable diff for edit validation.

### 3.6 `config.yaml`

- **Purpose**: Central configuration file for all guardrails.
- **Sample Structure**:
  ```yaml
  secrets:
    patterns:
      - "API_KEY_[A-Z]+"
      - "SECRET_.*"
      - "token.*"
  allowed_commands:
    - "git add"
    - "git commit"
    - "python3 -m pip install"
  disallowed_commands:
    - "rm -rf"
    - "sudo"
    - "sh -c"
  edit_scope:
    allowed: ["function", "variable", "comment"]
    forbidden: ["class", "import", "module"]
  ```

> **Note**: The error message `Lookup failed for Cls 'DocGenerator' from the 'wiki-agent' app: App 'wiki-agent' not found` suggests that this project may have been intended to be integrated into a larger agent system (possibly named `wiki-agent`), but that system is not present or not properly configured. This is likely a **legacy or placeholder error** — the guardrails themselves are self-contained and do not require the `DocGenerator` class to function.

---

## 4. Data Flow

The system is designed to be **intercepted by an orchestrator** (e.g., an agent controller) that triggers validation before/after code operations.

### Typical Flow:

1. **Pre-Write Phase**:
   - Agent generates code or proposes an edit.
   - Orchestrator calls `validate_secrets.py` → checks for secrets.
   - Orchestrator calls `validate_edit.py` → checks edit scope and safety.
   - Orchestrator calls `validate_command.py` → if command is involved (e.g., `git commit`).

2. **Post-Write Phase**:
   - Code is written to disk.
   - Orchestrator calls `post_write.py` → validates syntax, structure, formatting.

3. **Failure Handling**:
   - If any validator returns `False`, the orchestrator halts the operation and logs the failure.
   - Optionally, the orchestrator can return a detailed error message or suggestion for correction.

> **Data Format**: All validators expect a string (code content) or a tuple (old/new code, command string) as input. Output is a boolean and optionally a dict with metadata.

---

## 5. Getting Started

### Prerequisites

- Python 3.8+
- `pyyaml` (for `config.yaml` parsing)
- Optional: `black`, `ast`, `difflib` for enhanced validation

Install dependencies:

```bash
pip install pyyaml black ast difflib
```

### Usage

#### 1. Configure `config.yaml`

Edit `config.yaml` to define your security policies, allowed commands, and edit scope.

#### 2. Validate Secrets

```python
from validate_secrets import validate_secrets

code = "def api_call():\n    return requests.get('https://api.example.com', headers={'API_KEY': 'abc123'})"
result = validate_secrets(code)
print(result)  # True or False
```

#### 3. Validate Edits

```python
from validate_edit import validate_edit

old_code = "def hello():\n    print('Hello')"
new_code = "def hello():\n    print('Hello, world!')"
result = validate_edit(old_code, new_code)
print(result)  # True or False
```

#### 4. Validate Commands

```python
from validate_command import validate_command

cmd = "git commit -m 'feat: add guardrails'"
result = validate_command(cmd)
print(result)  # True or False
```

#### 5. Post-Write Validation

```python
from post_write import post_write

code = "def foo():\n    print('bar')"
result = post_write(code)
print(result)  # True or False
```

#### 6. Utility Functions

```python
from utils import load_config, log

config = load_config("config.yaml")
log("Starting validation...")
```

### Integration with Agent

To integrate with an agent system:

- Hook into agent’s `generate_code`, `edit_code`, or `execute_command` methods.
- Call appropriate validators before proceeding.
- Use `utils.load_config()` to load dynamic rules.

Example pseudo-code:

```python
def safe_generate_code(agent, prompt):
    code = agent.generate_code(prompt)
    if not validate_secrets(code):
        raise ValueError("Code contains secrets!")
    if not validate_edit(old_code, code):
        raise ValueError("Edit violates scope rules!")
    return code
```

---

## Notes

- The project is **not production-ready** without a proper orchestrator or agent system.
- The `DocGenerator` error is likely a **placeholder or legacy artifact** — the guardrails do not depend on it.
- Consider extending with:
  - Linting integration (e.g., `flake8`, `pylint`)
  - Secret scanning with `secrets-detect` or `truffleHog`
  - AI-based code review (e.g., using Claude’s API for semantic validation)

---

This project provides a **minimal, modular, and extensible guardrail system** for autonomous code agents — ideal for research, prototyping, or enterprise use cases requiring strict code hygiene.