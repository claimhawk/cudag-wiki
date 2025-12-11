```markdown
# `agents.md` — Mole-Trainer Subagent Execution & Workflow Protocol

## 1. Purpose

This document defines the mandatory subagent execution model and three-step implementation protocol for AI agents working on the `mole-trainer` project. It enforces structured, auditable, and context-isolated task decomposition to maintain code quality, traceability, and separation of concerns during distributed LoRA model training.

## 2. Key Components

- **Subagent Execution Model**: Mandates task decomposition into atomic subagents with explicit contracts and isolated context.
- **Three-Step Protocol**: Enforces `research → plan → implementation` workflow with dedicated `.claude/` output files.
- **Pipeline Scripts**: Bash wrappers (`generate.sh`, `train.sh`, etc.) for auto-chaining training stages.
- **Architecture**: Directory structure and data flow from dataset generation → upload → preprocess → train → eval → deploy.
- **Code Standards**: Python 3.11+, type hints, max complexity 10, docstrings required, `uvx`/`uv run` for execution.

## 3. Algorithm Notes

- **Subagent Isolation**: Each sub-task runs statelessly with minimal inputs; outputs are composed by orchestrator.
- **Deterministic Planning**: Plans must be complete, scoped, and validated via subagents before implementation.
- **Traceability**: All actions logged in `./.claude/implementation/progress.md` with timestamps and commit-sized entries.
- **Pipeline Auto-chaining**: Scripts chain stages unless `--dry` flag is used.

## 4. Dependencies

- Python 3.11+
- `ruff`, `mypy`, `docstring-checker` (via `scripts/pre-commit.sh`)
- `uvx` or `uv` for Python execution
- Modal SDK for cloud deployment
- `config/` files: `dataset.yaml`, `loras.json`
- External tools: `flash-attn`, `Qwen processor`

## 5. Usage Example

```bash
# Full training pipeline
./scripts/generate.sh
./scripts/upload.sh datasets/20240615_100000
./scripts/preprocess.sh --dataset-name my_dataset
./scripts/train.sh --run-name run_001 --dataset-name my_dataset --fast

# Evaluate and deploy
./scripts/eval.sh --run-name run_001 --dataset-name my_dataset
./scripts/deploy-latest.sh
```
```