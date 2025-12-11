```markdown
1. **Purpose**  
This script enables fast, lightweight training for the Router trainer by running only 20 steps (including one checkpoint and evaluation). It’s designed for rapid iteration and testing, bypassing full training cycles.

2. **Key Components**  
- `set -e`: Ensures script exits on any command failure.  
- `SCRIPT_DIR`: Resolves absolute path to script’s directory.  
- `exec "$SCRIPT_DIR/train.sh" --fast "$@"`: Delegates execution to `train.sh` with `--fast` flag and all passed arguments.

3. **Algorithm Notes**  
None — this is a wrapper script with no algorithmic logic. It leverages the `--fast` flag in `train.sh` to constrain training steps.

4. **Dependencies**  
Requires `train.sh` in the same directory (`$SCRIPT_DIR/train.sh`). No external libraries or modules.

5. **Usage Example**  
```bash
./scripts/train-fast.sh --config config.yaml --device cuda
```
Runs fast training with the given config and device.
```