```markdown
1. **Purpose**  
This script generates a balanced routing dataset for Mixture-of-Experts (MoE) training on Modal, pulling data from adapter-specific datasets and OCR-generated samples. It creates train/val/test splits based on config, ensuring each adapter contributes proportionally, with 100 test samples per adapter.

2. **Key Components**  
- `check_script_invocation()`: Warns if script is run directly (not via shell wrapper).  
- `find_latest_dataset()`: Finds the most recent dataset for an adapter by timestamp.  
- `find_ocr_folders()`: Locates directories containing OCR-generated images.  
- `generate_ocr_samples()`: Creates routing samples from OCR images using shuffled prompts.  
- `generate_routing_dataset()`: Main Modal function that orchestrates dataset generation and split creation.

3. **Algorithm Notes**  
- Uses `random.Random` with a seed for reproducibility.  
- Samples are balanced per adapter using configured counts and split ratios.  
- OCR samples are generated from pre-existing image folders, shuffled, and copied to avoid collisions.  
- Test set is fixed at 100 samples per adapter, regardless of config.

4. **Dependencies**  
- `modal`: For Modal cloud execution and volumes.  
- `json`, `os`, `random`, `shutil`, `datetime`, `pathlib`: Standard libraries.  
- `pyyaml`: For loading `dataset.yaml` (optional fallback).  
- `sdk.modal_compat` (optional): For volume/adapter config via SDK (fallback if not available).  
- `re`: For parsing dataset filenames by timestamp.

5. **Usage Example**  
```bash
modal run modal/generate_routing.py --seed 42
```
Run via shell script `./scripts/generate_dataset.sh` for full pipeline (generation + preprocessing + training).
```