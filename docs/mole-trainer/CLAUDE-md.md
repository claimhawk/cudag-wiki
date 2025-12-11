```markdown
# CLAUDE.md — Technical Documentation

## 1. Purpose

This file defines the mandatory operational protocol for AI assistants (including Claude Code) working on the `mole-trainer` repository. It enforces a strict subagent execution model and three-phase coding workflow to ensure traceable, auditable, and modular development. All code changes must follow this protocol to maintain repository integrity and separation of concerns.

## 2. Key Components

- **Subagent Execution Model**: Mandates decomposition of tasks into isolated, stateless subagents with explicit contracts.
- **Three-Step Implementation Protocol**: Enforces `research`, `plans`, and `implementation/progress.md` phases before any code is written.
- **Pipeline Scripts**: `generate.sh`, `upload.sh`, `preprocess.sh`, `train.sh`, `eval.sh`, `deploy-latest.sh` — automate end-to-end training workflow.
- **Architecture**: Defines directory structure, data flow (generate → upload → preprocess → train → eval → deploy), and config locations.
- **Code Standards**: Enforces Python 3.11+, type hints, cyclomatic complexity limits, and tooling via `ruff`, `mypy`, `uvx`.

## 3. Algorithm Notes

- **Subagent Decomposition**: Tasks are broken into atomic sub-tasks, each assigned to a dedicated subagent with no shared context beyond inputs.
- **Stateless Execution**: Subagents operate on inputs only, producing minimal, scoped outputs — no global state or inference beyond scope.
- **Deterministic Planning**: Implementation plans must be complete, deterministic, and validated via subagents before code generation.
- **Pipeline Auto-Chaining**: Scripts chain automatically (e.g., `generate.sh` → `upload.sh` → `preprocess.sh` → `train.sh`) unless `--dry` flag is used.

## 4. Dependencies

- **Python**: 3.11+
- **Tools**: `ruff`, `mypy`, `uvx`, `docstring-checker`, `flash-attn`
- **External**: Modal cloud platform for distributed training, Qwen processor for tokenization
- **Config**: `config/dataset.yaml`, `config/loras.json`
- **Scripts**: `scripts/`, `modal/`, `utils/`, `generator.py`, `routing-generator/`

## 5. Usage Example

```bash
# Run full training pipeline
./scripts/generate.sh
./scripts/upload.sh datasets/20240615_1000
./scripts/preprocess.sh --dataset-name my_dataset
./scripts/train.sh --run-name run_001 --dataset-name my_dataset --fast

# Evaluate and deploy
./scripts/eval.sh --run-name run_001 --dataset-name my_dataset
./scripts/deploy-latest.sh
```
```