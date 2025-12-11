```markdown
1. **Purpose**  
This script evaluates a router LoRA adapter on held-out test or validation data from linked expert datasets. It loads a checkpoint, processes samples, and measures accuracy by comparing predicted expert labels against ground truth, supporting both test-set (unseen during training) and validation-set modes.

2. **Key Components**  
- `evaluate_router`: Main function that orchestrates loading, evaluation, and reporting.  
- `LABEL_TO_ADAPTER`, `ADAPTER_TO_LABEL`: Mappings between numeric labels and expert names.  
- `manifest.json` loader: Parses expert-specific train/val/test paths and indices.  
- `PeftModel` + `Qwen3VLForConditionalGeneration`: Loads base model + router LoRA adapter.  
- `tqdm` + `torch`: For progress tracking and inference.  

3. **Algorithm Notes**  
- Uses `generate` with `max_new_tokens=10` to predict a single label token.  
- Truncates input to exclude labels, using `label_start` to find the first non-padded token.  
- Supports two evaluation modes: test-set (held-out) or validation-set (with sample limits).  
- Random shuffling and sampling per expert to ensure balanced evaluation.  

4. **Dependencies**  
- `torch`, `transformers`, `peft`, `qwen-vl-utils`, `Pillow`, `tqdm`  
- Hugging Face `AutoProcessor`, `Qwen3VLForConditionalGeneration`  
- Modal for distributed execution, volumes, and secrets  
- Requires `huggingface-secret` for model access  

5. **Usage Example**  
```bash
modal run modal/eval_router.py --run-name router-tb-fixed --manifest router_linked_20251209_233704 --checkpoint-step 40
```
Evaluates checkpoint `40` on validation data from the specified manifest.
```