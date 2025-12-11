```markdown
1. **Purpose**  
This script automates the cleanup of old TensorBoard logs specifically for the Router trainer by invoking a Python cleanup script via `uvx modal run` within the TensorBoard server environment. It ensures disk space is managed and logs are not indefinitely retained.

2. **Key Components**  
- `cleanup.py`: Python script executed remotely via Modal to delete old TensorBoard logs.  
- `uvx modal run`: Command-line utility to execute the cleanup script in a Modal-managed environment.  
- `SCRIPT_DIR` and `PROJECT_ROOT`: Environment variables resolving script and project root paths.  

3. **Algorithm Notes**  
None. The script follows a linear, deterministic execution flow with no complex algorithms — it simply navigates to the correct directory and triggers a remote cleanup process.

4. **Dependencies**  
- `uvx` (Modal CLI tool) for remote execution.  
- `modal` (Python library) for running the cleanup script in a managed environment.  
- `cleanup.py` (Python script in `tensorboard-server/scripts/`).

5. **Usage Example**  
Run the script directly from the project root:  
```bash
./scripts/cleanup-tensorboard.sh
```
This will trigger cleanup of TensorBoard logs for the Router trainer.
```