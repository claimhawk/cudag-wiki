```markdown
1. **Purpose**  
This script aggregates preprocessed expert datasets and converts them into router training format by replacing expert labels with numeric routing labels. It leverages existing `.pt` files to avoid reprocessing images, enabling faster, scalable, and consistent router training using all available expert data.

2. **Key Components**  
- `preprocess_router_from_experts`: Main function that collects, shuffles, splits, and converts expert samples into router format.  
- `create_router_label`: Generates label tensors for router training using a tokenizer and a special end token.  
- `process_sample`: Converts an expert sample by truncating input to label start and appending new router labels.  
- `main`: Local entrypoint to run the preprocessing remotely via Modal.

3. **Algorithm Notes**  
- Uses stratified sampling by expert name (via `max_per_expert` and random shuffling).  
- Splits data into train/val/test using fixed proportions (configurable).  
- Reuses existing expert `.pt` files without reprocessing images.  
- Label encoding uses tokenizer + special end token (151645) to signal router output.

4. **Dependencies**  
- `torch`, `transformers`, `tqdm`, `json`, `os`, `random`, `pathlib`, `datetime`, `modal`  
- Requires `Qwen/Qwen2.5-0.5B` tokenizer for label encoding.  
- Uses Modal for distributed execution with volumes for persistent storage.

5. **Usage Example**  
```bash
modal run modal/preprocess_router.py \
  --output_name "router_v1" \
  --val_split 0.1 \
  --test_split 0.05 \
  --seed 42 \
  --max_per_expert 1000
```
```

This documentation is concise, technical, and structured for technical audiences familiar with ML pipelines and Modal.