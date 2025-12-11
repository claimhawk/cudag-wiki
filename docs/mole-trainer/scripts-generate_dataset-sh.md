```markdown
1. **Purpose**  
This script automates the generation of a routing dataset on Modal, downloads the output locally, and auto-starts preprocessing. It supports a dry-run mode for previewing commands without execution.

2. **Key Components**  
- `DRY_RUN`: Flag to enable dry-run mode (preview only).  
- `TRAIN_TASKS`, `VAL_TASKS`, `EVAL_TASKS`: Default task counts for train/val/eval splits.  
- `uvx modal run`: Executes Modal remote script to generate dataset.  
- `DATASET_NAME`: Extracted from Modal output via regex.  
- `exec ./scripts/preprocess.sh`: Auto-launches preprocessing after dataset generation.

3. **Algorithm Notes**  
Uses regex to extract dataset name from stdout. Relies on `set -euo pipefail` for strict error handling. Auto-continues preprocessing without manual intervention.

4. **Dependencies**  
- `uvx` (Modal CLI)  
- `bash` with `grep`, `tee`, `tail`, `echo`, `exit`  
- `scripts/preprocess.sh` (must exist and be executable)

5. **Usage Example**  
```bash
./scripts/generate_dataset.sh --dry
# → Shows what would be run, exits without execution.

./scripts/generate_dataset.sh
# → Generates dataset, auto-starts preprocessing.
```
```