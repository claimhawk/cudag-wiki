```markdown
1. **Purpose**  
This script automates the deployment of the most recent routing LoRA checkpoint to the inference path using Modal, a cloud-based execution platform. It leverages `uvx` (a Python virtual environment tool) to run a Modal deployment script, ensuring the latest model weights are made available for inference.

2. **Key Components**  
- `set -e`: Ensures the script exits immediately if any command fails.  
- `exec uvx modal run modal/deploy.py --latest`: Executes the Modal deployment script with the `--latest` flag to target the newest checkpoint.

3. **Algorithm Notes**  
None — this is a utility shell script with no algorithmic logic. It relies on external tools (`uvx`, `modal`) to perform deployment.

4. **Dependencies**  
- `uvx` (for managing Python environments and running Modal commands)  
- `modal` (for cloud deployment and orchestration)  
- `modal/deploy.py` (the target deployment script, assumed to handle checkpoint selection and inference path updates)

5. **Usage Example**  
```bash
./scripts/deploy-latest.sh
```
Run the script in the project root to deploy the latest routing LoRA checkpoint.
```