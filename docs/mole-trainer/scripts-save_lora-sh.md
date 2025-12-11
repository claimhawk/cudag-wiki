```markdown
1. **Purpose**  
This script uploads a LoRA checkpoint (either from local disk or another Modal volume) to a designated Modal volume for cross-app sharing, and updates `config/loras.json` to track the uploaded model with metadata including source, dataset, and timestamp.

2. **Key Components**  
- `usage()`: Displays help and exits on invalid arguments.  
- `update_config()`: Updates the JSON config file with LoRA metadata.  
- `modal_bin detection`: Auto-selects `modal` or `uvx modal` CLI based on availability.  
- `volume copy logic`: Uses `modal volume get/put` to transfer files between volumes or from local disk.  
- `checkpoint validation`: Ensures source path exists and is either a file or directory.

3. **Algorithm Notes**  
- Uses `realpath` to resolve symlinks and validate file paths.  
- Uses Python for JSON manipulation to safely handle nested updates and remove `None` values.  
- Supports two upload modes: local-to-volume and volume-to-volume.  
- Timestamps entries with UTC ISO format for reproducibility.

4. **Dependencies**  
- `bash` (standard shell)  
- `modal` CLI or `uvx modal` (required for volume operations)  
- `python3` (for JSON config updates)  
- `jq` not required — Python handles JSON updates directly.

5. **Usage Example**  
```bash
scripts/save_lora.sh --checkpoint /path/to/ckpt --name calendar-tasks --dataset datasets/xyz --volume moe-lora-checkpoints
```
or
```bash
scripts/save_lora.sh --from-volume claimhawk-checkpoints --checkpoint path/to/checkpoint-40 --name calendar-tasks --volume moe-lora-checkpoints
```
```