```markdown
1. **Purpose**  
This script deploys a routing LoRA checkpoint from a training volume to an inference volume in a Modal-managed environment. It supports automatic detection of the latest checkpoint or manual specification, copying the checkpoint to a designated inference path and committing the volume for server readiness.

2. **Key Components**  
- `find_latest_checkpoint()`: Locates the most recent routing checkpoint by timestamp and run number.  
- `deploy_checkpoint(checkpoint_path)`: Copies the specified checkpoint to `/inference/routing/adapter` and commits the volume.  
- `main(checkpoint, latest)`: CLI entrypoint that either deploys a specified checkpoint or auto-selects the latest one.

3. **Algorithm Notes**  
- Directory traversal with sorting by timestamp (`routing_YYYYMMDD_HHMMSS`) and checkpoint number (`checkpoint-N`).  
- Fallback logic: prefers `final` checkpoint, otherwise selects highest-numbered checkpoint directory.  
- Uses `shutil.copytree` for recursive file copying and `modal.Volume.commit()` to persist changes.

4. **Dependencies**  
- `modal` (for Modal cloud orchestration)  
- `pathlib.Path` (for filesystem path operations)  
- `shutil` (for file copying)  
- `sdk.modal_compat` (optional; provides volume names from config, fallbacks if unavailable)

5. **Usage Example**  
```bash
modal run modal/deploy.py --latest
# or
modal run modal/deploy.py --checkpoint checkpoints/datasets/routing_20251202_185620/routing-20251202-185620/final
```
```