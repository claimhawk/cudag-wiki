```markdown
## Purpose

This script generates a balanced MoE routing dataset by querying Modal’s training history for the latest deployed dataset per expert, downloading it from `claimhawk-lora-training`, and converting tasks into routing samples with adapter labels. It caches datasets locally and supports configurable sample counts per adapter, with fallbacks for missing files or configurations.

## Key Components

- `_load_adapters_config()`: Loads adapter config from `adapters.yaml` (fallback paths).
- `_get_adapter_labels()`: Maps adapter names to integer labels.
- `_get_adapter_generators()`: Maps adapters to their generator scripts.
- `get_researcher_name()`: Reads researcher name from `.researcher` file.
- `find_generators_root()`: Locates the generators project root.
- `get_deployed_dataset_from_modal()`: Fetches latest dataset name from Modal training history.
- `download_dataset_from_modal()`: Downloads dataset from `lora_training` volume.
- `find_latest_dataset()`: Combines history lookup and download with caching.
- `convert_to_routing_sample()`: Converts task to routing format with image path resolution and conversation filtering.

## Algorithm Notes

- Uses **caching** to avoid re-downloading datasets.
- **Balanced sampling** via configurable `test_samples_per_adapter` (default 25).
- **Image path normalization** for Modal resolution: `{dataset}/images/{filename}`.
- **Conversation filtering** removes system prompts and replaces `gpt` messages with adapter names.
- **OCR prompt templates** are pre-defined for Chandra routing (not used in current code).

## Dependencies

- `argparse`, `json`, `os`, `random`, `subprocess`, `tempfile`, `datetime`, `pathlib.Path`
- `yaml` (via `pyyaml>=6.0`)
- `uvx` CLI tool (for Modal volume operations)
- `adapters.yaml` (required config)
- `.researcher` file (optional for researcher name)

## Usage Example

```bash
python generator.py --config config/dataset.yaml
```

Or use defaults:

```bash
python generator.py
```
```