```markdown
1. **Purpose**  
This script launches or deploys a standalone TensorBoard server for the Router trainer using Modal. It supports two modes: `serve` (temporary local server) or `deploy` (persistent, scalable server with auto-scaling to zero when idle).

2. **Key Components**  
- `MODE="${1:-deploy}"`: Determines whether to serve or deploy TensorBoard.  
- `uvx modal serve/deploy`: Executes Modal CLI commands to run or deploy the TensorBoard server.  
- `TB_SERVER`: Path to the TensorBoard server project directory.  
- `cleanup.py`: Optional cleanup script for old runs (invoked post-deploy).

3. **Algorithm Notes**  
Uses conditional logic (`if [[ "$MODE" == "serve" ]]`) to branch execution. Relies on Modal’s serverless infrastructure for scalability and URL stability. No complex algorithms — purely orchestration logic.

4. **Dependencies**  
- `uvx` (Modal CLI tool) — required to run `modal serve` or `modal deploy`.  
- `modal/tb_server.py` — the TensorBoard server script to be executed.  
- `scripts/cleanup.py` — optional cleanup script (called only in deploy mode).

5. **Usage Example**  
```bash
# Deploy TensorBoard (recommended)
./scripts/tensorboard.sh deploy

# Start temporary server (for quick debugging)
./scripts/tensorboard.sh serve
```
```