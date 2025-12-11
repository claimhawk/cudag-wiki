```markdown
## Purpose

This script retroactively saves training history records for MoE router runs that completed before the history tracking system was implemented. It fetches metadata (trainer state, eval results, cost reports, hyperparameters) from remote Modal volumes, constructs a structured history record, and uploads it to a dedicated history volume — optionally in dry-run mode.

## Key Components

- `run_modal_command(cmd, capture=True)`: Executes Modal CLI commands and captures output.
- `list_runs_for_dataset(dataset_name)`: Lists run directories under a dataset.
- `get_*_results(...)`: Fetches and parses JSON files (trainer_state, eval, train, cost) from remote volumes.
- `check_history_exists(...)`: Checks if a history record already exists.
- `save_history_record(...)`: Constructs and uploads a history record to the history volume.
- `backfill_run(...)`: Orchestrates fetching data and saving history for a single run.

## Algorithm Notes

- **Fallback Path Resolution**: Each data file is searched across multiple possible remote paths (e.g., `final/` vs root).
- **Dry Run Support**: Records are printed but not saved, enabling preview before execution.
- **Atomic Logging**: History records are appended to a JSONL log file (`runs.jsonl`) to maintain chronological order.
- **Metadata Extraction**: Metrics like `val_loss`, `cost_usd`, and `hyperparams` are extracted from multiple sources with fallbacks.

## Dependencies

- `argparse`: For CLI argument parsing.
- `json`: For parsing and serializing JSON.
- `subprocess`: To execute Modal CLI commands.
- `sys`, `tempfile`, `datetime`, `pathlib.Path`: For system utilities and file handling.
- `uvx` (Modal CLI): To interact with Modal volumes.

## Usage Example

```bash
# Backfill a specific run
python scripts/backfill_history.py --dataset routing_20251205_115325 --run routing-overnight-001

# Dry run (preview only)
python scripts/backfill_history.py --dataset routing_20251205_115325 --run routing-overnight-001 --dry-run
```
```