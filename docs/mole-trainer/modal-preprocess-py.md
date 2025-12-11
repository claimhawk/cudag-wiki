```markdown
## Purpose

This file implements a Modal-based data preprocessing pipeline for Qwen3-VL datasets, converting raw JSONL + image data into preprocessed tensors stored on persistent volumes. It avoids GPU preprocessing overhead by running on CPU instances and enables fast training startup via volume caching.

## Key Components

- `_read_yaml_value`: Extracts config values from YAML using regex (for environments without `pyyaml`).
- `_load_local_config_values`: Loads essential config (volumes, model) from local `adapters.yaml`.
- `_build_system_prompt`: Constructs a system prompt for the Mixture-of-Experts router from config.
- `preprocess_dataset_impl`: Main Modal function that loads, reassembles (if chunked), splits, and preprocesses dataset into tensors.

## Algorithm Notes

- **Chunk Reassembly**: For datasets split into chunks, reassembles `.tar.gz` from chunk files before extraction.
- **Auto-Splitting**: Falls back to 90/10 train/val split if no pre-split files exist (e.g., Ramiro format).
- **Volume Reloading**: Explicitly reloads Modal volumes to ensure latest data is visible.
- **Processor-based Preprocessing**: Uses `AutoProcessor` and `process_vision_info` to convert images + text into model-ready tensors.

## Dependencies

- `modal` (for cloud execution and volumes)
- `torch`, `transformers`, `qwen-vl-utils`, `Pillow`, `numpy`, `tqdm`
- `pyyaml` (inside Modal container only)
- `gzip`, `tarfile`, `json`, `os`, `pathlib`, `re`

## Usage Example

```bash
modal run modal/preprocess.py --dataset-dir datasets/routing_xxx
```

This runs the preprocessing function on Modal, using the dataset mounted at `/dataset_data`, and saves outputs to the `moe_data` volume.
```