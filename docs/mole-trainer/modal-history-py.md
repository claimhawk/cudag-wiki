```markdown
## Purpose

This module persists and visualizes training history for MoE router models, storing metrics like validation loss, eval accuracy, cost, and hyperparameters in a shared volume to survive dataset cleanups. It supports querying, summarizing, and plotting historical runs for analysis and comparison.

## Key Components

- `save_training_record`: Saves a training run’s metrics as JSON and JSONL for persistence.
- `get_router_history`: Retrieves all saved router training records.
- `plot_router_history`: Generates 4-panel plots (accuracy, loss, cost, per-class) from history.
- `print_summary`: Prints a formatted summary of training history including best/largest metrics.
- `main`: CLI entrypoint to either print summary or generate plots.

## Algorithm Notes

- Uses a **JSONL log** for efficient append-only querying.
- **Per-class accuracy** is visualized as a horizontal bar chart with color-coded thresholds (green ≥80%, orange ≥60%, red <60%).
- **Time-series plots** are sorted chronologically and include error handling for missing data.
- **Volume-backed persistence** ensures data survives container restarts and dataset deletions.

## Dependencies

- `modal` (for cloud-native volume persistence and functions)
- `json`, `datetime`, `pathlib` (standard library)
- `pandas`, `matplotlib` (for data analysis and plotting)
- Python 3.11+ (via Docker image)

## Usage Example

```bash
# Save a training record
uvx modal run modal/history.py -- save_training_record \
    --dataset_name "routing_20251205_115325" \
    --run_name "routing-overnight-001" \
    --metrics '{"val_loss": 0.23, "eval_accuracy": 92.5, "cost_usd": 150.0, ...}'

# View summary
uvx modal run modal/history.py

# Generate plots
uvx modal run modal/history.py --plot
```
```