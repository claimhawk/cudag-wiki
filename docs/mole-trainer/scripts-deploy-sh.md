```markdown
1. **Purpose**  
This script deploys a routing LoRA checkpoint to the inference path using Modal, a cloud-based execution platform. It abstracts the deployment logic by invoking a Python script (`modal/deploy.py`) with the provided checkpoint path, ensuring the model is ready for inference.

2. **Key Components**  
- `deploy.sh`: Bash script orchestrating deployment via `uvx modal run`.  
- `modal/deploy.py`: Python script executed by Modal to handle checkpoint deployment (not shown here).  
- `set -e`: Ensures script exits on any command failure.

3. **Algorithm Notes**  
None. This is a utility script with no algorithmic logic — it simply passes arguments to a Modal deployment job.

4. **Dependencies**  
- `uvx`: A tool for running Python scripts via Modal.  
- `modal`: Python library for cloud-based execution.  
- `bash`: Shell environment for script execution.

5. **Usage Example**  
```bash
./scripts/deploy.sh --checkpoint checkpoints/datasets/routing_20251202_185620/routing-20251202-185620/final
```
```

```