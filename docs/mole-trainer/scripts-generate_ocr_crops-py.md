```markdown
## Purpose

This script generates OCR training data by cropping text-heavy regions (procedure grids and provider dropdowns) from generator dataset images. It outputs cropped PNGs and associated prompts for training OCR models or routing systems like Chandra, with configurable limits and random prompt variation.

## Key Components

- `crop_claim_window_grid`: Crops procedure grids from claim-window images and returns metadata with OCR prompts.
- `crop_provider_dropdown`: Crops dropdown overlays from provider-select images, using field metadata to position the crop and includes expected text.
- `find_generator_datasets`: Scans generator directories to locate claim-window and provider-select datasets.
- `process_claim_window_dataset`: Iterates through a claim-window dataset, extracts grid crops, and limits output.
- `process_provider_select_dataset`: Processes provider-select datasets, filtering for select-provider tasks and extracting dropdown crops.
- `main`: Parses CLI args, initializes output directories, and orchestrates dataset processing.

## Algorithm Notes

- Uses deterministic cropping bounds defined in `CROP_CONFIGS` and field positions from `FIELD_POSITIONS`.
- Deduplicates images by tracking seen image paths to avoid redundant processing.
- Randomly selects OCR prompts from a template list for variability.
- Supports early termination based on `max_samples` per dataset type.
- Uses `Path` objects for robust filesystem navigation.

## Dependencies

- `argparse` – for CLI argument parsing
- `json` – for reading sample metadata
- `random` – for prompt randomization and reproducibility
- `datetime` – for potential timestamping (unused in current code)
- `pathlib.Path` – for filesystem path handling
- `PIL.Image` – for image cropping and saving

## Usage Example

```bash
python scripts/generate_ocr_crops.py --output-dir datasets/ocr --max-grids 150 --max-dropdowns 150 --seed 42
```
This generates up to 150 grid and 150 dropdown crops, saving them to `datasets/ocr/`, with random prompts and reproducible behavior via seed 42.
```