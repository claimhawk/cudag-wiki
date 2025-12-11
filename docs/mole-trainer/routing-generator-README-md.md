```markdown
## Purpose

The `routing-generator` script merges multiple task-specific datasets (e.g., `calendar`, `claim-window`) into a unified routing dataset, augmenting each row with an `adapter` label for Stage-1 router training. It supports image path resolution, optional filtering by task type, and shuffling with a seed for reproducibility.

## Key Components

- `generate.py`: Main CLI script that orchestrates dataset merging and labeling.
- `--source`: CLI flag to specify input datasets with metadata (label, JSONL path, image root, filters).
- `--output`: CLI flag to define output JSONL file.
- `--shuffle` / `--seed`: Flags to enable randomized row ordering with reproducible results.
- `resolve_image_path()`: Helper function to resolve relative image paths from `image_root`.

## Algorithm Notes

- Merges datasets row-by-row, appending `adapter`, `source_dataset`, and resolved `image_path` fields.
- Supports fallback image path resolution: first tries `image_root`, then parent of `image_root` if needed.
- Ignores unresolved image paths (warns but writes row).
- Optional filtering via `tasks` parameter to restrict rows by `metadata.task_type`.

## Dependencies

- Standard Python libraries: `argparse`, `os`, `json`, `random`, `pathlib`.
- No external dependencies beyond Python stdlib.

## Usage Example

```bash
python routing-generator/generate.py \
  --source label=calendar,jsonl=../generators/calendar/datasets/mike-im-day-clicks-system-prompt-8B_20251120_180854/data.jsonl,image_root=../generators/calendar/datasets \
  --source label=claim-window,jsonl=../generators/edit-claim-window/datasets/edit-claim-window-prod_20251122_234849/data.jsonl,image_root=../generators/edit-claim-window/datasets \
  --output routing-data.jsonl \
  --shuffle \
  --seed 42
```
```