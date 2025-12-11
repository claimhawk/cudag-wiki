```markdown
1. **Purpose**  
This script evaluates a routing LoRA adapter on held-out evaluation data, measuring accuracy per adapter class. It loads a base vision-language model, applies a fine-tuned routing LoRA, and compares predicted adapter labels against ground truth from the dataset.

2. **Key Components**  
- `evaluate_routing_lora`: Main Modal function that runs inference and computes accuracy.  
- `LABEL_TO_ADAPTER`, `ADAPTER_TO_LABEL`: Mapping between numeric labels and adapter names.  
- `EVAL_SYSTEM_PROMPT`: System prompt guiding the model to output only numeric adapter labels.  
- `balanced_data`: Ensures equal sampling across adapter classes for fair evaluation.  
- `PeftModel.from_pretrained`: Loads the LoRA adapter onto the base model.  
- `processor.apply_chat_template`: Formats input messages for Qwen-VL model.  

3. **Algorithm Notes**  
- Uses stratified sampling to ensure balanced evaluation across adapter classes.  
- Generates predictions with `max_new_tokens=50` and deterministic sampling (`do_sample=False`).  
- Decodes only new tokens to extract numeric labels from model output.  
- Handles missing or malformed samples gracefully by logging errors.  

4. **Dependencies**  
- `modal`: For cloud deployment and volume management.  
- `torch`, `transformers`, `peft`, `qwen-vl-utils`, `Pillow`, `tqdm`: For model loading, inference, and utilities.  
- `sdk.modal_compat` (optional): For centralized config (fallbacks provided).  
- Hugging Face `AutoProcessor` and `Qwen3VLForConditionalGeneration` for model loading.  

5. **Usage Example**  
```bash
modal run modal/test.py --run-name routing-20251125-052946 --dataset-name datasets/routing_20251125_052946
```
```

```