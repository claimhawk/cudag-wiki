```markdown
## Purpose

This script dynamically links the latest preprocessed expert datasets (by timestamp) into a manifest file for router training, without reprocessing. It reads configuration from `config/dataset.yaml` to determine sample counts and test splits, supports CLI overrides, and outputs a deterministic JSON manifest to a Modal volume.

## Key Components

- `load_dataset_config_local()`: Loads dataset config from local YAML or returns defaults.
- `find_latest_dataset()`: Finds the most recent dataset folder for an expert by timestamp.
- `link_expert_data()`: Modal function that discovers datasets, applies sample limits, holds out test samples, and saves manifest.
- `main()`: Local entrypoint that parses CLI args, loads config, and triggers remote dataset linking.

## Algorithm Notes

- **Timestamp-based discovery**: Uses regex to parse folder names and sorts by timestamp (lexicographic order).
- **Deterministic holdout**: Uses `random.seed(42)` to randomly select test samples for reproducibility.
- **Config precedence**: CLI overrides > config > defaults (500 samples per expert).
- **Proportional validation**: Val samples capped at 1/4 of train samples per expert.

## Dependencies

- `modal` (for cloud execution and volumes)
- `pyyaml` (for loading config)
- `pathlib`, `re`, `datetime`, `json`, `os`, `random` (standard library)
- Requires Python 3.11+

## Usage Example

```bash
modal run modal/link_router_data.py \
  --output-name router_manifest_20250405 \
  --test-samples 30 \
  --desktop 600 \
  --claim-window 400
```
```

This generates a manifest named `router_manifest_20250405.json` with 30 held-out test samples per expert, overriding desktop to 600 and claim-window to 400 samples.
```