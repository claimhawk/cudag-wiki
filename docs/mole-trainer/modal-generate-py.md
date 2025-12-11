```markdown
## Purpose

This file defines a Modal app that generates a routing dataset for training a router LoRA model. It performs four steps in one command: verifies expert datasets exist, assembles a routing dataset by sampling from each expert’s data, generates synthetic OCR samples, and saves the dataset with train/val/test splits. All data remains on Modal volumes, avoiding local I/O.

## Key Components

- `verify_datasets()`: Checks existence of expert datasets via training history and volume paths.
- `assemble_dataset()`: Assembles routing dataset by sampling from expert datasets, generating synthetic OCR samples, and saving splits.
- `convert_sample()`: Converts expert task into routing sample format with image path and numeric label.
- `generate_ocr_samples()`: Generates synthetic OCR routing samples using prompts from `OCR_PROMPTS`.
- `EXPERTS`: Maps expert names to numeric routing labels (e.g., "calendar" → 0).
- `OCR_PROMPTS`: List of prompt templates for synthetic OCR generation.

## Algorithm Notes

- **Stratified Sampling**: Each expert contributes equally to train/val/test splits, with test samples drawn first to ensure independence.
- **Synthetic Data Generation**: OCR samples are generated from images in other expert datasets using randomized prompts.
- **Randomization**: Uses a fixed seed for reproducibility and shuffles samples to avoid ordering bias.
- **Volume-Based I/O**: All data is stored and accessed via Modal volumes (`training_history`, `lora_training`, `moe_data`), ensuring no local downloads/uploads.

## Dependencies

- `json`, `random`, `datetime`, `pathlib` (Python standard library)
- `modal` (for Modal App, functions, volumes, and image configuration)
- `typing` (implicit via `dict[str, int]`, etc.)

## Usage Example

```bash
modal run modal/generate.py --samples-per-adapter 500 --test-per-adapter 25
```

This runs the full pipeline, verifying datasets, assembling a routing dataset with 500 samples per adapter and 25 test samples per adapter, and saving it to Modal’s `moe-lora-data` volume.
```