```markdown
1. **Purpose**  
This script orchestrates the generation of a MoE routing dataset on Modal, including dataset assembly, tokenization with Qwen processor, and optional LoRA router training — all performed entirely within Modal volumes to avoid local I/O. It supports skipping training or adjusting sampling parameters for adapters.

2. **Key Components**  
- `uvx modal run modal/generate.py "$@"`: Executes the core Modal function with passed arguments, handling all pipeline stages (dataset verification, assembly, preprocessing, training).

3. **Algorithm Notes**  
None explicitly defined — the script is a workflow orchestrator. Key pattern: all operations are delegated to `modal/generate.py`, which likely implements the core logic (e.g., expert dataset merging, tokenization, router training).

4. **Dependencies**  
- `uvx` (Modal CLI tool)  
- `modal` (Python library for Modal platform)  
- `modal/generate.py` (custom Modal function implementing pipeline logic)  
- Qwen tokenizer (via Hugging Face `transformers`)

5. **Usage Example**  
```bash
./scripts/generate.sh                          # Full pipeline
./scripts/generate.sh --skip-train             # Generate dataset only
./scripts/generate.sh --samples-per-adapter 500 --test-per-adapter 25
```
```