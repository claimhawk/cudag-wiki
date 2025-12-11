```markdown
1. **Purpose**  
This script orchestrates a test run for routing LoRA adapters on held-out test data using Modal, a cloud computing platform. It accepts required run and dataset identifiers, validates inputs, and executes a Modal job via `uvx modal run`.

2. **Key Components**  
- `set -e`: Ensures script exits on any command failure.  
- `while [[ $# -gt 0 ]]`: Parses command-line arguments with positional matching.  
- `uvx modal run modal/test.py`: Executes the core test job with passed parameters.  
- Input validation: Checks for required `--run-name` and `--dataset-name`.

3. **Algorithm Notes**  
None. The script is purely orchestration logic with no algorithmic processing. It relies on external Modal job (`modal/test.py`) for actual routing and evaluation.

4. **Dependencies**  
- `uvx` (Modal CLI tool) for remote job execution.  
- `bash` for script execution.  
- `modal/test.py` (external Python script invoked via Modal).

5. **Usage Example**  
```bash
./scripts/test.sh --run-name my_lora_test --dataset-name imdb
```
```

```